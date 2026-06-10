<div align="center">

# AURA — 硬體信任根

**182 GE · 四層側信道防禦 · 標準CMOS / FPGA**

[![專利](https://img.shields.io/badge/專利-CN%202026106956971-blue.svg)](https://github.com/AgentLex/aura-public)
[![模擬](https://img.shields.io/badge/模擬-22%2F22%20全PASS-brightgreen.svg)](./docs/)
[![面積](https://img.shields.io/badge/面積-182%20GE-gold.svg)](./docs/synthesis_report.png)

---

🌐 **語言切換**

[English](./README.md) · [简体中文](./README.zh-CN.md) · [繁體中文](./README.zh-TW.md) · [日本語](./README.ja.md) · [Español](./README.es.md)

---

</div>

## 產業痛點

全球晶片仿冒每年造成約 **$420億** 損失。絕大多數晶片的身份認證邏輯在韌體層——任何控制了軟體的人，也就控制了晶片身份。

現有方案在最需要安全的場景裡根本放不進去：

| 方案 | 閘等效數 | 可嵌入MCU/IoT | 抗DPA側信道 | 防SEU |
|---|---|---|---|---|
| TPM 2.0 | ~50,000 GE | ✗ 太大 | ✗ | ✗ |
| ARM TrustZone | 軟體開銷 | 部分 | ✗ | ✗ |
| PUF | ~10,000 GE | 部分 | ✗ | ✗ |
| **AURA** | **182 GE** | **✓ 完全相容** | **✓ 四層防禦** | **✓ L2雙軌邏輯** |

> 單一 AES-128 的最小硬體實作約需 **2,400 GE**。  
> AURA 用 **182 GE** 同時實現身份綁定 + 仿冒防護 + 四層SCA防禦——僅為 AES-128 面積的 7.6%。

---

## 核心機制

在標準二進位CMOS上實現三值狀態編碼，**無需特殊製程**。

```
2'b01  →  合法 · 通過       正常運作
2'b10  →  隔離 · 保護       非授權存取永遠無法通過；合法所有者可憑證申請恢復
2'b11  →  非法 · 告警       偵測到異常 / SEU事件
```

> **關鍵**：`2'b10` 一旦注入MAC鏈，任何軟體指令均無法清除。  
> 這是硬體編碼約束，與韌體旗標或軟體enum有本質區別。

---

## 四層側信道防禦

| 層級 | 機制 | 防禦目標 |
|---|---|---|
| L1 | 遮罩LFSR | DPA — 攻擊複雜度 O(2³²) → O(2⁴⁸) |
| L2 | 雙軌邏輯 | DPA + 故障注入 + SEU（0週期偵測） |
| L3 | 常數時間MAC | 時序側信道洩漏 |
| L4 | 隨機延遲插入 | DPA軌跡對齊（難度提升約100倍） |

---

## 關鍵數據

| 指標 | 數值 |
|---|---|
| 邏輯面積 | **182 GE**（Vivado 2023.2，Artix-7 35T：46 LUT + 22 FF）|
| ASIC估算 | 28nm製程約 300–500 GE |
| 矽晶面積 | **< 0.002 mm²** @28nm |
| 單片成本增加 | **< ¥0.1 / 片** |
| 功耗 | **< 1 mW**（TPM 2.0 為 5–15 mW）|
| 模擬驗證 | **22/22 場景全PASS**（Icarus Verilog）|

---

## 驗證數據

<div align="center">

![模擬結果](./docs/simulation_results.png)
*22/22 模擬場景全PASS — Icarus Verilog 全棧驗證*

![合成報告](./docs/synthesis_report.png)
*Vivado 2023.2：Artix-7 35T，46 LUT + 22 FF = 182 GE*

![波形圖](./docs/waveform_s1_s6.png)
*GTKWave：S1–S6 核心場景*

</div>

---

## 知識產權

- 🇨🇳 **中國發明專利** — 申請號 2026106956971（2026年5月）
- 🌍 **PCT五國佈局** 進行中 — 中 / 美 / 歐 / 日 / 韓

---

## 聯絡方式

**超矩陣（上海）高科技有限公司 / OptiAura Tech**

📩 lexxu@optiaura.tech  
🌐 optiaura.tech  
👤 [徐凌之 LinkedIn](https://www.linkedin.com/in/lex-xu-optiaura/)
