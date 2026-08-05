<div align="center">

# AURA — Raíz de Confianza Hardware Compacta

**182 GE (Base) · ~300 GE endurecido contra SCA · Vinculación de identidad a nivel de silicio · CMOS / FPGA estándar**

[![Patente](https://img.shields.io/badge/Patente-CN%20Solicitada%20%C3%972-blue.svg)](https://github.com/AgentLex/aura-public)
[![Simulación](https://img.shields.io/badge/Simulación-22%2F22%20PASS-brightgreen.svg)](./docs/)
[![Base HRoT](https://img.shields.io/badge/Base%20HRoT-182%20GE-gold.svg)](./docs/synthesis_report.png)
[![SCA](https://img.shields.io/badge/Endurecido%20SCA-~300%20GE-orange.svg)](./docs/arch_system_overview.png)

---

🌐 **Idioma / Language**

[English](./README.md) · [简体中文](./README.zh-CN.md) · [繁體中文](./README.zh-TW.md) · [日本語](./README.ja.md) · [Español](./README.es.md)

---

</div>

## El problema

La mayoría de los dispositivos embebidos autentican en firmware. Quien controla el software controla la identidad del dispositivo. Para equipos industriales basados en FPGA y nodos IoT sensibles al coste, la clonación a nivel de placa es un problema comercial persistente, y las contramedidas disponibles no resultan satisfactorias:

- **Cifrado del bitstream**: existen vulnerabilidades de canal lateral publicadas para las principales familias de FPGA.
- **Chips de autenticación externos**: baratos, pero expuestos en el bus y vulnerables a ataques de intermediario.
- **Subsistemas de seguridad completos** (TPM, elementos seguros, IP RoT comercial): están diseñados —y tarifados— para silicio de alta garantía. No encajan en el presupuesto de área, consumo ni licencia de los dispositivos donde la clonación realmente ocurre.

AURA se dirige a ese hueco: vinculación de identidad situada en la propia lógica, con una huella lo bastante pequeña como para convivir con la aplicación.

## Qué es AURA — y qué no es

AURA es un **ancla de identidad y estado**, no un motor criptográfico ni un sustituto de TPM.

**Proporciona:** vinculación de identidad del dispositivo, detección de clonación, un estado de aislamiento impuesto por hardware que el software no puede borrar, y detección de anomalías SEU.

**No proporciona:** cifrado de propósito general, almacenamiento de claves con garantías de elemento seguro, biblioteca criptográfica certificada, ni garantía Common Criteria a día de hoy.

Posicionamiento frente a tecnologías adyacentes:

| | Huella | ¿Cabe en MCU / FPGA pequeña? | Función principal |
|---|---|---|---|
| TPM 2.0 | ~50.000 GE | ✗ | Subsistema de seguridad completo |
| IP RoT comercial (tRoot, serie RT, clase PUFrt) | miles de GE | Parcial | Integridad de arranque + gestión de claves |
| Macro PUF sola | ~10.000 GE | Parcial | Fuente de clave inclonable |
| Chip de autenticación externo | componente aparte | ✓ | Autenticación desafío-respuesta (expuesta en bus) |
| **AURA (Base)** | **182 GE** | **✓** | **Vinculación de identidad + estado de aislamiento** |
| **AURA (endurecido SCA)** | **~280–320 GE** | **✓** | **Lo anterior, con endurecimiento de canal lateral** |

Esta tabla compara huella y encaje, **no equivalencia funcional**. AURA ocupa otro nivel: no compite con un TPM en garantía, sino que atiende diseños en los que un TPM nunca fue una opción.

**Base del recuento de puertas:** 182 GE = HRoT base, medido en FPGA (Vivado 2023.2, Artix-7 35T: 46 LUT + 22 FF). ~280–320 GE = las cuatro capas SCA activas (FPGA). ASIC a 28 nm: ~300–500 GE — **estimación**, no medida en silicio.

## Mecanismo central

Codificación de estado ternario sobre CMOS binario estándar — sin necesidad de proceso especial.

| Estado | Significado | Comportamiento |
|---|---|---|
| `2'b01` | Legítimo | Operación normal |
| `2'b10` | **Aislado** | Acceso bloqueado; **el propietario autorizado puede recuperar mediante verificación de credenciales** |
| `2'b11` | Ilegal / Alerta | Anomalía o evento SEU detectado |

**Propiedad clave:** una vez que `2'b10` entra en la cadena MAC, ninguna instrucción software puede borrarlo. La recuperación exige verificación de credenciales por una ruta definida en hardware.

**Esto es deliberado, y no es un ladrillo.** Para el propietario del producto, un evento de aislamiento es una condición reparable con un procedimiento de recuperación definido —y al ser recuperable, puede gestionarse como flujo de soporte en lugar de como devolución. Para un atacante con una unidad clonada, no existe ninguna vía software de retorno.

## Endurecimiento de canal lateral en cuatro capas

| Capa | Mecanismo | Objetivo |
|---|---|---|
| L1 | LFSR enmascarado | Análisis diferencial de potencia (DPA) |
| L2 | Lógica de doble raíl | DPA, inyección de fallos, SEU (detección en 0 ciclos) |
| L3 | MAC de tiempo constante | Canales laterales temporales |
| L4 | Inserción de retardo aleatorio | Alineación de trazas DPA |

Las cuatro capas operan en RTL — sin dependencia de firmware.

> **Estado de verificación.** Estos mecanismos están implementados y verificados funcionalmente en simulación. **La evaluación física de canal lateral (TVLA, según ISO/IEC 17825) está en curso.** Hasta que esos datos se publiquen aquí, ninguna afirmación de resistencia medida frente a ataques debe considerarse validada. Preferimos declararlo abiertamente antes que un evaluador lo descubra.

## Cifras

| Métrica | Valor | Base |
|---|---|---|
| Puertas — HRoT base (FPGA) | 182 GE (46 LUT + 22 FF) | Medido, Vivado 2023.2, Artix-7 35T |
| Puertas — endurecido SCA (FPGA) | ~280–320 GE | Medido |
| ASIC a 28 nm | ~300–500 GE | Estimación |
| Área de silicio a 28 nm | < 0,003 mm² | Estimación |
| Consumo | < 1 mW | Estimación |
| Simulación funcional | 22/22 escenarios PASS | Icarus Verilog |
| Módulos RTL | 9 completos | — |
| SCA física (TVLA) | En curso | — |

## Material de validación

- 22/22 escenarios de simulación funcional — Icarus Verilog
- Informe de síntesis Vivado 2023.2 (Artix-7 35T) — [`docs/synthesis_report.png`](./docs/synthesis_report.png)
- Trazas GTKWave, escenarios S1–S6 — [`docs/waveform_s1_s6.png`](./docs/waveform_s1_s6.png)

## Arquitectura

- Visión general — Capa de defensa SCA → Aura 2 (sensar) → ESM (decidir) → Aura 1 (ejecutar)
- Correspondencia tipo de ataque → mecanismo de defensa → efecto de seguridad
- Autocomprobación en arranque y flujo de escritura del estado de aislamiento — el estado sobrevive al ciclado de alimentación
- Comparación de perfil de consumo: raíl simple frente a doble raíl

Véase [`docs/`](./docs/).

## Demostración en hardware

Grabaciones en Artix-7 35T real: autenticación normal, aislamiento tras fallos repetidos, persistencia tras corte de alimentación, recuperación autorizada, rechazo de repetición y bloqueo de escalada de privilegios. Grabado en hardware real con salida UART a 115200 baudios — no es simulación.

En preparación para publicarse en GitHub Releases. ⭐ Marca con estrella para recibir aviso.

## A quién va dirigido

**Fabricantes de equipos basados en FPGA** — controladores industriales, instrumentación, equipos médicos y de ensayo que sufren clonación de placa. Desplegable sobre hardware existente; no requiere nuevo tape-out. **Es nuestro foco más inmediato.**

**IoT y silicio embebido** — cerraduras inteligentes y nodos de alta seguridad donde un subsistema de seguridad completo no cabe en el presupuesto de área o consumo. La norma EU EN 18031 es aplicable bajo la Directiva de Equipos Radioeléctricos desde agosto de 2025; SESIP L2 es una vía de certificación soportada.

**Automoción** — el endurecimiento SCA en cuatro capas aborda los prerrequisitos hardware de ISO 21434, con una penalización de área mínima en el silicio de la ECU. Ciclo de cualificación largo; se aborda como vía estratégica, no inmediata.

**Proyectos RISC-V SoC** — wrapper de integración AMBA APB en desarrollo.

## Cómo empezar

**Evaluación en FPGA (Artix-7 / Basys 3)** — contacto → NDA → paquete de evaluación: definiciones de interfaz RTL, documentación de integración, scripts de simulación. Evaluación típica: 2–4 semanas.

**Integración ASIC** — conviene iniciar 3–6 meses antes del tape-out. Bajo NDA se entrega el paquete RTL completo, restricciones de síntesis e informes de temporización.

## Propiedad intelectual

- Solicitud de patente de invención china n.º 202610850983.0 (presentada en 2026)
- Solicitud de patente de invención china n.º 2026106956971 (presentada en mayo de 2026)
- Solicitud PCT en curso — CN / US / EU / JP / KR

El código de este repositorio se publica únicamente para evaluación técnica. El uso comercial requiere licencia escrita independiente. Véase [LICENSE](./LICENSE).

## Contacto

**OptiAura Tech** — 上海若爻高科技有限公司

📩 lexxu@optiaura.tech · 🌐 optiaura.tech · 👤 [Lex Xu on LinkedIn](https://www.linkedin.com/in/lex-xu/)

Revisión completa del RTL disponible bajo NDA. Soporte de ingeniería de integración durante toda la evaluación.
