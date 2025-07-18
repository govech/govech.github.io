---
title: productFlavors的使用
date: 2025-07-19
---



在 Android Gradle 插件中（通常是 `com.android.application` 插件），`productFlavors` 是一个强大的概念，用于在同一项目中创建和管理应用的不同变体。这些变体共享核心代码和资源，但在某些方面（如包名、目标 API、版本、包含的特性、资源、配置常量等）可以存在差异。常见的用例包括：

1. **免费版和付费版：** 共享大部分代码，但付费版解锁特定功能。
2. **特定客户的定制版本：** 为不同客户提供品牌、主题或功能略有差异的应用。
3. **不同 API/平台版本：** 针对不同级别的 API 进行优化或提供备用实现。
4. **不同的分发渠道：** 为应用商店、企业分发等设置不同的配置（例如渠道 ID）。
5. **开发/测试环境与生产环境：** 使用不同的后端 URL、日志级别等。

**使用方法详解：**

1. **定义产品风味 (Define Flavors):**
   在 `app` 模块的 `build.gradle` (`build.gradle.kts` 对于 Kotlin DSL) 文件中，位于 `android { }` 块内，添加 `productFlavors { }` 块。在此块内，使用 `flavorDimensions` 定义维度（可选但推荐），然后声明具体的 `flavor`。

   ```groovy
   // Groovy DSL (build.gradle)
   android {
       ...
       flavorDimensions "version", "environment" // 定义两个维度：版本和环境
       productFlavors {
           // 定义 "version" 维度下的风味
           free {
               dimension "version"
               // 覆盖应用的 applicationId（包名）或添加后缀
               applicationIdSuffix ".free"
               // 定义资源常量 (BuildConfig.FLAVOR_version 会是 "free", 可以用在代码里判断)
               buildConfigField "String", "FLAVOR_version", "\"free\""
               // 替换清单文件中的占位符（如果需要）
               manifestPlaceholders = [appLabelSuffix: " (Free)"]
           }
           paid {
               dimension "version"
               applicationIdSuffix ".paid"
               buildConfigField "String", "FLAVOR_version", "\"paid\""
               manifestPlaceholders = [appLabelSuffix: " (Pro)"]
           }
   
           // 定义 "environment" 维度下的风味
           dev {
               dimension "environment"
               applicationIdSuffix ".dev"
               // 定义不同环境的 API URL
               buildConfigField "String", "API_BASE_URL", "\"https://dev.api.example.com/\""
               // 可能启用调试特性或更详细的日志
               resValue "string", "app_name_suffix", " Dev"
           }
           prod {
               dimension "environment"
               buildConfigField "String", "API_BASE_URL", "\"https://api.example.com/\""
               resValue "string", "app_name_suffix", ""
           }
       }
       ...
   }
   ```

   

   ```kotlin
   // Kotlin DSL (build.gradle.kts)
   android {
       ...
       flavorDimensions += listOf("version", "environment")
       productFlavors {
           create("free") {
               dimension = "version"
               applicationIdSuffix = ".free"
               buildConfigField("String", "FLAVOR_version", "\"free\"")
               manifestPlaceholders["appLabelSuffix"] = " (Free)"
           }
           create("paid") {
               dimension = "version"
               applicationIdSuffix = ".paid"
               buildConfigField("String", "FLAVOR_version", "\"paid\"")
               manifestPlaceholders["appLabelSuffix"] = " (Pro)"
           }
   
           create("dev") {
               dimension = "environment"
               applicationIdSuffix = ".dev"
               buildConfigField("String", "API_BASE_URL", "\"https://dev.api.example.com/\"")
               resValue("string", "app_name_suffix", " Dev")
           }
           create("prod") {
               dimension = "environment"
               buildConfigField("String", "API_BASE_URL", "\"https://api.example.com/\"")
               resValue("string", "app_name_suffix", "")
           }
       }
       ...
   }
   ```

