<div align="center">

# AURA — Compact Hardware Root of Trust

**182 GE (Base) · ~300 GE SCA-hardened · Silicon-level identity binding · Standard CMOS / FPGA**

[![Patent](https://img.shields.io/badge/Patent-CN%20Filed%20%C3%972-blue.svg)](https://github.com/AgentLex/aura-public)
[![Simulation](https://img.shields.io/badge/Simulation-22%2F22%20PASS-brightgreen.svg)](./docs/)
[![Base HRoT](https://img.shields.io/badge/Base%20HRoT-182%20GE-gold.svg)](./docs/synthesis_report.png)
[![SCA-hardened](https://img.shields.io/badge/SCA--hardened-~300%20GE-orange.svg)](./docs/arch_system_overview.png)

---

🌐 **Language / 语言 / 言語 / Idioma**

[English](./README.md) · [简体中文](./README.zh-CN.md) · [繁體中文](./README.zh-TW.md) · [日本語](./README.ja.md) · [Español](./README.es.md)

---

</div>

## The Problem

Most embedded devices authenticate in firmware. Whoever controls the software controls the device's identity. For FPGA-based industrial equipment and cost-sensitive IoT endpoints, board-level cloning is a persistent commercial problem, and the available countermeasures are unsatisfying:

- **Bitstream encryption** on mainstream FPGA families has published side-channel weaknesses.
- **External authentication chips** are cheap but vulnerable to man-in-the-middle interposition on the bus.
- **Full security subsystems** (TPM, secure elements, commercial RoT IP) are architected — and priced — for high-assurance silicon. They do not fit the area, power, or licensing budget of the devices where cloning actually happens.

AURA targets that gap: identity binding placed in the logic fabric itself, at a footprint small enough to sit alongside the application.

## What AURA Is — and Is Not

AURA is an **identity and state anchor**, not a cryptographic engine and not a TPM replacement.

**It provides:** device identity binding, clone detection, a hardware-enforced isolation state that software cannot clear, and SEU anomaly detection.

**It does not provide:** general-purpose encryption, key storage at secure-element assurance levels, a certified crypto library, or Common Criteria assurance today.

Positioning against adjacent technologies:

| | Footprint | Fits in MCU / small FPGA | Primary function |
|---|---|---|---|
| TPM 2.0 | ~50,000 GE | ✗ | Full security subsystem |
| Commercial RoT IP (tRoot, RT-series, PUFrt class) | thousands of GE | Partial | Boot integrity + key management |
| PUF macro alone | ~10,000 GE | Partial | Unclonable key source |
| Standalone authentication chip | external part | ✓ | Challenge–response auth (bus-exposed) |
| **AURA (Base)** | **182 GE** | **✓** | **Identity binding + isolation state** |
| **AURA (SCA-hardened)** | **~280–320 GE** | **✓** | **Same, with side-channel hardening** |

This table compares footprint and fit, not equivalent functionality. AURA occupies a different tier: it is not competing with a TPM on assurance, it is addressing designs where a TPM was never an option.

**Gate count basis:** 182 GE = Base HRoT, measured on FPGA (Vivado 2023.2, Artix-7 35T: 46 LUT + 22 FF). ~280–320 GE = all four SCA layers active (FPGA). ASIC estimate @ 28 nm: ~300–500 GE — estimate, not silicon-measured.

## Core Mechanism

Ternary-state encoding over standard binary CMOS — no special process required.

| State | Meaning | Behaviour |
|---|---|---|
| `2'b01` | Legitimate | Normal operation |
| `2'b10` | **Isolated** | Access blocked; **authorized owner can recover via credential verification** |
| `2'b11` | Illegal / Alert | Anomaly or SEU event detected |

**Key property:** once `2'b10` enters the MAC chain, no software instruction can clear it. Recovery requires credential verification through a hardware-defined path.

**This is deliberate, and it is not a brick.** For the product owner, an isolation event is a serviceable condition with a defined recovery procedure — and a recoverable one, which means it can be operated as a support workflow rather than an RMA. For an attacker holding a cloned unit, there is no software path back.

## Four-Layer Side-Channel Hardening

| Layer | Mechanism | Target |
|---|---|---|
| L1 | Masked LFSR | Differential power analysis |
| L2 | Dual-rail logic | DPA, fault injection, SEU (0-cycle detection) |
| L3 | Constant-time MAC | Timing side channels |
| L4 | Random delay insertion | DPA trace alignment |

All four layers operate in RTL — no firmware dependency.

> **Verification status.** These mechanisms are implemented and functionally verified in simulation. **Physical side-channel evaluation (TVLA, per ISO/IEC 17825) is in progress.** Until that data is published here, no claim of measured attack resistance should be considered validated. We would rather state this plainly than have an evaluator discover it.

## By the Numbers

| Metric | Value | Basis |
|---|---|---|
| Gate count — Base HRoT (FPGA) | 182 GE (46 LUT + 22 FF) | Measured, Vivado 2023.2, Artix-7 35T |
| Gate count — SCA-hardened (FPGA) | ~280–320 GE | Measured |
| ASIC @ 28 nm | ~300–500 GE | Estimate |
| Silicon area @ 28 nm | < 0.003 mm² | Estimate |
| Power | < 1 mW | Estimate |
| Functional simulation | 22/22 scenarios PASS | Icarus Verilog |
| RTL modules | 9 complete | — |
| Physical SCA (TVLA) | In progress | — |

## Validation Artifacts

- 22/22 functional simulation scenarios — Icarus Verilog
- Vivado 2023.2 synthesis on Artix-7 35T — see [`docs/synthesis_report.png`](./docs/synthesis_report.png)
- GTKWave traces, scenarios S1–S6 — see [`docs/waveform_s1_s6.png`](./docs/waveform_s1_s6.png)

## Architecture

- System overview — SCA Defense Layer → Aura 2 (sense) → ESM (decide) → Aura 1 (execute)
- Attack type → defense mechanism → security effect mapping
- Power-on boot check and isolation-state write flow — state survives power cycling
- Single-rail vs. dual-rail power profile comparison

See [`docs/`](./docs/).

## Hardware Demonstration

Live recordings on Artix-7 35T covering normal authentication, repeated-failure isolation, power-cycle persistence, authorized recovery, replay rejection, and privilege-escalation blocking. Recorded on real hardware with UART output at 115200 baud — not simulation.

Publishing to GitHub Releases. ⭐ Star to be notified.

## Who This Is For

**FPGA-based equipment manufacturers** — industrial controllers, instrumentation, medical and test equipment facing board-level cloning. Deployable on existing hardware today; no respin required. This is our nearest-term focus.

**IoT and embedded silicon** — smart locks and high-security endpoints where a full security subsystem does not fit the area or power budget. EU EN 18031 became applicable under the Radio Equipment Directive in August 2025; SESIP L2 is a supported certification path.

**Automotive** — 4-layer SCA hardening addresses ISO 21434 hardware prerequisites, at a small area penalty on ECU silicon. Longer qualification cycle; engaged as a strategic path rather than a near-term one.

**RISC-V SoC projects** — AMBA APB integration wrapper under development.

## Getting Started

**FPGA evaluation (Artix-7 / Basys 3)** — contact us → NDA → evaluation package: RTL interface definitions, integration docs, simulation scripts. Typical evaluation: 2–4 weeks.

**ASIC integration** — engage 3–6 months before tape-out. Full RTL package, synthesis constraints, and timing reports provided under NDA.

## Intellectual Property

- Chinese invention patent application No. 202610850983.0 (filed 2026)
- Chinese invention patent application No. 202610695697.1 (filed May 2026)
- PCT filing in progress — CN / US / EU / JP / KR

Source in this repository is published for technical evaluation only. Commercial use requires a separate written license. See [LICENSE](./LICENSE).

## Contact

**OptiAura Tech**

📩 lexxu@optiaura.tech · 🌐 optiaura.tech · 👤 [Lex Xu on LinkedIn](https://www.linkedin.com/)

Full RTL review available under NDA. Integration engineering support provided throughout evaluation.
