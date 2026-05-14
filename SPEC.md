# 冰箱制冷能力計算系統 - 規格書

## 1. 系統概述

- **目的**：冰箱制冷能力計算系統（雙艙室版），以 ASHRAE 2022 Handbook - Refrigeration 為理論基礎
- **用戶**：冰箱設計工程師、制冷技術人員
- **部署**：`D:\docker-volumn\ubuntu-apache2\html\refrigerator\` → `http://10.35.32.11/refrigerator/`
- **技術**：純 HTML + CSS + Vanilla JavaScript，Chart.js（CDN），無框架依賴

---

## 2. 系統架構

```
refrigerator/
├── index.html              # 主頁（iframe 導航）
├── css/style.css           # 全域樣式
├── js/calculator.js        # 核心計算引擎
├── pages/
│   ├── cabinet.html        # Step 1：機構尺寸與隔熱設計
│   ├── cooling_load.html    # Step 2：熱負荷計算
│   ├── compressor.html     # Step 3：壓縮機選型
│   ├── energy.html         # Step 4：能耗評估
│   └── report.html         # Step 5：綜合報告
└── assets/                 # 靜態資源
```

---

## 3. Step 1：機構尺寸與隔熱設計（cabinet.html）

### 輸入欄位

| 欄位 | 單位 | 預設值 | 說明 |
|------|------|--------|------|
| 寬度 | mm | 700 | 外部總寬度 |
| 深度 | mm | 650 | 外部總深度 |
| 高度 | mm | 1850 | 外部總高度 |
| 冷藏室高度比例 | % | 55 | 佔總高度的百分比 |
| 冷藏室庫內溫度 Ti | °C | 5 | |
| 冷藏室隔熱厚度 | mm | 45 | |
| 冷藏室隔熱材類型 | - | PU foam (cyclopentane) | λ=0.021 W/(m·K) |
| 冷藏室每日開門次數 | 次/日 | 20 | |
| 冷藏室每次開門時間 | min | 0.5 | |
| 冷藏室照明功率 | W | 5 | |
| 冷藏室風扇功率 | W | 5 | |
| 冷藏室除霜峰值 | W | 150 | |
| 冷藏室每日除霜次數 | 次/日 | 2 | |
| 冷藏室每次除霜時間 | min | 20 | |
| 冷藏室門面積比例 | - | 0.70 | |
| 冷凍室庫內溫度 Ti | °C | -18 | |
| 冷凍室隔熱厚度 | mm | 60 | |
| 冷凍室隔熱材類型 | - | PU foam (cyclopentane) | λ=0.021 W/(m·K) |
| 冷凍室每日開門次數 | 次/日 | 5 | |
| 冷凍室每次開門時間 | min | 0.3 | |
| 冷凍室照明功率 | W | 0 | |
| 冷凍室風扇功率 | W | 2 | |
| 冷凍室門面積比例 | - | 0.70 | |
| 環境溫度 To | °C | 30 | |
| 相對濕度 RH | % | 70 | |

### 輸出欄位（自動計算）

| 欄位 | 公式 |
|------|------|
| 露點溫度 Tdp | `γ = (a·To)/(b+To) + ln(RH/100)`，其中 a=17.27, b=237.7；`Tdp = b·γ/(a-γ)` |
| 冷藏室容積 VR | `VR = (W-2·ins_mm_R) × (D-2·ins_mm_R) × (H·R_pct-2·ins_mm_R) / 10^6` L |
| 冷凍室容積 VF | `VF = (W-2·ins_mm_F) × (D-2·ins_mm_F) × (H-H_R-H_mech-2·ins_mm_F) / 10^6` L |
| 冷藏室內部尺寸 | W-2·ins_mm_R × D-2·ins_mm_R × (H·R_pct-2·ins_mm_R) mm |
| 冷凍室內部尺寸 | 同上 |
| 冷藏室表面積 | `A_total = 2·(w·d + d·h + h·w) / 10^6` m²（外部尺寸） |
| 冷藏室 U 值（牆壁） | `U_wall = 1 / (1/h_i + d_m/λ + 1/h_o)`，h_i=10, h_o=6 W/(m²·K) |
| 冷凍室 U 值（牆壁） | 同上，各自代入參數 |
| 冷藏室防結露最小厚度 | `d_min = λ × [ (T_o - T_i) / (h_o × (T_o - T_dp)) - (1/h_o) - (1/h_i) ] × 1000` mm |
| 冷凍室防結露最小厚度 | 同上，各自代入參數 |
| 等效內容積（K=1.78） | `Veq = VR + 1.78 × VF` L |

---

## 4. Step 2：熱負荷計算（cooling_load.html）

### 輸入欄位（來自 Step 1 的 localStorage）

使用 Step 1 儲存的參數，包含 W, D, H, To, 各艙室 Ti, ins_mm, λ, open_times, open_min, light_W, fan_W, defrost_W, defrost_times, defrost_min。

### 輸出欄位（自動計算，頁面載入時執行）

