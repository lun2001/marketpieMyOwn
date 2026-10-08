# 主辦相關 View 和所需後端 API 的對照表

盤點日期：2026-10-08。

## 1. 盤點範圍與判讀方式

- 以目前工作目錄的實際內容為準，盤點 `src/views/organizer/` 的 11 個 View，以及公開主辦入口 `src/views/OrganizerView.vue`、主辦側欄與路由。這是依功能範圍整理，不以 Git 作者資訊認定每個檔案的作者。
- 同時讀取同專案後端 `../../API/`、`../../Controllers/`、`../../Services/`、`../../DTO/`、`../../Models/`，區分「後端已有實作」與「只有前端預留」。
- 「畫面已有」代表有版面或操作程式，不代表已存入資料庫；「API 已有」代表找到實作，不代表已通過實際連線測試。
- 本次僅靜態閱讀程式，未啟動服務、呼叫 API、操作資料庫或執行建置。
- 下文標示「建議」的 API 路徑與 DTO 是待製作規格，不是目前已存在的端點。
- 依先前分工，平台管理員後台由全員共同處理，本文僅列出主辦端送審、讀取結果與交接需求。付款、公告與通知交接祺元；攤商端報名與文件交接惠勝。
- 曝光功能排除；刊登方案／額度與方案購買依先前取消方案購買的整理暫不排入核心製作順序。程式目前仍保留刊登管理，與曝光管理是不同頁面，詳見第 7 節。

## 2. 已有後端及可重用功能

| 功能 | 已有 API／程式 | 主辦頁面使用狀況 | 接下來要做什麼 |
| --- | --- | --- | --- |
| 登入者與角色 | `GET /api/ApiAccount/GetCurrentUser`；`ApiAccountController` | `auth` store 與路由已呼叫；主辦路由檢查 `roleId === 5` | 新增主辦 API 延用 Session 身分；後端還要檢查資料是否屬於該主辦 |
| 登入／登出 | `POST /api/ApiAccount/UserLogIn`、`POST /api/ApiAccount/LogOut` | 共用 `accountService`、`auth` store 已接入 | 沿用，不必在主辦模組重寫登入系統 |
| 主辦註冊 | `POST /api/ApiAccount/OrganizerRegister` | 後端已有；本次盤點的主辦 View 沒有呼叫此端點 | 交由帳號模組整合；不能代替主辦資料編輯 API |
| 招募分類 | `GET /api/CategoryApi` | Create、Edit 已透過 `getCategories()` 載入 | 沿用；新增活動時傳送選取的 `categoryIds`，由後端再次驗證 |
| 報名所需文件與狀態 | `GET /api/organizer/applications/{applicationId}/documents` | ApplicationDetail 還是讀假資料的 `application.documents` | 新增前端 service/store 呼叫，處理 loading、失敗與缺件狀態 |
| 主辦查閱報名文件 | `GET /api/organizer/applications/{applicationId}/documents/{documentTypeId}/file` | 頁面已有文件連結處理，但正式文件清單尚未載入 | 使用文件 DTO 回傳的 `fileUrl`；後端已有主辦與報名歸屬檢查 |
| 主辦 API 身分檢查 | `OrganizerAuthorizeAttribute` | 文件 API 已使用 | 新 API 可沿用；此 Filter 檢查登入與角色，個別資料歸屬仍須由 Service 驗證 |
| 縣市／行政區 | 前端呼叫 `https://api.nlsc.gov.tw/other/ListCounty`、`.../ListTown/{code}` | Create、Edit 已有外部 XML 資料解析 | 這不是本專案後端 API；補載入錯誤、空結果與重試即可，是否改走後端代理另議 |
| 圖片上傳基礎 | `POST /api/uploads/images`、`ImageUploadService` | 主辦 Create／Edit 目前只有選檔與預覽 | **現有 Controller 使用 `VendorAuthorize`，且依攤商 ID 儲存，不能直接當主辦上傳端點**；需新增主辦端點或調整為明確支援主辦的共用設計 |
| 平台活動審核 | `AdminMarketController` 已有 MVC 活動查看、編輯及審核 Action | 不屬於主辦 JSON API，也未替主辦頁面提供列表／編輯端點 | 共同後台工作沿用現況協作；主辦端只送審與讀取審核結果，不自行決定審核通過 |