2. **风味维度的作用 (Flavor Dimensions):**

   - 维度不是强制性的，但**强烈推荐**使用它们来组织风味。它创建了一个风味“矩阵”。
   - 每个风味都必须分配到一个维度。
   - Gradle 会组合所有维度的风味以及 `buildTypes` 来生成最终的 **构建变体 (Build Variants)**。
   - 在这个例子中，维度是 `version`（`free` 或 `paid`）和 `environment`（`dev` 或 `prod`）。
   - 可能的组合变体（假设buildTypes有debug和release）：
     - `freeDevDebug`
     - `freeDevRelease`
     - `freeProdDebug`
     - `freeProdRelease`
     - `paidDevDebug`
     - `paidDevRelease`
     - `paidProdDebug`
     - `paidProdRelease`

3. **风味特定配置 (Flavor-Specific Configuration):**
   在上面的代码片段中，你可以在每个 `flavor` 块内设置各种属性：

   - **`applicationId` / `applicationIdSuffix`:** 控制包名。`applicationId` 直接设置完整包名，`applicationIdSuffix` 在默认包名后附加后缀（非常常用）。这使得同一设备上可以同时安装免费版和付费版。
   - **`buildConfigField(type, name, value)`:** 向 `BuildConfig` 类添加自定义常量。这些常量可以在你的 Java/Kotlin 代码中直接使用（如 `BuildConfig.API_BASE_URL`）。
   - **`resValue(type, name, value)`:** 生成一个在运行时可以访问的新资源值（如 `@string/app_name_suffix`）。
   - **`manifestPlaceholders`:** 一个键值对映射，用于替换 AndroidManifest.xml 文件中的占位符（如 `${appLabelSuffix}`）。
   - **`dimension`:** 指定该风味属于哪个维度（**必须设置**）。
   - **`minSdkVersion`, `targetSdkVersion`, `versionCode`, `versionName` 等:** 可以覆盖默认设置或其它维度的设置（但需小心使用）。
   - **`proguardFiles`, `consumerProguardFiles`:** 指定风味特定的混淆规则文件。
   - **`signingConfig`:** 可以为特定风味配置不同的签名（例如马甲包）。

4. **风味特定源集 (Flavor-Specific Source Sets):**

   - 默认情况下，所有风味的代码和资源都放在 `main/` 目录下。
   - 你可以为特定风味或维度的组合创建专属的源代码和资源目录：
     - `src/[flavorName]/java/` - 风味特定 Java 代码
     - `src/[flavorName]/kotlin/` - 风味特定 Kotlin 代码
     - `src/[flavorName]/res/` - 风味特定资源（图片、布局、字符串等）
     - `src/[flavorName]/AndroidManifest.xml` - 风味特定的清单文件（通常会继承 main 的，然后添加/覆盖特定元素）
   - 对于维度组合（例如 `freeProd`），目录名为 `src/[firstDimension][secondDimension]/...`（如 `src/freeProd/`）。
   - **合并规则**：
     - 构建变体时，Gradle 会合并以下来源的资源、清单和代码（优先级从高到低）：
       1. Build Type（例如 `src/debug/`)
       2. Product Flavor(s)（如果有多个维度，按维度顺序合并）
       3. Main (`src/main/`)
       4. Library Dependencies
     - 这意味着风味目录下的文件可以**覆盖** `main/` 中的同名文件。代码可以相互补充。
   - **示例：** 在 `src/free/res/values/strings.xml` 中定义一个名为 `app_name` 的字符串会覆盖 `main/res/values/strings.xml` 中的同名字符串。`src/paid/java/com/example/PaidFeature.java` 可以包含只在付费版中使用的类。

5. **选择构建变体 (Selecting Build Variants):**

   - 在 Android Studio 的左侧（通常），打开 **Build Variants** 工具窗口（可以通过 View -> Tool Windows -> Build Variants 打开）。
   - 在此窗口中，你会看到一个下拉列表，列出了项目中所有模块的所有可用构建变体（如 `freeDevDebug`, `paidProdRelease`）。
   - 选择你当前想要构建、运行或调试的变体。这决定了 Android Studio 将使用哪个配置来编译和安装应用。

