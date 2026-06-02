# 🔮 LINE Crystal Astrology Expert Bot (Google ADK & Cloud Firestore 記憶型智慧占星專家)

這是一個強大的、具備**長效永久記憶力**的 LINE 智慧占星與水晶能量諮詢專家機器人。
本專案基於 **Node.js (CJS/ESM 混血架構)** 實作，導入了 Google 最新的 **ADK (Agent Development Kit)** 智慧代理框架，結合 **Google Cloud Firestore** 雲端資料庫，並完美託管於 **Google Cloud Run**。

機器人採用了如「國師」唐綺陽老師般，溫暖、理性、優雅且具備深度洞察力與療癒感的專業水晶占星師人設，能主動且永遠記住使用者之前透露的星座、生日和收集過的水晶，為使用者量身打造細緻且不重複的心靈共鳴諮詢。

---

## 🌟 核心特色功能

* 🧠 **國師級專業水晶占星人設**：說話語調優雅、沈穩且富含占星學、五行學與脈輪（Chakra）共振知識。告別喧鬧浮誇的「神婆」形象，以理性與同理心切入，為使用者提供最溫暖的心靈引導。
* 🔄 **Google ADK + PreloadMemoryTool**：導入 Google 最新 Agent 代理開發框架，透過 `PreloadMemoryTool` 只要一行程式碼，即可自動在每次會話開始前預載使用者的歷史記憶與資料。
* 🔥 **Cloud Firestore 永久記憶體**：為了解決 Cloud Run 無狀態（Stateless）容器重啟、多實例負載平衡或縮容至零時導致記憶體遺忘的問題，自製了 **`ChineseFirestoreMemoryService`**，將使用者的每一次對話、上傳的水晶特徵、生日與星盤，永久且安全地刻在 Google Cloud Firestore 資料庫中。
* 🇨🇳 **中繁體中文智慧分詞檢索**：徹底修正了 ADK 內建 `InMemoryMemoryService` 分詞英文正規表示式（`/[A-Za-z]+/g`）完全過濾中文漢字的 Bug。實作基於中文字串與高頻占星詞彙（如：占卜、運勢、粉晶、紫水晶、黃水晶、綠幽靈、生日）的檢索匹配器。
* 🧩 **Node 22 黑科技啟動**：在 Docker 容器中使用 `node:22-alpine`，配合 `--experimental-require-module` 啟動旗標，完美解決 CJS (`require`) 同步載入 ESM (`lodash-es` 依賴) 時所發生的 `ERR_REQUIRE_ESM` 崩潰。
* 📸 **多模態水晶影像分析**：傳送水晶實拍照，機器人會主動經由 LINE Blob API 下載，轉換為 Base64 格式送至 Vertex AI Gemini 2.5 Flash 進行精細外觀與脈輪、五行共振特徵的鑑定。
* 🔒 **免金鑰安全認證 (ADC)**：部署至 Cloud Run 時，不需設定或在代碼中暴露 `GEMINI_API_KEY` 或 Firestore 金鑰 json 檔案，而是直接利用 GCP 帳戶預設憑證 (Application Default Credentials, ADC) 的 IAM 角色安全呼叫。

---

## 🛠️ 本地開發與環境設定

在開始運行之前，請確保您已完成以下準備：