主要依據：[路由](src/router/index.js)、[auth store](src/stores/auth.js)、[分類 service](src/services/categoryService.js)、[文件 Controller](../../API/OrganizerApplicationDocumentsController.cs)、[圖片 Controller](../../API/UploadsController.cs)、[主辦權限 Filter](../../API/Filters/OrganizerAuthorizeAttribute.cs)。

## 3. View 與待完成 API 對照

以下主辦業務 API，除第 2 節明列已有者外，本次未在目前後端找到對應實作。

| View／路由 | 已有畫面與行為 | 待完成內容 | 對應 API（建議；已存在者另註） |
| --- | --- | --- | --- |
| [OrganizerProfileView.vue](src/views/organizer/OrganizerProfileView.vue)；`/organizer/profile` | 只有「主辦資料」標題 | 資料讀取、表單、驗證、Logo 上傳、儲存與未儲存提示；側欄同步名稱與圖片 | `GET /api/organizer/profile`；`PUT /api/organizer/profile`；`POST /api/organizer/uploads/images` |
| [OrganizerMarketsView.vue](src/views/organizer/OrganizerMarketsView.vue)；`/organizer/markets` | 草稿與一般活動分區、審核／上架篩選、分頁、刪除草稿確認；資料為本地陣列 | 從後端取得自己的活動與頁數；刪除目前只移除前端陣列，需真的保存結果 | `GET /api/organizer/markets`；`DELETE /api/organizer/markets/{marketId}`，限草稿 |
| [OrganizerMarketCreateView.vue](src/views/organizer/OrganizerMarketCreateView.vue)；`/organizer/market/create` | 基本／活動／招募／攤位設定、日期時段、分類、縣市行政區、圖片預覽、編號批次建立、驗證、離頁確認；分類已接 API | 上傳圖片、保存草稿、取得新活動 ID、保存關聯資料、送審；現有儲存／送審仍是 `console.log` | `GET /api/CategoryApi`（已有）；`POST /api/organizer/uploads/images`；`POST /api/organizer/markets`；`POST /api/organizer/markets/{marketId}/submit-review` |
| [OrganizerMarketEditView.vue](src/views/organizer/OrganizerMarketEditView.vue)；`/organizer/market/:id/edit` | 假資料帶入表單、分類 API、狀態控制、驗證與離頁提示、圖片及攤位設定；草稿送審／待審核修改／退回重送／通過後僅改攤位的分支 | 真實載入及儲存；後端按狀態限制可改欄位；成功後才重設未儲存狀態或跳頁 | `GET /api/organizer/markets/{marketId}`；`PUT /api/organizer/markets/{marketId}`；`POST .../{marketId}/submit-review`；`PUT .../{marketId}/booths`；圖片上傳與分類 API |
| [OrganizerMarketDetailView.vue](src/views/organizer/OrganizerMarketDetailView.vue)；`/organizer/market/:id` | 活動資訊、場次、招募條件、審核結果、攤位資料、圖片檢視、上下架、修改申請視窗 | 取得真實活動／攤位；上下架目前只改本地 `status`；修改申請只印訊息並顯示成功 | `GET /api/organizer/markets/{marketId}`；`GET .../{marketId}/booths`；`PATCH .../{marketId}/publication`；`POST .../{marketId}/change-requests` |
| [OrganizerApplicationsView.vue](src/views/organizer/OrganizerApplicationsView.vue)；`/organizer/applications` | 依市集展開報名、搜尋與招募狀態篩選、待審核／待付款／已付款／逾期／未通過分組、未讀筆數、連至攤位安排 | 市集及報名真實資料、筆數、已讀保存；目前使用假市集與 notification store 的示範報名資料 | `GET /api/organizer/markets`（帶報名摘要）；`GET /api/organizer/markets/{marketId}/applications`；`PATCH /api/organizer/applications/{applicationId}/read` |
| [OrganizerApplicationDetailView.vue](src/views/organizer/OrganizerApplicationDetailView.vue)；`/organizer/applications/:id` | 報名內容、品牌照片與商品、文件區、付款／攤位結果、審核意見與確認；退件意見必填 | 正式詳情、已讀、文件清單及檔案、審核保存；現在審核只是修改 store 物件 | `GET /api/organizer/applications/{applicationId}`；`PATCH .../{applicationId}/read`；`POST .../{applicationId}/review`；`GET .../{applicationId}/documents` 與文件讀取（已有） |
| [OrganizerBoothAssignmentView.vue](src/views/organizer/OrganizerBoothAssignmentView.vue)；`/organizer/markets/:marketId/booths` | 已通過且已付款的示範攤商、拖放／點選安排、交換／移除、搜尋、統計、未儲存提示、暫存與完成確認 | 載入合格名單及攤位；儲存安排；完成狀態持久化；通知交接；目前儲存只印 payload 並顯示提示 | `GET /api/organizer/markets/{marketId}/booths`；`GET .../{marketId}/applications?eligibleForBooth=true`；`PUT .../{marketId}/booths/assignments`；`POST .../{marketId}/booths/complete` |
| [OrganizerDashboardView.vue](src/views/organizer/OrganizerDashboardView.vue)；`/organizer/dashboard` | 活動數、待平台審核／退回數、待審報名、待攤位安排、即將開始活動及入口；目前是假資料，已不只是占位標題 | 統計與摘要 API、安排期限／完成狀態來源、日期邏輯及資料更新；刊登額度區另列非核心 | `GET /api/organizer/dashboard`，建議由後端彙整 |

