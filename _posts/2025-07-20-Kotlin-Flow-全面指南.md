---
title: Kotlin Flow 全面指南
date: 2025-07-20
categories: [Kotlin]
---



## Flow 是什么？

Flow 是 Kotlin 协程中的一种异步数据流。它代表了一个可以按顺序发出多个值的数据流。

## 核心概念

### 1. 冷流（Cold Flow）

- **定义**：只有在被收集（collect）时才会执行其上游逻辑的流。

- **行为**：每次调用 `collect` 都会重新执行其构建逻辑，相当于懒加载。

- **示例**：

  ```kotlin
  val coldFlow = flow {
      println("Flow started")
      emit(1)
      emit(2)
  }
  
  runBlocking {
      coldFlow.collect { println(it) }
      // Output:
      // Flow started
      // 1
      // 2
      coldFlow.collect { println(it) }
      // Output again:
      // Flow started
      // 1
      // 2
  }
  ```

### 2. 热流（Hot Flow）

- **定义**：数据的发射和订阅无关，类似广播，数据会持续发送，不管有没有人收集。
- **种类**：
  - **SharedFlow**
  - **StateFlow**
- **区别于冷流**：热流不依赖 collect 的调用。即使没有 collect，也可以发射和缓存值。

#### SharedFlow

- 类似广播。

- 支持多个收集者。

- 可配置缓存（replay）、缓冲区、丢弃策略等。

  ```kotlin
  val sharedFlow = MutableSharedFlow<Int>()
  
  scope.launch {
      sharedFlow.collect { println("collector1: $it") }
  }
  
  scope.launch {
      sharedFlow.emit(1)
  }
  ```

#### StateFlow

- 类似 LiveData：保存当前状态值。

- 只有一个当前值，收集时会立即收到当前值。

- 所有新收集者都会收到当前状态值。

  ```kotlin
  val stateFlow = MutableStateFlow(0)
  
  scope.launch {
      stateFlow.collect { println("State: $it") }
  }
  
  scope.launch {
      delay(1000)
      stateFlow.value = 1
  }
  ```

## Flow 使用方式

### 构建 Flow

```kotlin
val flow = flow {
    emit(1)
    emit(2)
}
```

### 收集 Flow

```kotlin
flow.collect { value ->
    println(value)
}
```

### 结合协程

```kotlin
lifecycleScope.launch {
    flow.collect { println(it) }
}
```

## 常用操作符（Operators）

### 转换类操作符

- `map { }`: 对流中每个元素进行转换
- `transform { emit(...) }`: 更灵活的转换操作符
- `flatMapConcat / flatMapMerge / flatMapLatest`: 用于展开内部流

```kotlin
flowOf(1, 2, 3).map { it * 2 }.collect { println(it) }
```

### 过滤类操作符

- `filter { }`: 过滤元素
- `take(n)`: 取前 n 个元素
- `drop(n)`: 跳过前 n 个元素

### 组合类操作符

- `zip(flow2) { a, b -> }`: 将两个流压缩合并
- `combine(flow2) { a, b -> }`: 当任一流发射值时合并发射

### 终止类操作符

- `collect { }`: 启动收集操作
- `toList()`: 转换为 List
- `first() / last()`: 获取首/尾元素

### 异常处理

- `catch { e -> }`: 捕获上游异常
- `onCompletion { }`: 流结束时调用

```kotlin
flow.catch { e ->
    println("Error: $e")
}.collect {
    println(it)
}
```

### 背压（Backpressure）相关操作符

- `buffer()`: 缓冲发射值，加快上游速度
- `conflate()`: 跳过中间值，仅保留最新值
- `collectLatest { }`: 如果新的值来了，取消之前的收集

```kotlin
flow {
    emit(1)
    delay(100)
    emit(2)
}.collectLatest {
    delay(200)
    println(it) // 只打印2，因为1在delay中被取消
}
```

## Flow vs SharedFlow vs StateFlow 对比

| 特性               | Flow（冷）         | SharedFlow（热）   | StateFlow（热）         |
| ------------------ | ------------------ | ------------------ | ----------------------- |
| 是否立即触发       | 否                 | 是                 | 是                      |
| 是否缓存值         | 否                 | 可选               | 是                      |
| 是否有默认值       | 否                 | 否                 | 是                      |
| 是否支持多个收集者 | 是                 | 是                 | 是                      |
| 应用场景           | 网络请求、单次任务 | 多个订阅者接收事件 | 表示状态，替代 LiveData |

## 使用建议

- UI 状态：推荐使用 StateFlow
- 多个订阅者共享事件：使用 SharedFlow
- 普通异步数据流或网络请求：使用 Flow

------

如需更多 Flow 进阶实战和自定义操作符封装，可继续拓展。