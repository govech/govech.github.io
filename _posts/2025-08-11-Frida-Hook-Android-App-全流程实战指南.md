---
title: Frida Hook Android App 全流程实战指南
date: 2025-08-11
---

如何从 0 到实战使用 **Frida** 动态 Hook Android 应用?

------

## 1. Frida 基本原理

Frida 是一款 **动态插桩（Dynamic Instrumentation）** 工具，可以在 App 运行时注入 JavaScript 代码，从而：

- Hook/修改 **方法参数** 和 **返回值**
- 调用 App 内部方法
- 绕过安全检测（Root 检测、签名校验等）
- 打印/修改内存数据

在 Android 环境下：

1. 在设备端运行 **frida-server**（root 模式或非 root gadget 模式）。
2. 在 PC 端通过 USB/TCP 与设备通信，将 Hook 脚本注入目标进程。

------

## 2. 环境准备

### 2.1 安装 PC 端 Frida

```bash
pip install frida-tools
pip install frida
```

> `frida-tools` 提供命令行工具（frida、frida-ps、frida-trace），`frida` 提供 Python API。

### 2.2 下载并部署 frida-server

1. **查看当前 PC 端 Frida 版本**

```bash
frida --version
```

1. **下载对应版本的 frida-server**
    访问：https://github.com/frida/frida/releases
    下载与 PC 端版本一致的 `frida-server-<version>-android-arm64`（或 arm/x86）。
2. **推送到设备**

```bash
adb push frida-server-16.x.x-android-arm64 /data/local/tmp/frida-server
adb shell "chmod 755 /data/local/tmp/frida-server"
```

1. **启动 frida-server**

```bash
adb shell
su   # 如果有 root
/data/local/tmp/frida-server
```

后台运行：

```bash
nohup /data/local/tmp/frida-server &
```

------

## 3. 查看设备与进程

- 列出已连接设备：

```bash
frida-ls-devices
```

- 列出设备进程：

```bash
frida-ps -U   # -U 表示 USB 设备
```

启动目标 App 后可找到进程名，例如：

```
com.example.targetapp
```

------

## 4. 编写 Hook 脚本

Frida 使用 JavaScript 编写 Hook 逻辑，Java 开发者可直接类比理解。

示例：Hook 实例方法与静态方法

```javascript
Java.perform(function () {
    var TargetClass = Java.use("com.example.targetapp.SomeClass");
    TargetClass.someMethod.implementation = function (arg1, arg2) {
        console.log("原参数:", arg1, arg2);
        var result = this.someMethod(arg1, arg2);
        console.log("原返回值:", result);
        return "被 Hook 修改的返回值";
    };
    TargetClass.staticMethod.implementation = function (arg) {
        console.log("静态方法参数:", arg);
        return this.staticMethod(arg);
    };
});
```

保存为 `hook.js`。

------

## 5. 注入并运行脚本

```bash
frida -U -n com.example.targetapp -l hook.js --no-pause
```

参数说明：

- `-U` → USB 设备
- `-n` → 指定进程名
- `-l` → 加载脚本
- `--no-pause` → 启动后不暂停执行

------

## 6. Python 调用 Frida（进阶）

```python
import frida

def on_message(message, data):
    print("[*] Message:", message)

device = frida.get_usb_device()
session = device.attach("com.example.targetapp")

script_code = """
Java.perform(function () {
    var TargetClass = Java.use("com.example.targetapp.SomeClass");
    TargetClass.someMethod.implementation = function (a, b) {
        send("Hooked! 参数: " + a + ", " + b);
        return "changed";
    };
});
"""

script = session.create_script(script_code)
script.on("message", on_message)
script.load()

input("[*] Press Enter to exit...\n")
```

------

## 7. 常见实战场景

### 7.1 绕过 Root 检测

```javascript
Java.perform(function () {
    var RootCheck = Java.use("com.example.security.RootUtils");
    RootCheck.isDeviceRooted.implementation = function () {
        return false;
    };
});
```

### 7.2 打印加密函数明文

```javascript
Java.perform(function () {
    var AES = Java.use("javax.crypto.Cipher");
    AES.doFinal.overload("[B").implementation = function (data) {
        console.log("加密前数据:", bytesToString(data));
        return this.doFinal(data);
    };
});
```

### 7.3 修改网络请求参数（okhttp3 示例）

可 Hook `okhttp3.Request` 构造方法或 `RequestBody` 相关方法。

------

## 8. 实战案例：抓取登录明文密码

### 场景假设

- 目标 App：`com.example.securelogin`
- 通过反编译发现：

```java
package com.example.securelogin.util;
public class SecurityUtils {
    public static String encryptPassword(String pwd) {
        // AES 加密逻辑
    }
}
```

- 目标：在运行时截获 `pwd` 的明文。

### Hook 脚本 `hook_encrypt.js`

```javascript
Java.perform(function () {
    var SecurityUtils = Java.use("com.example.securelogin.util.SecurityUtils");
    SecurityUtils.encryptPassword.overload("java.lang.String").implementation = function (pwd) {
        console.log("[*] 明文密码: " + pwd);
        var encrypted = this.encryptPassword(pwd);
        console.log("[*] 加密后: " + encrypted);
        return encrypted;
    };
});
```

### 注入运行

```bash
frida -U -n com.example.securelogin -l hook_encrypt.js --no-pause
```

### 运行结果

输入：

```
用户名: test
密码: 123456
```

输出：

```
[*] 明文密码: 123456
[*] 加密后: Q2VjUldZTlNEaGZsZWkz
```

### 应用扩展

- 抓包前打印 HTTPS 明文数据
- 绕过反调试检测
- 动态修改加密参数

------

## 9. 注意事项

- **非 Root 设备** 可用 `frida-gadget` 注入 APK
- 混淆代码需先用 `jadx` / `jeb` 查看方法签名
- Android 12+ 可能需禁用 SELinux 或用 Magisk Hide
- 可用 Objection、frida-detection-bypass 等模块绕过 Frida 检测

------

## 10. 自动 Hook 框架思路

如果需要批量 Hook：

1. 列出所有类和方法
2. 自动生成 Hook 模板
3. 自动注入脚本并输出日志
4. 加入常用绕过（Root、SSL Pinning）模块

这样可极大提升渗透测试与逆向效率。