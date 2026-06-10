<div align="center">

# AURA — 硬件信任根

**182 GE · 四层侧信道防御 · 标准CMOS / FPGA**

[![专利](https://img.shields.io/badge/专利-CN%202026106956971-blue.svg)](https://github.com/AgentLex/aura-public)
[![仿真](https://img.shields.io/badge/仿真-22%2F22%20全PASS-brightgreen.svg)](./docs/)
[![面积](https://img.shields.io/badge/面积-182%20GE-gold.svg)](./docs/synthesis_report.png)

---

🌐 **语言切换**

[English](./README.md) · [简体中文](./README.zh-CN.md) · [繁體中文](./README.zh-TW.md) · [日本語](./README.ja.md) · [Español](./README.es.md)

---

</div>

## 行业痛点

全球芯片克隆每年造成约 **$420亿** 损失。绝大多数芯片的身份认证逻辑在固件层——任何控制了软件的人，也就控制了芯片身份。

现有方案在最需要安全的场景里根本放不进去：

| 方案 | 门等效数 | 可嵌入MCU/IoT | 抗DPA侧信道 | 防SEU |
|---|---|---|---|---|
| TPM 2.0 | ~50,000 GE | ✗ 太大 | ✗ | ✗ |
| ARM TrustZone | 软件开销 | 部分 | ✗ | ✗ |
| PUF | ~10,000 GE | 部分 | ✗ | ✗ |
| **AURA** | **182 GE** | **✓ 完全兼容** | **✓ 四层防御** | **✓ L2双轨逻辑** |

> 单一 AES-128 的最小硬件实现约需 **2,400 GE**。  
> AURA 用 **182 GE** 同时实现身份绑定 + 克隆防护 + 四层SCA防御——仅为 AES-128 面积的 7.6%。

---

## 核心机制

在标准二进制CMOS上实现三值状态编码，**无需特殊工艺**。

```
2'b01  →  合法 · 通过       正常运行
2'b10  →  隔离 · 保护       非授权访问永远无法通过；合法所有者可凭证申请恢复
2'b11  →  非法 · 告警       检测到异常 / SEU事件
```

> **关键**：`2'b10` 一旦注入MAC链，任何软件指令均无法清除。  
> 这是硬件编码约束，与固件标志位或软件enum有本质区别。

---

## 四层侧信道防御

| 层级 | 机制 | 防御目标 |
|---|---|---|
| L1 | 掩码LFSR | DPA — 攻击复杂度 O(2³²) → O(2⁴⁸) |
| L2 | 双轨逻辑 | DPA + 故障注入 + SEU（0周期检测） |
| L3 | 常数时间MAC | 时序侧信道泄漏 |
| L4 | 随机延迟插入 | DPA轨迹对齐（难度提升约100倍） |

全部四层均在RTL层面实现，**无需固件支持**。

---

## 关键数据

| 指标 | 数值 |
|---|---|
| 逻辑面积 | **182 GE**（Vivado 2023.2，Artix-7 35T：46 LUT + 22 FF）|
| ASIC估算 | 28nm工艺约 300–500 GE |
| 硅片面积 | **< 0.002 mm²** @28nm |
| 单片成本附加 | **< ¥0.1 / 片** |
| 功耗 | **< 1 mW**（TPM 2.0 为 5–15 mW） |
| 仿真验证 | **22/22 场景全PASS**（Icarus Verilog）|
| RTL模块 | 9个完成 |

---

## 验证数据

<div align="center">

![仿真结果](./docs/simulation_results.png)
*22/22 仿真场景全PASS — Icarus Verilog 全栈验证*

![综合报告](./docs/synthesis_report.png)
*Vivado 2023.2 综合结果：Artix-7 35T，46 LUT + 22 FF = 182 GE*

![波形图](./docs/waveform_s1_s6.png)
*GTKWave：S1–S6 核心场景 — 完全匹配 / 轻度偏离 / 中度 / 重度 / 故障注入 / 复位恢复*

</div>

---

## 适用场景

**智能门锁 / 高安全IoT**  
182 GE 完美嵌入任何门锁MCU。2025年国内强制安全规范，欧盟EN 18031 2025年8月强制。支持SESIP L2认证路径。

**汽车电子 / ASIL-D / ISO 21434**  
四层SCA防御满足ISO 21434硬件层前置要求。在ECU硅片上附加面积 < 0.002 mm²。

**工业控制器 / 国防电子**  
FPGA版本现已可用。身份绑定从硬件层阻断供应链伪造。

---

## 开始评估

AURA 提供 NDA 下的技术评估。

**FPGA 评估（Artix-7 / Basys 3）**
```
联系我们 → 签署NDA → 获取 FPGA Starter Kit：
RTL接口定义 + 集成文档 + 仿真验证脚本
典型评估周期：2–4 周
```

**ASIC 集成**
```
建议流片前 3–6 个月启动技术对接
NDA后提供完整RTL代码包 + 综合约束 + 时序报告
支持 SESIP L2 / ISO 21434 认证资料配套
```

---

## 知识产权

- 🇨🇳 **中国发明专利** — 申请号 2026106956971（2026年5月申请）
- 🌍 **PCT五国布局** 进行中 — 中 / 美 / 欧 / 日 / 韩

---

## 联系方式

**超矩阵（上海）高科技有限公司 / OptiAura Tech**

📩 lexxu@optiaura.tech  
🌐 optiaura.tech  
👤 [徐凌之 LinkedIn](https://www.linkedin.com/in/lex-xu-optiaura/)

> *完整RTL代码在NDA签署后提供审阅，全程技术对接支持。*
