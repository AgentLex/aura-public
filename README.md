<div align="center">

# AURA — Hardware Root of Trust

**182 GE (Base) · ~300 GE SCA-hardened · Four-Layer SCA Defense · Standard CMOS / FPGA**

[![License](https://img.shields.io/badge/License-Proprietary-red.svg)](./LICENSE)
[![Patent](https://img.shields.io/badge/Patent-CN%20Filed%20×2-blue.svg)](https://github.com/AgentLex/aura-public)
[![Simulation](https://img.shields.io/badge/Simulation-22%2F22%20PASS-brightgreen.svg)](./docs/)
[![GE Base](https://img.shields.io/badge/Base%20HRoT-182%20GE-gold.svg)](./docs/synthesis_report.png)
[![GE SCA](https://img.shields.io/badge/SCA--hardened-~300%20GE-orange.svg)](./docs/arch_system_overview.png)

---

🌐 **Language / 语言 / 言語 / Idioma**

[English](./README.md) · [简体中文](./README.zh-CN.md) · [繁體中文](./README.zh-TW.md) · [日本語](./README.ja.md) · [Español](./README.es.md)

---

</div>

## The Problem

$42B in annual chip cloning losses. Most chips today authenticate in firmware — which means anyone who controls the software controls the chip's identity.

Existing solutions don't fit where the problem is worst:

| Solution | Gate Count | Embeddable in MCU/IoT? | Anti-DPA? | Anti-SEU? |
|---|---|---|---|---|
| TPM 2.0 | ~50,000 GE | ✗ Too large | ✗ | ✗ |
| ARM TrustZone | SW overhead | Partial | ✗ | ✗ |
| PUF alone | ~10,000 GE | Partial | ✗ | ✗ |
| **AURA (Base)** | **182 GE** | **✓ Full fit** | **✓ 4 layers** | **✓ L2 Dual-Rail** |
| **AURA (SCA-hardened)** | **~280–320 GE** | **✓ Full fit** | **✓ 4 layers active** | **✓ L2 Dual-Rail** |

> **Gate count clarification:** 182 GE = Base HRoT (FPGA, Vivado measured: 46 LUT + 22 FF). ~280–320 GE = all four SCA layers fully active (FPGA). ASIC estimate @ 28nm: ~300–500 GE.
>
> A standalone AES-128 hardware implementation requires ~2,400 GE. Even the SCA-hardened AURA delivers full identity binding + clone protection + 4-layer SCA defense at **13% of AES-128 alone**.

---

## Core Mechanism

Ternary-state encoding over standard binary CMOS — **no special process required**.

```
2'b01 → Legitimate · Pass   Normal operation
2'b10 → Isolated · Protected   Unauthorized access blocked; authorized owner may recover
2'b11 → Illegal · Alert   Anomaly detected / SEU event
```

> **Key insight:** Once `2'b10` enters the MAC chain, no software instruction can clear it.
> This is a hardware constraint — fundamentally different from a firmware flag or software enum.

---

## Four-Layer SCA Defense

| Layer | Mechanism | Defends Against |
|---|---|---|
| L1 | Masked LFSR | DPA — lifts complexity O(2³²) → O(2⁴⁸) |
| L2 | Dual-Rail Logic | DPA + Fault Injection + SEU (0-cycle detection) |
| L3 | Constant-Time MAC | Timing side-channel leakage |
| L4 | Random Delay Insertion | DPA trace alignment (~100× harder) |

All four layers operate entirely in RTL — **no firmware dependency**.

---

## By the Numbers

| Metric | Value |
|---|---|
| Gate count — Base HRoT (FPGA) | **182 GE** (Vivado 2023.2, Artix-7 35T: 46 LUT + 22 FF) |
| Gate count — SCA-hardened (FPGA) | **~280–320 GE** (all 4 SCA layers active) |
| ASIC estimate @ 28nm | **~300–500 GE** (SCA-hardened) |
| Silicon area @ 28nm | **< 0.003 mm²** (SCA-hardened) |
| Per-chip cost addition | **< ¥0.1 / < $0.01** |
| Power consumption | **< 1 mW** (vs. TPM 2.0: 5–15 mW) |
| Simulation | **22/22 scenarios PASS** (Icarus Verilog) |
| RTL modules | 9 complete |

---

## Validation

<div align="center">

![Simulation Results](./docs/simulation_results.png)
*22/22 simulation scenarios — Icarus Verilog full-stack verification*

![Synthesis Report](./docs/synthesis_report.png)
*Vivado 2023.2 synthesis on Xilinx Artix-7 35T: 46 LUT + 22 FF = 182 GE (Base HRoT)*

![Waveform](./docs/waveform_s1_s6.png)
*GTKWave: S1–S6 core scenarios — Full match / Light deviation / Medium / Heavy / Fault injection / Reset*

</div>

---

## Architecture Diagrams

<div align="center">

![System Architecture](./docs/arch_system_overview.png)
*Four-layer active defense: SCA Defense Layer → Aura 2 (Sense) → ESM (Decide) → Aura 1 (Execute / HRoT)*

![SCA Defense Detail](./docs/arch_sca_defense.png)
*Attack type → Defense mechanism → Security effect — all 4 SCA layers mapped*

![Persistent Lock Mechanism](./docs/arch_persistent_lock.png)
*Power-on boot check + lock-trigger write flow — lockdown survives power cycling*

![Dual-Rail Encoding](./docs/arch_dual_rail.png)
*Standard single-rail (power varies = DPA vulnerable) vs. Dual-Rail Logic (constant power = DPA impossible)*

</div>

---

## Demo Videos

*Live FPGA recordings on Artix-7 35T — uploading to GitHub Releases soon. ⭐ Star to get notified.*

| # | Scenario | Description | Link |
|---|---|---|---|
| 01 | Normal Auth | Correct credential → authenticated pass | 🔜 Coming soon |
| 02 | Brute-Force Lockdown | 3 failures → irreversible hardware lockdown | 🔜 Coming soon |
| 03 | Power-Cycle Persistence | Lockdown survives full power-off / power-on | 🔜 Coming soon |
| 04 | Unlock & Recovery | Authorized credential → restored operation | 🔜 Coming soon |
| 05 | Replay Attack Blocked | Reused token → detected and rejected | 🔜 Coming soon |
| 06 | Privilege Escalation Blocked | Unauthorized state jump → hardware barrier | 🔜 Coming soon |
| 00 | Full Demo | All 7 scenarios — complete walkthrough | 🔜 Coming soon |

> Recorded live on real hardware with UART serial output at 115200 baud. Not simulation.

---

## Use Cases

**Smart Locks / High-Security IoT**
182 GE fits inside any lock MCU. EU EN 18031 mandatory from Aug 2025. SESIP L2 certification path supported.

**Automotive / ASIL-D / ISO 21434**
4-layer SCA defense meets ISO 21434 hardware prerequisites. < 0.003 mm² area penalty on ECU silicon.

**Industrial Controllers / Defense Electronics**
FPGA version available today. Identity binding prevents supply chain counterfeiting.

---

## Getting Started

AURA is available for technical evaluation under NDA.

**FPGA Evaluation (Artix-7 / Basys 3)**
```
Contact us → Sign NDA → Receive FPGA Starter Kit:
RTL interface definitions + integration docs + simulation scripts
Typical evaluation: 2–4 weeks
```

**ASIC Integration**
```
Engage 3–6 months before tape-out
Full RTL package + synthesis constraints + timing reports provided under NDA
SESIP L2 / ISO 21434 certification documentation included
```

---

## IP Protection

- 🇨🇳 **Chinese Invention Patent** — Application No. **202610850983.0** (filed 2026)
- 🇨🇳 **Chinese Invention Patent** — Application No. **2026106956971** (filed May 2026)
- 🌍 **PCT 5-country filing** in progress — CN / US / EU / JP / KR

> Full RTL source code review, synthesis constraints, and IP licensing terms available under NDA.

---

## Contact

**OptiAura Tech — 超矩阵（上海）高科技有限公司**

📩 lexxu@optiaura.tech
🌐 optiaura.tech
👤 [Lex Xu on LinkedIn](https://www.linkedin.com/in/lex-xu-optiaura/)

> *Full RTL code review available after NDA. Integration engineers supported throughout.*
