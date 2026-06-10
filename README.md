# AURA — Hardware Root of Trust

**182 GE. Four-layer SCA defense. Standard CMOS/FPGA.**

AURA is a hardware security IP core designed to be small enough to embed 
in any chip — including IoT MCUs and automotive silicon where existing 
solutions (TPM 2.0: ~50,000 GE) simply don't fit.

## What problem does this solve?

$42B in annual chip cloning losses. Most chips today authenticate in 
firmware — which means anyone who controls the software controls the 
identity. AURA puts the authentication layer below the software stack, 
in silicon.

## Core mechanism

Ternary-state encoding over standard binary CMOS:
- `2'b01` — Legitimate, pass
- `2'b10` — Security isolation (authorized owners can recover via 
   credential verification; unauthorized access permanently blocked)
- `2'b11` — Illegal state / SEU alert

Once the isolation state (`2'b10`) enters the MAC chain, no software 
instruction can clear it. This is a hardware constraint, not a software 
flag — fundamentally different from a firmware-based security check.

## Four-layer SCA defense

| Layer | Mechanism | Defends against |
|-------|-----------|----------------|
| L1 | Masked LFSR | DPA — lifts complexity O(2³²)→O(2⁴⁸) |
| L2 | Dual-rail logic | DPA + Fault injection + SEU |
| L3 | Constant-time MAC | Timing side-channel |
| L4 | Random delay insertion | DPA trace alignment (~100× harder) |

## Validation

- 22/22 simulation scenarios PASS (Icarus Verilog)
- Synthesized: 182 GE on Artix-7 35T (46 LUT + 22 FF), Vivado 2023.2
- 9 RTL modules complete
- Chinese Invention Patent No. 2026106956971
- PCT 5-country filing in progress (CN/US/EU/JP/KR)

## Getting Started

AURA is available for technical evaluation under NDA.

**For FPGA evaluation (Artix-7 / Basys 3):**
Contact us to receive the FPGA Starter Kit:
RTL interface definitions + integration documentation + simulation scripts

**For ASIC integration:**
Engage 3–6 months before tape-out.
Full RTL package + synthesis constraints + timing reports provided under NDA.

**For certification support (SESIP L2 / ISO 21434):**
Certification documentation package available upon request.

## Contact

Technical review available under NDA.  
lexxu@optiaura.tech | optiaura.tech
