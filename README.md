<div align="center">

# AURA — Hardware Root of Trust

**182 GE · Four-Layer SCA Defense · Standard CMOS / FPGA**

[![License](https://img.shields.io/badge/License-Proprietary-red.svg)](./LICENSE)
[![Patent](https://img.shields.io/badge/Patent-CN%202026106956971-blue.svg)](https://github.com/AgentLex/aura-public)
[![Simulation](https://img.shields.io/badge/Simulation-22%2F22%20PASS-brightgreen.svg)](./docs/)
[![GE](https://img.shields.io/badge/Footprint-182%20GE-gold.svg)](./docs/synthesis_report.png)

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
| **AURA** | **182 GE** | **✓ Full fit** | **✓ 4 layers** | **✓ L2 Dual-Rail** |

> A standalone AES-128 hardware implementation requires ~2,400 GE.  
> AURA delivers identity binding + clone protection + 4-layer SCA defense in **182 GE** — 7.6% of AES-128 alone.

---

## Core Mechanism

Ternary-state encoding over standard binary CMOS — **no special process required**.

```
2'b01  →  Legitimate · Pass        Normal operation
2'b10  →  Isolated · Protected     Unauthorized access blocked; authorized owner may recover
2'b11  →  Illegal · Alert          Anomaly detected / SEU event
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
| Gate count | **182 GE** (Vivado 2023.2, Artix-7 35T: 46 LUT + 22 FF) |
| ASIC estimate | ~300–500 GE @ 28nm |
| Silicon area @ 28nm | **< 0.002 mm²** |
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
*Vivado 2023.2 synthesis on Xilinx Artix-7 35T: 46 LUT + 22 FF = 182 GE*

![Waveform](./docs/waveform_s1_s6.png)
*GTKWave: S1–S6 core scenarios — Full match / Light deviation / Medium / Heavy / Fault injection / Reset*

</div>

---

## Use Cases

**Smart Locks / High-Security IoT**  
182 GE fits inside any lock MCU. EU EN 18031 mandatory from Aug 2025. SESIP L2 certification path supported.

**Automotive / ASIL-D / ISO 21434**  
4-layer SCA defense meets ISO 21434 hardware prerequisites. < 0.002 mm² area penalty on ECU silicon.

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

- 🇨🇳 **Chinese Invention Patent** — Application No. 2026106956971 (filed May 2026)
- 🌍 **PCT 5-country filing** in progress — CN / US / EU / JP / KR

---

## Contact

**OptiAura Tech — Shanghai Opti Aura Technology Co., Ltd.**

📩 lexxu@optiaura.tech  
🌐 optiaura.tech  
👤 [Lex Xu on LinkedIn](https://www.linkedin.com/in/lex-xu-optiaura/)

> *Full RTL code review available after NDA. Integration engineers supported throughout.*