### 共用元件與入口

- [OrganizerSidebar.vue](src/components/OrganizerSidebar.vue)：主辦名稱、Logo 與 `hasNew` 目前是假資料；名稱／Logo 可共用 `GET /api/organizer/profile`，未讀數可由 dashboard 或報名摘要取得。點擊選單清掉圖示不等於後端報名全部已讀。
- [OrganizerLayout.vue](src/layouts/OrganizerLayout.vue)：共用版型，不需要獨立業務 API。
- [OrganizerView.vue](src/views/OrganizerView.vue)：公開主辦入口仍有方案卡片，刊登按鈕開啟舊 `listing` modal；之後應在範圍確認後銜接正式建立活動頁，不要讓舊示範表單和正式活動各存一份。本次未修改。

## 4. 待完成 API 的資料與規則

下列是製作時的建議契約。`marketId`、`applicationId` 皆為後端資料 ID；登入者與 `organizerId` 從 Session 查出，不接受前端指定他人的主辦身分。

| API（皆為建議新增） | 輸入／輸出重點 | 必要規則 |
| --- | --- | --- |
| `GET /api/organizer/profile` | 主辦 ID、名稱、聯絡人、電話、Email、介紹、Logo、網站、統編等頁面需要的欄位 | 可參考現有 `Organizer` Model；不要直接把整個資料庫 Entity 當 DTO 回傳 |
| `PUT /api/organizer/profile` | 可編輯的主辦資料；回傳儲存後 DTO | 登入帳號 Email 與主辦聯絡 Email 是否同一欄位先釐清；不可讓此端點修改角色 |
| `POST /api/organizer/uploads/images` | multipart `file`，用途如 poster／layout／logo；回傳 `{ url }` | 主辦權限、檔案類型／大小檢查、儲存歸屬；可評估重用圖片服務，不能直接重用攤商 Controller |
| `GET /api/organizer/markets` | `reviewStatus`、`status`、`keyword`、`page`、`pageSize`；回傳 `items`、`totalCount`，需要時帶報名摘要 | 僅限自己的活動；草稿與一般活動的分頁計算需一致 |
| `GET /api/organizer/markets/{marketId}` | 活動基本資料、`categoryIds/categories`、`schedules`、招募資訊、圖片、審核意見與狀態 | 查無或不屬於本人時不洩漏資料；攤位可由獨立端點取得 |
| `POST /api/organizer/markets` | 活動表單、分類、場次及初始攤位設定；回傳 `marketId` 與完整 DTO | 建議先建立草稿再送審；編號、擁有人、建立時間由後端產生；關聯資料須一致保存 |
| `PUT /api/organizer/markets/{marketId}` | 可編輯活動欄位、分類與場次 | 草稿／待審核／退回修改可改哪些欄位由後端檢查；不得直接修改平台審核結果 |
| `DELETE /api/organizer/markets/{marketId}` | 刪除指定草稿 | 只允許草稿；後端成功後前端才移除；有關聯資料時應完整處理 |
| `POST /api/organizer/markets/{marketId}/submit-review` | 活動 ID；回傳更新後審核狀態 | 草稿／退回修改才能送審；完整驗證必填、場次、日期、名額與分類；避免重複送審 |
| `PATCH /api/organizer/markets/{marketId}/publication` | 例如 `{ status: 1 }` 上架、`{ status: 2 }` 下架 | 只有符合平台審核及活動規則者可上架；不可自行解除強制下架 |
| `POST /api/organizer/markets/{marketId}/change-requests` | `{ requestContent }`；回傳申請 ID／狀態 | 保存申請並交給共同平台後台處理，不能只跳成功視窗 |
| `GET /api/organizer/markets/{marketId}/applications` | 報名 ID／編號、攤商摘要、審核／付款狀態、已讀、攤位；支援狀態或安排資格條件 | 筆數與列表來源一致；資格條件由後端計算，不只由前端篩選 |
| `GET /api/organizer/applications/{applicationId}` | 報名內容、攤商品牌／照片／商品、分類、審核、付款摘要與攤位結果 | 文件沿用現有專用 API；付款摘要來自付款模組，不由此 View 自行造狀態 |
| `PATCH /api/organizer/applications/{applicationId}/read` | `{ isRead: true }` 或固定已讀動作 | 與 `VendorApplication.IsRead` 對應；可重複呼叫；不是通知已讀的同義操作 |
| `POST /api/organizer/applications/{applicationId}/review` | `{ reviewStatus: 1 或 2, reviewNote }`；回傳新狀態及審核時間 | 驗證可審核狀態、退件原因、名額；通過後與祺元約定建立付款資料及期限的時點 |
| `GET /api/organizer/markets/{marketId}/booths` | 配置圖、攤位 ID／編號／已分配報名 ID、安排期限及完成狀態 | 供詳情、編輯、安排頁共用；配置圖只是圖片，不代表已有持久化攤位清單 |
| `PUT /api/organizer/markets/{marketId}/booths` | `{ boothLayoutUrl, booths: [{ marketBoothId, boothNumber }] }` | 編號同場唯一；限制刪除已分配攤位；設定端點不應順便任意改報名分配 |
| `PUT /api/organizer/markets/{marketId}/booths/assignments` | `{ assignments: [{ marketBoothId, vendorApplicationId }] }`；null 表示清空 | 此路徑已有前端註解構想，但沒有呼叫或後端實作；整批更新要用交易、檢查同場、資格、重複分配及同時編輯衝突 |
| `POST /api/organizer/markets/{marketId}/booths/complete` | 已保存的安排版本，或連同最終安排一起提交；回傳完成狀態 | 再檢查所有應安排者皆已分配；完成與保存須一致，成功後再交通知模組；重送不可重複發通知 |
| `GET /api/organizer/dashboard` | 活動／審核／報名統計、待安排活動、近期活動與未讀摘要 | 依登入主辦計算；沿用業務狀態，避免 dashboard 自己定另一套規則 |

