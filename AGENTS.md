# Repository Guidelines

## 專案結構與模組
- 主要程式碼位於 `src/`（Go module：`6.5840`，Go 1.22）。
- 可執行程式在 `src/main/`（如 `mrcoordinator.go`、`mrworker.go`）。
- 重要套件/實驗：`mr/`、`mrapps/`、`raft1/`、`kvsrv1/`、`kvraft1/`、`shardkv1/`，輔助：`labrpc/`、`labgob/`、`tester1/`。
- 其他：`doc/` 放說明；`Makefile` 用於打包繳交。

## 建置、測試與開發命令
- 格式化：`cd src && go fmt ./...`（提交前必跑）。
- 靜態檢查：`cd src && go vet ./...`（建議）。
- 全域測試：`cd src && go test ./...`。
- 競態檢查：`cd src && go test -race ./...`（並行/一致性實驗強烈建議）。
- 套件測試：`cd src && go test ./raft1 -run TestName`（依需求替換套件）。
- 打包繳交：`make lab1`（或 `lab2`、`lab3a`…），輸出 `labX-handin.tar.gz`。

## 程式風格與命名
- 依 Go 標準風格；統一以 `go fmt`。縮排與空白遵循 gofmt。
- 套件名一律小寫短名（如 `raft1`）；檔名小寫（必要時才用底線）。
- 輸出符號使用 `CamelCase`，未輸出使用 `camelCase`；避免重覆前綴。
- 錯誤以 `error` 回傳，不在函式庫中 `panic`；必要時以 `fmt.Errorf` 補上下文。
- 紀錄：優先使用各套件的偵錯工具（如 `DPrintf`），預設關閉。

## 測試指引
- 測試檔採 `*_test.go`，測試函式 `TestXxx`；可採表格化測試。
- 於 `src/` 執行測試以確保模組路徑正確；並行元件請搭配 `-race`。
- 測試應可重現、具時限；避免任意 `sleep`，以通道/超時控制。

## Commit 與 PR 規範
- 訊息：`<pkg>: <變更摘要>`，例：`raft1: fix leader election timing`。
- 內容：說明動機/取捨與測試方式（如 `go test -race ./...` 結果）。
- PR：清楚描述、連結任務/章節、附重現步驟與必要紀錄。

## 代理與安全規範（重要）
- 全程中文回覆；嚴禁提供任何「函式實作代碼」。
- 僅在你明確下達要求時，提供「語法用法提示」，且僅限語法層面。
- 勿修改評測/繳交相關檔（`Makefile`、`.check-build`）除非明確要求。
- 非必要勿新增外部相依；保持變更最小、介面相容以配合課程工具。

## CLAUDE/代理補充指引（整合自 CLAUDE.md）
- 語言偏好：優先使用中文與用戶溝通，除非用戶明確要求英文。
- 教學/學習原則（不提供實作答案或完整解法）：
  - 解釋理論概念與需求
  - 分析測試案例的目的
  - 討論設計思路與權衡
  - 回答概念性問題
  - 幫助理解現有代碼結構
  - 僅在用戶主動請求時，為現有代碼添加解釋性註釋
- 測試與建構指令補充：
  - 套件/子目錄測試：`cd src/<lab> && go test -v`；或以 `-run <TestName>` 指定測試。
  - MapReduce 測試：`cd src/main && ./test-mr.sh`（或 `./test-mr.sh quiet`）。
  - MapReduce 建構：`cd src/main && go build -buildmode=plugin ../mrapps/<app>.go`、`go build mrcoordinator.go`、`go build mrworker.go`。
- 架構與相依補充：
  - RPC 框架：`labrpc/`；測試框架：`tester1/`；持久化：`labgob/`。
  - 實驗順序：1) MapReduce（`mr/`）→ 2) K/V 服務（`kvsrv1/`）→ 3) Raft（`raft1/`）→ 4) 複製 K/V（`kvraft1/`）→ 5) 分片 K/V（`shardkv1/`）。
  - 外部依賴：`github.com/anishathalye/porcupine` 用於線性化檢查（於相關測試中使用）。
