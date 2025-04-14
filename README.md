# ThreadsPipe-py

ThreadsPipe-py 是一個基於 Python 語言的函式庫，旨在簡化與 Meta 官方 Threads API 的整合。透過此函式庫，開發者能夠輕鬆執行發文、回覆、引用、轉發以及查詢數據等操作，並提供自動處理超長文字、媒體上傳與 Hashtag 配置等實用功能。

---

## 目錄

- [專案介紹](#專案介紹)
- [功能列表](#功能列表)
- [安裝指南](#安裝指南)
- [使用前準備](#使用前準備)
  - [取得 Facebook/Threads 授權](#取得-facebookthreads-授權)
  - [GitHub 媒體上傳設定](#github-媒體上傳設定)
- [快速上手](#快速上手)
  - [初始化與授權流程](#初始化與授權流程)
  - [發佈貼文與回覆](#發佈貼文與回覆)
- [進階 API 功能](#進階-api-功能)
  - [發文、引用、轉發](#發文引用轉發)
  - [資料查詢與洞察](#資料查詢與洞察)
- [CLI 命令行工具](#cli-命令行工具)
- [環境變數與設定檔](#環境變數與設定檔)
- [開發與貢獻](#開發與貢獻)
- [授權條款](#授權條款)

---

## 專案介紹

ThreadsPipe-py 為開發者提供一套高階介面，使得與 Threads API 進行整合變得簡便。該函式庫解決了純手動調用官方 API 可能遇到的限制，例如文字超長問題、自動拆分串貼、上傳本地媒體檔案（結合 GitHub 作為暫存平台）以及 Hashtag 分配處理等。使用 ThreadsPipe-py，開發者可以更加專注於業務邏輯，而無需過多關注 API 接口細節。

---

## 功能列表

### 發文與互動
- **發佈新貼文：** 支援純文字、附帶圖片或影片貼文。
- **回覆貼文：** 透過指定 `reply_to_id` 進行貼文回覆。
- **引用貼文：** 引用其他貼文並附加自定義評論，支援選擇性在多則串貼中持續引用。
- **轉發貼文：** 方便地轉發原有貼文以擴大影響。
- **自動拆分文字：** 當文字超出 Threads 平台上限時，自動拆分為多則串貼，同時處理 Hashtag 的分配。

### 資料查詢
- **查詢個人資訊：** 取得當前授權帳戶的個人資料。
- **查詢貼文列表：** 取得使用者發佈的所有貼文，並支援日期篩選與數量限制。
- **查詢單則貼文：** 根據貼文 ID 獲取詳細內容。
- **查詢貼文回覆：** 檢索指定貼文下的回覆留言，可設定僅顯示第一層或全部回覆。

### 互動洞察
- **貼文互動數據：** 查詢貼文的瀏覽數、按讚數、回覆數、轉發數及引用數等數據。
- **帳戶整體數據：** 獲取包含追蹤者統計、互動數據在內的整體帳戶洞察，並支援細分地域、年齡、性別等參數分析。

### 輔助工具
- **意圖連結生成：** 生成貼文意圖 URL（用於分享或網站嵌入）與追蹤意圖 URL，以便快速引導使用者操作。
- **上傳本地媒體：** 結合 GitHub 儲存庫實現本地媒體檔案上傳，解決直接上傳限制。

---

## 安裝指南

### 基本需求
- **Python 版本：** 需 3.8 以上版本。
- **Threads API 認證：** 需擁有 Facebook 開發者帳號並建立 Threads 應用。

### 安裝步驟
1. 使用 pip 安裝函式庫：
   ```bash
   pip install threadspipepy
   ```
2. 若需使用 CLI 工具，可執行：
   ```bash
   pip install threadspipepy[cli]
   ```

---

## 使用前準備

### 取得 Facebook/Threads 授權
1. **建立 Threads 應用：**  
   前往 [Facebook Developers](https://developers.facebook.com/) 建立專案，並依照說明取得 **App ID** 與 **App Secret**，同時申請相應 Threads 權限。
2. **設定 Redirect URI：**  
   在應用設定中填寫一個正確可用的 redirect URI，系統將在 OAuth 授權流程後返回授權碼（auth code）。

### GitHub 媒體上傳設定
為解決本地媒體檔案上傳問題，請準備以下資訊：
- GitHub 帳號與相應儲存庫（用作檔案暫存）
- GitHub 細粒度個人存取權杖（PAT）
  
若未配置上述資訊，調用上傳本地檔案功能時系統將會拋出錯誤。

---

## 快速上手

### 初始化與授權流程
首先，在程式中初始化 ThreadsPipe 物件：
```python
from threadspipepy.threadspipe import ThreadsPipe

# 建立 API 實例（授權相關參數稍後更新）
api = ThreadsPipe(
    access_token="",    # 待取得後更新
    user_id="",         # 待取得後更新
    handle_hashtags=True,
    auto_handle_hashtags=False,
    # 若需上傳本地檔案，請配置下列 GitHub 資訊：
    # gh_bearer_token="YOUR_GITHUB_TOKEN",
    # gh_repo_name="YOUR_GITHUB_REPO",
    # gh_username="YOUR_GITHUB_USERNAME"
)
```

#### OAuth 授權步驟
1. **取得授權碼：**
   ```python
   auth_code = api.get_auth_token(
       app_id="YOUR_APP_ID",
       redirect_uri="https://your.domain/handle",
       scope="all"
   )
   ```
   系統會開啟瀏覽器提示您登入並同意授權，完成後從返回的 URL 擷取授權碼（注意移除 URL 尾部的多餘片段）。

2. **交換存取權杖：**
   ```python
   tokens = api.get_access_tokens(
       app_id="YOUR_APP_ID",
       app_secret="YOUR_APP_SECRET",
       auth_code=auth_code,
       redirect_uri="https://your.domain/handle"
   )
   ```
   該方法將返回短期與長期存取權杖，建議使用長期權杖（約 60 天有效期）。

3. **更新 API 實例：**
   ```python
   api.update_param(
       user_id=tokens["user_id"],
       access_token=tokens["tokens"]["long_lived"]["access_token"]
   )
   ```

完成以上步驟後，您即可開始調用各項 API 功能。

---

## 發佈貼文與回覆

### 發佈純文字貼文
```python
api.pipe(post="Hello Threads! 這是我的第一則貼文。")
```

### 發佈附帶圖片的貼文
```python
api.pipe(
    post="分享一張精美照片。",
    files=["./local/path/to/image.jpg"]
)
```

### 發佈回覆貼文
```python
api.pipe(
    post="這是一則回覆。",
    reply_to_id="1234567890123456"  # 指定回覆目標貼文的 ID
)
```

### 發佈引用貼文
```python
api.pipe(
    post="以下是我的看法：",
    quote_post_id="1234567890123456"
)
```

### 轉發貼文
```python
api.repost_post(post_id="1234567890123456")
```

---

## 進階 API 功能

### 發文、引用與轉發
- **自動拆分超長文字：**  
  若貼文超過 Threads 平台字數限制（約 500 字元），函式庫將自動將內容拆分為多則貼文，並合理分配 Hashtag。
- **設定回覆權限：**  
  發佈貼文時可指定 `who_can_reply` 參數（例如 `"everyone"` 或 `"accounts_you_follow"`），以控制回覆權限。

### 資料查詢與洞察
- **取得個人資訊：**
  ```python
  profile = api.get_profile()
  print(profile)
  ```
- **查詢貼文列表：**
  ```python
  posts = api.get_posts(limit=10)
  for p in posts:
      print(p)
  ```
- **取得單則貼文詳細資訊：**
  ```python
  post_detail = api.get_post(post_id="1234567890123456")
  print(post_detail)
  ```
- **查詢貼文回覆：**
  ```python
  replies = api.get_post_replies(post_id="1234567890123456", top_levels=True)
  for reply in replies:
      print(reply)
  ```
- **取得貼文互動數據：**
  ```python
  insights = api.get_post_insights(post_id="1234567890123456", metrics="all")
  print(insights)
  ```
- **取得帳戶整體洞察：**
  ```python
  user_insights = api.get_user_insights(since_date="2025-01-01", until_date="2025-03-01")
  print(user_insights)
  ```
- **生成貼文意圖 URL：**
  ```python
  intent_url = api.get_post_intent(text="前往 Threads 發文", link="https://your.domain/somepage")
  print("分享連結:", intent_url)
  ```
- **生成追蹤意圖 URL：**
  ```python
  follow_url = api.get_follow_intent(username="your_threads_username")
  print("追蹤連結:", follow_url)
  ```

---

## CLI 命令行工具

ThreadsPipe-py 附帶 CLI 工具，可在命令行環境下執行存取權杖交換、刷新等操作。

### 取得存取權杖
```bash
threadspipepy access_token \
  --app_id=YOUR_APP_ID \
  --app_secret=YOUR_APP_SECRET \
  --auth_code="YOUR_AUTH_CODE" \
  --redirect_uri="https://your.domain/handle" \
  --env_path="./.env" \
  --env_variable="THREADS_TOKEN"
```
此指令將幫助您取得長期存取權杖，並儲存至指定的 `.env` 檔中。

### 刷新存取權杖
```bash
threadspipepy refresh_token \
  --access_token="YOUR_CURRENT_LONG_LIVED_TOKEN" \
  --app_id=YOUR_APP_ID \
  --app_secret=YOUR_APP_SECRET
```
刷新成功後請確保將新權杖更新到您的環境變數檔中。

### 查看 CLI 幫助
```bash
threadspipepy -h
```
此指令會顯示所有可用命令與選項。

---

## 環境變數與設定檔

為方便管理敏感資料（如 App Secret 與 Access Token），建議使用環境變數或 `.env` 檔案。  
例如，在 Linux/Unix 環境下可設定環境變數：
```bash
export THREADS_TOKEN="YOUR_LONG_LIVED_TOKEN"
```
程式中讀取方式：
```python
import os
token = os.environ.get("THREADS_TOKEN")
api.update_param(access_token=token)
```
CLI 工具亦支援直接寫入 `.env` 檔案。

---

## 開發與貢獻

若您有意為本專案貢獻代碼，歡迎透過以下方式參與：
- **原始碼儲存庫：** [GitHub Repository](https://github.com/paulosabayomi/ThreadsPipe-py)
- **問題回報：** 請在 GitHub Issues 中提出建議或回報問題。
- **Pull Requests：** 請參考專案的貢獻指南提交 PR。

我們期待各位開發者能夠共同維護與改進本函式庫。

---

## 授權條款

ThreadsPipe-py 採用 **MIT 授權條款**，請參閱 [LICENSE](LICENSE) 文件以瞭解詳細權限與條件。使用本函式庫時，請確保依據授權條款妥善引用版權聲明。
