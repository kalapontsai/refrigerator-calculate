# 冰箱制冷能力計算系統

一套完整的冰箱制冷能力計算網頁系統，涵蓋從機構尺寸規劃到能耗評估的完整流程。

**存取網址：** `http://localhost/refrigerator/`

---

## 功能架構（5步驟）

```
Step 1: 機構尺寸與隔熱設計
Step 2: 制冷能力計算（熱負荷分析）
Step 3: 壓縮機選型
Step 4: 能耗評估
Step 5: 綜合報告輸出
```

---

## 目錄結構

```
refrigerator/
├── index.html              # 主入口頁面（分頁導航 iframe 架構）
├── SPEC.md                 # 規格文件
├── css/
│   └── style.css           # 全域樣式（響應式設計）
├── js/
│   └── calculator.js       # 核心計算引擎（熱力學計算）
├── pages/
│   ├── cabinet.html        # Step 1：機構尺寸
│   ├── cooling_load.html   # Step 2：制冷能力
│   ├── compressor.html     # Step 3：壓縮機選型
│   ├── energy.html          # Step 4：能耗評估
│   └── report.html          # Step 5：報告輸出
└── assets/                 # 圖片資源（預留）
```

---

## Step 1：機構尺寸與隔熱設計

**功能：**
- 輸入外部尺寸（寬 × 深 × 高，mm）
- 選擇隔熱材料（PU泡沫、VIP、EPS、XPS）
- 設定隔熱厚度
- 輸入環境溫度、濕度

**輸出：**
- 內部容積計算（利用率）
- **防結露最小隔熱厚度**（Magnus 公式）
- 箱壁 U 值、熱傳係數
- VIP 效益分析

**公式來源（已修正）：**
```
防結露（維度修正版）：
d_min = λ × [ (T_o - T_i) / (h_o × (T_o - T_dp)) - (1/h_o) - (1/h_i) ] × 1000
熱傳：U = 1 / (1/h_i + d/λ + 1/h_o)
ASHRAE 2022 Chapter 17

開門負荷（修正版）：
Q_door = V × open_times × f × ρ_air × Cp × ΔT × 1.4 / (24×3600)
f=0.5冷藏/0.3冷凍，1.4=潛熱補償

內部熱源（修正版）：
Q_internal = avg_light + fan + avg_defrost
avg_light = light_W × (open_times × open_min / 60) / 24
```

---

## Step 2：制冷能力計算

**功能：**
- 設定每日開門次數與時間
- 設定內部負載（照明、風扇、除霜功率）
- 設定安全係數

**熱負荷來源（ASHRAE Ch17）：**

| 負荷類型 | 符號 | 說明 |
|---------|------|------|
| 箱壁傳導 | Q_wall | U × A × ΔT |
| 開門侵入 | Q_door | 容積 × open_times × f × ρ × Cp × ΔT × 1.4 / (24×3600) |
| 內部負載 | Q_internal | avg_light + fan + avg_defrost（照明僅開門時耗電） |
| 食品負荷 | Q_product | 約 Q_wall × 3%（呼吸熱） |
| 安全餘裕 | Q_safety | Q_base × 15%（獨立顯示） |
| **總計** | **Q_total** | **Q_base + Q_safety** |

**輸出：**
- 四項熱負荷分項數值（W）
- 總制冷能力（W / kcal/h / BTU/h / HP）
- 熱負荷分佈圓餅圖（Chart.js）

---

## Step 3：壓縮機選型

**功能：**
- 設定蒸發溫度、冷凝溫度
- 選擇冷媒（R-600a / R-134a / R-290 / R-410A）

**輸出：**
- 建議馬力（HP）及輸入功率（kW）
- 壓縮機類型建議（旋轉式 / 往復式 / 渦旋式 / 變頻渦旋）
- COP 分析（卡諾理論 vs 實際估算）
- 四種壓縮機類型比較表

**支援冷媒：**

| 冷媒 | 名稱 | GWP | 應用 |
|------|------|-----|------|
| R-600a | 異丁烷 | 3 | 目前家用冰箱主流 |
| R-134a | 四氟乙烷 | 1430 | 逐步淘汰中 |
| R-290 | 丙烷 | 3 | 小型冰箱，出口歐盟 |
| R-410A | R-32/R-125 | 2088 | 高溫制冷 |

**公式來源：**
```
理論 COP = T_evap(K) / (T_cond(K) - T_evap(K))
P_in = Q_e / COP_actual
ASHRAE 2022 Chapter 11, Chapter 17
```

---

## Step 4：能耗評估

