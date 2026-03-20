# 📝 Feature Spec: Coupon 系統優化與 App 歸戶整合

| Meta | Details |
| :--- | :--- |
| **Version** | 1.0 |
| **Status** | Ready for Dev |
| **Owner** | Product Team |
| **Last Updated** | 2026-03-13 |

## 1. Overview

* **Objective:** 完善優惠券系統的端到端體驗，從後台管理新增自訂欄位，到 App 端的「推播歸戶機制」、「我的優惠券」專屬列表、詳情頁，並透過「會員頁面入口」提升優惠券領取率與整體互動黏著度。
* **Success Metrics:**
  * Primary: 優惠券推播點擊轉換率提升、優惠券領取與使用率提升。
* **Scope (In/Out):**
  * **In:**
    * 後端管理平台 (Web) 新增優惠券欄位
    * App 端推播攔截、點擊分流與自動歸戶機制
    * App 實作「我的優惠券」列表、「使用說明」詳情頁
    * 會員中心清單新增「我的優惠券」入口
  * **Out:** 結帳流程優惠券折抵邏輯 (原功能)、發送推播後端派發邏輯。
* **Related Project:**
  * N/A

---

## 2. Cross Sync (跨部門協作)

| | Info (資訊流) | Operation (營運操作) | Airpa Capacity (系統量能) | Platform Features (平台功能) |
| :--- | :--- | :--- | :--- | :--- |
| **Demand**<br>(流量/用戶端) | [3.2] 確認推播點擊的 URI 是否夾帶特殊參數 (coupon_id) | - | - | [3.2] App 提供歸戶用的暫存與深度跳轉流程 |
| **Supply**<br>(供給/產品端) | [3.3] 依據點擊帶入之 ID，串接內部 API 動態呈現優惠券資料 | [3.1] 管理員需於後台設定「優惠券顯示名稱」、「簡要說明」及「詳細規範」 | - | - |
| **Purchase**<br>(交易/金流端) | [3.3] 頁面頂部常駐文案，提示於結帳流程可折抵 | - | - | - |

**上線後 (Post-release):** 驗證推播通知點擊後的跳轉行為、歸戶 API 正確性，以及長時間運行下本地推播資料庫 (100 筆上限) 的淘汰機制定時清理是否穩定。

---

## 3. Feature Details (功能規格)

### 3.1 後台 Coupon 欄位擴充

#### **A. User Story (使用者情境)**

| Actor (角色) | Action (行為) | Outcome/Value (預期結果) |
| :--- | :--- | :--- |
| 後台管理員 | 設定 Coupon 顯示名稱、說明以及詳細規範 | 可以設定 Coupon 顯示名稱、說明以及詳細規範，以利前端能夠正確顯示這些資訊。 |

#### **B. UI & Interaction (介面互動)**

