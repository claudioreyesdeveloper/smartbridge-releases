# SmartBridge release files

[Download SmartBridge Setup](https://github.com/claudioreyesdeveloper/smartbridge-setup/releases/latest) to install SmartBridge.

This repository supplies the application and supporting files that Setup downloads. Setup reads `smartbridge-release-manifest.json`, downloads the selected components, and verifies their sizes and SHA-256 checksums before installation.

## Included files

- SmartBridge application and plug-ins, with the encrypted phrase library.
- The current Cubase template (`SmartBridge.cpr`) and MIDI Remote script.
- Synthesizer V integration files, default configuration and help files.
- Private Python runtimes for macOS 14 or newer on Apple Silicon and Windows x64.
- Composer’s Assistant and SkyTNT melody models, with their upstream license and source information.
- Platform feature metadata and release checksums.

AI melody helpers install automatically through Setup. **AI Lyrics (Ollama) is optional and off by default.** Setup downloads Ollama and its lyrics models from their upstream distributors only if the user selects AI Lyrics.

The downloadable application is covered by the SmartBridge license. Third-party runtimes, libraries and model files retain their respective upstream licenses and notices.

The source archives automatically added to releases in this repository contain its documentation; use SmartBridge Setup to install the application.
