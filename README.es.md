<div align="center">

# AURA — Raíz de Confianza en Hardware

**182 GE (Base) · ~300 GE reforzado SCA · Defensa de 4 capas SCA · CMOS estándar / FPGA**

[![Patente](https://img.shields.io/badge/Patente-CN%20Solicitada%20×2-blue.svg)](https://github.com/AgentLex/aura-public)
[![Simulación](https://img.shields.io/badge/Simulación-22%2F22%20PASS-brightgreen.svg)](./docs/)
[![Base HRoT](https://img.shields.io/badge/Base%20HRoT-182%20GE-gold.svg)](./docs/synthesis_report.png)
[![SCA Reforzado](https://img.shields.io/badge/SCA%20Reforzado-~300%20GE-orange.svg)](./docs/arch_system_overview.png)

---

🌐 **Idioma / Language**

[English](./README.md) · [简体中文](./README.zh-CN.md) · [繁體中文](./README.zh-TW.md) · [日本語](./README.ja.md) · [Español](./README.es.md)

---

</div>

## El Problema

**42.000 millones de dólares** en pérdidas anuales por clonación de chips. La mayoría de los chips actuales autentican en la capa de firmware — quien controla el software, controla la identidad del chip.

Las soluciones existentes no caben donde el problema es más grave:

| Solución | Puertas Lógicas | Embebible en MCU/IoT | Anti-DPA | Anti-SEU |
|---|---|---|---|---|
| TPM 2.0 | ~50.000 GE | ✗ Demasiado grande | ✗ | ✗ |
| ARM TrustZone | Sobrecarga SW | Parcial | ✗ | ✗ |
| PUF solo | ~10.000 GE | Parcial | ✗ | ✗ |
| **AURA (Base)** | **182 GE** | **✓ Totalmente compatible** | **✓ 4 capas** | **✓ L2 Dual-Rail** |
| **AURA (SCA Reforzado)** | **~280–320 GE** | **✓ Totalmente compatible** | **✓ 4 capas activas** | **✓ L2 Dual-Rail** |

> **Aclaración sobre puertas lógicas:** 182 GE = Base HRoT (FPGA, medido con Vivado: 46 LUT + 22 FF). ~280–320 GE = las cuatro capas SCA completamente activas (FPGA). Estimación ASIC @ 28nm: ~300–500 GE.
>
> Una implementación hardware mínima de AES-128 requiere ~2.400 GE. Incluso la versión AURA reforzada con SCA ofrece vinculación de identidad + protección contra clonación + defensa SCA de 4 capas en solo el **13% del área de AES-128**.

---

## Mecanismo Central

Codificación de estados ternarios sobre CMOS binario estándar — **sin proceso especial requerido**.

```
2'b01 → Legítimo · Pasa    Operación normal
2'b10 → Aislado · Protegido    Acceso no autorizado bloqueado permanentemente; propietario autorizado puede recuperar
2'b11 → Ilegal · Alerta    Anomalía detectada / Evento SEU
```

> **Clave:** Una vez que `2'b10` entra en la cadena MAC, ninguna instrucción de software puede borrarlo.
> Es una restricción de hardware — fundamentalmente diferente de un flag de firmware o un enum de software.

---

## Defensa de 4 Capas SCA

| Capa | Mecanismo | Defiende Contra |
|---|---|---|
| L1 | LFSR enmascarado | DPA — eleva la complejidad O(2³²) → O(2⁴⁸) |
| L2 | Lógica Dual-Rail | DPA + Inyección de fallos + SEU (detección en 0 ciclos) |
| L3 | MAC en tiempo constante | Filtración por canal lateral de temporización |
| L4 | Inserción de retardo aleatorio | Alineación de trazas DPA (~100× más difícil) |

Las cuatro capas operan completamente en RTL — **sin dependencia de firmware**.

---

## Datos Clave

| Métrica | Valor |
|---|---|
| Área lógica — Base HRoT (FPGA) | **182 GE** (Vivado 2023.2, Artix-7 35T: 46 LUT + 22 FF) |
| Área lógica — SCA reforzado (FPGA) | **~280–320 GE** (4 capas SCA activas) |
| Estimación ASIC @ 28nm | **~300–500 GE** (SCA reforzado) |
| Área de silicio @ 28nm | **< 0,003 mm²** (SCA reforzado) |
| Coste adicional por chip | **< ¥0,1 / < $0,01** |
| Consumo de energía | **< 1 mW** (vs. TPM 2.0: 5–15 mW) |
| Simulación | **22/22 escenarios PASS** (Icarus Verilog) |
| Módulos RTL | 9 completos |

---

## Validación

<div align="center">

![Resultados de simulación](./docs/simulation_results.png)
*22/22 escenarios de simulación — verificación completa con Icarus Verilog*

![Informe de síntesis](./docs/synthesis_report.png)
*Síntesis Vivado 2023.2 en Xilinx Artix-7 35T: 46 LUT + 22 FF = 182 GE (Base HRoT)*

![Forma de onda](./docs/waveform_s1_s6.png)
*GTKWave: escenarios S1–S6 — Coincidencia total / Desviación leve / Media / Grave / Inyección de fallo / Reinicio*

</div>

---

## Diagramas de Arquitectura

<div align="center">

![Arquitectura del sistema](./docs/arch_system_overview.png)
*Defensa activa de 4 capas: Capa SCA → Aura 2 (Detección) → ESM (Decisión) → Aura 1 (Ejecución / HRoT)*

![Detalle de defensa SCA](./docs/arch_sca_defense.png)
*Tipo de ataque → Mecanismo de defensa → Efecto de seguridad — las 4 capas SCA mapeadas*

![Mecanismo de bloqueo persistente](./docs/arch_persistent_lock.png)
*Comprobación al arrancar + flujo de escritura del bloqueo — el bloqueo sobrevive al ciclo de energía*

![Codificación Dual-Rail](./docs/arch_dual_rail.png)
*Carril único estándar (consumo varía = vulnerable a DPA) vs. Lógica Dual-Rail (consumo constante = DPA imposible)*

</div>

---

## Vídeos de Demostración

*Grabaciones en vivo en FPGA Artix-7 35T — subiéndose a GitHub Releases próximamente. ⭐ Dale Star para recibir notificaciones.*

| # | Escenario | Descripción | Enlace |
|---|---|---|---|
| 01 | Autenticación normal | Credencial correcta → autenticación exitosa | 🔜 Próximamente |
| 02 | Fuerza bruta → bloqueo | 3 fallos → bloqueo hardware irreversible | 🔜 Próximamente |
| 03 | Persistencia tras corte de luz | El bloqueo persiste tras apagar/encender | 🔜 Próximamente |
| 04 | Desbloqueo y recuperación | Credencial autorizada → operación restaurada | 🔜 Próximamente |
| 05 | Ataque de repetición bloqueado | Token reutilizado → detectado y rechazado | 🔜 Próximamente |
| 06 | Escalada de privilegios bloqueada | Salto de estado no autorizado → barrera hardware | 🔜 Próximamente |
| 00 | Demo completa | Los 7 escenarios — demostración completa | 🔜 Próximamente |

> Grabado en hardware real con salida serie UART a 115200 baudios. No es simulación.

---

## Propiedad Intelectual

- 🇨🇳 **Patente de invención china** — Solicitud No. **202610850983.0** (presentada en 2026)
- 🇨🇳 **Patente de invención china** — Solicitud No. **2026106956971** (presentada en mayo de 2026)
- 🌍 **Presentación PCT en 5 países** en curso — CN / EE.UU. / UE / JP / KR

> Código fuente RTL completo, restricciones de síntesis y términos de licencia de IP disponibles bajo NDA.

---

## Contacto

**OptiAura Tech — 超矩阵（上海）高科技有限公司**

📩 lexxu@optiaura.tech
🌐 optiaura.tech
👤 [Lex Xu en LinkedIn](https://www.linkedin.com/in/lex-xu-optiaura/)

> *Revisión completa del código RTL disponible tras la firma del NDA. Ingenieros de integración disponibles durante todo el proceso.*
