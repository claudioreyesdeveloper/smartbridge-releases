# SmartBridge Releases

Public release artifacts and component manifest for **SmartBridge** —
the macOS / Windows companion app that bridges Cubase, Synthesizer V,
and a Yamaha Tyros / Genos workstation through a single plugin and a
small set of optional integration components.

> Source code lives in
> [claudioreyesdeveloper/SmartBridge-Plugin](https://github.com/claudioreyesdeveloper/SmartBridge-Plugin)
> (private). This repository hosts only the publicly-downloadable build
> artifacts.

---

## Quick install (recommended)

Most users only need **SmartBridge Setup** — a small bootstrapper
(~3 MB) that detects which components are already on the machine and
installs the rest with one click.

| Platform | Download |
|---|---|
| macOS (Apple Silicon) | [`SmartBridge_Setup_<latest>_arm64.dmg`](../../releases) |
| Windows (x64) — installer | [`SmartBridge_Setup_<latest>_x64-setup.exe`](../../releases) |
| Windows (x64) — MSI | [`SmartBridge_Setup_<latest>_x64.msi`](../../releases) |

Open the [Releases](../../releases) page and pick the latest
`setup-v*` release for your platform.

After install, launch **SmartBridge Setup** and the app guides you
through enabling whichever components you want (the main app, Cubase
integration, Synthesizer V side panel, AI Lyrics local model, help
files, etc.). It downloads each component on demand from the
release tracks below — only what you opted in to.

---

## Manual install (advanced)

Prefer to install the SmartBridge plugin directly without the Setup
helper? Each `v<version>` release contains the native installers:

```
v2.0.0/
├── SmartBridge_2.0.0.pkg              ← macOS Apple Silicon, productbuild
├── SmartBridge_2.0.0_Setup.exe        ← Windows x64, NSIS
├── build_features.macos.json
├── build_features.windows.json
├── smartbridge-release-manifest.json
├── checksums-sha256.txt
├── config-default.json                ← seed config (no secrets)
├── SmartBridge.cpr                    ← Cubase project template
├── SmartBridge_GenosSlotRename.js     ← Cubase MIDI Remote script
├── synthv_smartbridge_sidepanel.lua   ← Synthesizer V side panel
├── SmartBridge_Getting_Started_One_Page.txt
├── Installation_guide.zip
└── smartbridge_multilingual_manual.zip
```

Download the file you need, double-click to install (or unzip in
the case of help bundles).

---

## Repository layout

This repo hosts **two parallel release tracks**, each with its own
tag prefix:

| Track | Tag prefix | Contents | Audience |
|---|---|---|---|
| **Plugin asset bundle** | `v<X.Y.Z>` | Native plugin installers + integration assets + release manifest | End users (manual installs) and the Setup app (automatic downloads) |
| **Setup installer** | `setup-v<X.Y.Z>` | The Tauri-based **SmartBridge Setup** application itself (`.dmg`, `.exe`, `.msi`) | End users who want guided component-by-component installation |

The two tracks evolve independently: a UI-only Setup tweak can ship
without a plugin rebuild and vice versa.

---

## Release manifest

Each `v<version>` plugin release publishes a top-level
**`smartbridge-release-manifest.json`** that the Setup app downloads
first to learn what's available. Schema (excerpt):

```jsonc
{
  "manifest_version": 1,
  "product": "SmartBridge",
  "release_version": "2.0.0",
  "release_channel": "beta",
  "generated_at": "2026-05-01T08:00:00Z",
  "asset_provider": "github_release",
  "components": [
    {
      "id": "main-app",
      "title": "SmartBridge",
      "required": true,
      "platforms": ["macos", "windows"],
      "install_action": "launch_native_installer"
    },
    {
      "id": "cubase-connection",
      "title": "Cubase connection",
      "required": false,
      "platforms": ["macos", "windows"],
      "install_action": "file_copy"
    },
    /* ai-lyrics, synthv-connection, smartbridge-resources, help-files, ... */
  ],
  "assets": [
    {
      "asset_id": "main-app.macos.installer",
      "component_id": "main-app",
      "platforms": ["macos"],
      "filename": "SmartBridge_2.0.0.pkg",
      "object_key": "main-app/macos/SmartBridge_2.0.0.pkg",
      "release_asset_name": "SmartBridge_2.0.0.pkg",
      "size_bytes": 197132288,
      "sha256": "628a6b33f8ed3e...",
      "delivery_method": "github_release_asset",
      "install_action": "launch_native_installer",
      "requires_admin": true,
      "signature_status": "signed",
      "signature_required": true
    }
    /* ... */
  ]
}
```

Every asset declares its **size**, **SHA256 digest**, target
**platforms**, and **delivery method** (`github_release_asset`,
`http`, or `ollama_pull`). Setup downloads via streaming HTTP and
verifies the digest before any file is written into a target location.

---

## Verifying downloads

Each Setup installer ships a `.sha256` sidecar in the standard
`<hex>  <basename>` format. To verify on macOS:

```bash
shasum -a 256 -c SmartBridge_Setup_2.0.0_arm64.dmg.sha256
# -> SmartBridge_Setup_2.0.0_arm64.dmg: OK
```

On Windows (PowerShell):

```powershell
$expected = (Get-Content .\SmartBridge_Setup_2.0.0_x64-setup.exe.sha256).Split(' ')[0]
$actual   = (Get-FileHash .\SmartBridge_Setup_2.0.0_x64-setup.exe -Algorithm SHA256).Hash.ToLower()
if ($expected -eq $actual) { "OK" } else { "MISMATCH" }
```

For plugin assets, `checksums-sha256.txt` in each `v<version>`
release lists every file's expected digest in the same format.

---

## Component model

SmartBridge Setup understands these components. Detection is
runtime; nothing is forced.

| Component | Required | What it installs |
|---|---|---|
| `main-app` | Yes | The SmartBridge application + plugin formats (Standalone, VST3, AU on macOS) via the native installer. |
| `cubase-connection` | No | Cubase MIDI Remote driver script and project template. |
| `windows-cubase-runtime` | No | The Bome BMIDI runtime needed by the Windows Cubase integration. Card is hidden when the active SmartBridge build was compiled without `bmidi` support. |
| `synthv-connection` | No | Side-panel `.lua` script for Synthesizer V Studio 1 and 2. |
| `smartbridge-resources` | No | Seed `config-default.json` placed only on first install (never overwrites user config). |
| `ai-lyrics` | No | Local lyric-generation model via Ollama (`ollama pull <tag>`). Skipped silently if Ollama isn't installed. |
| `help-files` | No | Getting-started guide, installation guide, and multilingual manual. |

Cards display plain-language status (Ready / Not installed / Needs
repair / Not available in this build). Technical details live in the
**Diagnostics** tab.

---

## `build_features.json`

Each plugin release publishes per-platform feature manifests
(`build_features.macos.json`, `build_features.windows.json`) that
record which optional features were compiled into that specific
binary:

```json
{
  "schema_version": 1,
  "smartbridge_version": "2.0.0",
  "platform": "windows",
  "features": {
    "bmidi": false,
    "tracktion": true,
    "essentia": true
  }
}
```

The Setup app reads this and gates platform-specific cards
accordingly (e.g. the `windows-cubase-runtime` card stays hidden if
`bmidi=false`).

---

## Versioning

* **Format:** `MAJOR.MINOR.PATCH` with optional `-prerelease` suffix
  (`2.0.0`, `2.0.1`, `2.0.0-rc1`, `2.1.0-beta.3`).
* **Plugin** tags are `v<version>`; **Setup** tags are
  `setup-v<version>`. The two are versioned independently.
* All Setup releases are currently marked **Pre-release** until the
  product reaches general availability.

---

## Platform support

| Platform | Architectures | Minimum OS |
|---|---|---|
| macOS | Apple Silicon (arm64) | macOS 11 Big Sur |
| Windows | x64 | Windows 10 1809+ |

Intel macOS and Windows ARM are not currently shipped.

---

## Code signing & notarization

When the project's signing identities are configured at build time:

* macOS `.app`, `.pkg`, and `.dmg` are signed with a **Developer ID
  Application** / **Developer ID Installer** certificate and
  notarized via Apple `notarytool`.
* Windows `.exe` and `.msi` are **Authenticode**-signed with an
  RFC 3161 timestamp.

Pre-1.0 releases may ship unsigned. On macOS, right-click the dmg
and choose **Open** to bypass Gatekeeper on first launch.

---

## Source code

All source code, build scripts, and CI workflows live in the private
[`SmartBridge-Plugin`](https://github.com/claudioreyesdeveloper/SmartBridge-Plugin)
repository. This releases repo holds only the resulting binaries and
manifests.

For commercial licensing, integration questions, or access to the
source, contact the author.

---

## License

Released artifacts are distributed under the terms of the
SmartBridge end-user license, included in each plugin release
(`license.txt` inside `Installation_guide.zip`).

The Tauri-based Setup application is © 2026 Claudio Reyes; binary
redistribution outside this repository is not permitted.

---

*Built and published by automated CI from the SmartBridge source
repository on every tagged release.*
