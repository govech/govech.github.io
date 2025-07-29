---
title: Android SDK开发
date: 2025-07-28
---



Android SDK 开发按**技术形态**、**功能定位**和**应用场景**可分为多种类型，以下是系统性分类及代表案例：

## **一、按技术形态分（交付方式）**

|    **类型**    |                           **描述**                           |    **文件格式**     |                           **特点**                           |            **典型场景**            |
| :------------: | :----------------------------------------------------------: | :-----------------: | :----------------------------------------------------------: | :--------------------------------: |
|   **AAR 库**   | Android 专属库，含代码、资源（布局/图片/字符串）、清单文件等 |       `.aar`        | 1. 支持资源隔离 2. 可包含 Android 组件（Activity/Service） 3. 主流 SDK 形态 |        支付宝 SDK、微信 SDK        |
|   **JAR 库**   |           纯 Java/Kotlin 代码库，不含 Android 资源           |       `.jar`        |    1. 轻量级 2. 无资源冲突风险 3. 不支持 Android 特有组件    |       Gson、OkHttp、Retrofit       |
| **Native SDK** |  通过 JNI 调用 C/C++ 开发的底层库（性能敏感或硬件交互场景）  | `.so` (Android NDK) | 1. 高性能 2. 直接操作硬件 3. 需处理 CPU 架构兼容性（armeabi-v7a, arm64-v8a） |       OpenCV、音视频编解码库       |
|  **远程依赖**  |           通过 Maven/Gradle 直接从仓库拉取（常用）           |          -          |             1. 自动解决传递依赖 2. 支持版本管理              | Firebase SDK、Google Play Services |

## **二、按功能定位分（核心用途）**

#### **1. 基础能力 SDK**

|    **子类**    |             **核心功能**             |     **代表 SDK**      |
| :------------: | :----------------------------------: | :-------------------: |
| **网络请求库** |       封装 HTTP 请求、数据解析       |   Retrofit + OkHttp   |
| **图片加载库** |         图片下载、缓存、变换         |    Glide、Picasso     |
| **本地存储库** |         数据库、文件存储封装         |      Room、Realm      |
| **工具链 SDK** | 提供通用工具（加密、日志、权限处理） | AndroidX Core、Timber |

#### **2. 业务服务 SDK**

|     **子类**     |              **核心功能**              |               **代表 SDK**                |
| :--------------: | :------------------------------------: | :---------------------------------------: |
|   **支付 SDK**   | 对接第三方支付渠道（支付宝/微信/银联） |         支付宝 SDK、微信支付 SDK          |
|   **登录 SDK**   |    提供社交账号登录（微信/QQ/微博）    |   微信 OpenSDK、OneLogin（本机号认证）    |
|   **推送 SDK**   |   消息推送服务（后台保活、厂商通道）   | 华为 Push、个推、Firebase Cloud Messaging |
| **地图定位 SDK** |      地图渲染、路线规划、位置服务      |        高德地图 SDK、百度地图 SDK         |
|   **广告 SDK**   |   展示广告（Banner、插屏、激励视频）   |          腾讯优量汇、穿山甲 SDK           |

#### **3. 系统增强 SDK**

|     **子类**     |              **核心功能**               |            **代表 SDK**             |
| :--------------: | :-------------------------------------: | :---------------------------------: |
|  **UI 组件库**   | 提供预制 UI（图表、日历、富文本编辑器） |      MPAndroidChart、PhotoView      |
| **性能监控 SDK** |     收集 Crash、卡顿、内存泄漏数据      | Sentry、Bugly、Firebase Crashlytics |
| **安全风控 SDK** |       设备指纹、反欺诈、代码防护        |      顶象加固、阿里聚安全 SDK       |
|  **AR/VR SDK**   |          增强现实/虚拟现实支持          |           ARCore、Vuforia           |

## **三、按发布主体分（生态角色）**

|      **类型**       |        **开发主体**         |                         **特点**                          |               **案例**                |
| :-----------------: | :-------------------------: | :-------------------------------------------------------: | :-----------------------------------: |
| **Google 官方 SDK** |           Google            |             1. 系统级深度集成 2. 兼容性保障高             | Android Jetpack、Google Play Services |
|  **硬件厂商 SDK**   | 手机/芯片厂商（华为、高通） |     1. 利用硬件特性（GPU、AI芯片） 2. 设备定制化接口      |    华为 HMS Core、Qualcomm骁龙 SDK    |
| **第三方服务 SDK**  |  互联网公司（腾讯、阿里）   | 1. 解决业务需求（支付/登录） 2. 需适配碎片化 Android 系统 |        微信 SDK、友盟统计 SDK         |
|  **开源社区 SDK**   |         开发者社区          |           1. 免费可定制 2. 迭代快但稳定性需验证           |       RxJava、Coil（图片加载）        |

## **四、特殊类型 SDK**

#### **1. 插件化 SDK**