| 欄位 | 說明 |
|------|------|
| 冷藏室箱壁傳導負荷 | `Q_wall = (U_wall·A_wall + U_door·A_door) × ΔT`，其中 ΔT = To - Ti |
| 冷藏室開門侵入負荷 | `Q_door = (V × open_times × f × ρ_air × Cp_air × ΔT × 1.4) / (24×3600)`，f=0.5冷藏/0.3冷凍，ρ_air=1.2 kg/m³, Cp_air=1005 J/(kg·K)，1.4=潛熱凝結補償 |
| 冷藏室內部熱源 | `Q_internal = avg_light_W + fan_W + avg_defrost_W`<br>avg_light = light_W × (open_times×open_min/60) / 24 |
| 冷藏室食品熱負荷 | `Q_product = Q_wall × 3%`（冷藏室） |
| 冷藏室總負荷（含安全係數） | `Q_base = Q_wall + Q_door + Q_internal + Q_product`<br>`Q_safety = Q_base × 15%`（安全餘裕，獨立顯示）<br>`Q_R = Q_base + Q_safety` |
| 冷凍室箱壁傳導負荷 | 同上，各自代入參數 |
| 冷凍室開門侵入負荷 | 同上 |
| 冷凍室內部熱源 | 照明+風扇（無除霜） |
| 冷凍室食品熱負荷 | `Q_product = Q_wall × 0.3%`（冷凍室） |
| 冷凍室總負荷（含安全係數 1.15） | `Q_F = (Q_wall + Q_door + Q_internal + Q_product) × 1.15` W |
| 系統總制冷負荷 | `Q_total = Q_R + Q_F` W |

> 熱負荷圖表由 Chart.js 甜甜圈圖呈現 10 項分類（8項熱負荷 + 2項安全餘裕）。

---

## 5. Step 3：壓縮機選型（compressor.html）

### 輸入欄位

| 欄位 | 單位 | 預設值 |
|------|------|--------|
| 冷凝溫度 Tcond | °C | 45 |
| 蒸發溫度 Tevap | °C | -10 |
| 冷媒種類 | - | R-600a |

### 輸出欄位（自動計算，頁面載入時執行）

| 欄位 | 公式 |
|------|------|
| 系統總制冷負荷 | 取自 localStorage `s.Q.Q_total` W |
| 卡諾 COP（理論上限） | `COP_carnot = T_evap_K / (T_cond_K - T_evap_K)`，T 為絕對溫度 |
| 實際 COP | `COP = 2.5 - 0.018×ΔT - 0.03×(T_evap+20)`，ΔT = Tcond - Tevap |
| 壓縮機輸入功率 | `P_in = Q_total / COP_actual` W |
| 建議功率（HP） | 級距：1/8, 1/6, 1/4, 1/3, 1/2, 3/4, 1 HP |
| 估計年耗電 | `(P_in/1000) × 24 × 365 × RF`，RF=0.55（冷凍室）或 0.40（冷藏室） |
| 冷媒 GWP | 取自 REFRIGERANTS 常數 |
| 冷媒 ODP | 取自 REFRIGERANTS 常數（均為 0） |
| 壓縮機類型建議 | P≤0.25HP→旋轉式，≤0.5HP→渦旋式，＞0.5HP→往復式 |

---

## 6. Step 4：能耗評估（energy.html）

### 輸入欄位

| 欄位 | 單位 | 預設值 |
|------|------|--------|
| 輸入功率 P_in | W | 0（自動帶入 Step 3 結果） |
| 運行係數 RF | - | 0.40（手動可調） |
| 冷藏室容積 VR | L | 自動帶入 Step 1 |
| 冷凍室容積 VF | L | 自動帶入 Step 1 |
| 等效容積法參數 | K 值 | 1.78（視星級） |
| 星級 | - | 3（三星級） |
| 冷卻方式 | - | 風扇式 |
| 目前 U 值 | W/(m²·K) | 自動帶入 Step 1 冷藏室 U 值 |
| VIP 保溫厚度 | mm | 50 |

### 輸出欄位（點擊「計算能耗」按鈕）

| 欄位 | 公式 |
|------|------|
| 年耗電量 | `AE = (P_in/1000) × 24 × 365 × RF` kWh/年 |
| EU ErP EEI | `EEI = (AE / (124 + 0.233 × V_total)) × 100` |
| EU ErP 等級 | A(<30), B(<42), C(<55), D(<80), E(<95), F(<110), G(>110) |
| 等效內容積 Veq | `Veq = VR + K × VF` L（K=星級係數） |
| 月耗電量 | `AE_monthly = AE / 12` kWh/月 |
| E.F.（實測） | `EF_actual = Veq / AE_monthly` L/kWh/月 |
| MEPS 基準 E.F. | `EF_base = (1.3 × Veq) / (a × Veq + b) × 效率百分比`<br>風扇式<400L: a=0.037, b=24.3；風扇式≥400L: a=0.031, b=21.0<br>直冷式<400L: a=0.033, b=19.7；直冷式≥400L: a=0.029, b=17.0 |
| 與 MEPS 比率 | `ratio = (EF_actual / EF_base) × 100` % |
| 台灣能效合格 | ratio ≥ 100% 為合格 |
| 每升耗電 | `per_L = AE / (VR + VF)` kWh/L/年 |
| VIP 節省厚度 | `d_PU - d_VIP` mm，視保溫效益分析 |
| 變頻年省金額 | `AE × 變頻節省比例 × 電價` 元/年 |