建議統一使用 400（資料驗證）、401（未登入）、403（角色不符）、404（查無／無權讀取該筆）、409（狀態或分配衝突），並沿用目前後端 `ProblemDetails`／欄位驗證回應模式。

## 5. 寫 API 前先處理的資料落差

| 落差 | 本次看到的證據 | 建議處理 |
| --- | --- | --- |
| 攤位實體尚未在 EF Model 出現 | View 使用 `marketBoothId`、`boothNumber`、`vendorApplicationId`；目前 `Models` 與 `MarketpiedbContext` 未找到 MarketBooth 實體／映射 | 先確認實際 DB 是否已有尚未同步的表；若無，再設計攤位表、關聯與唯一限制。不能直接判定資料庫也沒有 |
| 攤位號碼有兩種保存構想 | `VendorApplication` 已有 `BoothNumber`；新安排頁使用獨立攤位及報名 ID 關聯 | 決定正式來源；若保留舊欄位，相容同步必須在同一交易內，避免報名頁與安排頁不同步 |
| 安排期限／完成狀態沒有對應 Model 欄位 | Dashboard 假資料使用 `boothAssignmentDeadline`、`boothAssignmentCompleted`，目前 `Market` Model 無對應欄位 | 定義期限是計算值或持久化設定，完成時間／狀態如何保存，再做 dashboard |
| 草稿允許不完整資料，但活動 Model 有多個必填欄位 | Create 的草稿驗證較寬鬆；`Market` 多個欄位非 nullable | 先選擇草稿儲存策略與 DB 約束，不能用完整送審 DTO 直接擋掉合法草稿，也不要填虛構資料過關 |
| 自訂報名欄位是先前分工需求，但目前 Create 沒找到設定程式 | 目前表單有固定招募欄位，未找到自訂題目設定；攤商 `ApplyFormDTO` 也未提供自訂題目 | 若仍列入範圍，需新增題目／選項／答案設計與前後端 UI，和惠勝一起安排；不是「已有畫面只缺 API」 |
| 活動審核與報名審核不是同一組狀態 | 活動頁：0 草稿、1 待審核、2 退回修改、3 通過、4 不通過；報名頁：0 待審核、1 通過、2 不通過 | 分開定義 Enum／DTO。攤商 API 又輸出 `pending/approved/rejected` 字串，要有明確轉換，不要跨頁直接共用數字 |
| 付款狀態表示不同 | 主辦假資料含 `overdue/refunded`；後端 `VendorApplicationService` 由付款資料轉換攤商 DTO | 與祺元、惠勝統一逾期、已付款及免費活動的安排資格；退款文字存在不代表必須新增退款業務 |
| 修改申請保存格式尚未確定 | Detail 只有 `requestContent` 與 console 輸出，未找到對應主辦 API | 和共同後台確認使用哪個資料模型、如何查看與處理申請，再串接 |