6. **构建特定变体 (Building Specific Variants):**

   - **通过 Android Studio:** 使用工具栏的运行按钮或 `Build -> Make Project` / `Build Bundle(s) / APK(s)`。它会使用 **Build Variants** 窗口中选择的变体。
   - 通过 Gradle 命令行:
     - 构建所有变体: `./gradlew assemble` (Linux/macOS) or `gradlew assemble` (Windows)
     - 构建特定变体 (如所有 `free` 变体): `./gradlew assembleFree` (会构建 `freeDevDebug`, `freeDevRelease`, `freeProdDebug`, `freeProdRelease`)
     - 构建更具体的变体 (如 `freeProdRelease`): `./gradlew assembleFreeProdRelease`
     - 安装特定变体到设备: `./gradlew installFreeProdDebug`

**关键点总结：**

- **`productFlavors`** 用于定义同一应用的不同版本（变体）。
- **`flavorDimensions`**（推荐）用于将风味分组，创建风味组合矩阵。
- 每个 **构建变体 (Build Variant)** 是 `buildType` 和 `productFlavors`（所有维度）的组合（如 `freeProdRelease`）。
- 使用 **风味特定源集目录** (`src/[flavorName]/`) 覆盖资源、添加/覆盖代码和清单设置。
- 使用 `buildConfigField`, `resValue`, `manifestPlaceholders`, `applicationIdSuffix` 等配置风味特定的属性和常量。
- 在 **Build Variants** 窗口或通过 Gradle 任务（如 `assemble[VariantName]`）选择、构建、运行或安装特定的变体。
- **优先覆盖原则：** 构建变体时，较低优先级目录（`main`）的配置会被较高优先级目录（风味、构建类型）的同名配置覆盖。

**最佳实践：**

- **明智地命名：** 使用清晰描述风味目的的维度名和风味名（如 `customer`、`tier`、`env`）。
- **充分利用 `buildConfigField` 和 `resValue`：** 避免在风味代码中进行大量的硬编码环境判断。
- **在 `applicationId` 中使用后缀而非完全覆盖：** 这样更容易维护基础包名。
- **小心依赖：** 确保不同风味的依赖配置不会导致冲突或非预期行为。
- **使用维度：** 除非你只有一个非常简单的风味需求，否则使用维度能让你的配置结构更清晰、灵活。



以下是一个详细的 `productFlavors` 示例（Groovy DSL），每行代码都带有注释说明：

```groovy
android {
    // 基本配置
    compileSdkVersion 31
    defaultConfig {
        applicationId "com.example.myapp" // 基础包名
        minSdk 21
        targetSdk 31
        versionCode 1
        versionName "1.0"
    }

    // 定义两个风味维度：应用类型（免费/付费）和运行环境（开发/生产）
    flavorDimensions "tier", "env"
    
    // 产品风味配置块
    productFlavors {
        
        /***** 应用类型维度（免费版/付费版）*****/
        
        free {
            dimension "tier"  // 指定所属维度
            applicationIdSuffix ".free" // 包名后缀：com.example.myapp.free
            versionNameSuffix "-free"  // 版本名后缀：1.0-free
            
            // 在BuildConfig类中添加常量
            buildConfigField "boolean", "IS_PREMIUM", "false" 
            
            // 在资源文件中添加值（可在代码中通过R.string.app_name获取）
            resValue "string", "app_name", "@string/app_name_free"
            
            // 清单文件占位符，在AndroidManifest.xml中可使用${adAppId}
            manifestPlaceholders = [adAppId: "ca-app-pub-3940256099942544~3347511713"]
        }

        paid {
            dimension "tier" // 必须指定维度
            applicationIdSuffix ".pro" // 包名后缀：com.example.myapp.pro
            versionNameSuffix "-pro"  // 版本名后缀：1.0-pro
            buildConfigField "boolean", "IS_PREMIUM", "true"
            resValue "string", "app_name", "@string/app_name_pro"
            manifestPlaceholders = [adAppId: "no_ads"] // 付费版无广告
        }

        /***** 环境维度（开发环境/生产环境）*****/
        
        dev {
            dimension "env" // 指定环境维度
            applicationIdSuffix ".dev" // 最终包名：com.example.myapp.[tier].dev
            
            // 定义不同环境的API地址
            buildConfigField "String", "API_BASE_URL", "\"https://dev.api.example.com/\""
            
            // 添加构建变体标识到应用名称
            resValue "string", "env_suffix", " (Dev)"
            
            // 启用调试功能
            buildConfigField "boolean", "DEBUG_FEATURES", "true"
        }

        prod {
            dimension "env" // 环境维度
            buildConfigField "String", "API_BASE_URL", "\"https://api.example.com/\""
            resValue "string", "env_suffix", ""
            buildConfigField "boolean", "DEBUG_FEATURES", "false" // 生产环境关闭调试功能
        }
    }

    // 构建类型配置（所有风味共享）
    buildTypes {
        debug {
            debuggable true
            minifyEnabled false
        }
        release {
            debuggable false
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

// 风味特定依赖配置（可选）
dependencies {
    // 免费版依赖广告SDK
    freeImplementation 'com.google.android.gms:play-services-ads:21.5.0'
    
    // 付费版依赖高级功能模块
    paidImplementation project(":premium-features")
    
    // 开发环境专用调试工具
    debugImplementation 'com.facebook.stetho:stetho:1.5.1'
}
```

