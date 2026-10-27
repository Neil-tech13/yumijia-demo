# 玉米家叫號網頁：正式版

規格見 `TECH-HANDOFF.md`。這個資料夾是**完整的正式版網站**，之後要整包搬到新的 repo `yumijia/yumijia.github.io` 的根目錄。網址會是 `https://yumijia.github.io/`。

> 目前暫放在 `yumijia-demo` 的 `claude/determined-ptolemy-480ph2` 分支。這個分支**不要合併到 main**，否則示範網址會被影響。

## 檔案

| 檔案 | 用途 |
|---|---|
| `index.html` | 客人頁。只能看，即時更新，記得自己的號碼（網址 `#n23`） |
| `staff.html` | 店家頁。輸入密碼登入，發號、叫號、復原、歸零、設定 |
| `common.js`、`style.css` | 兩頁共用的 Firebase 連線、計算和樣式 |
| `firestore.rules` | 和 Firebase 後台已發布的規則一樣，給本機測試用 |
| `tests/` | 本機模擬器測試，跟網站本身無關 |
| `../print/玉米家QR貼紙_A4.pdf` | A4 QR 貼紙，指向 `https://yumijia.github.io/`，有 25／30／40／50 mm 四種大小，一頁一種。不用放進網站 repo |

貼紙上的 QR 要指向 `https://yumijia.github.io/`。店家頁的網址是 `https://yumijia.github.io/staff.html`，只給老闆娘。

## 做到哪裡

- ✅ 客人頁、店家頁都已完成，照 `TECH-HANDOFF.md` 的規格
- ✅ 用 Firebase 本機模擬器測過，沒有碰到正式資料（見下面「測試」）
- ❌ 還沒用真的 Firebase 測過，要等網站上線，再用老闆娘的密碼登入，拿兩支手機實測
- ❌ 新 repo 還沒建立

## 上線步驟

1. 使用者到 https://github.com/account/organizations/new 建立免費組織 `yumijia`
2. 在組織底下建 **Public** repo，名稱要剛好是 `yumijia.github.io`，什麼都不要勾
3. 到 claude.ai 的 GitHub 設定，讓 Claude 可以存取這個新 repo
4. 把這個資料夾的內容放到新 repo 的根目錄，`.nojekyll` 也要放進去
5. repo 的 Settings → Pages，Source 選 `main` 分支的 `/ (root)`
6. 到 Firebase 後台 → Authentication → 設定 → 授權網域，加入 `yumijia.github.io`。密碼登入通常不需要這一步，加了比較保險
7. **關掉自行註冊**（安全性）：Firebase 後台 → Authentication → 設定 → 使用者動作，把「啟用建立（註冊）」取消勾選。安全規則只檢查 email，萬一店家帳號被刪掉，別人可以用同一個 email 註冊來改號碼。關掉註冊就不會有這個問題，店家帳號已經建好，不受影響
8. 用兩支手機測：一支開 `staff.html` 登入，按發號；另一支開客人頁，看號碼有沒有在 2 秒內更新

## 測試（本機模擬器）

```bash
cd tests
npm ci
./start-emulators.sh      # 啟動 Auth (9099) 和 Firestore (8080) 模擬器
node seed-staff.mjs       # 在模擬器建測試用的店家帳號（只存在模擬器，密碼是假的）
npm test                  # 安全規則測試
node e2e.cjs              # 兩支手機的端對端測試（Playwright）
./stop-emulators.sh
```

客人頁、店家頁的網址只要加上 `?emu`，**而且是在 localhost 開的**，就會連到本機模擬器，不會連到正式的 Firebase。

端對端測試會檢查這些：
- 客人頁沒有店家按鈕
- 密碼錯誤時登入會被擋下
- 重新整理後店家頁仍然保持登入
- 發號後，客人頁在 2 秒內更新
- 叫號（烤好了）、已取餐、復原上一步、兩次確認才會歸零
- 兩支手機同時按發號，拿到不同的號碼
- 改設定後，客人頁出現 LINE 按鈕
- 換了一天（早上 6 點換日）時，店家頁會提示從 1 號開始，客人頁不會顯示前一天的號碼
- 斷線時客人頁出現提示，連回來後提示消失
- 沒登入就直接寫資料，會被安全規則擋下
- 頁面上沒有「Claude」或「示範」字樣
- 320、375、1280px 寬都不會出現橫向捲軸
