# Lab2 超詳細學習計劃

適合上班族的 MIT 6.5840 Lab2 學習安排，每天 20-45 分鐘，14天完成。

## 📚 Lab2 概述

Lab2 要實現一個**單機鍵值服務器**，支持線性化操作和至多一次語義。包含4個任務：

1. **基本 KV 服務器**（可靠網絡）- 簡單
2. **分散式鎖實現**（可靠網絡）- 中等
3. **處理網絡故障的 KV 服務器** - 中等
4. **不可靠網絡下的分散式鎖** - 簡單

## 📅 詳細學習計劃（14天完成）

### **第1週：理解與基礎實現**

**Day 1 (週一) - 20分鐘：環境準備**

- [X] `cd ~/6.5840 && git pull`
- [X] `cd src/kvsrv1 && go test -v` (看失敗訊息，理解當前狀態)
- [X] 閱讀中文 Lab2 文檔前半部分（Introduction + KV Server 章節）

**Day 2 (週二) - 30分鐘：理解代碼結構**

- [X] 仔細讀 `rpc/rpc.go`，理解所有結構體和錯誤類型
- [ ] 看 `client.go` 和 `server.go` 的骨架代碼，理解 RPC 調用方式
- [ ] 畫圖理解：Client → Clerk → RPC → Server 的流程

**Day 3 (週三) - 35分鐘：實現 Server 端 Get**

- [ ] 在 `KVServer` 結構體中添加 map 存儲
- [ ] 實現 `Get` 方法處理邏輯：
  ```go
  func (kv *KVServer) Get(args *rpc.GetArgs, reply *rpc.GetReply) {
      // 1. 加鎖
      // 2. 查找 key
      // 3. 設置 reply
  }
  ```
- [ ] 運行 `go test -v -run TestReliablePut` 確認 Get 部分正常

**Day 4 (週四) - 40分鐘：實現 Server 端 Put**

- [ ] 實現 `Put` 方法處理版本檢查：
  ```go
  func (kv *KVServer) Put(args *rpc.PutArgs, reply *rpc.PutReply) {
      // 1. 加鎖
      // 2. 檢查版本號規則
      // 3. 更新或創建 key
  }
  ```
- [ ] 測試版本檢查邏輯（新key版本0，已存在key版本匹配）

**Day 5 (週五) - 30分鐘：實現 Client 端 Get**

- [ ] 在 `Clerk` 結構體中實現 `Get` 方法：
  ```go
  func (ck *Clerk) Get(key string) (string, rpc.Tversion, rpc.Err) {
      // 1. 構建 args
      // 2. 調用 RPC
      // 3. 返回結果
  }
  ```
- [ ] 運行測試確認 Get 功能正常

**Day 6 (週六) - 35分鐘：實現 Client 端 Put**

- [ ] 實現 `Put` 方法（暫時不考慮重試）：
  ```go
  func (ck *Clerk) Put(key string, value string, version rpc.Tversion) rpc.Err {
      // 1. 構建 args
      // 2. 調用 RPC
      // 3. 返回錯誤
  }
  ```
- [ ] 運行 `go test -v -run Reliable` 確認基礎功能

**Day 7 (週日) - 25分鐘：修復和驗證**

- [ ] 修復任何測試失敗的問題
- [ ] 確保通過所有 Reliable 測試
- [ ] 運行 `go test -race` 檢查併發安全

### **第2週：分散式鎖與網絡故障**

**Day 8 (週一) - 30分鐘：理解 Lock 需求**

- [ ] 閱讀 `lock/lock.go` 骨架代碼
- [ ] 理解分散式鎖的概念：互斥、唯一擁有者
- [ ] 設計鎖的狀態表示：用什麼key存儲、存儲什麼值

**Day 9 (週二) - 40分鐘：實現 Lock Acquire**

- [ ] 在 `MakeLock` 中生成唯一客戶端ID
- [ ] 實現 `Acquire` 方法：
  ```go
  func (lk *Lock) Acquire() {
      // 1. 生成唯一ID (如果還沒有)
      // 2. 嘗試Put(lockKey, clientID, 0) 獲取鎖
      // 3. 如果失敗就重試
  }
  ```

**Day 10 (週三) - 25分鐘：實現 Lock Release**

- [ ] 實現 `Release` 方法：
  ```go
  func (lk *Lock) Release() {
      // 1. Get當前鎖狀態
      // 2. 驗證是自己持有的鎖
      // 3. Put(lockKey, "", version+1) 釋放
  }
  ```
- [ ] 運行 `cd lock && go test -v -run Reliable`

**Day 11 (週四) - 45分鐘：處理網絡重試**

- [ ] 修改 `client.go` 的 `Get` 方法添加重試邏輯：
  ```go
  for {
      ok := ck.clnt.Call(ck.server, "KVServer.Get", &args, &reply)
      if ok {
          break
      }
      time.Sleep(100 * time.Millisecond)
  }
  ```

**Day 12 (週五) - 45分鐘：處理 Put 重試和 ErrMaybe**

- [ ] 修改 `Put` 方法處理重試邏輯：
  ```go
  firstTry := true
  for {
      ok := ck.clnt.Call(ck.server, "KVServer.Put", &args, &reply)
      if ok {
          if reply.Err == rpc.ErrVersion && !firstTry {
              return rpc.ErrMaybe  // 重試時的版本錯誤
          }
          return reply.Err
      }
      firstTry = false
      time.Sleep(100 * time.Millisecond)
  }
  ```

**Day 13 (週六) - 35分鐘：調整 Lock 處理網絡故障**

- [ ] 檢查 Lock 實現是否能正確處理 `ErrMaybe`
- [ ] 運行 `cd lock && go test -v` 確認不可靠網絡測試通過
- [ ] 修復任何失敗的測試

**Day 14 (週日) - 30分鐘：最終測試和整理**

- [ ] 運行所有測試：`go test -v` 和 `cd lock && go test -v`
- [ ] 運行 `go test -race` 檢查併發安全
- [ ] 整理代碼，添加必要註釋
- [ ] 準備提交：`make lab2`

## 🎯 每日小提示

**工作日晚上 (20-45分鐘)**：

- 先花5分鐘回顧昨天做了什麼
- 專注完成當天的單一任務
- 如果卡住了，做筆記明天繼續

**週末 (30-45分鐘)**：

- 時間稍長一些，可以做整合測試
- 修復累積的小問題

**遇到問題時**：

- 先看測試失敗訊息
- 檢查 RPC 參數和返回值類型
- 用 `DPrintf` 添加調試訊息

## 💡 重要概念提醒

### KV Server 核心邏輯

- **版本控制**：每個key都有版本號，用於實現條件更新
- **至多一次語義**：通過版本檢查確保同一操作不會執行多次
- **線性化**：操作效果如同單線程順序執行

### 分散式鎖設計

- **互斥性**：同時只有一個客戶端能持有鎖
- **唯一標識**：每個客戶端需要唯一ID來區分鎖的擁有者
- **條件釋放**：只有鎖的持有者才能釋放鎖

### 網絡故障處理

- **重試機制**：客戶端在超時後重新發送請求
- **ErrMaybe**：當無法確定操作是否成功時返回的特殊錯誤
- **冪等性**：Get操作天然冪等，Put通過版本檢查實現冪等

## 📋 進度追蹤

你可以把這個checklist打印出來或複製到筆記軟件中，每天完成任務後打勾。

**第1週完成度**：□□□□□□□ (7/7)
**第2週完成度**：□□□□□□□ (7/7)

祝你學習順利！🎉