上述對照僅依目前程式模型判斷，沒有查詢實際資料庫。

## 6. 建議製作順序與驗收點

以先打通一條實際可保存的流程，再完成審核及攤位安排為原則。每階段都包含 API 與 View 串接，不建議把所有 API 做完才開始接頁面。

| 順序 | 製作內容 | 先決條件／原因 | 完成驗收點 |
| --- | --- | --- | --- |
| 0 | 確認範圍、狀態契約、草稿策略、攤位資料模型、付款資格與 API DTO；確認刊登方案是否維持排除 | 避免依假資料格式寫完後重做 DB／前端 | 分清活動審核／報名審核；草稿能保存；攤位與報名關聯定案 |
| 1 | 主辦資料 GET／PUT、圖片上傳、前端主辦 service/store；沿用登入、分類與 `OrganizerAuthorize` | 建立 Session 歸屬與檔案上傳基礎 | 主辦資料修改後重整仍在，側欄同步；主辦能上傳圖片；其他角色不能操作 |
| 2 | 活動列表／詳情 GET、建立／更新草稿、刪除草稿；接 Markets、Create、Edit、Detail | 先讓四個活動頁共用同一份真實資料；攤位編號可在第 6 階段補完整 | 建立取得真實 ID，列表可見，編輯保存後重整可讀；只能刪自己的草稿 |
| 3 | 完整招募／場次／分類驗證、活動送審／重新送審、上下架、修改申請 | 依賴活動 CRUD；平台審核端由共同後台配合 | 草稿 → 待審核 → 平台審核結果 → 上架；退回後可修改重送；主辦不能自行改成通過 |
| 4 | 報名列表／詳情／已讀、接上現成文件 API | 活動資料已真實化，搭配惠勝的攤商報名資料 | 主辦只看到報到自己活動的攤商；詳情與列表一致；文件可讀且跨主辦不可存取 |
| 5 | 報名審核 API、名額與狀態限制；付款及通知交接 | 先能查看真實報名與文件，才進行審核 | 通過／退件保存；退件原因可見；重複審核與名額衝突有處理；付款端收到正確交接 |
| 6 | 攤位清單／配置圖保存、合格名單、批次安排暫存、完成 API；接 Edit／Detail／BoothAssignment | 攤位模型已定案，報名與付款資格有來源 | 拖放、交換、清空後可保存；重整保留；同一攤位／報名不重複；完成後攤商顯示與通知一致 |
| 7 | Dashboard 統計、待安排期限／完成狀態、近期活動、側欄未讀摘要 | 最後依真實活動、報名、付款及攤位計算，避免先做假的統計 API | 審核與攤位異動後統計正確；跨帳號只顯示自己的數字 |
| 8 | 串接收尾與流程驗證、公開主辦入口調整、取消範圍的殘留入口處理 | 核心功能完成後整理入口與體驗；本次只列計畫 | 未登入、錯誤角色、跨主辦、查無資料、網路失敗與同時更新都有合理結果；API 失敗不跳成功或清掉未儲存狀態 |

