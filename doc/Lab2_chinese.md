好的 ✅ 我已經幫你把 **6.5840 Lab 2: Key/Value Server** 的整份說明轉成了 **結構化 Markdown**，保持了格式與內容一致，只是排版更清楚、方便閱讀：

---

# 6.5840 - Spring 2025

* Lab 2: Key/Value Server

### Collaboration policy // Submit lab // Setup Go // Guidance // Piazza

---

## Introduction

在本實驗中，你將建立一個單機的 Key/Value (KV) server，確保每個 `Put` 操作在網路故障下仍 **at-most-once**，並且操作需具備 **linearizability**。
之後你將用這個 KV server 來實作一個 **lock**。後續實驗會擴展到容錯。

---

## KV Server

### RPC 介面

* Client 與 server 透過 **Clerk** 互動。
* RPC:

  * `Put(key, value, version)`
  * `Get(key)`

### Server 狀態

* Server 維護一個 in-memory map：`key -> (value, version)`
* Key/Value: 字串
* Version: key 被寫入的次數

### Put 規則

* 僅當 `Put` 的 version 與 server 當前 version 一致時，才安裝/取代 value。
* 成功後，server 將 **version+1**。
* 特殊情況:

  * `Put(..., version=0)` 可建立新 key → server 版本從 1 開始
  * 若 `version > 0` 且 key 不存在 → 回傳 `rpc.ErrNoKey`
  * 若 version 不符 → 回傳 `rpc.ErrVersion`

### Get 規則

* 回傳 key 的 `(value, version)`
* 若 key 不存在 → 回傳 `rpc.ErrNoKey`

---

## Linearizability

* 對 client 而言，`Clerk.Get` 與 `Clerk.Put` 的效果如同單一 server 順序執行。
* **非並行**操作：必須看到前序操作的效果。
* **並行**操作：回傳值與最終狀態應符合某個序列化順序。
* 提供 linearizability 對應用程式而言就像單一執行緒的 server。

---

## Getting Started

* Skeleton code: `src/kvsrv1/`

  * `client.go`: Clerk → `Put`、`Get`
  * `server.go`: Server handlers → `Put`、`Get`
  * `rpc/rpc.go`: RPC 定義（不需修改）

### 初始測試

```bash
cd ~/6.5840
git pull
cd src/kvsrv1
go test -v
```

範例錯誤：

```
=== RUN   TestReliablePut
One client and reliable Put (reliable network)...
    kvsrv_test.go:25: Put err ErrNoKey
```

---

## Key/Value Server with Reliable Network (Easy)

### 任務

1. 實作 Clerk 的 `Put/Get` → 發送 RPC
2. 實作 Server 的 `Put/Get` handlers

### 驗收

通過 `Reliable` 測試：

```bash
go test -v -run Reliable
```

### 範例輸出

```
=== RUN   TestReliablePut
... Passed
=== RUN   TestPutConcurrentReliable
... Passed
=== RUN   TestMemPutManyClientsReliable
... Passed
PASS
```

---

## Implementing a Lock (Moderate)

### 說明

* 分散式應用程式常使用 KV server 來協調，例如 **ZooKeeper**、**Etcd**。
* 這裡要基於 `Clerk.Put/Get` 實作：

  * `Acquire`
  * `Release`

### 條件

* 同一時間僅允許一個 client `Acquire` 成功
* 其他 client 必須等待直到 lock 被釋放
* 若 client crash，lock 會永久卡住（本實驗忽略此問題）

### Skeleton Code

* 位置：`src/kvsrv1/lock/lock.go`
* 測試：

```bash
cd lock
go test -v -run Reliable
```

### 範例輸出

```
=== RUN   TestOneClientReliable
... Passed
=== RUN   TestManyClientsReliable
... Passed
PASS
```

### 提示

* 每個 client 需有唯一 ID（用 `kvtest.RandValue(8)` 產生）
* Lock 狀態存放在 KV server 的某個固定 key

---

## Key/Value Server with Dropped Messages (Moderate)

### 挑戰

網路可能：

* 重新排序
* 延遲
* 丟失 **request/reply**

### 解法

* **Request 丟失** → Client 重送即可
* **Reply 丟失**：

  * Get：安全
  * Put：透過 version 檢查

    * 若 server 已執行過，重送會回 `rpc.ErrVersion`
    * **Ambiguous case**:

      * Client 重送 → 收到 `rpc.ErrVersion`
      * 不確定第一次 Put 是否已成功（reply 可能丟了）
      * 此時必須回 **`rpc.ErrMaybe`**

### Clerk 規則

* 初次 Put 若收到 `rpc.ErrVersion` → 回傳 `rpc.ErrVersion`
* 重送 Put 收到 `rpc.ErrVersion` → 回傳 `rpc.ErrMaybe`
* Clerk 必須 **重試直到有回覆**
* 重試間隔：`time.Sleep(100 * time.Millisecond)`

### 驗收

```bash
go test -v
```

### 範例輸出

```
=== RUN   TestUnreliableNet
One client (unreliable network)...
... Passed
PASS
```

---

## Implementing a Lock with Unreliable Network (Easy)

### 說明

* 修改 lock 使其在 **不可靠網路** 下也正確工作
* 基於修改後的 KV client

### 驗收

```bash
cd lock
go test -v
```

### 範例輸出

```
=== RUN   TestOneClientUnreliable
... Passed
=== RUN   TestManyClientsUnreliable
... Passed
PASS
```

---

這樣的結構把內容切清楚（Introduction → KV server → Reliable → Lock → Dropped messages → Lock+Unreliable），方便邊看邊做實驗 👍

要不要我幫你整理成 **實作 checklist**（每個階段要改哪個檔案、通過哪個測試）？這樣你可以邊做邊打勾。
