# SmartBridge — Asset Feed

> **Looking for the SmartBridge installer?**
> ➜ Go to **[claudioreyesdeveloper/smartbridge-setup](https://github.com/claudioreyesdeveloper/smartbridge-setup/releases/latest)**.

This repository is **not** for direct end-user downloads. It hosts the
machine-readable **asset feed** that the SmartBridge Setup application
reads from at runtime: the release manifest, the native plugin
installers, integration scripts, help bundles, and per-platform
feature flags.

End users should download **SmartBridge Setup** from the link above —
that bootstrapper handles everything in this repo automatically, with
SHA256 verification on every download.

---

## What's in each release

Each `v<version>` tag here represents one SmartBridge plugin version
and contains the assets the Setup application fetches on demand:

```
v2.0.0/
├── smartbridge-release-manifest.json   ← Setup app reads this first
├── checksums-sha256.txt
├── SmartBridge_2.0.0.pkg               ← macOS native installer
├── SmartBridge_2.0.0_Setup.exe         ← Windows native installer
├── build_features.macos.json           ← compiled-in features
├── build_features.windows.json
├── config-default.json                 ← seed config (no secrets)
├── SmartBridge.cpr                     ← Cubase project template
├── SmartBridge_GenosSlotRename.js      ← Cubase MIDI Remote script
├── synthv_smartbridge_sidepanel.lua    ← Synthesizer V side panel
├── SmartBridge_Getting_Started_One_Page.txt
├── Installation_guide.zip
└── smartbridge_multilingual_manual.zip
```

---

## Release manifest

Each release publishes a `smartbridge-release-manifest.json` that the
Setup application downloads first to learn what's available. Schema
(excerpt):

```jsonc
{
  "manifest_version": 1,
  "product": "SmartBridge",
  "release_version": "2.0.0",
  "release_channel": "beta",
  "asset_provider": "github_release",
  "components": [ /* main-app, cubase-connection, ai-lyrics, ... */ ],
  "assets": [
    {
      "asset_id": "main-app.macos.installer",
      "component_id": "main-app",
      "platforms": ["macos"],
      "filename": "SmartBridge_2.0.0.pkg",
      "size_bytes": 197132288,
      "sha256": "628a6b33f8ed3e...",
      "delivery_method": "github_release_asset",
      "install_action": "launch_native_installer"
    }
  ]
}
```

Every asset declares its size, SHA256, target platforms, and delivery
method. The Setup app streams downloads, verifies digests before any
file is written, and never modifies user data.

---

## `build_features.json`

Per-platform compile-time feature manifests:

```json
{
  "schema_version": 1,
  "smartbridge_version": "2.0.0",
  "platform": "windows",
  "features": { "bmidi": false, "tracktion": true, "essentia": true }
}
```

Used by the Setup app to gate platform-specific component cards
(e.g. the Windows Cubase runtime card stays hidden when
`bmidi: false`).

---

## Why two public repositories?

* **[smartbridge-setup](https://github.com/claudioreyesdeveloper/smartbridge-setup)** — End-user-facing.
  Contains only the SmartBridge Setup installer (`.dmg`, `.exe`, `.msi`).
* **smartbridge-releases** (this repo) — Backend asset feed.
  Contains the manifest and components that the Setup app downloads
  programmatically.

Splitting them keeps the end-user download page clean (one product,
three platforms) while letting the asset feed evolve independently.

The "Source code" zip and tar.gz that GitHub auto-generates on every
release here contain only this README — they are an unavoidable
artefact of the GitHub Releases system and can be ignored.

---

## Source code

All SmartBridge source code, build scripts, and CI workflows live in
the private
[`SmartBridge-Plugin`](https://github.com/claudioreyesdeveloper/SmartBridge-Plugin)
repository.

---

## License

Released artifacts are distributed under the SmartBridge end-user
license, included with each install (`license.txt` inside
`Installation_guide.zip`).
