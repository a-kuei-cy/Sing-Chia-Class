# Sing Chia Class V1.1 School
興嘉國小課堂即時互動系統｜免 Google OAuth 校內帳密版

## 這一版改了什麼
- 不需要 Google Cloud Console
- 不需要 OAuth Client ID
- 不需要教師使用 Google 帳號登入
- 管理者可匯入教師帳號與初始密碼
- 密碼在 Google Sheets 中只保存 SHA-256 雜湊與隨機 Salt，不保存明碼
- 登入成功後使用 8 小時 Session Token
- 管理者可新增、更新、停用帳號及重設密碼
- 學生仍免登入，使用 6 位數 PIN 或 QR Code 加入

## 檔案
- `index.html`：前端主頁
- `styles.css`：介面樣式
- `app.js`：前端功能
- `config.js`：只需設定 GAS Web App 網址
- `Code.gs`：Google Apps Script 後端
- `teacher-import-template.csv`：教師帳號匯入範本

---

# 一、建立 Google 試算表
1. 建立一份新的 Google 試算表，例如「Sing Chia Class 資料庫」。
2. 從網址複製試算表 ID：
   `https://docs.google.com/spreadsheets/d/這一段就是ID/edit`
3. 試算表選「擴充功能 → Apps Script」。
4. 將 `Code.gs` 全部貼入 Apps Script。

> 建議 V1.1 使用新的空白試算表，不要直接沿用 V1.0 OAuth 版資料表，以免欄位名稱不同。

# 二、設定 Apps Script 指令碼屬性
Apps Script 左側「專案設定」→「指令碼屬性」，新增：

- `SPREADSHEET_ID`：你的試算表 ID
- `INITIAL_ADMIN_ACCOUNT`：例如 `admin`
- `INITIAL_ADMIN_PASSWORD`：第一次管理者登入密碼，至少 6 碼
- `INITIAL_ADMIN_NAME`：例如 `興嘉教務處`

例如：

```text
SPREADSHEET_ID = 你的試算表ID
INITIAL_ADMIN_ACCOUNT = admin
INITIAL_ADMIN_PASSWORD = 請自行設定安全密碼
INITIAL_ADMIN_NAME = 興嘉教務處
```

注意：第一次執行 `setupSingChiaClass()` 後，程式會自動刪除 `INITIAL_ADMIN_PASSWORD` 指令碼屬性。真正密碼只留下雜湊值。

# 三、建立資料表與第一個管理者
1. Apps Script 編輯器上方函式選 `setupSingChiaClass`。
2. 按「執行」。
3. 第一次會要求 Google 授權，這是 Apps Script 存取你自己的試算表所需授權，不是 Google Cloud OAuth Client ID。
4. 完成後試算表會自動建立：
   - Teachers
   - AuthSessions
   - Quizzes
   - Questions
   - Sessions
   - Participants
   - Answers

# 四、部署 Apps Script Web App
1. Apps Script →「部署」→「新增部署作業」。
2. 類型選「網頁應用程式」。
3. 執行身分：**我**。
4. 誰可以存取：**任何人**。
   - 原因：學生需要免 Google 登入加入活動。
   - 教師管理功能仍由本系統帳號、密碼及 Session Token 保護。
5. 按「部署」。
6. 複製最後 `/exec` 結尾的網址。

# 五、設定前端
開啟 `config.js`：

```javascript
window.SCC_CONFIG = {
  GAS_URL: "https://script.google.com/macros/s/AKfycbzVMLaFZ3FgcGvTT5eSDQ440W3H0EhQWmzkZ6uwBc_FlA_aXfnoevp4CNFhSqjMfrIe/exec",
  APP_NAME: "Sing Chia Class",
  VERSION: "V1.1 School"
};
```

本程式包已填入你指定的 `GAS_URL`，不必再次修改。只有改用另一個部署網址時才需更新。

若後端已部署：將本包 `Code.gs` 更新到原 Apps Script 專案，使用「部署 → 管理部署 → 編輯 → 新版本」更新原部署，可保留目前 `/exec` 網址。既有 V1.1 School 資料表可保留；不需重新建立管理者。