### 1. LINE Developers 設定
1. 登入 [LINE Developers Console](https://developers.line.biz/)。
2. 建立 **Provider**，並在下方建立一個 **Messaging API** 頻道。
3. 在 **Basic settings** 分頁中，找到 **Channel secret**。
4. 在 **Messaging API** 分頁最下方，點擊 **Channel access token (v2)** 的 **Issue** 產生權杖。

### 2. Google Cloud Firestore 啟用
1. 進入 [Google Cloud Console](https://console.cloud.google.com/)。
2. 搜尋並進入 **Firestore**。
3. 如果尚未建立資料庫，請點擊「建立資料庫」，並選擇 **「Native (原生) 模式」**。
4. 選擇適合的資料庫儲存區域（建議與您的 Cloud Run 同一區域，例如 `asia-east1` 台灣），並建立預設資料庫 `(default)`。

### 3. 環境變數設定 (`.env`)
複製 `.env.example` 並重新命名為 `.env`，填入您取得的金鑰與 GCP 設定：

```env
PORT=8080

# LINE Channel 金鑰
LINE_CHANNEL_SECRET=您的_Channel_Secret
LINE_CHANNEL_ACCESS_TOKEN=您的_Channel_Access_Token

# Google Cloud 設定
GCP_PROJECT=您的_GCP_專案ID
GCP_LOCATION=us-central1
VERTEX_AI_MODEL=gemini-2.5-flash
```

> [!NOTE]  
> 建議將 `GCP_LOCATION` 設定為 `us-central1` 或 `asia-northeast1` (東京)，以確保 Gemini 2.5 Flash 多模態模型有最佳的區域支援與低延遲！

---

## 🚀 本地執行步驟

### 1. 安裝套件
在專案根目錄下執行（需附帶 `--legacy-peer-deps` 與專屬快取以避開部分權限衝突）：
```bash
npm install --legacy-peer-deps --cache ./.npm-cache
```

### 2. 本地認證 (呼叫 Vertex AI 與 Firestore)
在本地執行時，由於沒有 API Key，您必須確保本機電腦已設定好 Google Cloud 驗證：
```bash
# 登入您的 GCP 帳號並產生 Application Default Credentials (ADC)
gcloud auth application-default login
```

### 3. 啟動開發伺服器
```bash
# 本地使用 Node 22 啟動（帶有 experimental flag）
node --experimental-require-module --watch index.js
```
或者利用我們在 `package.json` 中配置的監聽啟動指令：
```bash
npm run dev
```

### 4. 設定 ngrok 與 LINE Webhook
1. 啟動 ngrok 將 8080 埠口映射至外網：
   ```bash
   ngrok http 8080
   ```
2. 將 ngrok 產生的 `https://xxxx.ngrok-free.app/webhook` 貼入 LINE Developers Console 的 **Webhook URL**。
3. 點擊 **Update** 並啟用 **Use Webhook**。

---

## ☁️ 部署到 Google Cloud Run 🚀

本專案已完全優化 Cloud Run 的一鍵容器化部署，無縫整合 Firestore 安全授權。

### 1. 授與 Vertex AI 與 Firestore 權限 (IAM)
在部署前，您需要讓 Cloud Run 預設的服務帳戶擁有呼叫 Vertex AI 與讀寫 Firestore 的權限。請在終端機執行：

```bash
# 授權呼叫 Vertex AI
gcloud projects add-iam-policy-binding 您的_GCP_專案ID \
  --member="serviceAccount:您的_GCP_專案編號-compute@developer.gserviceaccount.com" \
  --role="roles/aiplatform.user"

# 授權讀寫 Firestore
gcloud projects add-iam-policy-binding 您的_GCP_專案ID \
  --member="serviceAccount:您的_GCP_專案編號-compute@developer.gserviceaccount.com" \
  --role="roles/datastore.user"
```

### 2. 執行 Cloud Run 部署
使用以下指令即可在幾分鐘內完成雲端打包與部署：

```bash
gcloud run deploy line-echo-bot \
  --source . \
  --region asia-east1 \
  --allow-unauthenticated \
  --update-env-vars="GCP_PROJECT=您的_GCP_專案ID,GCP_LOCATION=us-central1,VERTEX_AI_MODEL=gemini-2.5-flash"
```

> [!IMPORTANT]  
> 為了避免洩漏金鑰，部署指令中的 `--update-env-vars` **只需填寫 GCP 專案設定**。Cloud Run 服務會自動安全地繼承您原先設定於伺服器上的 `LINE_CHANNEL_SECRET` 與 `LINE_CHANNEL_ACCESS_TOKEN`！

---

## 🧪 玩轉您的智慧占星國師

1. 打開手機 LINE 掃描 **Messaging API** 分頁最上方的 QR code 加好友。
2. **告知生日**：輸入「*老師你好，我是1995年10月12日出生的天秤座*」。
3. **傳送照片**：傳送一張您的粉晶、紫水晶或礦石照片。機器人會提示：「*📸 老師收到照片，正在為你解析能量...*」，並回覆詳細的能量與脈輪鑑定。
4. **長效記憶測試**：隔段時間輸入：「*老師，我今天適合帶什麼水晶出門？*」
   * 機器人會根據 Google ADK 與 Firestore 的持久記憶，**主動調取你之前告知的 10月12日 天秤座生日星盤，並主動對照你先前上傳的水晶收藏**。
   * 它會精確地對你說：「*親愛的，結合你落在天秤座的穩重星盤配置，今天特別適合攜帶你上次傳給我的那顆粉晶出門...*」這完美展現了長效、不重複、流暢的 AI 助理魅力！