### 对应目录结构说明：

```
src/
├── main/                  # 公共资源
│   ├── java/              # 公共代码
│   ├── res/               # 公共资源
│   └── AndroidManifest.xml
│
├── free/                  # 免费版专属
│   ├── res/
│   │   └── values/
│   │       └── strings.xml  # 覆盖应用名称 <string name="app_name">App Free</string>
│   └── java/              # 免费版特定功能实现
│
├── paid/                  # 付费版专属
│   ├── res/
│   │   └── drawable/      # 付费版专属图标
│   └── java/              # 付费功能解锁代码
│
├── dev/                   # 开发环境配置
│   └── java/
│       └── DebugUtil.java # 开发环境专用调试工具
│
├── freeDev/               # 免费+开发环境组合特有代码
│   └── java/
│       └── AdManager.java # 免费开发版广告管理
│
└── prodPaid/              # 付费+生产环境组合资源
    └── res/
        └── values/
            └── strings.xml # 覆盖付费生产版文案
```

### 实际使用示例：

```java
// 在代码中访问风味配置
public class ApiClient {
    private static final String BASE_URL = BuildConfig.API_BASE_URL;
    
    public void fetchData() {
        if (BuildConfig.IS_PREMIUM) {
            // 付费版专享功能
            PremiumFeatures.unlock();
        }
    }
}
```

### 在AndroidManifest.xml中使用占位符：

```xml
<application
    android:label="${app_name}" <!-- 由resValue动态注入 -->
    ... >
    
    <!-- 广告SDK配置（免费版生效） -->
    <meta-data
        android:name="com.google.android.gms.ads.APPLICATION_ID"
        android:value="${adAppId}" /> <!-- 由manifestPlaceholders注入 -->
</application>
```

### 构建变体示例：

1. **freeDevDebug**
   - 包名: `com.example.myapp.free.dev`
   - 应用名称: "MyApp Free (Dev)"
   - API地址: `https://dev.api.example.com/`
   - 包含广告，启用调试功能
2. **paidProdRelease**
   - 包名: `com.example.myapp.pro`
   - 应用名称: "MyApp Pro"
   - API地址: `https://api.example.com/`
   - 无广告，启用代码混淆
3. **freeProdDebug**
   - 包名: `com.example.myapp.free`
   - 应用名称: "MyApp Free"
   - API地址: `https://api.example.com/`
   - 包含广告但禁用调试功能

### 操作方式：

1. 在Android Studio的 **Build Variants** 窗口选择：

   ```
   Module: app
   Active Build Variant: freeDevDebug
   ```

2. 或使用Gradle命令构建：

   ```bash
   ./gradlew assembleFreeDevDebug  # 构建特定变体
   ./gradlew installFreeDevDebug   # 安装到设备
   ./gradlew assembleProdRelease    # 构建所有生产环境发布包
   ```

通过此配置，可实现：

- 免费版包含广告/付费版无广告
- 开发环境与生产环境使用不同服务端点
- 每个变体有独立的应用名称和包名
- 特定变体启用调试功能
- 按需包含依赖库（如仅免费版依赖广告SDK）