# 富邦大樓環繞飛行 排位賽

第一人稱飛行小遊戲：從富邦大樓頂樓起飛，沿黃色虛線繞一圈降落 1F 大廳，途中穿越 5 個金環解鎖 Alphabet 4 的債券條件。
任何人開網址就能玩，輸入名字就上共同排行榜，**不需要註冊任何帳號**。

**排名規則**：成績＝飛行時間＋每漏掉一個金環加 5 秒，秒數越少排越前面。同一個名字只留最好的一次。

---

## 檔案有哪些

| 檔案 | 做什麼的 | 你會不會需要改 |
|---|---|---|
| `app.py` | Streamlit 外殼，負責把遊戲顯示出來 | 幾乎不用 |
| `game.html` | 遊戲本體（大樓、航線、債券條件、排行榜） | 換債券時改這支 |
| `schema.sql` | Supabase 建表用的指令 | 只跑一次 |
| `requirements.txt` | 要安裝的套件 | 不用 |

---

## 部署步驟（約 10 分鐘）

跟「債速配」用**同一個 Supabase 專案**就好，不用再開新的。

### 第一段：在原本的 Supabase 專案多建一張表

1. 到 [supabase.com](https://supabase.com) 打開「債速配」用的那個專案
2. 左邊選單點 **SQL Editor** → **New query**
3. 把 `schema.sql` **整份貼進去** → 按右下角 **Run**
4. 看到綠色的 `Success` 就成功了（它會新建一張 `flight_scores`，不會動到原本的 `flip_scores`）

### 第二段：把程式放上 GitHub

1. 在 GitHub 開一個新的 repository，例如 `flight-game`
2. **Add file → Upload files**，把 `app.py`、`game.html`、`requirements.txt`、`schema.sql`、`README.md` 拖進去 → **Commit changes**

### 第三段：部署到 Streamlit Cloud 並接上資料庫

1. 到 [share.streamlit.io](https://share.streamlit.io) → **Create app**
2. 選剛剛那個 repository，Main file path 填 `app.py` → **Deploy**
3. 部署完成後，右下角 **⋮ → Settings → Secrets**
4. 貼上這兩行 —— **跟債速配那個 App 的 Secrets 一模一樣**，直接複製過來：

```toml
SUPABASE_URL = "https://你的專案代號.supabase.co"
SUPABASE_ANON_KEY = "eyJhbGciOi...（很長一串）"
```

5. 按 **Save**，App 會自己重開
6. 填名字飛一輪，排行榜出現你的名字就成功了

網址直接發給同事，他們點開就能玩。

---

## 平常怎麼維護

### 換成另一檔債券

打開 `game.html`，搜尋這兩個地方：

**① 排行榜代號** — 換一檔債就換一個 id，排行榜才不會混在一起：

```javascript
var BOARD_ID="alphabet4-flight";   // 改成例如 "msft-bond-03-flight"
```

**② 五項債券條件** — 搜尋 `const INFO=[`：

```javascript
const INFO=[
  {k:'債券名稱',v:'Alphabet 4'},
  {k:'債券代碼',v:'WMBB26040004'},
  {k:'Offer Price',v:'94.88'},
  {k:'YTM',v:'5.51%'},
  {k:'剩餘年期',v:'9.38 年'}
];
```

`k` 是欄位名稱、`v` 是數值，照順序對應第 1～5 個金環。
降落後結果卡上的報價日期也要一起更新：搜尋 `20260929 本行報價`，改成新的報價日。
開始畫面的小標 `WMBB26040004 · 飛行任務` 和說明文字裡的 `Alphabet 4` 也記得一起改（搜尋一下就找得到）。

### 清掉排行榜

Supabase → **Table Editor** → `flight_scores` → 勾選要刪的列 → 刪除。
或在 SQL Editor 跑：

```sql
delete from public.flight_scores where board = 'alphabet4-flight';
```

---

## 常見狀況

**排行榜一直空的 / 顯示「尚未設定連線」**
Secrets 沒設定好。回 Streamlit 的 Settings → Secrets 檢查那兩行，值要用雙引號包起來。

**顯示「找不到資料表 flight_scores」**
第一段的 `schema.sql` 還沒跑，或跑到別的專案去了。

**電腦上按方向鍵飛機不動**
先用滑鼠點一下遊戲畫面（或按「起飛」），鍵盤才會對到遊戲。

**同事說打不開**
Streamlit Cloud 免費版太久沒人用會睡著，第一個人打開要等 30 秒左右喚醒。

**有人用同一個名字**
排行榜用名字認人，同名只留最好的成績。請大家名字加上分行或單位。

---

## 安全性說明

- 程式裡放的 `anon` 金鑰本來就是設計給瀏覽器用的，公開沒關係
- 資料庫只開放「讀取」和「新增一筆」，**沒有開放修改和刪除**，沒人能洗掉別人的成績
- 資料表有檢查條件：名字最多 12 字、時間要在合理範圍，而且「成績」一定要等於「飛行時間＋漏環罰秒」，亂送的數字會被擋下
- 這裡面沒有任何客戶資料，只有名字和成績
- 遊戲頁面附註「僅供內部教育訓練使用，非投資建議」
