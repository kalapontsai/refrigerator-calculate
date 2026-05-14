# 冰箱制冷能力計算系統 - 完整驗證報告
# SR-C48DV 案例：V_R=350L, V_F=125L, Panasonic TKF100E23 變頻壓縮機

---

## 驗證結果總覽

| 題目 | 項目 | 計算值 | 基準解答 | 結果 |
|------|------|--------|---------|------|
| **第一題** | 等效內容積 V_eq | 572.5 L | 572.5 L | PASS |
| **第一題** | 基準 E.F. | 25.12 | 25.12 | PASS |
| **第一題** | 96%目標月耗電 | 21.88 kWh/月 | 21.88 | PASS |
| **第一題** | 目標實測EF | 26.17 | 26.16 | PASS |
| **第二題** | 冷媒質量流量 G | 2.414 kg/h | 2.41 | PASS |
| **第二題** | 蒸發器有效冷量 Q_E | 139.0 kcal/h | 138.8 | PASS |
| **第二題** | 運轉比 PR | 38.4% | 38.4% | PASS |
| **第三題** | VIP複合牆體 K_Fv | 0.1587 | 0.1587 | PASS |
| **第三題** | VIP區域熱負荷 Q_Fv | 3.81 kcal/h | 3.81 | PASS |
| **防結露** | 露點溫度 T_dp | 27.19°C | 27.2°C | PASS |
| **防結露** | 最小保溫厚度 d | 44.0 mm | 44.0 mm | PASS |

**全部 11 項驗證通過**

---

## 各題公式與程式對應

### 第一題：2027 年能效基準

```javascript
// 等效內容積
V_eq = V_R + K × V_F = 350 + 1.78 × 125 = 572.5 L

// 基準 E.F.（≥400L 風扇式）
EF_base = (1.3 × V_eq) / (0.031 × V_eq + 21.0) × 1.308 = 25.12

// 月耗電上限
monthly_limit = V_eq / EF_base = 572.5 / 25.12 = 22.79 kWh/月

// 96% 目標
target_monthly = 22.79 × 0.96 = 21.88 kWh/月
target_EF = V_eq / 21.88 = 26.17
```

對應函式：`calcEquivVolume()`, `calcTaiwanMEPSKcal()`, `generateSelfCheckSRC48DV()`

### 第二題：變頻壓縮機能力匹配

```javascript
// Panasonic TKF100E23：V_c=10cm³, η_v=0.827, v=0.740m³/kg
G = η_v × V_c × N × 60 / (v × 10^6)
  = 0.827 × 10.0 × 3600 × 60 / (0.740 × 10^6)
  = 2.414 kg/h

// 有效冷量 Q_E = G × ΔH
Q_E = 2.414 × (124.44 - 66.85) = 139.0 kcal/h

// 運轉比 PR = Q_L / Q_E
PR = 53.34 / 139.0 = 0.384 = 38.4%
```

對應函式：`calcCompressorMassFlow()`, `calcEvaporatorCooling()`, `calcRequiredPR()`, `calcInverterCompressorMatch()`

### 第三題：VIP 複合牆體熱傳

```javascript
// 複合牆體總熱阻
K_Fv = 1 / (1/α_o + d₁/λ_u + d₂/λ_v + 1/α_i)
     = 1 / (1/6 + 0.06/0.0165 + 0.012/0.005 + 1/10)
     = 1 / (0.167 + 3.636 + 2.400 + 0.100)
     = 0.1587 kcal/h·m²·°C

// VIP 區域熱負荷
Q_Fv = K_Fv × S × ΔT
     = 0.1587 × 0.5 × (30 - (-18))
     = 3.81 kcal/h
```

對應函式：`calcCompositeWallK()`, `calcVIPWallLoad()`, `calcVIPCompositeWall()`

### 防結露厚度驗證

```javascript
// 露點（Magnus 公式，T_o=30°C, RH=85%）
T_dp = 27.19°C

// 最小厚度
d = (1/K - 1/α_i) × λ = (1/0.372 - 1/10) × 0.017 = 44.0 mm
```

對應函式：`calcDewPoint()`, `calcMinInsulationKcal()`

---

## 程式碼檔案結構

| 檔案 | 說明 |
|------|------|
| `js/calculator.js` | 核心計算引擎（22KB），包含所有函式 |
| `pages/cabinet.html` | Step 1：冷藏/冷凍雙艙室尺寸與隔熱參數 |
| `pages/cooling_load.html` | Step 2：熱負荷計算明細（Chart.js 圖表）|
| `pages/compressor.html` | Step 3：壓縮機選型與 COP 分析 |
| `pages/energy.html` | Step 4：能耗評估 + **自檢模式（含 SR-C48DV 完整驗證）**|
| `pages/report.html` | Step 5：計算報告（JSON 匯入/匯出）|
| `css/style.css` | 樣式 |
| `index.html` | 主入口（iframe 導航）|

---

## 自檢功能使用方法

在 `http://localhost/refrigerator/pages/energy.html` 頁面底部：

1. **執行自檢**：手動填入參數進行單題驗證
2. **執行完整自檢（SR-C48DV）**：一鍵填入 SR-C48DV 全套參數，自動比對 6 大項目基準解答，螢幕顯示 PASS/FAIL 結果

---

## 驗證案例數據來源

| 項目 | 數據 | 來源 |
|------|------|------|
| V_R, V_F | 350L, 125L | SR-C48DV 開發規格 |
| K | 1.78 | 台灣 2027 法規（四星級）|
| T_E, T_C | -25°C, 38°C | 測試工況 |
| V_c | 10.0 cm³ | Panasonic TKF100E23 規格 |
| η_v | 0.827 | 壓縮機效能表 |
| v | 0.740 m³/kg | R600a 熱力表（@ -25°C）|
| ΔH | 57.59 kcal/kg | R600a 焓差（H8-H7）|
| Q_L | 53.34 kcal/h | 機構模組計算結果 |
| d₁, λ_u | 0.06m, 0.0165 | PU 發泡（環戊烷）|
| d₂, λ_v | 0.012m, 0.005 | VIP 真空隔熱板 |
| S_Fv | 0.5 m² | VIP 側板面積 |