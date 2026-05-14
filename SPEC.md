# 冰箱制冷能力計算系統 - 規格書

## 1. Objective

建立一套完整的冰箱制冷能力計算網頁系統，涵蓋從機構尺寸規劃、制冷能力計算、壓縮機選型、保溫設計到能耗評估的完整流程。

目標用戶：冰箱設計工程師、制冷技術人員。

成功標準：用戶輸入箱體尺寸、隔熱規格、環境條件後，系統輸出完整制冷能力報告，包含壓縮機規格建議、能耗預估、散熱器選型。

---

## 2. Tech Stack

- 前端：純 HTML + CSS + Vanilla JavaScript（無框架依賴）
- 後端計算：客戶端 JavaScript 直接計算
- 圖表：Chart.js（CDN）
- 樣式：響應式設計，行動裝置友善
- 部署：Apache2（/mnt/d/docker-volumn/ubuntu-apache2/html/refrigerator/）

---

## 3. Commands

- 開發：直接編輯 .html 檔案，瀏覽器即時刷新
- 測試：http://localhost/refrigerator/
- 除錯：瀏覽器開發者工具（F12）

---

## 4. Project Structure

```
/mnt/d/docker-volumn/ubuntu-apache2/html/refrigerator/
├── SPEC.md                      # 本規格書
├── index.html                   # 主入口頁面（分頁導航）
├── css/
│   └── style.css                # 全域樣式
├── pages/
│   ├── cabinet.html             # 頁面1：機構尺寸與隔熱設計
│   ├── cooling_load.html        # 頁面2：制冷能力計算
│   ├── compressor.html          # 頁面3：壓縮機選型
│   ├── energy.html              # 頁面4：能耗評估
│   └── report.html              # 頁面5：綜合報告輸出
├── js/
│   ├── calculator.js             # 核心計算引擎（熱力學計算）
│   ├── compressor_data.js        # 壓縮機資料庫（模拟数据）
│   ├── chart.js                  # Chart.js 圖表渲染
│   └── ui.js                    # UI 互動邏輯
└── assets/
    └── logo.png                 # 選用logo
```

---

## 5. Code Style

- 變數命名：camelCase（JavaScript 標準）
- HTML id 命名：snake_case（如 `cabinet_width`）
- CSS class 命名：kebab-case（如 `input-group`）
- 中文字說明內嵌於 HTML，不抽出成 JS 字串
- 計算公式以函數包裝，註解標示公式來源（ASHRAE Handbook 章節）

---

## 6. Testing Strategy

- 單一 HTML 頁面可獨立開啟測試
- 計算結果與 ASHRAE Handbook 公式交叉驗證
- 手動輸入測試：已知案例（SR-C39）比對輸出

---

## 7. Boundaries

**Always do:**
- 所有計算結果顯示計算公式與參考來源
- 預設值基於 R-600a（異丁烷）系統
- 单位清楚標示（SI 制為主）

**Ask first:**
- 修改現有 refrig 目錄下的任何檔案
- 添加外部函式庫依賴

**Never do:**
- 不覆寫 refrig/ 目錄下的任何檔案

---

## 8. 計算流程（5步驟）

### Step 1 - 機構尺寸（cabinet.html）
用戶輸入：
- 外部尺寸（寬 × 深 × 高，mm）
- 內部容積（公升）
- 門數與門型（冷藏/冷凍配置）
- 隔熱材料種類（PU 泡沫、VIP 等）
- 隔熱厚度（mm）

輸出：
- 防結露最小保溫厚度計算
- 內部容積利用率
- 箱壁熱傳係數（U-value）

公式：
```
防結露：d_min = (1/K - 1/α_o) × λ
K = 外表面熱傳係數
α_o = 外部空氣熱傳係數（約6 kcal/hm²°C）
λ = 保溫材導熱係數（W/m·K）
```

### Step 2 - 制冷能力計算（cooling_load.html）
輸入：
- 環境溫度（°C）
- 環境濕度（%RH）
- 冷藏室目標溫度（°C）
- 冷凍室目標溫度（°C）
- 每日開門次數與時間

輸出：
- 通過箱壁的熱傳導負荷
- 門開啟侵入負荷
- 內部負載（照明、馬達、食物）
- 總制冷能力需求（W 或 kcal/h）

ASHRAE Handbook 熱負荷分類：
- Q_wall：箱壁傳導
- Q_infiltration：滲透與開門
- Q_internal：內部負載
- Q_product：食品負荷

### Step 3 - 壓縮機選型（compressor.html）
輸入：
- 制冷能力需求（W）
- 蒸發溫度（°C）
- 冷凝溫度（°C）
- 冷媒種類（R-600a / R-134a / R-290）

輸出：
- 壓縮機類型建議（往復/旋轉/渦旋）
- 建議功率（HP/kW）
- COP 預估值
- 廠牌型號建議（模擬資料庫）

ASHRAE 壓縮機選擇原則：
```
COP = Q_e / P_in
Q_e = 制冷能力（W）
P_in = 輸入功率（W）
```

### Step 4 - 隔熱機構選用（energy.html）
輸入：
- 已選壓縮機規格
- 目標能耗分級（EU ErP A+/A++/A+++）
- 年運行小時數

輸出：
- 預估年耗電量（kWh/年）
- 能耗指數（EEI）計算
- 改善建議（VIP 選用、變頻壓縮機評估）

### Step 5 - 綜合報告（report.html）
輸出完整計算報告（可列印格式）：
- 輸入參數摘要
- 每一步的計算結果
- 壓縮機與配件建議清單
- 能耗標籤預估

---

## 9. Open Questions

- 是否需要支援多種冷媒切換（R-600a / R-134a / R-290）？
- 是否需要儲存計算歷史？
- 壓縮機資料庫要從真實資料庫抓取，或先用模擬資料？