# LianXia SDK

An embeddable licensing and device-identity SDK for multi-platform apps — device fingerprinting, PoW human-verification, license evaluation (fail-closed signature checks), anonymous sessions, and upgrade management — over a uniform C ABI with thin bindings for TypeScript, Go, Swift, Kotlin (JNI), and Rust.

可嵌入多端应用的授权与设备身份 SDK：设备指纹、PoW 人机校验、许可证评估（fail-closed 签名校验）、匿名会话与升级管理，通过统一 C ABI 提供 TypeScript / Go / Swift / Kotlin (JNI) / Rust 薄绑定。

## Install / 安装

Pick your language bundle from the [latest release](https://github.com/KylinLabAI/LianXia-SDK/releases/latest) (the `-any` wrapper), and let the tooling resolve the native core slice for your platform (`darwin-x64`, `darwin-arm64`, `ios-arm64`, `android-arm64`, `windows-x64`, `ohos-arm64`, `linux-x64`):

```bash
# central binary store (KylinArtifacts)
kyart pull --product lianxia --version v0.1.0 --platform ts-any
```

Or download the bundle directly from the release page and extract it into your project's `vendor/lianxia/` — the archive layout follows the Kylin package standard (`include/`, `lib/<os>-<arch>/`, root `LICENSE` / `README.md` / `lianxia.json`).

## Quick start / 快速开始

```java
// Kotlin (JNI) — full lifecycle
try (LianXiaClient c = new LianXiaClient(
        "https://api.example.com/api/v1",      // baseUrl (required)
        "9b2f5c1e-0000-4000-8000-000000000001", // appId  (required)
        null, null, null)) {                    // userToken / appSecret / publicKeyHex
    String fp = LianXiaSdk.deriveFingerprint("macos", "SN-XXXX");
    String view = c.licenseGetView();           // JSON in / JSON out
}   // close() releases the native handle (idempotent)
```

```typescript
// TypeScript (koffi over the Rust C ABI)
import { LianXiaSdk } from "lianxia-sdk";
LianXiaSdk.version();                 // semantic version
LianXiaSdk.powHash(token, salt, nonce);
```

See the [API reference](user_manual/API_REFERENCE.md) for the full surface: device identity (S6), PoW (S3), license (S8), session (S7), upgrade (S1), config validation (S4/S16), and the host Storage/LogSink adapters (S9).

## Verification / 校验

Every release asset's `sha256` is published with the artifact and pinned in the KylinArtifacts manifest; verify before linking.

## License / 许可

See [LICENSE](https://github.com/KylinLabAI/LianXia-SDK/blob/main/LICENSE).
