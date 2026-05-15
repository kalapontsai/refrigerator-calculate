# 冰箱制冷能力計算系統 - 完整公式總覽
# 冰箱網頁版計算引擎 v2（雙艙室版）+ 四大欠缺模組

---

## 一、容積與幾何

### 1.1 等效內容積（台灣 MEPS）
$$V = V_R + K \times V_F$$
- $K = 1.78$（ 四星/三星級）、$1.67$（超二星）、$1.56$（二星）

### 1.2 實際內容積
```
V_R = (W - 2×ins) × (D - 2×ins) × (H_R - ins) × door_ratio
V_F = (W - 2×ins) × (D - 2×ins) × H_F  × door_ratio
H_R = H × R_pct,  H_mech = H × 0.12,  H_F = H - H_R - H_mech
```

---

## 二、防結露厚度

### 2.1 露點溫度（Magnus 公式）
$$T_{dp} = \frac{237.7 \times \gamma}{17.27 - \gamma}, \quad \gamma = \frac{17.27 T_o}{237.7 + T_o} + \ln\frac{RH}{100}$$

### 2.2 最小隔熱厚度
$$K_{cond} = \frac{Q/S}{T_{dp} - T_i}, \quad d_{min} = \left(\frac{1}{K_{cond}} - \frac{1}{\alpha_i}\right) \lambda$$

---

## 三、熱負荷（各艙室分開計算）

### 3.1 牆體傳熱負荷
$$Q_{wall} = \frac{\Delta T \times A}{R_{total}}, \quad R_{total} = \frac{ins}{\lambda} + \frac{1}{\alpha_i} + \frac{1}{\alpha_o}$$

### 3.2 開門負荷（修正版）
$$Q_{door} = \frac{V \times open\_times \times f \times \rho_{air} \times C_p \times \Delta T \times 1.4}{24 \times 3600}$$
- $f = 0.5$（冷藏室）/ $0.3$（冷凍室）：換氣係數
- $\rho_{air} = 1.2$ kg/m³, $C_p = 1005$ J/(kg·K)
- $1.4$：潛熱凝結補償係數（水氣凝結潛熱效應）

### 3.3 內部熱負荷（修正版）
$$Q_{internal} = avg\_light\_W + W_{fan} + avg\_defrost\_W$$
- $avg\_light\_W = W_{light} \times \frac{open\_times \times open\_min / 60}{24}$（照明僅開門時耗電）
- $avg\_defrost\_W = \frac{W_{defrost} \times n_{def} \times t_{def}}{24 \times 60}$（除霜熱攤提）

### 3.4 艙室總熱負荷（安全係數獨立版）
$$Q_{base} = Q_{wall} + Q_{door} + Q_{internal} + Q_{product}$$
$$Q_{safety} = Q_{base} \times 15\% \quad \text{（安全餘裕，獨立顯示）}$$
$$Q_{total} = Q_{base} + Q_{safety}$$

---

## 四、壓縮機匹配

### 4.1 COP（經驗公式）
$$COP \approx 2.5 - 0.018 \times \Delta T - 0.03 \times (T_{evap} + 20)$$

### 4.2 質量流量
$$G = \frac{\eta_v \times V_c \times N_{rpm} \times 60}{v_{suction} \times 10^6} \quad \text{kg/h}$$

### 4.3 有效冷卻能力
$$Q_E = G \times \Delta H_{evap} \quad \text{kcal/h}$$

### 4.4 運轉比（Running Ratio, PR）
$$PR = \frac{Q_L}{Q_E}$$

### 4.5 PR=70%（低頻長時策略）匹配
$$Q_U = \frac{Q_L}{0.70}, \quad G = \frac{Q_U}{\Delta H_{evap}}, \quad N = \frac{G \times v \times 10^6}{\eta_v \times V_c \times 60}$$

---

## 五、能效標準（台灣 2027 MEPS）

### 5.1 基準 E.F.（風扇式雙門，V ≥ 400L）
$$E.F._{base} = \frac{1.3 \times V}{0.031 \times V + 21.0} \times 130.8\%$$

### 5.2 月均能耗
$$AE_{monthly} = \frac{V}{E.F._{actual}}, \quad E.F._{actual} = \frac{V}{AE_{monthly}}$$

### 5.3 是否合格
$$E.F._{actual} \geq E.F._{base}$$

---

## 六、VIP 複合牆體

### 6.1 複合牆體熱傳係數
$$R = \frac{1}{\alpha_o} + \frac{d_{PU}}{\lambda_{PU}} + \frac{d_{VIP}}{\lambda_{VIP}} + \frac{1}{\alpha_i}, \quad K_{Fv} = \frac{1}{R}$$

### 6.2 VIP 節省熱負荷
$$\Delta Q = (K_{old} - K_{new}) \times S \times \Delta T$$

---