**功能：**
- 設定每日運行小時數與負載係數
- 設定電價
- 評估改善方案（VIP / 變頻壓縮機）

**輸出：**
- 年度耗電量（kWh/年）
- EU ErP 能耗等級（A ~ G）
- EEI 能耗指數
- 年度電費預估
- VIP 效益分析（節省厚度 / 能耗）
- 變頻效益分析（COP 提升 / 省電量）

**EU ErP 等級（2021後）：**

| 等級 | EEI 範圍 |
|------|---------|
| A | < 30 |
| B | 30 ~ 42 |
| C | 42 ~ 55 |
| D | 55 ~ 80 |
| E | 80 ~ 95 |
| F | 95 ~ 110 |
| G | >= 110 |

**公式來源：**
```
AE(kWh/年) = P_in(kW) × 24h × 365天 × Runtime Factor
EEI = AE / AE_std × 100
EU ErP Regulation 2019/2019
```

---

## Step 5：綜合報告

**功能：**
- 完整計算摘要（所有步驟結果）
- 可列印格式（Print 按鈕）
- JSON 匯出功能

**輸出內容：**
- 機構尺寸與隔熱規格
- 熱負荷分項數值
- 壓縮機選型建議
- 能耗與 ErP 等級
- 備註與工程建議

---

## 計算引擎（calculator.js）

核心函式：

| 函式 | 功能 |
|------|------|
| `calcWallLoad()` | 箱壁傳導負荷計算 |
| `calcMinInsulation()` | 防結露最小厚度 |
| `calcDoorLoad()` | 開門侵入負荷 |
| `calcInternalLoad()` | 內部負載（照明/風扇/除霜） |
| `calcTotalCoolingLoad()` | 總制冷能力 |
| `calcCompressorSpec()` | 壓縮機選型計算 |
| `calcEnergyRating()` | 能耗與 ErP 等級 |
| `calcVolume()` | 內外部容積計算 |
| `calcVIPBenefit()` | VIP 效益分析 |

---

## 知識來源

1. **ASHRAE 2022 Handbook — Refrigeration (SI)**
   - Chapter 17：Household Refrigerators and Freezers
   - Chapter 19：Thermal Properties of Foods
   - Chapter 20：Cooling and Freezing Times
   - Chapter 11：Refrigerant Control Devices
   - 原始 PDF 位於：`/mnt/d/docker-volumn/ubuntu-apache2/html/ashrae2022/`

2. **EU ErP Regulation 2019/2019**

3. **ASHRAE Chapter Notes**（翻譯笔记）：
   `/mnt/d/docker-volumn/ubuntu-apache2/html/ashrae2022/chapters/`

---

## 技術說明

- **前端架構：** 純 HTML + CSS + Vanilla JavaScript（無框架依賴）
- **圖表：** Chart.js（CDN）
- **字體：** 支援中文字（WenQuanYi / AR PL UKai）
- **計算模式：** 全客戶端計算（所有數據存在 localStorage）
- **響應式設計：** 支援桌機與平板（手機适配中）
- **部署方式：** Apache2，URL `http://localhost/refrigerator/`

---

## 計算預設值

| 參數 | 預設值 |
|------|--------|
| 外部尺寸 | 700 × 650 × 1850 mm |
| 隔熱材料 | PU 泡沫（環戊烷）λ=0.021 |
| 隔熱厚度 | 50 mm |
| 環境溫度 | 30 °C |
| 箱內溫度 | 5 °C |
| 環境濕度 | 70% RH |
| 每日開門 | 20 次 |
| 安全係數 | 1.15 |
| 冷媒 | R-600a |
| 蒸發溫度 | -10 °C |
| 冷凝溫度 | 45 °C |

---

## 使用限制與注意事項

1. **計算結果為估算值**，實際制冷能力應以壓縮機廠商樣本與實測為準
2. **防結露計算**假設環境相對濕度70%，高濕環境建議增加隔熱厚度10-15%
3. **海拔修正**：高度 > 1000m 時，壓縮機功率需降功約 10%
4. **R-600a** 具有可燃性，維修時需注意通風
5. **R-290** 主要用於出口歐盟的小型冰箱（< 50L）

---

## 未來擴充方向

- [ ] 支援冷藏室 + 冷凍室雙溫區計算
- [ ] 加入壓縮機廠商實際型號資料庫
- [ ] 增加冷藏 / 冷凍溫度自訂（多艙室）
- [ ] 增加 Cost/Benefit 分析（VIP 額外成本 vs 節能效益）
- [x] 匯入/匯出 JSON（支援重新編輯）
- [ ] 支援冷藏室 + 冷凍室雙溫區計算