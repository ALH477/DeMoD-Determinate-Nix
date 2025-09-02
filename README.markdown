# DeMoD Communications Framework - Determinate Nix Flake

**Version 5.0.0 | September 2, 2025**  
**Developed by DeMoD LLC**  
**Contact:** info@demodllc.example  
**License:** MIT License  
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)  
[![Nix Flake](https://img.shields.io/badge/Nix-Flake-blue.svg)](https://nixos.org)

## Overview

This repository provides a standalone **Nix Flake** for reproducible development environments for the [DeMoD Communications Framework (DCF)](https://github.com/ALH477/DeMoD-Communication-Framework), a free and open-source framework for low-latency, modular, and interoperable data exchange. Built with [Determinate Systems](https://determinate.systems)' Nix installer and best practices, this flake ensures deterministic builds and development setups for DCF's SDKs in Perl, Python, C/C++, Node.js, Go, Rust, Java/Kotlin (Android), and Swift (iOS). It supports DCF's goals of modularity, hardware/language agnosticism, and performance (<1ms latency, <5% CPU) across embedded devices, cloud servers, and mobile platforms.

Key features:
- **Isolated Development Shells**: Per-language environments (e.g., `nix develop .#python`) to prevent dependency conflicts.
- **Pinned Dependencies**: Locked versions (e.g., `protobuf=3.21.12`, `grpc=1.54.0`) for consistent Protobuf/gRPC interoperability.
- **Cross-Platform Support**: Linux, macOS, and ARM (e.g., Raspberry Pi) via cross-compilation.
- **CI Integration**: Automated tests for C and Python SDKs (`nix flake check`) for reliable contributions.
- **Plugin Support**: Configured paths for DCF's dynamic plugin system (e.g., `libcustom_transport.so`).
- **Mobile Development**: Ready-to-use Android and iOS environments for DCF's mobile bindings.

This flake is licensed under the MIT License, allowing flexible use and modification while complementing the GPL-3.0-licensed DCF project.

## Prerequisites

- **Nix**: Install via the [Determinate Nix Installer](https://install.determinate.systems/nix) for a reproducible setup:
  ```bash
  curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install
  ```
- **Direnv** (optional, recommended): For auto-activation of dev environments:
  ```bash
  nix profile install nixpkgs#direnv
  echo 'use flake' > .envrc && direnv allow
  ```
- **Git**: To clone this repository.
- **DCF Repository**: Clone the DCF mono repo separately:
  ```bash
  git clone --recurse-submodules https://github.com/ALH477/DeMoD-Communication-Framework
  ```

## Installation

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/ALH477/DeMoD-Determinate-Nix
   cd DeMoD-Determinate-Nix
   ```

2. **Verify Nix Flake Support**:
   Ensure `flakes` and `nix-command` are enabled:
   ```bash
   nix --version
   ```
   If needed, add to `~/.config/nix/nix.conf`:
   ```
   experimental-features = nix-command flakes
   ```

3. **Lock Dependencies** (optional):
   Generate `flake.lock` for full determinacy:
   ```bash
   nix flake lock
   ```

## Usage

### Development Shells
Activate a shell for a specific DCF SDK or shared tools:
- **Default (Minimal Shared Tools)**:
  ```bash
  nix develop
  ```
  Includes `cmake`, `git`, `protobuf`, `grpc`, `jq`, `valgrind` for general DCF workflows.
- **Language-Specific**:
  ```bash
  nix develop .#<sdk>
  ```
  Available SDKs: `perl`, `python`, `c`, `nodejs`, `go`, `rust`, `android`, `ios` (macOS only).

  Example:
  ```bash
  nix develop .#c  # C/C++ SDK environment with plugin support
  nix develop .#python  # Python SDK environment
  ```

### Building SDKs
Build the C SDK as a reproducible package (requires DCF repo at `../DeMoD-Communication-Framework/c_sdk`):
```bash
nix build .#c-sdk
```
For ARM (e.g., Raspberry Pi):
```bash
nix build .#c-sdk-aarch64
```
Outputs are stored in `./result`.

### Running Tests
Run automated tests for CI (requires DCF repo):
```bash
nix flake check
```
This executes:
- C SDK tests (`test_redundancy`, `test_plugin` with Valgrind).
- Python SDK tests (`pytest tests/`).

### Generating Protobuf/gRPC Stubs
Inside a shell (e.g., `nix develop .#c`), with the DCF repo cloned:
```bash
protoc --c_out=../DeMoD-Communication-Framework/c_sdk/src messages.proto
protoc --cpp_out=../DeMoD-Communication-Framework/cpp/src --grpc_out=../DeMoD-Communication-Framework/cpp/src --plugin=protoc-gen-grpc=grpc_cpp_plugin messages.proto services.proto
```
See `docs/dcf_design_spec.md` in the DCF repo for full instructions.

### Plugin Development
For C SDK plugins, place shared libraries in the DCF repo’s `c_sdk/plugins/`:
```bash
gcc -shared -o ../DeMoD-Communication-Framework/c_sdk/plugins/libcustom_transport.so ../DeMoD-Communication-Framework/c_sdk/plugins/custom_transport.c
```
The `c` shell sets `DCF_PLUGIN_PATH` to the DCF repo’s plugin directory.

### Mobile Development
- **Android**: The `android` shell sets `ANDROID_HOME`. Run `sdkmanager --licenses` to accept licenses if prompted.
- **iOS**: The `ios` shell (macOS only) includes `swift` and `xcodebuild` for DCF’s Swift bindings.

### Flake Information
View usage details:
```bash
nix run .#flake-info
```

## Flake Structure
The `flake.nix` defines:
- **Inputs**:
  - `nixpkgs`: Pinned to `nixos-24.05` for stable dependencies.
  - `flake-utils`: For cross-platform support (Linux, macOS, ARM).
- **Outputs**:
  - `devShells`: Isolated environments for each SDK and a minimal default shell.
  - `packages`: `c-sdk` and `c-sdk-aarch64` for native and ARM builds.
  - `checks`: Automated tests for C and Python SDKs.
  - `formatter`: `nixpkgs-fmt` for consistent flake formatting.
  - `apps.flake-info`: Quick reference for flake usage.

## Why Use This Flake?

This flake, built with Determinate Systems’ principles, enhances DCF development by:
- **Reproducibility**: Pinned dependencies (`protobuf=3.21.12`, `grpc=1.54.0`) ensure consistent builds across machines, aligning with DCF’s interoperability requirements for Protobuf/gRPC.
- **Modularity**: Isolated shells mirror DCF’s modular components (Networking, Serialization, Plugins), supporting SDK development without conflicts.
- **Performance**: Lightweight shells and Valgrind ensure DCF’s <1ms latency and <5% CPU goals, especially for embedded devices.
- **Hardware/Language Agnosticism**: Cross-platform and cross-compilation support (e.g., `c-sdk-aarch64`) enables DCF’s deployment on Raspberry Pi, cloud servers, and mobile platforms.
- **Usability**: Direnv integration and clear shell messages streamline workflows for DCF’s CLI (`dcf`) and TUI (`dcf tui`).
- **Compliance**: No encryption dependencies, ensuring DCF’s adherence to U.S. export regulations (EAR, ITAR).
- **Collaboration**: CI-ready checks and clear documentation encourage contributions to DCF’s open-source ecosystem.

## Contributing

Contributions are welcome! Follow these steps:
1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/xyz`.
3. Update `flake.nix` or add tests (ensure paths reference the DCF repo).
4. Format with `nix fmt`.
5. Submit a pull request using the [PR template](docs/PR_TEMPLATE.md).
6. Discuss via [GitHub Issues](https://github.com/ALH477/DeMoD-Determinate-Nix/issues).

Ensure changes align with DCF’s goals (e.g., modularity, performance) and maintain the MIT License.

## Documentation

- **DCF Design Specification**: See `docs/dcf_design_spec.md` in the [DCF repo](https://github.com/ALH477/DeMoD-Communication-Framework).
- **C SDK Guide**: See `c_sdk/C-SDKreadme.markdown` in the DCF repo.
- **Nix Resources**: [Determinate Systems](https://determinate.systems), [NixOS Docs](https://nixos.org).

## License

This flake is licensed under the MIT License, allowing flexible use and modification. See [LICENSE](LICENSE) for details. Note: The DCF project is licensed under GPL-3.0; ensure compliance when integrating with DCF.