* **Visual Reference:** [Lovable Prototype] link [https://preview--artful-asset-space.lovable.app/?__lovable_token=eyJhbGciOiJ...]
* **UI States (關鍵狀態定義):**
  * **Trigger Timing:** 進入 `prog` > `create coupon` 頁面。
  * **Default:**
    * 常駐顯示：`Display for Customer`（預設 No）、`Coupon Name`
    * 隱藏：`Coupon Display Description`、`Coupon Details`
  * **Interaction:**
    * `Display for Customer = Yes` → 即時展開 `Coupon Display Description` 與 `Coupon Details`
    * `Display for Customer = No` → 即時隱藏上述兩欄位
    * 切換無需重整頁面；各欄位詳細規則與 Tooltip 見下方 **Logic**
  * **Empty/No Data:**
    * 各欄位留空均可正常送出
    * `Coupon Details` 留空時 fallback 至 Coupon Plan 描述（詳見 **Business Rules**）
  * **Error:**
    * 超字數上限 → 顯示 inline 錯誤並阻止送出
    * 上限：`Coupon Name` 60 字 / `Coupon Display Description` 100 字 / `Coupon Details` 2000 字
* **Logic:**
  * **Display for Customer (Radio: Yes / No):**
        1. 常駐顯示，為所有前台內容的主開關。
        2. 選 **Yes** → 展開顯示 Coupon Display Description 與 Coupon Details 兩個從屬欄位。
        3. 選 **No** → 隱藏 Coupon Display Description 與 Coupon Details 兩個從屬欄位。
        4. 切換時即時動態顯示／隱藏從屬欄位（不需重整頁面）。
        5. Tooltip 提示：「勾選 Yes 會顯示於前台 App 通知中心、我的優惠券以及優惠券細節頁面。」
  * **Coupon Name (Single-line Input):**
        1. 常駐顯示。
        2. 欄位下方輔助文：*上限 60 個字，若選擇 Display for Customer = Yes，此段文字會出現於前台展示給消費者。*
  * **Coupon Display Description (Single-line Input):**
        1. 僅在 Display for Customer = **Yes** 時顯示。
        2. 欄位下方輔助文：*上限 100 個字，此欄位用於摘要 coupon 如何使用，例如：首次購物折抵 $200。*
  * **Coupon Details (Textarea):**
        1. 僅在 Display for Customer = **Yes** 時顯示。
        2. 欄位下方輔助文：*上限 2000 個字，若留空則系統會自動抓取 Coupon Plan 內所提供的文字展示。*
  * **送出行為:** 儲存時將以上欄位整合進現有資料流程，一併送往 API。

#### **C. Data & Business Logic (資料邏輯)**

* **Data Source:** 從`prog` > `create_coupon`表單輸入。
* **Business Rules (商業規則):**
  * **Display for Customer 開關機制:** 勾選 Yes 後，三個欄位的資料才會透過 API 實際傳遞至前台渲染，應用於：我的優惠券、優惠券詳情、通知中心 Activity Tab。
  * **欄位字數驗證 (Server-side):**
    * `Coupon Name`: 上限 **60 字**
    * `Coupon Display Description`: 上限 **100 字**
    * `Coupon Details`: 上限 **2000 字**
  * **Coupon Details Fallback:** 若 `Coupon Details` 留空，系統自動以 Coupon Plan 所有 `coupon_plan_description` 的排列文字作為資料來源填入；若有輸入則優先使用後台內容。

#### **D. Analytics (數據埋點)**

* 請跳過這邊不用追蹤。

#### **E. Acceptance Criteria (驗收標準)**

* **Happy Path:**
  * [ ] 新欄位成功渲染於畫面上並可供輸入。
  * [ ] 確實驗證提交後之文案，能與既有欄位一併送往 API，且寫入成功。
* **Edge Cases:**
  * [ ] 字數恰好等於上限（Coupon Name 60 字 / Description 100 字 / Details 2000 字）時應可正常送出，不觸發錯誤。
  * [ ] 切換 `Display for Customer` 為 No 時，已填寫的 `Coupon Display Description` 與 `Coupon Details` 內容應保留（再次切回 Yes 時可回復）。
  * [ ] 編輯既有 Coupon 時，各欄位能正確回帶已儲存的值，且 `Display for Customer` 狀態與從屬欄位顯示一致。

---

### 3.2 推播通知分流與歸戶機制

#### 3.2.1

A. User Story (使用者情境)

| Actor (角色) | Action (行為) | Outcome/Value (預期結果) |
| :--- | :--- | :--- |
| 使用者 | 點擊帶有優惠券的推播通知後 | 身為一個使用者，我希望點擊帶有優惠券的推播通知後，系統可以聰明地引導我登入並自動幫我領取優惠券，不用讓我重頭尋找頁面。 |

B. UI & Interaction (介面互動)

* **Visual Reference:**
  * **返回邏輯 (Back Navigation):**

        ```mermaid
        flowchart TD
            A["使用者點擊推播通知進入頁面"] --> B{"推播 url / uri\n= coupon 列表？"}
            B -- "是 (uri = /coupons/)" --> C["進入「我的 coupon」頁面"]
            B -- "否 (uri ≠ /coupons/)" --> D["進入指定 url 頁面\n(view, list, index 等)"]
            C --> E["點擊返回 ←"]
            D --> F["點擊返回 ←"]
            E --> G["/member/\n（會員中心）"]
            F --> H["/notification/\n（通知中心）"]

            style G fill:#4CAF50,color:#fff,stroke:#388E3C
            style H fill:#2196F3,color:#fff,stroke:#1565C0
        ```

* **UI States (關鍵狀態定義):**
  * **Trigger Timing:** 使用者點擊夾帶 `coupon={{coupon_code}}` 與 `coupon_claim=on` 參數的推播通知。
  * **Default:** 依推播 URI 類型導向對應目標頁面。
  * **Interaction：**
    * **A. 目標 = 我的 coupon (`/coupons/`)**
            1. 已登入 + 歸戶成功 → 進入 coupon 列表 + Toast ({{ 歸戶成功文案 (1) }})
            2. 已登入 + 歸戶失敗 → 進入 coupon 列表 + Toast ({{ 歸戶失敗文案 (2) }})
            3. 未登入 → 導向登入頁 ({{ 登入提示通知文案 (3) }})，登入後跳回 coupon 列表並歸戶 + Toast
            4. 放棄登入 → 返回首頁
    * **B. 目標 ≠ 我的 coupon（其他頁面）**
            1. 已登入 + 歸戶成功 → 導向目標頁 + 背景歸戶 + Toast ({{ 歸戶成功文案 (1) }})
            2. 已登入 + 歸戶失敗 → 導向目標頁 + Toast ({{ 歸戶失敗文案 (2) }})
            3. 未登入 → 導向目標頁 + 彈出歸戶登入提示 ({{ 歸戶登入提示 (4) }})（含「登入」按鈕），登入後於原頁顯示歸戶 Toast
            4. 放棄登入 → 返回首頁
  * **Empty/No Data:** 缺少 `coupon` 或 `coupon_claim=on` → 視為一般推播，不觸發歸戶。
  * **Error:** 歸戶 API 失敗 / 優惠券已過期或已使用 → Toast ({{ 歸戶失敗文案 (2) }})，頁面跳轉不受影響。
* **Logic:**
  * 攔截使用者點擊 Notification 的行為，並解析該 notification 所夾帶的 URI 參數。判斷若同時帶有 `coupon={{coupon_code}}` 且 `coupon_claim=on`，則進入歸戶機制：

  * 畫面上對應的提示訊息如下：
    * **{{ 歸戶成功文案 (1) }}**
    * -- 定義: 跳轉進入「我的 coupon」，背景歸戶完成後的 Toast 提示
    * -- example value: 優惠券已領取！請在結帳頁面勾選使用，即可享有優惠價格！
    * -- Lokalise key:
    * **{{ 歸戶失敗文案 (2) }}**
    * -- 定義: 背景歸戶發生錯誤或條件不符時的 Toast 提示
    * -- example value: 歸戶失敗！此優惠券已過期或被使用過。
    * -- Lokalise key:
    * **{{ 登入提示通知文案 (3) }}**
    * -- 定義: 跳轉至「登入頁面」時的提示
    * -- example value: 為了自動領取您的專屬優惠券，請先完成登入！
    * -- Lokalise key:
    * **{{ 歸戶登入提示 (4) }}**
    * -- 定義: 在非 coupon 頁面時，提示使用者登入並提供「登入」按鈕點擊前往登入頁面
    * -- example value: 為了自動領取您的專屬優惠券，請先完成登入！[去登入]
    * -- Lokalise key:

C. Data & Business Logic (資料邏輯)

* **Data Source:** 推播通知本身夾帶之 URL 或 URI 內容資訊。
* **Business Rules (商業規則):**
  * **特殊參數判定:** 檢查 URI 中是否有同時夾帶 `coupon={{coupon_code}}` 且 `coupon_claim=on` 作為歸戶觸發判斷點。
  * **歸戶觸發條件:**
    * 若只有 `coupon={{coupon_code}}` 但 `coupon_claim=off` (或非 on)，則不執行背景歸戶，僅執行頁面跳轉。
    * 若同時擁有 `coupon={{coupon_code}}` 且 `coupon_claim=on`，系統才會在進入跳轉目標頁面時同步驅動歸戶 API。
  * **優惠券狀態檢查:** 歸戶時，需檢查該 coupon 是否已經過期、已使用（避免無效歸戶）。
  * **重複領取放行:** 若會員已經領取過此優惠券，系統不做阻擋，一樣可讓客人新增優惠券。但客人實際是否能重複使用，受優惠券本身的設定限制。
  * **支援頁面限制（僅限原生 App 頁面）:** 推播歸戶跳轉僅支援以下原生 App 頁面，不支援非原生（WebView）頁面：
    * `index` (首頁)
    * `list` (列表頁)
    * `view` (詳情頁)
    * `/zh-tw/packpage/...` (套裝行程)
    * `/zh-tw/special/...` (特惠專區)
    * `/zh-tw/cruise/` (郵輪首頁)
    * `/zh-tw/cruise/line/...` (郵輪航線)
    * `/zh-tw/journey/` (旅程)
    * `/coupons/` (優惠券列表，未來功能)

D. Analytics (數據埋點)

* Event Name: `BebitTech_click_push`
* **Parameters:**
  * `visitor_id` [String]
  * `crm_id` [Number]
  * `aff_id` [Number]
  * `utm_source` [String]
  * `utm_medium` [String]
  * `utm_campaign` [String]
  * `DPL` [String]
  * `url` [String]

E. Acceptance Criteria (驗收標準)

* **Happy Path:**
  * [ ] 檢測未登入點擊具歸戶參數之推播，若目標為「我的 coupon」，系統應順利要求登入，完成後跳回清單頁。
  * [ ] 檢測未登入點擊具歸戶參數之推播，若目標非「我的 coupon」，系統應先跳轉至該頁面並彈出「歸戶登入提示 (4)」。
  * [ ] 若目標非「我的 coupon」點擊提示訊息中的「登入」後順利前往登入頁，完成後返回原頁面並正確顯示歸戶成功/失敗之 Toast。
  * [ ] 登入成功（或已登入狀態）跳至目標頁面後，背景成功呼叫歸戶 API 並跳出「歸戶成功」或「失敗」提示。
  * [ ] 會員點擊已歸戶過之推播優惠券，依然順利導向目標頁面，且系統判定重複歸戶不報錯或崩潰。
  * [ ] 缺少歸戶參數的普通推播，不管是否有登入均直達一般活動或商品頁面無誤。
  * [ ] 透過 Universal Link (iOS) 或 Android App Link 進入 App 時，系統能比照推播邏輯，於進入之 Deep Link 頁面完成背景歸戶並正常顯示成功/失敗之 Toast。
* **Edge Cases:**
  * [ ] 若客人於跳出的「登入頁面」點擊關閉或取消放棄登入，系統需正確退回首頁，不作後續導向或歸戶動作。
  * [ ] 點擊參數對應之優惠券若已過期或狀態無效時，系統導回目標頁面後，背景需拋出對應的「歸戶失敗」Toast。
  * [ ] 推播 URI 指向不支援的非原生頁面（WebView）時，系統應僅執行一般跳轉，不觸發歸戶流程且不報錯。
  * [ ] 網路斷線或歸戶 API 逾時時，頁面跳轉行為不受影響，僅顯示歸戶失敗 Toast。
  * [ ] 使用者短時間內連續點擊多則歸戶推播，系統應逐則處理且不產生重複歸戶衝突或 App 崩潰。
  * [ ] 登入過程中 session / token 過期導致登入失敗時，系統應正確回到登入前狀態，不觸發歸戶。

---

### 3.3 「我的優惠券」清單與詳情頁

#### 3.3.1 「我的優惠券」清單

A. User Story (使用者情境)

| Actor (角色) | Action (行為) | Outcome/Value (預期結果) |
| :--- | :--- | :--- |
| 會員 | 進入「我的優惠券」頁面 | 身為一個會員，我想要有一個專屬頁面可以查看我目前擁有或已經歸戶的優惠券。 |

B. UI & Interaction (介面互動)

* **Visual Reference:** [Lovable Prototype](https://preview--link-claim.lovable.app/?__lovable_token=eyJhbGciOiJ...)
* **UI States (關鍵狀態定義):**
  * **Trigger Timing:** 會員進入「我的優惠券」頁面（需已登入，否則導向登入頁）。
  * **Default (List State):**
    * 頂部常駐顯示提示文案 ({{ 靜態文案 (1) }})
    * 下方依序列出優惠券卡片（名稱、代碼、描述、效期、使用說明按鈕）
  * **Interaction:**
    * 點擊 Coupon Code → 複製至剪貼簿 + Toast ({{ 複製成功 Toast }})
    * 點擊「使用說明」→ 進入優惠券詳情頁（3.3.2）
  * **Empty/No Data:** 顯示空狀態插圖 + 提示文案 ({{ 靜態文案 (2) }})
  * **Error:** API 呼叫失敗時，顯示通用錯誤提示並提供重試按鈕。
* **Logic:**
  * 畫面頂端常駐顯示提示文案 ({{ 靜態文案 (1) }})：「結帳時，您可以選擇已存入的優惠券。提醒！已使用、已過期的優惠券會自動清除。」
  * **分頁與加載 (Infinite Scroll):**
    * 此頁面採用無限捲動機制。
    * 每頁一次顯示 **5 筆** coupon。
    * 使用者往下滾動至接近底部時，系統自動啟動 Loading 狀態並加載後續 5 筆 coupon。
  * 提供**複製功能**: 點擊列表之 Coupon Code 拷貝字串至剪貼簿並閃出 Toast 提示成功 ({{ 複製成功 Toast }})。
  * 畫面上對應的靜態文案如下：
    * **{{ 靜態文案 (1) }}**
    * -- 定義: 頁面頂部常駐提示文案
    * -- example value: 結帳時，您可以選擇已存入的優惠券。提醒！已使用、已過期的優惠券會自動清除。
    * -- Lokalise key:
    * **{{ 靜態文案 (2) }}**
    * -- 定義: Empty State 沒有優惠券時顯示的提示
    * -- example value: 目前沒有可使用的優惠券喔！
    * -- Lokalise key:
    * **{{ 複製成功 Toast }}**
    * -- 定義: 複製優惠碼成功後的 Toast 提示
    * -- example value: 已複製優惠碼
    * -- Lokalise key:
  * ***Copywriting — 優惠券卡片 (Coupon Card) 欄位定義：***
    * 每張優惠券卡片依序顯示以下內容：
            1. **{{ 卡片標題 }}**
      * -- 定義: 優惠券名稱（動態帶入）
      * -- example value: 新會員首購優惠
      * -- data field: `coupon_name`

            1. **{{ 卡片描述 }}**
      * -- 定義: 優惠券顯示描述（動態帶入 coupon_display_description）
      * -- example value: 僅限新會員首購使用，折抵上限 100 元
      * -- data field: `coupon_display_description`；若為空依 3.1 **Business Rules** fallback
            1. **{{ 卡片優惠碼 }}**
      * -- 定義: 優惠碼代碼（動態帶入），點擊可複製
      * -- example value: NEW2026
      * -- data field: `coupon_code`
      * -- 互動: 點擊觸發複製 + Toast
            1. **{{ 卡片有效期限 }}**
      * -- 定義: 有效期限日期（動態帶入）
      * -- 格式: `YYYY/MM/DD GMT+8`
      * -- example value: 2026/02/28 GMT+8
      * -- data field: `end_date`
            1. **{{ 使用說明按鈕 }}**
      * -- 定義: 點擊進入優惠券詳情頁（3.3.2）
      * -- 靜態文字: 使用說明
      * -- Lokalise key:

C. Data & Business Logic (資料邏輯)

* **Data Source:** 待確認。
* **Business Rules (商業規則):**
  * **排序邏輯:** 統一按照 **加入日期 (Date Added)** 由新到舊排列（Newest first）。
  * **有效日期顯示:** 拼接為 `Start_date ~ End_date (GMT +8)` 格式。
  * **資料保留政策:** 系統不會保留任何已經過期、已使用的 coupon；一旦狀態轉為過期或已使用，該筆資料將從清單中移除。因此清單中僅會出現「可使用」狀態的優惠券。

D. Analytics (數據埋點)

* Event Name: `screen_view`
* **Parameters:**
  * `firebase_screen` : `My_coupon_list`
  * `visitor_id` [String]
  * `crm_id` [Number]
  * `aff_id` [Number]
  * `utm_source` [String]
  * `utm_medium` [String]
  * `utm_campaign` [String]
  * `DPL` [String]
  * `url` [String]

* **Acceptance Criteria (驗收標準)**
* **Happy Path:**
  * [ ] App正確解析傳回之 JSON 列出相符卡片（並反應出後台輸入的標題與說明）。
  * [ ] 清單正確按照「加入日期」由新到舊排序。
  * [ ] 每頁顯示 5 筆資料，下拉後能觸發 Loading 並成功加載後續資料。
  * [ ] 優惠券效期後方正確顯示 `(GMT +8)` 時區文字。
* **Edge Cases:**
  * [ ] 已使用或已過期的優惠券，應確認會在資料更新後立即從清單中消失（不予保留）。
  * [ ] API 回傳空陣列 vs 載入失敗應分別導向 Empty State 與 Error State，不可混淆。
  * [ ] 優惠券數量較多時（如 50+ 張），清單應可正常滿版捲動，不影響效能。

#### 3.3.2 優惠券詳情與使用說明

A. User Story (使用者情境)

| Actor (角色) | Action (行為) | Outcome/Value (預期結果) |
| :--- | :--- | :--- |
| 會員 | 點擊查看使用說明 | 身為一個會員，我希望能在優惠券頁面中找到明確的使用說明，以了解優惠券的限制與規範。 |

B. UI & Interaction (介面互動)

* **Visual Reference:** [Lovable Prototype](https://preview--link-claim.lovable.app/?__lovable_token=eyJhbGciOiJ...)
* **UI States (關鍵狀態定義):**
  * **Trigger Timing:** 由「我的優惠券」清單點擊「使用說明」進入。
  * **Default（載入成功）：**
    * 依序顯示以下區塊：
            1. 優惠券標題 ({{ 優惠券標題 }})
            2. 顯示描述 ({{ 優惠券顯示描述 }})
            3. 優惠碼 ({{ 優惠碼實際代碼 }})
            4. 複製按鈕 ({{ 按鈕文案 }})
            5. 有效期限 ({{ 有限期限日期 }})
            6. 優惠詳情 ({{ 優惠詳情內容 }})
    * 各文案定義見下方 **Copywriting**
  * **Interaction：**
    * 點擊「複製優惠碼」按鈕 → 拷貝至剪貼簿 + Toast ({{ 複製成功 Toast }})
  * **Empty/No Data:** `Coupon Details` 為空 → fallback 至 Coupon Plan descriptions（同 3.1 **Business Rules**）
  * **Error:** API 載入失敗時，顯示通用載入錯誤提示並提供重試。
* **Logic:**
  * 提供**複製功能**: 點擊詳情內「複製優惠碼」按鈕 ({{ 按鈕文案 }})，拷貝字串至剪貼簿並閃出 Toast 提示成功 ({{ 複製成功 Toast }})。
  * 畫面上各區塊標題與對應的動態/靜態文案定義如下：
        ***Copywriting***
    * **[第一部分：優惠券資訊]**
    * **{{ 優惠券標題 }}**
    * -- 定義: 動態帶入顯示名稱
    * -- example value: 新會員首購優惠
    * -- Lokalise key: 無
    * **{{ 優惠券顯示描述 }}**
    * -- 定義: 動態帶入顯示描述 (coupon display description)
    * -- example value: 僅限新會員首購使用，折抵上限 100 元 (若欄位為空則依 3.1 規則抓取 plan descriptions)
    * -- Lokalise key: 無
    * **{{ 區塊標題 1 }}**
    * -- 定義: 優惠碼區塊標題
    * -- example value: 優惠碼
    * -- Lokalise key:
    * **{{ 優惠碼實際代碼 }}**
    * -- 定義: 動態帶入優惠碼
    * -- example value: NEW2026
    * -- Lokalise key: 無

    * **{{ 按鈕文案 }}**
    * -- 定義: 複製優惠碼按鈕
    * -- example value: 複製優惠碼
    * -- Lokalise key:
    * **{{ 複製成功 Toast }}**
    * -- 定義: 複製優惠碼成功後的 Toast 提示
    * -- example value: 已複製優惠碼
    * -- Lokalise key:

    * **[第二部分：有效期限]**
    * **{{ 區塊標題 2 }}**
    * -- 定義: 有效期限區塊標題
    * -- example value: 有效期限
    * -- Lokalise key:
    * **{{ 有限期限日期 }}**
    * -- 定義: 動態帶入日期區間
    * -- example value: 2026-01-31 ~ 2026-02-30
    * -- Lokalise key: 無

    * **[第三部分：優惠詳情]**
    * **{{ 區塊標題 3 }}**
    * -- 定義: 優惠詳情區塊標題
    * -- example value: 優惠詳情
    * -- Lokalise key:
    * **{{ 優惠詳情內容 }}**
    * -- 定義: 動態帶入詳細規範
    * -- example value: 1. 滿低消 500 元可折抵...
    * -- Lokalise key: 無

C. Data & Business Logic (資料邏輯)

* **Data Source:** 單張屬性 API。
* **Business Rules (商業規則):**
  * **文案資料來源:**
    * **動態文案：** 取自 `Coupon API`（包含：優惠券標題、優惠券顯示描述 (coupon_display_description)、優惠碼實際代碼、有限期限日期、優惠詳情內容）。
      * **備註:** `coupon_display_description` 邏輯同 3.1，優先使用後台輸入，次之為所有 `coupon_plan_description` 之排列文字。
    * **靜態文案：** 其餘區塊標題與按鈕文字皆使用 `Lokalise key` 管理對應多語系字串。

D. Analytics (數據埋點)

* Event Name: `screen_view`
* **Parameters:**
  * `firebase_screen` : `My_coupon_details`
  * `visitor_id` [String]
  * `crm_id` [Number]
  * `aff_id` [Number]
  * `utm_source` [String]
  * `utm_medium` [String]
  * `utm_campaign` [String]
  * `DPL` [String]
  * `url` [String]

E. Acceptance Criteria (驗收標準)

* **Happy Path:**
  * [ ] 「使用說明」點選能切入對應資料且右上可正確返回優惠券列表。
* **Edge Cases:**

  * [ ] `Coupon Details` 為空時，詳情頁應正確 fallback 顯示 Coupon Plan descriptions，不得顯示空白。
  * [ ] 導覽至已被後台刪除或不存在的優惠券時，應顯示適當的錯誤提示而非空白頁。
  * [ ] 返回按鈕應正確導回「我的優惠券」清單，而非 App 首頁或其他頁面。

---

### 3.4 會員頁入口

#### **A. User Story (使用者情境)**

| Actor (角色) | Action (行為) | Outcome/Value (預期結果) |
| :--- | :--- | :--- |
| 會員 | 尋找「我的優惠券」頁面入口 | 身為一個會員，我希望在會員中心的功能選項中可以輕鬆找到「我的優惠券」頁面的入口。 |

#### **B. UI & Interaction (介面互動)**

* **Visual Reference:** [Lovable Prototype](https://preview--link-claim.lovable.app/?__lovable_token=eyJhbGciOiJ...)
* **UI States (關鍵狀態定義):**
  * **Trigger Timing:** 使用者進入會員中心頁面。
  * **Default:**
    * 已登入 → 會員選單中顯示「我的優惠券」入口 ({{ 會員選單入口文案 }})
    * 未登入 → 不顯示此入口
  * **Interaction:** 點擊入口 → 跳轉至「我的優惠券」清單頁 (`/coupons/`)。
  * **Empty/No Data:** N/A（純入口導航）。
  * **Error:** N/A（純入口導航）。
* **Logic:**
  * 畫面上對應的靜態文案如下：
    * **{{ 會員選單入口文案 }}**
    * -- 定義: 登入後顯示的會員選單入口名稱
    * -- example value: 我的優惠券
    * -- Lokalise key:

#### **C. Data & Business Logic (資料邏輯)**

* **Data Source:** 無特定 API (純入口跳轉)。
* **Business Rules (商業規則):** -
* **Analytics (數據埋點):** -

#### **D. Acceptance Criteria (驗收標準)**

* **Happy Path:**
  * [ ] 未登入不會在會員中心看見優惠券入口。
  * [ ] 已登入會員點擊入口後，正確跳轉至「我的優惠券」清單頁。
* **Edge Cases:**
  * [ ] 登入 session 過期後，入口應隨之隱藏或點擊時導向重新登入，不得顯示空白清單。

---

## 4. SEO Functional Details

*(App 內開發為主，如 Web 則不適用於本次範圍)*

* [ ] **1. Workshop:** 確認 SEO 需求與關鍵字策略 (N/A)
* [ ] **2. Spec Planning:** 規劃 URL 結構、Meta Data 邏輯 (N/A)
* [ ] **3. 正式推版前 (Pre-release):** 檢查 staging 環境的 SEO 表現與 Tag 正確性 (N/A)
* [ ] **4. 上線後** (N/A)