---

## 7. Step 5：綜合報告（report.html）

### 輸出欄位（從 localStorage 讀取所有步驟資料）

| 章節 | 內容 |
|------|------|
| 1. 冰箱尺寸與容積 | 外部尺寸、VR, VF, Vtotal, Veq、各艙室隔熱厚度/類型、U值 |
| 2. 熱負荷計算 | 各艙室箱壁/開門/內部/食品明細、系統總負荷 |
| 3. 壓縮機選型 | Tcond/Tevap、冷媒種類、建議HP、輸入功率、COP、年耗電 |
| 4. 能耗評估 | 年耗電、EU ErP等級、MEPS E.F.比對結果、每升耗電 |

報告支援 JSON 匯入/匯出、列印功能。

---

## 8. 計算引擎核心公式（calculator.js）

### 露點溫度
```
T_dp = b·γ/(a-γ)
γ = a·T/(b+T) + ln(RH/100)
a = 17.27, b = 237.7
```

### 箱壁傳導
```
R_wall = 1/h_i + d_m/λ + 1/h_o
U_wall = 1 / R_wall
Q_wall = (U_wall·A_wall + U_door·A_door) × ΔT
```
h_i=10 W/(m²·K)（內部），h_o=6 W/(m²·K)（外部），門厚=0.5×壁厚

### 開門侵入
```
Q_door = (V × 10% × ρ_air × Cp_air × ΔT) / (24×3600)
ρ_air = 1.2 kg/m³, Cp_air = 1005 J/(kg·K)
```

### 內部熱源
```
Q_internal = light_W + fan_W + (defrost_W × defrost_times × defrost_min) / (24×60)
```

### 總制冷能力（雙艙室）
```
Q_chamber = (Q_wall + Q_door + Q_internal + Q_product) × 1.15
Q_total = Q_R + Q_F
```

### COP 模型（R-600a 實測修正）
```
COP_actual = max(0.7, min(2.0, 2.5 - 0.018×ΔT - 0.03×(T_evap+20)))
ΔT = T_cond - T_evap
```

### 台灣 2027 MEPS
```
Veq = VR + K × VF（K依星級：2★=1.56, 2.5★=1.67, 3★/4★=1.78）
EF_base = (1.3 × Veq) / (a × Veq + b)（依型式與容積級距）
EF_actual = Veq / (AE/12)
pass = EF_actual ≥ EF_base
```

---

## 9. 常數對照表（calculator.js）

### 冷媒（REFRIGERANTS）
| 冷媒 | 分子量 | Tc (°C) | Pc (MPa) | ODP | GWP |
|------|--------|---------|----------|-----|-----|
| R-600a | 58.12 | 134.7 | 3.63 | 0 | 3 |
| R-134a | 102.03 | 101.1 | 4.07 | 0 | 1430 |
| R-290 | 44.10 | 96.7 | 4.25 | 0 | 3 |
| R-410A | 72.58 | 72.1 | 4.95 | 0 | 2088 |

### 隔熱材（INSULATION_TYPES）
| 類型 | λ (W/m·K) | 說明 |
|------|-----------|------|
| PU foam | 0.022 | 聚氨酯泡沫（傳統） |
| PU foam (cyclopentane) | 0.021 | 聚氨酯泡沫（環戊烷） |
| VIP | 0.005 | 真空隔熱板 |
| EPS | 0.034 | 保麗龍 |
| XPS | 0.029 | 押出聚苯乙烯 |

### 壓縮機效率（COMP_RESULTS）
| 類型 | ηv | ηmech | 說明 |
|------|-----|-------|------|
| rotary | 0.72 | 0.85 | 旋轉式（最常用於冰箱） |
| reciprocating | 0.70 | 0.82 | 往復式 |
| scroll | 0.78 | 0.90 | 渦旋式 |
| inverter | 0.82 | 0.92 | 變頻渦旋式（最高效率） |

---

## 10. 資料流向

```
Step 1 (cabinet.html)
  → localStorage: width_mm, depth_mm, height_mm, T_o, RH, R/F params, vol, wall_R, wall_F

Step 2 (cooling_load.html，自動執行)
  → localStorage: Q (Q_R, Q_F, Q_total), Q_R_detail, Q_F_detail

Step 3 (compressor.html，自動執行)
  → localStorage: compressor (T_cond_C, T_evap_C, refrigerant, P_input_W, COP_actual, ...)

Step 4 (energy.html，按鈕執行)
  → localStorage: energy (AE_kWh_year, runtime_factor, per_L, EEI, MEPS {...})

Step 5 (report.html)
  ← 從 localStorage 讀取所有資料，渲染報告
```