# LianXia SDK

面向多端应用的可嵌入授权与设备身份 SDK —— 设备指纹、PoW 人机校验、许可证评估（fail-closed 签名校验）、匿名会话与升级管理，通过统一 C ABI 提供 TypeScript / Go / Swift / Kotlin (JNI) / Rust 薄绑定。

## 安装

从[最新发布页](https://gitee.com/KylinLab/LianXia-SDK/releases/latest)选择语言包（`-any` 薄包装），工具链会按平台自动解析原生核心切片（`darwin-x64`、`darwin-arm64`、`ios-arm64`、`android-arm64`、`windows-x64`、`ohos-arm64`、`linux-x64`）：

```bash
# 中央二进制仓库（KylinArtifacts）
kyart pull --product lianxia --version v0.1.0 --platform ts-any
```

或从发布页直接下载语言包，解压到项目的 `vendor/lianxia/` 目录 —— 包内布局遵循 Kylin 包标准（`include/`、`lib/<os>-<arch>/`，包根含 `LICENSE` / `README.md` / `lianxia.json`）。

## 快速开始

```java
// Kotlin (JNI) —— 完整生命周期
try (LianXiaClient c = new LianXiaClient(
        "https://api.example.com/api/v1",       // baseUrl（必填）
        "9b2f5c1e-0000-4000-8000-000000000001", // appId（必填）
        null, null, null)) {                     // userToken / appSecret / publicKeyHex
    String fp = LianXiaSdk.deriveFingerprint("macos", "SN-XXXX");
    String view = c.licenseGetView();            // JSON 进 / JSON 出
}   // close() 释放原生句柄（幂等）
```

完整接口面（设备身份 S6、PoW S3、许可证 S8、会话 S7、升级 S1、配置校验 S4/S16、宿主 Storage/LogSink 适配器 S9）见 [API 参考](user_manual/API_REFERENCE.md)。

## 许可

见 [LICENSE](https://gitee.com/KylinLab/LianXia-SDK/blob/master/LICENSE)。
