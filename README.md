# SmartBridge Releases

Public release binaries and manifests for SmartBridge.

The source code lives in [claudioreyesdeveloper/SmartBridge-Plugin](https://github.com/claudioreyesdeveloper/SmartBridge-Plugin) (private).

Releases are tagged as `v<version>-<channel>`, e.g. `v1.0.0-beta`. Each release contains:

- The SmartBridge installers (Windows `.exe`, macOS `.pkg`).
- Per-component asset files (Cubase scripts and template, Synthesizer V script, help files).
- The release manifest (`manifest__smartbridge-release-manifest.json`) and SHA256 checksum file (`manifest__checksums-sha256.txt`).

The Tauri Component Installer reads the manifest first, then downloads and SHA256-verifies each asset before installing.
