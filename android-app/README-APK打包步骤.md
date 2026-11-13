# 数学小勇士：李子大闯关 —— 安卓 APK 套壳工程说明

本工程是一个原生 Android WebView 套壳工程：把离线网页游戏装进 `assets`，打开 APP 即用全屏网页运行，不依赖网络、不依赖 Node.js。

## 一、工程结构

```
android-app/
├── settings.gradle            # 工程与仓库配置
├── build.gradle               # 顶层插件版本（AGP 8.5.2）
├── gradle.properties
└── app/
    ├── build.gradle           # 模块配置：compileSdk 34 / minSdk 24
    ├── proguard-rules.pro
    └── src/main/
        ├── AndroidManifest.xml
        ├── java/com/lizi/mathquest/MainActivity.java   # WebView 主页面
        ├── res/
        │   ├── values/strings.xml      # APP 名称
        │   ├── values/themes.xml       # 主题/状态栏颜色
        │   └── drawable/ic_launcher.xml # 应用图标（李子矢量图）
        └── assets/
            ├── index.html                 # 游戏网页（必须为最新版）
            ├── icon.svg
            └── manifest.webmanifest
```

包名：`com.lizi.mathquest`；应用名：李子大闯关。

## 二、构建环境要求（在出 APK 的电脑上准备）

1. **JDK 17**（AGP 8.5 要求；Android Studio 自带 JDK，一般无需单独安装）
2. **Android Studio** Hedgehog（2023.1）或更新版本
3. **Android SDK** 34（在 Studio 的 SDK Manager 下载）
4. **Gradle 8.7 或更新**（由 Android Studio 自动匹配；命令行构建时本机需已安装对应 Gradle）
5. 首次打开工程时 Studio 会自动下载对应 Gradle 与依赖，需联网

> 本机当前未检测到以上环境，因此未能在此直接编译 APK；在装好环境的电脑上按下面步骤即可出包。

## 三、准备游戏网页

1. 确认 `app/src/main/assets/index.html` 是最新版游戏。
2. 若该文件缺失：把桌面目录 `五年级数学互动答题/五年级数学闯关答题.html` 复制一份，重命名为 `index.html`，放入 `app/src/main/assets/`。
3. `icon.svg`、`manifest.webmanifest` 已随工程就位；如更新了网页图标，同步替换 `icon.svg` 即可。

> 网页内的脚本、样式、题目全部内联在 `index.html`，无需额外联网资源。

## 四、用 Android Studio 构建 APK（推荐）

1. 打开 Android Studio → **Open** → 选择本目录 `android-app`。
2. 等待 **Gradle Sync** 完成（首次会下载依赖，耐心等待；若提示 SDK 缺失，按提示安装 SDK 34）。
3. 菜单 **Build → Build App Bundle(s) / APK(s) → Build APK(s)**。
4. 构建成功后弹窗定位到产物：
   `app/build/outputs/apk/debug/app-debug.apk`
5. 把 `app-debug.apk` 传到安卓手机点击安装即可（debug 包可直接侧载安装）。

## 五、命令行构建（可选）

在工程根目录执行（需本机已装 Gradle 或由 Studio 生成过 wrapper）：

```
gradle assembleDebug
```

产物位置同上：`app/build/outputs/apk/debug/app-debug.apk`。

## 六、安装到手机

- **USB 线**：手机开启「开发者选项 → USB 调试」，连接电脑后执行：
  `adb install app/build/outputs/apk/debug/app-debug.apk`
- **传输安装**：把 apk 发到手机（微信/QQ/网盘/数据线），手机上点击安装；首次安装需允许「安装未知来源应用」。

## 七、发布正式版（可选）

1. 菜单 **Build → Generate Signed App Bundle / APK**。
2. 新建或选择签名密钥（keystore），设置别名与密码并妥善保管。
3. 选择 **Release** 构建，生成可上架/分发的签名包；release 包默认未开启混淆。

## 八、常见问题

