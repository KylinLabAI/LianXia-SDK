# LianXia SDK — API Reference (v0.1.0)

Language bindings: **TypeScript** (`ts-any`), **Go** (`go-any`), **Swift** (`swift-any`), **Kotlin/Java** (`kotlin-any`), **Rust** (core slices per platform). All bindings are thin wrappers over the Rust C ABI (`lianxia_sdk.h`); JSON in / JSON out, error codes mapped to language exceptions/returns.

## Lifecycle

| Call | Notes |
|---|---|
| `client_new(base_url, app_id, user_token, app_secret, public_key_hex, app_slug, data_dir)` | Creates the client handle. `base_url` and `app_id` are required; the rest optional. `data_dir` enables file-backed persistence (empty/NULL → in-memory storage). |
| `client_free(handle)` | Releases the handle. Idempotent at the language-wrapper level (`close()` is safe to call twice). |
| `version()` | Returns the semantic version string. |

## Device identity (S6)

- `device_identity_get_or_create(handle)` — stable device identity; persisted through the Storage adapter.
- `derive_fingerprint(os, serial)` / `get_fingerprint(handle)` — deterministic `lx1-<os>-<sn>` fingerprint.

## PoW human-verification (S3)

- `pow_hash(token, salt, nonce)` — deterministic SHA-256 layout hash.
- `pow_solve(token, salt, difficulty)` — returns the solving nonce.

## License (S8)

- `license_init / restore / refresh / evaluate / get_view / activate`
- `license_request_activation_key(app_slug, plan_id, email, name, phone, payment_method, payment_ref)` — S12 closed loop (submit → poll → delivered key code); blocks while polling.
- Signature verification is **fail-closed**: a missing/invalid signature never evaluates as licensed.
- `license_verify_signature(uuid, app_id, status, expire_at, key_code, platform, signature, public_key_hex)` — pure, no I/O; `LX_ERR_ARG` on missing arguments.

## Session (S7)

- `session_ensure_anonymous(device_sn, os_type)` / `session_ensure_anonymous_with_slider(device_sn, os_type, solution_json)` — registration; slider escalation surfaces `SliderRequiredException` / a `slider_required` envelope (`code: "slider_required"`).
- `session_restore / session_get_view / session_logout`.

## Upgrade (S1)

- `upgrade_init(handle, github_repo, github_token)`
- `upgrade_check(current_version, platform, channel, fingerprint, force)` — evaluates the newest release; honors force-update floors.
- `upgrade_download(url, dest_path, progress_cb)` — https-only streamed download.

## Config (S4/S16)

- `validate_config(cfg_json)` / `resolve_config(handle, baked_json)` — baked > dev precedence; dev sources are rejected for packaged builds (anti-redirect).

## Host adapters (S9)

Register **once**, before first use of the handle:

- `storage_set_callbacks(get_fn, set_fn, delete_fn, kind, ctx)` — host Storage backend (`kind`: `memory` / `keychain` / `app-data`). Callbacks stay valid until `client_free`; `ctx` ownership stays with the host (the SDK never frees it).
- `log_sink_set_callback(log_fn, ctx)` — metadata-only events (desensitized in the core); never installed → the core stays silent.

## Errors

JSON results return `LX_OK (0)` with `out_json`; failures return a non-zero code with `out_err` (JSON `{"code": ..., "message": ...}`). Language wrappers map codes to `LxException` / `LxError`; `slider_required` maps to `SliderRequiredException`.

## Concurrency & memory contract

- The handle is safe to call from any thread; callbacks must be (register before multi-threaded use, never concurrently with another call on the same handle).
- Strings returned through `out_*` parameters are owned by the caller and released with `lianxia_sdk_string_free` (wrappers handle this internally).
- The client owns its own tokio runtime; `close()`/`client_free` shuts it down — do not use the handle afterwards (wrappers throw `IllegalStateException`).
