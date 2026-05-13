# RPush-Pro

> 一个用纯 Swift 编写、可运行在 macOS 的 APNs 推送测试工具。
> 支持 **证书 (.cer)** 与 **Token (.p8)** 两种鉴权方式，配套现代化 UI、JSON 编辑器、推送历史等开发者友好特性。

本项目基于 [nevermore-imba/RPush](https://github.com/nevermore-imba/RPush) 二次开发，修复了若干生产环境会触发的 bug，并对整套 UI 做了重构。原仓库版权与 Apache-2.0 协议保留。

---

## ✨ 相对原版的改进

### 🛠 关键修复

- **修复 `TooManyProviderTokenUpdates`**：原实现每次 `send()` 都会重新签发 JWT，远超 APNs 要求的「最少 20 分钟刷新一次」窗口。现在按 `(keyId, teamId)` 缓存 `AuthenticationToken`，仅在 token 已签发超过 50 分钟时才刷新，天然落在 Apple 的 20 分钟下限和 60 分钟上限之间。
- **修正 JWT 的 `iat` 声明**：原版把 `iat` 错误地设置为 `now + 600s`（未来时间）；新版严格遵循 Apple 规范，使用签发当下时间。
- **修复 `JWTDecoder` 解析字段**：APNs JWT 没有 `exp` 字段，原版读 `exp` 导致 `isExpired` 永远为 `true`。新版改读 `iat`，按 50 分钟安全寿命判定过期。

### 🎨 全新 UI

- **顶部环境 tab**：`开发环境 / 生产环境` 用胶囊样式 SegmentedControl + `NSVisualEffectView` 毛玻璃，深浅模式自适应。
- **默认 Token 鉴权方式**：现代推送的推荐路径，启动即可用。
- **现代化表单**：所有表单控件用代码 + `NSStackView` 重构，自适应窗口大小，告别绝对定位的拥挤布局。
- **真正的 JSON 编辑器**：多行 `NSTextView` + 等宽字体，配套：
  - 模板下拉（Alert / Badge / Sound / Rich Alert / Background Push）
  - 一键 **格式化** / **压缩**
  - 实时 JSON 校验 + 字节数统计
- **Device Token 实时格式化**：粘贴即按 8 位分组显示，无需失焦触发；旁边带复制按钮。
- **状态栏**：连接指示灯（灰/绿/蓝）+ 文字 + spinner，推送过程中按钮自动禁用。
- **内联 banner 替代 NSAlert**：成功/警告/错误以顶部短横幅展示 2.4 秒后自动消失，不再阻塞操作。
- **快捷键**：`⌘↵` 发送、`⌘L` 清空日志。
- **推送历史侧边栏**：自动记录最近 100 条推送（成功/失败、Token、Payload、时间），双击回填到发送页面。

### 📦 工程改进

- **沿用 Apache-2.0**，并在文件头与本 README 中标注上游来源。
- 部署目标 macOS 10.15+（Swift Runtime 系统化，`.app` 体积仅 ~500 KB）。
- 仓库自带 `dist/` 打包脚本路径，可直接用 `xcodebuild + hdiutil` 输出 `.dmg`。

---

## 📸 截图

<!-- 截图请在新版 UI 下重新捕获，原版截图与当前 UI 已不一致。 -->

| 鉴权方式 | 截图 |
|---|---|
| 证书 (.cer) | `RPush/screenshots/certificate_based_screenshot.png`（旧版，待更新） |
| Token (.p8) | `RPush/screenshots/token_auth_based_screenshot.png`（旧版，待更新） |

---

## 🚀 使用方法

### 1. 下载

从 [Releases](https://github.com/<your-username>/RPush-Pro/releases) 页下载最新 `.dmg`，挂载后将 `RPush.app` 拖到 `Applications`。

> 当前 Release 为 ad-hoc 签名（未走 Notarization），首次打开请右键 → 打开 → 在弹窗里再点「打开」绕过 Gatekeeper。
> 如果遇到「已损坏」提示，运行 `xattr -dr com.apple.quarantine /Applications/RPush.app` 即可。

### 2. 选择鉴权方式

#### 🔹 Token (.p8) 模式（默认，推荐）

1. 顶部 tab 选择推送环境（开发 / 生产）
2. 填写 `Bundle ID`、`Key ID` (10 位)、`Team ID` (10 位)
3. 在「选择 P8 文件」下拉里选取你的 `.p8` 私钥
4. 粘贴目标设备的 `Device Token`
5. 编辑 / 选择 Payload 模板，可按 `⌘↵` 直接发送

参考：[Establishing a Token-Based Connection to APNs](https://developer.apple.com/documentation/usernotifications/establishing-a-token-based-connection-to-apns)

#### 🔹 证书 (.cer) 模式

1. 切换到「证书 (.cer)」tab
2. 在「选择推送证书」下拉里选取已导入钥匙串的证书，或选「从文件中选择…」上传 `.cer`
3. 粘贴 `Device Token`、编辑 Payload
4. 点击「连接服务器」与 APNs 建立 TLS 长连接
5. 点击「推送消息」（或 `⌘↵`）发送

参考：[Establishing a Certificate-Based Connection to APNs](https://developer.apple.com/documentation/usernotifications/establishing-a-certificate-based-connection-to-apns)

### 3. 历史回填

侧边栏「历史推送」会保存最近 100 条记录，双击任意一条即可把所有字段（Token / Payload / Bundle ID / Key ID / Team ID / 鉴权方式 / 环境）一键回填到发送页面，方便复测。

---

## 🧰 从源码构建

需求：Xcode 12 +、macOS 10.15 +。

```bash
git clone https://github.com/<your-username>/RPush-Pro.git
cd RPush-Pro
open RPush/RPush.xcodeproj
```

或在命令行直接打包 ad-hoc 签名的 `.dmg`：

```bash
# 1) Build Release
xcodebuild -project RPush/RPush.xcodeproj -scheme RPush \
    -configuration Release -destination 'platform=macOS' \
    -derivedDataPath build/DD \
    CODE_SIGN_STYLE=Manual CODE_SIGN_IDENTITY="-" \
    PROVISIONING_PROFILE_SPECIFIER="" DEVELOPMENT_TEAM="" build

# 2) Stage and create dmg
APP=build/DD/Build/Products/Release/RPush.app
STAGE=build/dmg-stage
rm -rf "$STAGE" && mkdir -p "$STAGE" dist
cp -R "$APP" "$STAGE/"
ln -s /Applications "$STAGE/Applications"
hdiutil create -volname "RPush" -srcfolder "$STAGE" -ov -format UDZO dist/RPush.dmg
```

### 正式分发（Developer ID + Notarization）

如需对外分发不弹 Gatekeeper 警告的版本：

1. 在 Apple Developer 后台创建 **Developer ID Application** 证书并导入钥匙串。
2. 用 `xcodebuild archive + exportArchive`，`ExportOptions.plist` 里 `method` 写 `developer-id`。
3. `xcrun notarytool submit dist/RPush.dmg --apple-id <id> --team-id <team> --wait`。
4. `xcrun stapler staple dist/RPush.dmg`。

---

## 🏗 架构概览

```
RPush/
├── ViewController.swift        # 主推送页（纯代码 UI、新版重构）
├── MainSplitViewController.swift  # 主分栏（侧边栏 + 内容）
├── SidebarViewController.swift    # 侧边栏导航
├── HistoryViewController.swift    # 推送历史
├── PushHistoryManager.swift       # 历史持久化（UserDefaults）
├── AuthenticationToken.swift      # JWT 缓存 / 刷新（修复 TooManyProviderTokenUpdates）
├── JSON Web Token/
│   ├── JWT.swift                  # 签发（修复 iat 语义）
│   ├── JWTDecoder.swift           # 解析（按 iat 判过期）
│   ├── ECPrivateKey.swift / ASN1.swift / ECKeyData.swift
├── P8.swift                       # .p8 文件解析
├── Sec.swift                      # 钥匙串证书读取
├── Socket.swift                   # 证书模式下与 APNs 的 TLS 长连接
└── Fomatter/                      # Legacy / Enhanced 二进制协议（证书模式）
```

---

## 🤝 致谢

- 原仓库：[nevermore-imba/RPush](https://github.com/nevermore-imba/RPush) — 提供基础架构与初始实现
- [SmartPush](https://github.com/shaojiankui/SmartPush) — 思路启发
- [appstoreconnect-swift-sdk](https://github.com/AvdLee/appstoreconnect-swift-sdk) — JWT 签名实现参考
- [iOS 远程推送 — APNs 详解](https://blog.csdn.net/weixin_37409570/article/details/96575120) — APNs 协议解析

---

## 📄 License

[Apache License 2.0](LICENSE) — 沿用上游协议。
