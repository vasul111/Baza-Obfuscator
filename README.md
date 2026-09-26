<div align="center">

```
=========================================================================================
 ____    _     _____     _       ___  ____  _____ _   _ ____   ____    _  _____ ___  ____  
| __ )  / \   |__  /    / \     / _ \| __ )|  ___| | | / ___| / ___|  / \|_   _/ _ \|  _ \ 
|  _ \ / _ \    / /    / _ \   | | | |  _ \| |_  | | | \___ \| |     / _ \ | || | | | |_) |
| |_) / ___ \  / /_   / ___ \  | |_| | |_) |  _| | |_| |___) | |___ / ___ \| || |_| |  _ < 
|____/_/   \_\/____| /_/   \_\  \___/|____/|_|    \___/|____/ \____/_/   \_\_| \___/|_| \_\
                         Created by baza2000ultrapro
=========================================================================================
```

# Baza Obfuscator
### High-Performance Luau Virtual Machine & Script Protection Engine

[![Platform](https://img.shields.io/badge/Platform-Roblox%20%7C%20Luau%20%7C%20Lua%205.1-black)](https://roblox.com)
[![Version](https://img.shields.io/badge/Version-2.4--beta-blue)](https://github.com)
[![Status](https://img.shields.io/badge/Status-Active%20Beta-success)](https://github.com)
[![Discord](https://img.shields.io/badge/Discord-Community-5865F2)](https://discord.gg/2GKg4h4rrr)

[Discord Community](https://discord.gg/2GKg4h4rrr) | [Architecture](#architecture) | [Security Audit](#security-audit) | [Performance](#performance-characteristics) | [Usage](#getting-started)

</div>

---

## Overview

Baza Obfuscator is a high-performance Luau virtualization and code protection suite designed for script creators, UI library maintainers, and reverse engineering research.

Rather than relying purely on AST scrambling or legacy Lua 5.1 forks, Baza compiles Luau scripts into an encrypted binary container interpreted by an isolated custom Virtual Machine. The pipeline prioritizes runtime stability, closure fidelity, and zero literal token leakage.

**Project Status**: `v2.4-beta`. Actively tested and maintained.

---

## Architecture

- **Custom Luau Virtual Machine**: Compiles source code into a virtualized instruction set featuring 16 polymorphic instruction memory layouts (`layout_id 0..15`) and randomized opcode permutations per build.
- **Cryptographic Bytecode Container**: Bytecode streams are encrypted using the ChaCha20 stream cipher with HalfSipHash-2-4 message authentication codes (MAC) and LFSR-derived key shares.
- **Zero Static Global Leaks**: Output code contains zero literal references to sensitive environment globals (`getgenv`, `getrenv`, `debug`, `traceback`, `getinfo`). All external references are resolved dynamically via XOR proxy tables.
- **Safe-Bounds Buffer Engine**: Physical buffer boundary enforcement (`_tot_bl`) prevents memory out-of-bounds reads and unhandled exceptions on truncated or modified payloads.
- **Environment Compatibility**: Tested against standard Roblox client runtimes, Luau CLI, and current executor environments (Wave, Solara, Swift, Delta, Fluxus). No false-positive environment assertion failures observed in test batteries.

---

## Security Audit

Static analysis results using automated token scanner (`static_scan.py`):

| Sensitive Literal | Public Forks / Common Protectors | Baza Obfuscator Output |
| :--- | :---: | :---: |
| `getgenv` | 3 - 8 | **0** |
| `getrenv` | 1 - 4 | **0** |
| `getfenv` | 2 - 6 | **0** |
| `shared` | 1 - 3 | **0** |
| `debug` | 4 - 12 | **0** |
| `traceback` | 2 - 5 | **0** |
| `getinfo` | 1 - 4 | **0** |
| `stack overflow` | 1 - 2 | **0** |

### Semantic Verification
All builds pass the complete Mimi Luau Semantic Battery:
- Deep nested closures with shared and independent upvalues
- Exported module closures executed post-cleanup
- Coroutine bidirectional state exchange (`yield` / `resume`)
- Multiret expansion and vararg forwarding

---

## Performance Characteristics

Benchmark measured on a 50,000 full-dispatch opcode loop workload:

| Benchmark Parameter | Native Interpreter | Baza Obfuscator VM |
| :--- | :--- | :--- |
| Workload | 50,000 loop iterations | 50,000 loop iterations |
| Execution Time | 5.00 ms | 3000.00 ms |
| Virtualized Throughput | ~10,000,000 iters/sec | 16,667 iters/sec |
| Operational Latency | 0.100 ms / 1k iters | 60.000 ms / 1k iters |
| Verification Check | Baseline | Exact match (`1000030000`) |

*Note: Virtualization introduces interpreted execution overhead. Recommended for application logic, security gates, licensing, and event handlers; heavy real-time graphics math loops should remain unvirtualized or run on lighter AST presets.*

---

## Protection Presets

- **Balanced**: Standard Virtual Machine compilation, AST renaming, constant splitting, and minification. Recommended for general usage.
- **Maximum**: Full Virtual Machine virtualization, dynamic anti-tamper, control flow flattening, string-to-expressions, and opaque predicates.
- **Double-VM**: Dual-layer protection pipeline (encrypted secondary payload loader + virtual machine core).

---

## Getting Started

### Discord Bot
The primary obfuscation service is accessible via our official Discord bot:
1. Join the server: [https://discord.gg/2GKg4h4rrr](https://discord.gg/2GKg4h4rrr)
2. Use `/obfuscate` in the dedicated channel and upload your `.lua` / `.luau` file.
3. Select your desired preset.

### REST API
Authenticated access is available for integration into automated build workflows:

```python
import requests

url = "https://api.baza.dev/v1/obfuscate"  # Replace with active API host
headers = {
    "X-API-Key": "YOUR_API_KEY",
    "Content-Type": "application/json"
}
payload = {
    "script": "print('Hello from Baza')",
    "preset": "balanced",
    "target": "luau"
}

response = requests.post(url, json=payload, headers=headers)
result = response.json()
if result.get("success"):
    print("Obfuscation completed successfully.")
```

---

## License & Notice

Baza Obfuscator core binaries and VM compiler components are proprietary. Public API wrappers and client tooling are provided for authorized users.

This project is maintained for educational software security research and intellectual property protection.