- **打开白屏 / 网页不运行**：确认 `assets/index.html` 存在且未改名；WebView 已开启 JavaScript 与文件访问。
- **中文乱码或路径异常**：工程路径、`index.html` 内部均使用 UTF-8；避免把工程放在含特殊符号的目录。
- **Gradle Sync 报 SDK/版本错误**：在 SDK Manager 安装 API 34；JDK 需为 17。
- **想改 APP 名称**：修改 `res/values/strings.xml` 的 `app_name`。
- **想改图标**：替换 `res/drawable/ic_launcher.xml`，或放入标准 PNG 多尺寸图标到 `res/mipmap-*`。
- **想改包名/版本号**：在 `app/build.gradle` 的 `applicationId`、`versionCode`、`versionName` 修改。

## 九、云端打包（本机不装 Android Studio）

有三条纯云端 / 轻客户端路线，按推荐顺序如下。

### 方案一：GitHub Actions 云构建（推荐，复用本工程源码，免费）

工程内已内置流水线配置：`.github/workflows/build-apk.yml`，云端自动装好 JDK 17、SDK 34、Gradle 8.7 并出包，本机不需要任何安卓构建环境。

1. 注册 / 登录 GitHub，新建一个仓库（公开仓库 Actions 额度更充裕；私有仓库有每月免费分钟数限制）。
2. 把本 `android-app` 目录的内容作为**仓库根**上传（仓库里直接能看到 `settings.gradle`、`app/`、`.github/`）。可用网页「Add file → Upload files」上传，或本地 `git push`。
3. 推送后 Actions 会自动运行；也可在仓库 **Actions → Build Android APK → Run workflow** 手动触发。
4. 运行完成后进入本次任务页，在 **Artifacts → app-debug-apk** 下载压缩包，解压即得 `app-debug.apk`，可直接侧载安装。
5. 说明：
   - 流水线构建的是 debug 包，已用调试签名，手机允许「未知来源」即可安装。
   - 若要上架 / 正式分发的 release 包，需在仓库 Secrets 配置签名 keystore 并把任务改为 `assembleRelease`，步骤同本地签名。
   - 若私有仓库 Actions 被禁用，可在仓库 **Settings → Actions → General** 允许工作流。

### 方案二：HBuilderX 云打包（国内常用，内嵌本地网页，需装客户端 + DCloud 账号）

这是另一套打包结构，**不复用本 Gradle 源码**，但用同一份 `index.html` 网页。

1. 下载安装 HBuilderX，注册并登录 DCloud 账号。
2. 新建「5+App」项目，把 `app/src/main/assets/` 下的 `index.html`、`icon.svg`、`manifest.webmanifest` 复制到项目 `www/` 目录。
3. 在项目 `manifest.json` 里设置应用名称「李子大闯关」、应用图标、启动配置，关闭用不到的权限。
4. 菜单 **发行 → 原生App-云打包**，平台选 Android，证书可选 DCloud 公共测试证书，提交后由云端编译。
5. 免费用户每日有打包次数限制、热门时段需排队；完成后在下载列表取 APK。

### 方案三：PWABuilder（适合已部署到公网的 PWA）

1. 先把 `index.html`、`manifest.webmanifest`、`icon.svg` 部署到一个 HTTPS 公网地址（本游戏为离线单文件，未配 Service Worker，走此方案需先上线）。
2. 打开 pwabuilder.com，输入站点 URL，选择打包 Android 包，按提示下载。
3. 该方案更适合有域名、长期在线的 PWA；纯本地离线小游戏一般优先选方案一或方案二。

### 路线对比

| 路线 | 是否复用本工程源码 | 本机要装什么 | 是否需要公网部署 | 费用 |
| --- | --- | --- | --- | --- |
| GitHub Actions | 是 | 无（网页上传即可） | 否 | 公开仓库免费 |
| HBuilderX 云打包 | 否（用同一份网页） | HBuilderX 客户端 | 否 | 免费次数 + 排队，加速付费 |
| PWABuilder | 否 | 无 | 是（需 HTTPS 站点） | 免费 |