`Code.gs` 不需要設定 `GAS_URL`；它由 `SPREADSHEET_ID` 指令碼屬性連到資料庫。前端以 `window.SCC_CONFIG` 讀取設定，`index.html` 先載入 `config.js`，再載入 `app.js`。

# 六、部署 GitHub Pages
將下列檔案放到 GitHub repository：
- index.html
- styles.css
- app.js
- config.js

GitHub → Settings → Pages → Deploy from a branch → `main` / root。

完全不需要設定 Google Cloud Console。

# 七、第一次登入
開啟網站 → 教師端。

使用你在 Apps Script 指令碼屬性設定的：
- `INITIAL_ADMIN_ACCOUNT`
- `INITIAL_ADMIN_PASSWORD`

登入後會看到「帳號管理」。

# 八、匯入教師帳號
可以直接使用 `teacher-import-template.csv`。

CSV 第一列必須是：

```csv
account,name,password,role,enabled
```

範例：

```csv
account,name,password,role,enabled
teacher01,王老師,12345678,teacher,TRUE
teacher02,陳老師,abc12345,teacher,TRUE
admin02,課程組長,Admin2026,admin,TRUE
```

欄位說明：
- `account`：登入帳號，建議英文、數字、`.`、`_`、`-`
- `name`：顯示姓名
- `password`：初始密碼，至少 6 碼
- `role`：`teacher` 或 `admin`
- `enabled`：`TRUE` / `FALSE`

如果帳號已存在，匯入會更新姓名、角色及啟用狀態；`password` 有填寫時才會更新密碼。

# 九、教師操作
1. 教師端 → 帳號密碼登入。
2. 新增題庫。
3. 建立單選、多選或是非題。
4. 按「開始」產生 6 位數 PIN 與 QR Code。
5. 教師按「下一題」。
6. 學生作答，教師端即時看到加入人數、作答人數與選項分布。
7. 活動結束可匯出 CSV。

# 十、學生操作
1. 掃描 QR Code，或開啟網站「學生加入」。
2. 輸入 PIN、班級、座號、姓名。
3. 不需要 Google 帳號。
4. 等待教師出題後作答。

# 安全提醒
- 不要把管理者密碼寫進 `config.js`、`app.js` 或 GitHub。
- 請使用至少 8 碼且不要和個人重要帳號相同的管理者密碼。
- 教師密碼不以明碼保存於 Sheets。
- Apps Script Web App 必須允許「任何人」才能讓學生免登入，因此請勿自行新增會回傳教師密碼雜湊或 Salt 的 API。
- 若管理者離職或帳號需停止使用，請立即在帳號管理中停用。

# 常見問題
## 1. 登入顯示「尚未設定 GAS_URL」
代表 `config.js` 還沒有貼上 Apps Script `/exec` 網址。

## 2. 執行 setupSingChiaClass 顯示 INITIAL_ADMIN_PASSWORD 錯誤
請先到「專案設定 → 指令碼屬性」加入 `INITIAL_ADMIN_PASSWORD`，至少 6 碼。

## 3. 忘記管理者密碼
可暫時在 Apps Script 編輯器新增一個一次性函式，或由另一個 admin 在系統後台重設。不要直接修改 Teachers 工作表的 passwordHash。

## 4. 老師無法登入
管理者可檢查帳號是否為「啟用」，或直接重設密碼。

# 本次封裝與檢查
- 已設定指定的 GAS 網址，保留原本七個檔案及資料表欄位。
- 修正 QR／PIN 連結自動開啟學生加入頁面、零分題目編輯、登出停止教師輪詢、舊成績顯示與結束活動按鈕狀態。
- 修正歷史 PIN 重複、活動結束後重新出題、無效學生寫入答案；寫入操作加鎖，避免同時作答重複計分。
- QR 圖片沿用原版 qrcodejs CDN；需能連線 cdnjs.cloudflare.com，載入失敗時仍可用 PIN 加入。
- 題目時間欄位沿用原版，尚無自動倒數截止；教師按「下一題」控制進度。兩種活動模式沿用原版計分與排名行為。
- 檢查採本機語法、前後端對照與模擬資料測試；未登入你的線上後端，未修改 Google 試算表，實際部署後仍須以教師與學生裝置試跑一次。