- **目标**：实现动态加载（热修复、模块热更新）
- **技术**：使用 `ClassLoader` 或字节码框架（如 RePlugin、VirtualAPK）
- **场景**：大型应用功能模块解耦

#### **2. Flutter/React Native 插件**

- **目标**：为跨平台框架提供原生能力桥接
- **实现**：通过 `MethodChannel` (Flutter) 或 `NativeModule` (React Native) 暴露接口
- **示例**：`flutter_map`（地图插件）、`react-native-camera`（相机）

## **五、开发关键差异对比**

|    **维度**    |  **基础库 SDK** (如 Retrofit)   |     **业务服务 SDK** (如 微信支付 SDK)     |
| :------------: | :-----------------------------: | :----------------------------------------: |
| **API 抽象层** | 技术功能接口（`@GET("/user")`） |     业务语义接口（`payOrder(amount)`）     |
|  **资源占用**  |     轻量级（通常 < 100KB）      |     较重（可能含资源文件、SO库，>2MB）     |
|  **依赖约束**  |    允许依赖底层库（OkHttp）     |      需严格控制依赖（避免与宿主冲突）      |
| **兼容性要求** |        支持 Android 4.0+        | 需适配厂商定制系统（华为 EMUI、小米 MIUI） |
| **初始化配置** |   简单（通常无需显式初始化）    |      复杂（需 AppKey、Context 配置）       |

------

### ▶ 总结：选择 SDK 类型的决策链

1. **明确目标**
   → 是提供通用能力（如图片加载）还是业务服务（如支付）？
2. **技术选型**
   → 是否需要资源/UI？选 AAR；纯逻辑？选 JAR；高性能？选 Native
3. **适配范围**
   → 是否覆盖老旧系统？是否处理厂商兼容性？
4. **依赖管理**
   → 避免引入冲突库，优先用 `implementation` 而非 `api`
5. **维护成本**
   → 官方 SDK > 开源 SDK > 自研 SDK

**示例决策流程**：

> 开发一个 **「人脸识别 SDK」**
> → 需调用摄像头：选 ​**​AAR​**​（包含 Camera 界面）
> → 需模型推理：集成 ​**​Native SDK​**​（.so 库处理 AI 计算）
> → 为兼容低端机：提供 ​**​纯云识别模式​**​（网络请求 JAR）
> → 避免资源冲突：资源文件名加前缀 `face_`

## 六、使用方式

### 1、本地直接依赖

```groovy
// app/build.gradle
implementation(project(":simplelogger"))
```

### 2、打包成AAR文件（注意：必须保留consumer-rules.pro规则文件，并且在gradle中设置consumerProguardFiles("consumer-rules.pro")）

#### 方法一：右侧点击gradle，选择simplelogger->Tasks->other->assembleRelease

![](https://raw.githubusercontent.com/spxcc/MyImages/main/img/Snipaste_2025-07-29_09-06-18.png)

#### 方法二：右侧点击gradle，选择simplelogger->Tasks->build->assemble 或者 simplelogger->Tasks->build->build都可以

![](https://raw.githubusercontent.com/spxcc/MyImages/main/img/Snipaste_2025-07-29_09-42-29.png)

#### 方法三：

- 右上角选择要打包的model
- 左侧Build Variants选择release(一般不会选debug)
- 在左上方选择Build->Assemble Module "Logtest.simplelogger"
- **AAR文件生成路径**/simplelogger/build/outputs/aar/

![](https://raw.githubusercontent.com/spxcc/MyImages/main/img/Snipaste_2025-07-29_00-42-35.png)

#### 如何使用AAR?

- 1、将 `.aar` 文件放入项目中某个模块的 `libs/` 目录下（没有就新建）

```
例如：app/libs/simplelogger.aar
```

- 2、在该模块的 `build.gradle` 中添加以下内容：(**注意：`name: 'simplelogger'` 要和 `.aar` 文件名匹配，不带 `.aar` 扩展名**)

```kotlin
android {
    ...
    repositories {
        flatDir {
            dirs 'libs'  // 指定 .aar 文件所在的目录
        }
    }
}

dependencies {
    implementation(name: 'simplelogger', ext: 'aar') // 文件名不需要加扩展名
}
```

### 3、发布到JitPack（推荐）

- **在 `simplelogger/build.gradle` 中添加（需要打包的Module）**：

```groovy
group = "com.github.yourname" // your GitHub 用户名，不能写错！
version = "1.0.0"             // 你要发布的版本号
```

- **发布版本（tag）**

  ```bash
  git tag v1.0.0
  git push origin v1.0.0
  ```

- **如何使用？**

  - 在根项目的 `settings.gradle` 或 `build.gradle` 中添加：

  ```kotlin
  dependencyResolutionManagement {
      repositories {
          google()
          mavenCentral()
          maven { url "https://jitpack.io" }
      }
  }
  ```

  - 在 App 中添加依赖

  ```kotlin
  implementation("com.github.yourname:SimpleLogger:1.0.0")
  ```

### 4、发布到Maven Central（比较繁琐个人项目不推荐）

### 5、打包成Jar包

