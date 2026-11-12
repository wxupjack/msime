# Agent Note: 个人 fork 构建触控板修复测试 APK

Status: implemented

## Problem

贡献者希望在手机上验收触控板振动强度修复，暂不创建 PR 或发布版本。本地缺少 Android 与 Rust 工具链，磁盘不足以启动新的完整构建树；上游 `release-android.yml` 即使关闭发布，也依赖正式签名密钥和专用 runner，不能直接在个人 fork 中使用。

## Decision

`android-test-apk.yml` 只在 `wxupjack/msime` 的 `codex/android-trackpad-haptic-apk` 分支推送或手动调用时运行，使用 GitHub 托管的 Ubuntu runner、仓库固定的工具链与原生 `build-apk.sh`。用户选择拼音版，打 arm64 瘦包，复用开发签名回退，不接触发布密钥，不创建 Release、tag 或 PR。工作流权限只有 `contents: read`。

修复留在 `codex/android-trackpad-haptic-strength`，构建分支从它派生；以后向上游提交修复 PR 时不带个人构建配置。APK 以 Actions artifact 保留 14 天，附构建提交、运行链接与 SHA-256。拼音版与完整版包名不同，可以并行安装；同包名正式版不能被开发签名覆盖。

## Alternatives considered

- 复用发布工作流并设 `publish=false`：可以沿用全部发布校验，但该工作流仍强制要求正式签名，并构建所有版本，不符合贡献者只取一份开发包的需求。
- 本地建立完整环境：适合后续持续开发，但当前缺工具链与磁盘余量，用户明确选择云端构建。

## Testing

工作流执行笔记校验、Android 宿主编译和 JVM 冒烟、拼音版原生 APK 构建及 `lintPinyinRelease`。这些检查不替代本地完整 quick 门禁，也不能验收真实马达手感。实际云端运行结果以本分支对应的 Actions 记录为准。

## Consequences

用户无需本地构建即可下载测试包；代价是首次运行要下载固定工具链、词库并冷编译 Rust。每次临时 runner 的开发证书不保证相同，后续测试包不能假定能覆盖前一份；本工作流不承担连续分发与发布签名管理。
