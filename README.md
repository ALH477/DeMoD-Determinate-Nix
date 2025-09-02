# DeMoD Communications Framework (DCF) — Nix Flake Setup

A Nix Flake configuration optimized for developing the **DeMoD Communications Framework (DCF)** using **Determinate Nix**.

---

##  Overview

- Uses **Deterministic System’s Determinate Nix**—a secure, enterprise‑ready Nix distribution with superior flake support and automation.  
- Targets **DCF v5.0.0**, a modular, low‑latency communication framework offering cross‑language and cross‑platform support, self‑healing peer‑to‑peer networking, and flexible transports (UDP, TCP, WebSocket, gRPC).  

---

##  Prerequisites

- [ ] **Deterministic Nix** installed (via Determinate Nix Installer on macOS/Linux)  
- [ ] Basic familiarity with Flakes in Nix  

---

##  Usage

```bash
git clone https://github.com/ALH477/DeMoD-Communication-Framework.git
cd DeMoD-Communication-Framework

# Enter the development shell defined by flakes
nix develop

# Or build DCF
nix build .#dcf
```

This assumes your `flake.nix` defines `devShells.default` and an output named `dcf` for building/testing.

---

##  Flake Configuration (flake.nix)

```nix
{
  description = "DCF development environment";

  inputs = {
    determinate.url = "https://flakehub.com/f/DeterminateSystems/determinate/*";
    nixpkgs.url = "github:NixOS/nixpkgs";
  };

  outputs = { self, determinate, nixpkgs }: {
    devShells.default = determinate.lib.mkShell {
      inherit (nixpkgs.legacyPackages.x86_64-linux) gcc protobuf grpc cmake;
      shellHook = ''
        export LC_ALL=C.UTF-8
        echo "Loaded DCF dev environment."
      '';
    };

    packages.x86_64-linux.dcf = nixpkgs.legacyPackages.x86_64-linux.callPackage ./default.nix {};
  };
}
```

- **Determinate Nix input** ensures consistent flake support and enterprise tooling.  
- **devShell** includes essential tools: compiler, Protocol Buffers, gRPC, CMake. Tailor as needed for other language SDKs (Python, Rust, Go, etc.).

---

##  Key Benefits

| Benefit              | Description |
|----------------------|-------------|
| **Reproducible Builds** | Leverages Determinate Nix’s flake-based reproducibility. |
| **Fast Evaluation** | Features like `nix flake prefetch-inputs` speed up builds. |
| **Language-Agnostic SDKs** | DCF supports C/C++, Rust, Go, Python, Java, Kotlin, Swift, JS/Node.js, Perl, Haskell. |
| **Low-Latency, Self-Healing Networking** | Designed for ultra-fast, resilient communication in distributed systems. |

---

##  Quick Commands

```bash
# Enter development environment
nix develop

# Prefetch all flake inputs to accelerate builds
nix flake prefetch-inputs

# Build the DCF package
nix build .#dcf
```

---

##  Contribution & Further Reading

- Review design and architecture in `docs/dcf_design_spec.markdown` or similar documentation within the repository.  
- Contributions welcome via issues or pull requests—especially for plugin development, transport enhancements, or new language bindings.

---

##  License & Compliance

- **License**: GNU GPL‑3.0 — fully open-source and community-oriented.  
- **Export Compliance**: DCF deliberately avoids encryption to remain compliant with U.S. export laws (EAR/ITAR). Custom extensions may require a legal review.  

---

###  Summary

This template provides a clean, flake-based setup for DCF development using Determinate Nix. It emphasizes simplicity, reproducibility, and fast iteration—ideal for contributors and maintainers working with the DeMoD Communications Framework.
