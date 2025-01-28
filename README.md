### 2. 設定 Google Apps Script
1. **進入 Google Apps Script 編輯器**：
   - 打開 [Google Apps Script](https://script.google.com/)。
   - 建立一個新專案。

2. **撰寫程式碼**：
   - 複製以下程式碼，並貼到 Apps Script 編輯器。

```
// LINE Notify 的 Token
var LINE_NOTIFY_TOKEN = "**************"; // 請替換為實際的 Token

// 搜尋 Gmail 郵件的條件
var query = "subject:您的 Netflix 暫時存取碼";

// 檢查郵件並將符合條件的郵件通知到 LINE
function getMail() {
  
  // 使用指定的條件搜尋 Gmail 郵件（取 10 封）
  var myThreads = GmailApp.search(query, 0, 10);

  // 取得每個郵件線程中的郵件內容
  var myMessages = GmailApp.getMessagesForThreads(myThreads);

  // 遍歷每個線程
  for (var i in myMessages) {
    for (var j in myMessages[i]) {

      // 僅處理未加星標的郵件
      if (!myMessages[i][j].isStarred()) {

        // 建立訊息字串，先加上郵件主題
        var strmsg = myMessages[i][j].getSubject() + "\n"; // 主題

        // 使用正則表達式從郵件內容中提取 URL
        const regex = /\[https:\/\/www\.netflix\.com\/account\/travel\/verify\?[^ ]+\]/g;
        const match = myMessages[i][j].getPlainBody().match(regex);

        if (match) {
          // 去除 URL 的中括號並添加到訊息
          const url = match[0].replace('[', '').replace(']', '');
          strmsg += url + "\n\n* 連結將於 15 分鐘後失效。"; // 顯示 URL 並加提示
        }

        // 傳送訊息到 LINE
        sendLineMessage(strmsg);

        // 將已處理的郵件加上星標，避免重複處理
        myMessages[i][j].star();
      }
    }
  }
}

// 傳送訊息到 LINE
function sendLineMessage(msg) {
  var payload = {
    "to": "**************", // LINE 群組或用戶的 ID，請替換為目標 ID
    "messages": [
      {
        "type": "text",
        "text": msg // 傳送的訊息內容
      }
    ]
  };

  // 設定請求參數
  var options = {
    "method": "post",
    "headers": {
      "Authorization": "Bearer " + LINE_NOTIFY_TOKEN, // 驗證用的 Bearer Token
      "Content-Type": "application/json" // 請求內容格式
    },
    "payload": JSON.stringify(payload) // 將 payload 轉換為 JSON 字串
  };

  // 傳送請求到 LINE Messaging API
  var response = UrlFetchApp.fetch("https://api.line.me/v2/bot/message/push", options);

  // 在日誌中輸出 LINE API 的回應內容
  console.log(response.getContentText());
}
```

3. **部署專案**：
   - 點擊「**部署**」→「新增部署」。
   - 選擇「**執行 API**」方式，並點擊「部署」。
   - 記錄部署後的 URL。

---

### 3. 自動化執行
1. **設置觸發器**：
   - 在 Apps Script 編輯器中，點擊「**時鐘圖示 (觸發器)**」。
   - 新增一個觸發器，選擇 `getMail`，設定每隔 5 分鐘執行一次。

---

## 注意事項
1. **確保權限**：
   - 第一次執行時，Google Apps Script 會要求授權訪問 Gmail 和外部 API，請允許相關權限。

2. **測試 BOT 功能**：
   - 執行 `getMail()` 測試功能是否正常，確認訊息是否成功推送到 LINE 群組。

3. **LINE Messaging API 配額**：
   - 免費帳號每日有推送訊息的限制，請注意流量使用。

---

## 結語
完成上述設定後，您的系統將自動接收 Netflix 的暫時存取碼，並推送到指定的 LINE 群組中，大幅提高便利性！
