# FSM Socket Server

一個基於 Socket 的有限狀態機服務器，接收外部的 Message 結構並處理狀態轉換。

## 架構

### Message 結構
```c
typedef struct {
    uint32_t msg_id;             // 訊息 ID
    uint32_t transaction_id;     // 外部交易 ID
    char data[MAX_DATA_LENGTH];  // 資料內容 (字串)
} Message;
```

### 狀態轉換
```
idle → connecting → connected → disconnecting → disconnected → idle
```

每個完整的狀態循環（idle → disconnected → idle）稱為一個 **transaction**。

## 編譯和運行

### 1. 編譯
```bash
make all
```

### 2. 運行服務器
```bash
./fsm_server
```

### 3. 運行客戶端（在另一個終端）
```bash
./fsm_client
```

## 功能特點

### 1. 多客戶端支持
- 服務器支持多個客戶端同時連接
- 每個客戶端有獨立的 FSM 實例

### 2. Transaction 追蹤
- 每個 transaction 有唯一的 ID
- 自動計數完成的 transaction 數量
- 支持中斷和重新開始 transaction

### 3. 狀態處理
- **MSG_1**: 一般消息，不觸發狀態轉換
- **MSG_2**: 一般消息，不觸發狀態轉換  
- **MSG_3**: 觸發狀態轉換到下一個狀態
- **MSG_POWEROFF (0xFFFFFFFF)**: 特殊消息，關閉服務器

### 4. 詳細日誌
- 服務器顯示每個客戶端的狀態變化
- 客戶端顯示發送的消息和服務器響應

## 測試場景

### 1. 單個完整 Transaction
客戶端發送完整的狀態序列：
```
MSG_3 → MSG_1 → MSG_3 → MSG_2 → MSG_1 → MSG_3 → MSG_2 → MSG_3 → MSG_1 → MSG_3
```

### 2. 多個連續 Transaction
測試多個 transaction 的連續執行。

### 3. 中斷的 Transaction
測試未完成的 transaction 和新 transaction 的處理。

### 4. Poweroff 測試
客戶端在完成所有測試後發送 POWEROFF 消息來優雅地關閉服務器。

## 輸出示例

### 服務器輸出
```
[SERVER] FSM Server started on port 8888
[SERVER] Waiting for clients...
[SERVER] New client connected from 127.0.0.1:12345
[CLIENT 4] Connected, FSM initialized
[CLIENT 4] Received: msg_id=3, transaction_id=1001, data='Start connecting'
[FSM] Transaction 1001: State 0 → State 1
  [idle] 收到 MSG_3：準備 connecting, data='Start connecting'
[CLIENT 4] Received: msg_id=1, transaction_id=1001, data='Connecting in progress'
[FSM] Transaction 1001: State 1 → State 1
  [connecting] 正在嘗試連線...忽略 MSG_1, data='Connecting in progress'
```

### 客戶端輸出
```
[CLIENT] Connected to server 127.0.0.1:8888
[CLIENT] Starting FSM tests...
=== Testing Transaction 1001 ===
[CLIENT] Sent: msg_id=3, transaction_id=1001, data='Start connecting'
[CLIENT] Server response: State: 1, Transaction: 1001, Total: 0
[CLIENT] Sent: msg_id=1, transaction_id=1001, data='Connecting in progress'
[CLIENT] Server response: State: 1, Transaction: 1001, Total: 0

### Poweroff 輸出
```
[CLIENT] === Sending POWEROFF message ===
[CLIENT] Sent POWEROFF message
[CLIENT] Server response: POWEROFF confirmed, server will shutdown
[CLIENT] Server shutdown initiated
[SERVER] [CLIENT 4] Received POWEROFF message: Shutdown server
[SERVER] [CLIENT 4] Initiating server shutdown...
```

## 配置

### 端口配置
在 `fsm_server.c` 中修改：
```c
#define SERVER_PORT 8888
```

### 客戶端 IP
在 `fsm_client.c` 中修改：
```c
#define SERVER_IP "127.0.0.1"
```

## 擴展

### 添加新狀態
1. 在 `State` 枚舉中添加新狀態
2. 在 `g_trans` 數組中定義轉換規則
3. 實現新的狀態處理函數
4. 更新 `STATE_TABLE`

### 添加新消息類型
1. 在狀態處理函數中添加新的 case
2. 更新轉換邏輯（如果需要）

## 故障排除

### 常見問題

1. **端口被佔用**
   ```bash
   # 檢查端口使用情況
   netstat -tuln | grep 8888
   # 或者修改端口號
   ```

2. **連接失敗**
   - 確保服務器正在運行
   - 檢查防火牆設置
   - 確認 IP 地址正確

3. **編譯錯誤**
   ```bash
   # 安裝必要的開發工具
   sudo apt-get install build-essential
   ```

## 許可證

此代碼僅供學習和測試使用。 
