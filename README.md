# ThreadsPipe-py 使用指南

[![繁體中文](https://img.shields.io/badge/繁體中文-點擊查看-orange)](README.md)  

這是一份針對 [paulosabayomi/ThreadsPipe-py](https://github.com/paulosabayomi/ThreadsPipe-py) 函式庫的完整繁體中文使用指南。ThreadsPipe-py 是一個 Python 函式庫，利用 Meta 官方 Threads API 進行用戶帳號操作，包括發文、回覆、引用、轉發、資料查詢及其他輔助功能。本文檔將介紹安裝、設定、快速上手、核心功能、CLI 工具、環境變數管理、開發與貢獻等詳盡內容。

- [什麼是 ThreadsPipe-py？](#什麼是-threadspipe-py)
- [安裝](#安裝)
- [使用前準備](#使用前準備)
  - [取得 Facebook/Threads 授權](#取得-facebookthreads-授權)
  - [GitHub 媒體上傳設定](#github-媒體上傳設定)
- [快速上手](#快速上手)
  - [初始化與授權流程](#初始化與授權流程)
- [核心功能](#核心功能)
  - [發文與互動](#發文與互動)
  - [資料查詢與洞察](#資料查詢與洞察)
  - [輔助工具](#輔助工具)
- [CLI 命令行工具](#cli-命令行工具)
- [環境變數與設定檔](#環境變數與設定檔)
- [開發與貢獻](#開發與貢獻)
- [授權條款](#授權條款)

---

## 什麼是 ThreadsPipe-py？

ThreadsPipe-py 提供開發者一套簡潔而高效的介面，用以整合 Meta 官方 Threads API。透過此函式庫，您可以方便地執行發文、回覆、引用、轉發，以及查詢用戶資訊和互動數據等操作，並且支援自動拆分超長貼文、附件批次上傳等功能。本文檔為繁體中文版本，內容來源與設計均參考原作者 [paulosabayomi/ThreadsPipe-py](https://github.com/paulosabayomi/ThreadsPipe-py) 的說明文件。

---

## 安裝

### 基本需求
- **Python 版本：** Python 3.8 以上
- **認證需求：** 需擁有 Facebook 開發者帳號並建立 Threads 應用

### 安裝步驟

- 使用 pip 安裝 ThreadsPipe-py：
  ```bash
  pip install threadspipepy
  ```
- 若需使用 CLI 工具，則執行：
  ```bash
  pip install threadspipepy[cli]
  ```

---

## 使用前準備

### 取得 Facebook/Threads 授權

1. **建立 Threads 應用：**  
   前往 [Facebook Developers](https://developers.facebook.com/apps) 建立一個專案，並取得 **App ID** 與 **App Secret**；依照官方文件 [設定 Threads Use Case](https://developers.facebook.com/docs/development/create-an-app/threads-use-case) 完成設定。

2. **設定 Redirect URI：**  
   在應用設定中填入正確的 Redirect URI。當用戶完成授權後，系統將重定向至該 URI，並在 URL 中附加 `code` 參數，例如：  
   `https://example.com/handler.php?code=Abcdef...#_`  
   請在程式使用前剝除尾端的 `#_`。

### GitHub 媒體上傳設定

若需上傳本地媒體檔案，請準備以下資訊：
- GitHub 帳號與專用儲存庫（用於臨時上傳）
- GitHub 細粒度個人存取權杖（PAT）  
**注意：** 若未正確配置，嘗試上傳本地檔案時將發生錯誤。

---

## 快速上手

### 初始化與授權流程

首先，在程式中引入函式庫並初始化 ThreadsPipe 物件：
```python
from threadspipepy.threadspipe import ThreadsPipe

# 初始化 API 物件（access_token 與 user_id 可暫留空，待授權後更新）
api = ThreadsPipe(
    access_token="",
    user_id="",
    handle_hashtags=True,
    auto_handle_hashtags=False,
    # 若需上傳本地檔案，請設定以下 GitHub 資訊：
    # gh_bearer_token="YOUR_GITHUB_TOKEN",
    # gh_repo_name="YOUR_GITHUB_REPO",
    # gh_username="YOUR_GITHUB_USERNAME"
)
```

#### OAuth 授權流程

1. **取得授權碼（Authorization Code）：**
   ```python
   auth_code = api.get_auth_token(
       app_id="YOUR_APP_ID",  # 從 Facebook 開發者儀表板獲取
       redirect_uri="https://your.domain/handler",  # 請確保與設定中的相符
       scope="all"  # 或傳入所需授權範圍列表
   )
   ```
   執行後系統會自動開啟瀏覽器，引導用戶登入並授權。從 Redirect URI 擷取授權碼（記得剝除尾端的 `#_`）。

2. **交換存取權杖：**
   ```python
   tokens = api.get_access_tokens(
       app_id="YOUR_APP_ID",
       app_secret="YOUR_APP_SECRET",
       auth_code=auth_code,
       redirect_uri="https://your.domain/handler"
   )
   ```
   此方法會返回包含短期（1 小時有效）與長期（60 天有效）權杖的資料，以及用戶的 `user_id`。建議使用長期權杖便於長期應用。

3. **更新 API 物件：**
   ```python
   api.update_param(
       user_id=tokens["user_id"],
       access_token=tokens["tokens"]["long_lived"]["access_token"]
   )
   ```

完成以上流程後，您便可調用 ThreadsPipe-py 進行各項操作。

---

## 核心功能

### 發文與互動

- **發佈貼文**  
  可發送純文字、圖片或影片貼文；當貼文文字超過 500 字或附件超過 20 個時，系統將自動拆分成串貼（類似 X 线程）。  
  **範例：**
  ```python
  api.pipe(post="Hello Threads! 這是我的第一則貼文。")
  ```

- **發佈回覆**
  ```python
  api.pipe(
      post="這是一則回覆貼文。",
      reply_to_id="1234567890123456"  # 指定要回覆的貼文 ID
  )
  ```

- **引用貼文**
  ```python
  api.pipe(
      post="以下是我的評論：",
      quote_post_id="1234567890123456"  # 指定要引用的貼文 ID
  )
  ```

- **轉發貼文**
  ```python
  api.repost_post(post_id="1234567890123456")
  ```

- **自動拆分功能**  
  當貼文內容超出限制時，系統會自動將超長文字或多個附件拆分為多則貼文，並合理分配 Hashtag。

### 資料查詢與洞察

- **查詢個人資訊**
  ```python
  profile = api.get_profile()
  print(profile)
  ```

- **查詢貼文列表**
  ```python
  posts = api.get_posts(limit=10)
  for post in posts:
      print(post)
  ```

- **查詢單則貼文詳情**
  ```python
  post_detail = api.get_post(post_id="1234567890123456")
  print(post_detail)
  ```

- **查詢貼文回覆**
  ```python
  replies = api.get_post_replies(post_id="1234567890123456", top_levels=True)
  for reply in replies:
      print(reply)
  ```

- **取得貼文互動數據**
  ```python
  insights = api.get_post_insights(post_id="1234567890123456", metrics="all")
  print(insights)
  ```

- **取得帳戶整體洞察**
  ```python
  user_insights = api.get_user_insights(since_date="2025-01-01", until_date="2025-03-01")
  print(user_insights)
  ```

### 輔助工具

- **生成貼文意圖 URL**  
  可用於嵌入網站的分享按鈕，讓用戶一鍵前往 Threads 發文：
  ```python
  intent_url = api.get_post_intent(text="前往 Threads 發文", link="https://your.domain/somepage")
  print("貼文意圖連結：", intent_url)
  ```

- **生成追蹤意圖 URL**  
  生成快捷追蹤用戶的 URL：
  ```python
  follow_url = api.get_follow_intent(username="your_threads_username")
  print("追蹤意圖連結：", follow_url)
  ```

---

## CLI 命令行工具

ThreadsPipe-py 附帶 CLI 工具，使用者可在終端機中進行存取權杖管理等操作。

### 取得存取權杖

```bash
threadspipepy access_token \
  --app_id=YOUR_APP_ID \
  --app_secret=YOUR_APP_SECRET \
  --auth_code="YOUR_AUTH_CODE" \
  --redirect_uri="https://your.domain/handler" \
  --env_path="./.env" \
  --env_variable="THREADS_TOKEN"
```

此指令將以授權碼換取短期與長期存取權杖，並可自動更新指定 `.env` 檔案內的環境變數。

### 刷新存取權杖

```bash
threadspipepy refresh_token \
  --access_token="YOUR_CURRENT_LONG_LIVED_TOKEN" \
  --env_path="./.env" \
  --env_variable="THREADS_TOKEN"
```

若設定 `--auto_mode=true`，CLI 將自動使用 `.env` 檔案中指定的變數值進行更新。

### 查看 CLI 幫助

```bash
threadspipepy -h
```

此指令會顯示所有可用的 CLI 命令與選項。

---

## 環境變數與設定檔

為了安全管理 App Secret 與 Access Token，建議使用環境變數或 `.env` 檔案。  
例如，在 Linux/Unix 中設定環境變數：
```bash
export THREADS_TOKEN="YOUR_LONG_LIVED_TOKEN"
```
在程式中可透過：
```python
import os
token = os.environ.get("THREADS_TOKEN")
api.update_param(access_token=token)
```
CLI 工具同時支援自動更新 `.env` 檔案中指定變數。

---

## 開發與貢獻

如果您有意參與本專案改進或提供建議，請參考下列資訊：

- **原始碼儲存庫：** [paulosabayomi/ThreadsPipe-py](https://github.com/paulosabayomi/ThreadsPipe-py)
- **問題回報：** 請至 GitHub Issues 提出您的建議或報告問題
- **Pull Requests：** 請按照原專案貢獻指南提交您的修改

本使用指南僅為繁體中文翻譯版本，內容來源均參考原作者說明。請在使用時尊重原始授權與版權資訊。

---

## 授權條款

ThreadsPipe-py 採用 **MIT 授權條款**。詳細內容請參閱 [LICENSE](https://github.com/paulosabayomi/ThreadsPipe-py/blob/main/LICENSE)。  
本繁體中文使用指南僅為翻譯與整理用途，原始專案內容請參考 [paulosabayomi/ThreadsPipe-py](https://github.com/paulosabayomi/ThreadsPipe-py)。

---

以上即為 ThreadsPipe-py 的完整繁體中文使用指南。希望本文件能協助您快速上手並有效整合 Threads API，同時感謝您對原作者工作的支持與尊重！

Happy Coding!