建議先完成的最小流程：**建立草稿 → 列表 → 編輯 → 送審 → 讀取審核結果／上架 → 查看報名與文件 → 審核 → 付款交接 → 攤位安排**。Dashboard 最後彙整。

如果確定要做自訂報名欄位，應在第 3 階段完成題目設定及資料契約，與惠勝串接填答後再進第 4 階段，不要等報名審核完成才補答案結構。

## 7. 已有檔案但本輪不排入核心製作

| 項目 | 現況 | 本輪處理方式 |
| --- | --- | --- |
| [OrganizerPromotionsView.vue](src/views/organizer/OrganizerPromotionsView.vue) | 只有標題；`/organizer/promotions` 路由仍在，側欄入口已註解 | 曝光功能已取消，不安排曝光 API；僅記錄殘留，不刪檔或改路由 |
| [OrganizerPlansView.vue](src/views/organizer/OrganizerPlansView.vue) | 刊登額度、購買紀錄、方案卡片、確認視窗皆使用假資料；`confirmPurchase()` 只印訊息，未接 Checkout | 刊登方案與曝光不是同一功能。依先前方案購買取消規劃暫不排程；若保留刊登收費，另需方案列表、購買紀錄／剩餘額度、建立訂單、付款結果與額度扣抵契約，不能混入攤商報名費付款 |
| Dashboard 刊登額度、Sidebar 刊登管理、公開主辦入口方案卡片 | 程式仍保留相關區塊／入口 | 範圍確認後另外處理；本次未修改 |
| 平台管理員後台 | 已有 MVC 管理端程式 | 全員共同負責；本文件不將其列成雅綸個人待做 API |

## 8. 實作與協作提醒

- 建議按 [API-INTEGRATION.md](docs/API-INTEGRATION.md) 的分層，新增主辦領域的 service/store，再由 View 呼叫；文件中的示例 API 名稱不等於實際已實作 API，本表的「已有」以 Controller 為準。
- 主辦報名 API 不直接沿用 `/api/applications`：目前該組是攤商端、使用攤商權限與本人資料範圍；主辦查閱使用自己的 `/api/organizer/...` 與歸屬條件。
- 報名「已讀」與通知「已讀」分開保存；不要繼續把報名業務資料放在通知示範 store 當正式來源。
- 付款狀態、公告與通知由祺元模組提供；雅綸的審核／安排 API 負責明確交接成功事件，不在多個 View 各自發送通知。
- 攤商資料、文件及報名由惠勝模組提供；現成主辦文件 API 應優先串接，避免重寫另一份文件讀取邏輯。
- 所有寫入都應等 API 成功才提示成功、清除 dirty 狀態或跳頁；目前多處 console／本地修改必須在正式串接時替換。
- 主辦跨帳號資料隔離、同一活動名額／攤位衝突與完成操作重送，是本流程最值得優先驗證的項目。

本次只新增此文件，未修改既有 View、路由、API、資料模型或資料庫。