## 七、四大欠缺模組（新增）

### 7.1 冷風迴路熱平衡（Cold Air Circuit）

**7.1.1 冷空氣密度（隨蒸發器出口溫度 T₂ 變動）**
$$\gamma = 1.2515 - 0.0049 \times T_2 + 0.000024 \times T_2^2 \quad \text{kg/m}^3$$

**7.1.2 蒸發器入口混合溫度**
$$T_1 = \frac{T_F \times M_F + T_R \times M_R}{M_F + M_R}$$

**7.1.3 風扇出口冷風溫度**
$$T_3 = T_2 + \frac{Q_{EV} + Q_{FM}}{M \times \gamma \times C_p \times PR}$$

**7.1.4 各艙室熱平衡反推風量**
$$M_R = \frac{Q_R}{\gamma \times C_p \times (T_R - T_3) \times PR}$$

### 7.2 細部機構熱負荷（取代 1.15 安全係數）

**7.2.1 門環熱負荷**
$$Q_{packing} = K_{fp} \times L_{packing} \times (T_o - T_i)$$

**7.2.2 法蘭防露冷凝管熱負荷**
$$T_{Flange} = \frac{K_1 T_o + K_2 T_F + K_3 T_R}{K_1 + K_2 + K_3}$$
$$Q_{dpc} = K_2 \times (T_C - T_{Flange}) \times PR \times L_{fr}$$

### 7.3 冷凝器熱交換與過冷度匹配

**7.3.1 冷凝器散熱分段匹配**
$$Q_C = G \times (H_3 - H_6), \quad Q_C = U_C \times SC \times (T_C - T_{air})$$

**7.3.2 R600a 焓值估算**
$$H_3 \approx 108 + 0.5 \times (T_C - 38), \quad H_6 \approx 44 + 0.3 \times (T_C - 38)$$

**7.3.3 疊代收斂冷凝溫度**（牛頓法）

### 7.4 動態貨物冷卻熱負荷（Product Cooling Load）

**7.4.1 產品冷卻熱負荷**
$$Q_{food} = \frac{m \times C_{food} \times (T_o - T_F)}{24 \times 3600} \quad \text{W}$$

**7.4.2 IEC 標準測試負載**
$$m_{test} = V_F \times \frac{3.5 \text{ kg}}{100 \text{ L}}$$
（冷凍食品含潜熱等效比熱 $C_{food} \approx 1940$ J/kg/°C）

---

## 八、程式函式對照表

| 函式 | 公式編號 | 功能 |
|------|---------|------|
| `calcDewPoint` | 2.1 | 露點溫度 |
| `calcVolumeV2` | 1.2 | 雙艙室內容積 |
| `calcMinInsulation` | 2.2 | 防結露厚度 |
| `calcCompartmentWallLoad` | 3.1 | 牆體傳熱 |
| `calcDoorLoad` | 3.2 | 開門負荷 |
| `calcInternalLoad` | 3.3 | 電氣部件熱負荷 |
| `calcCompartmentTotalLoad` | 3.4 | 艙室總熱負荷 |
| `calcTotalCoolingLoadV2` | - | 全系統熱負荷 |
| `calcCOP` | 4.1 | COP 經驗公式 |
| `calcCompressorSpecV2` | 4.2-4.4 | 壓縮機能量平衡 |
| `calcTaiwanMEPSV2` | 5.1-5.3 | 台灣 2027 MEPS |
| `calcVIPBenefit` | 6.1-6.2 | VIP 節能效益 |
| `calcAirDensity` | 7.1.1 | 冷空氣密度 |
| `calcMixedAirTemp` | 7.1.2 | 混合溫度 |
| `calcFanOutletTemp` | 7.1.3 | 風扇出口溫度 |
| `calcCompartmentAirFlow` | 7.1.4 | 風量反推 |
| `calcDoorPackingLoad` | 7.2.1 | 門環熱負荷 |
| `calcFlangeDPCLoad` | 7.2.2 | 防露管熱負荷 |
| `calcMechanismHeatLoad` | 7.2 | 機構漏熱汇总 |
| `getR600aEnthalpy` | 7.3.2 | R600a 焓值 |
| `calcCondenserHeat` | 7.3.1 | 冷凝器散熱 |
| `calcCondenserIteration` | 7.3.3 | 冷凝溫度疊代 |
| `calcProductCoolingLoad` | 7.4.1 | 產品冷卻熱負荷 |
| `calcIECKProductLoad` | 7.4.2 | IEC 測試負載熱負荷 |
| `generateSelfCheckSRC48DV` | - | SR-C48DV PR=38.4% 自檢 |
| `generateSelfCheckPR70` | - | PR=70% 策略自檢 |
| `generateSelfCheckColdAirCircuit` | - | 四大欠缺模組整合自檢 |

---

*文件更新：2026-05-10*