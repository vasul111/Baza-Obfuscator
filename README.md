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
### Luau Virtual Machine & Script Protection Engine

[![Platform](https://img.shields.io/badge/Platform-Roblox%20%7C%20Luau%20%7C%20Lua%205.1-black)](https://roblox.com)
[![Version](https://img.shields.io/badge/Version-2.4--beta-blue)](https://github.com)
[![Status](https://img.shields.io/badge/Status-Beta-success)](https://github.com)
[![Discord](https://img.shields.io/badge/Discord-Community-5865F2)](https://discord.gg/2GKg4h4rrr)

[Discord](https://discord.gg/2GKg4h4rrr) | [Architecture](#architecture) | [Security Audit](#security-audit) | [Performance](#performance-characteristics) | [Usage](#getting-started) | [License](#license--notice)

</div>

---

## Overview

Baza Obfuscator is a Luau virtualization and code protection suite designed for script creators and security research. Scripts are compiled into an encrypted binary container executed by a custom Virtual Machine, prioritizing runtime stability, closure fidelity, and zero literal token leakage.

**Project Status**: `v2.4-beta` (Under active development and testing).

---

## Architecture

- **Virtual Machine**: Compiles source code into virtualized bytecode with polymorphic instruction memory layouts and randomized opcode permutations per build.
- **Cryptographic Container**: Encrypted using the ChaCha20 stream cipher with HalfSipHash-2-4 message authentication codes (MAC) and dynamic LFSR key reconstruction.
- **Zero Static Global Leaks**: Output contains zero literal references to sensitive environment globals (`getgenv`, `getrenv`, `debug`, `traceback`, `getinfo`). All external symbols resolve dynamically via XOR proxy tables.
- **Safe-Bounds Buffer Engine**: Physical buffer boundary enforcement (`_tot_bl`) prevents out-of-bounds reads and unhandled exceptions on truncated payloads.
- **Environment Compatibility**: Tested against standard Roblox client runtimes, Luau CLI, and current executor environments (Wave, Solara, Swift, Delta, Fluxus). No false-positive environment assertion failures observed in test suites.

---

## Security Audit

Static analysis results using automated token scanner (`static_scan.py`):

| Sensitive Literal | Common Protectors | Baza Obfuscator Output |
| :--- | :---: | :---: |
| `getgenv`, `getrenv`, `getfenv` | 6 - 18 | **0** |
| `shared`, `debug`, `traceback`, `getinfo` | 8 - 25 | **0** |
| `stack overflow` | 1 - 2 | **0** |

### Semantic Verification
All builds pass a comprehensive Luau semantic verification battery:
- Deep nested closures with shared and independent upvalues
- Exported module closures executed post-cleanup
- Coroutine bidirectional state exchange (`yield` / `resume`)
- Multiret expansion and vararg forwarding

---

## Performance Characteristics

Benchmark measured on a 50,000 branch-and-arithmetic opcode loop workload:

```lua
-- Benchmark Workload:
for i = 1, 50000 do
    local step = i % 5
    if step == 0 then acc += (i * 2) - 1
    elseif step == 1 then acc -= (i + 3)
    elseif step == 2 then acc += (i * 3)
    else acc += 1 end
end
```

| Parameter | Native Interpreter | Baza Obfuscator VM |
| :--- | :--- | :--- |
| Workload | 50,000 loop iterations | 50,000 loop iterations |
| Execution Time | 5.00 ms | 3000.00 ms |
| Throughput | ~10,000,000 iters/sec | 16,667 iters/sec |
| Latency | 0.100 ms / 1k iters | 60.000 ms / 1k iters |
| Computed Result | `1000030000` | `1000030000` (Exact Match) |

*Note: Virtualization introduces interpreter overhead. Best suited for application logic, security gates, licensing, and event handlers. If near-native execution speed is required for heavy math loops, consider lighter AST-focused presets.*

---

## Protection Presets

- **Balanced**: Standard Virtual Machine compilation, AST renaming, constant splitting, and minification.
- **Maximum**: Full Virtual Machine virtualization, dynamic anti-tamper, control flow flattening, string-to-expressions, and opaque predicates.
- **Double-VM**: Dual-layer protection pipeline (encrypted secondary payload loader + virtual machine core).

---

## Getting Started

### Discord Bot
1. Join the server: [https://discord.gg/2GKg4h4rrr](https://discord.gg/2GKg4h4rrr)
2. Use `/obfuscate` in the bot channel and upload your `.lua` / `.luau` file.
3. Select your preset.

### REST API
Authenticated API endpoints are available for automated CI/CD workflows:

```python
import requests

url = "https://api.baza.dev/v1/obfuscate"  # Replace with active API host
headers = {"X-API-Key": "YOUR_API_KEY", "Content-Type": "application/json"}
payload = {"script": "print('Protected')", "preset": "balanced", "target": "luau"}

res = requests.post(url, json=payload, headers=headers)
if res.json().get("success"):
    print("Obfuscation completed successfully.")
```

---

## License & Notice

Baza Obfuscator core binaries and VM compiler components are proprietary. See [LICENSE](./LICENSE) for details.

This software is provided for educational security research and intellectual property protection.
