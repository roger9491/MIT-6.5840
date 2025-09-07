# CLAUDE.md

此文件為 Claude Code (claude.ai/code) 在此代碼庫中工作時提供指導。

**語言偏好**: 請優先使用中文與用戶溝通，除非用戶明確要求使用英文。

**學習原則**: 這是學習用的課程代碼庫。不要提供實作答案或完整解決方案。只能：
- 解釋理論概念和要求
- 分析測試案例的目的
- 討論設計思路和權衡
- 回答概念性問題
- 幫助理解現有代碼結構
- 當用戶主動請求時，可以為現有代碼添加解釋性註釋以幫助理解

## 概述

這是 MIT 6.5840 (分散式系統) 課程實驗，使用 Go 語言實現分散式系統概念。代碼庫包含 MapReduce、Raft 共識、鍵值服務器和分片系統的實現。

## 指令

### 測試
- `cd src/<lab>` 然後 `go test -v` - 執行特定實驗的所有測試
- `cd src/<lab>` 然後 `go test -v -run <TestName>` - 執行特定測試
- `cd src/<lab>` 然後 `go test -v -run Reliable` - 執行可靠網絡測試
- `cd src/main` 然後 `./test-mr.sh` - 執行 MapReduce 測試
- `cd src/main` 然後 `./test-mr.sh quiet` - 安靜模式執行 MapReduce 測試

### 建構
- `cd src/main` 然後 `go build -buildmode=plugin ../mrapps/<app>.go` - 將 MapReduce 應用建構為插件
- `cd src/main` 然後 `go build mrcoordinator.go` - 建構 MapReduce 協調器
- `cd src/main` 然後 `go build mrworker.go` - 建構 MapReduce 工作者

### 實驗提交
- `make <lab>` - 創建提交壓縮檔 (例如 `make lab1`, `make lab2`)

## 架構

### 實驗結構
- **kvsrv1/**: 單機鍵值服務器，具有至多一次語義和版本控制
- **kvraft1/**: 使用 Raft 共識的複製鍵值服務
- **raft1/**: Raft 共識算法實現
- **mr/**: MapReduce 框架，包含協調器和工作者節點
- **shardkv1/**: 具有動態重配置的分片鍵值服務
- **mrapps/**: MapReduce 應用程式 (字數統計、索引器等)

### 核心組件
- **RPC 框架**: `labrpc/` 中的自定義 RPC 系統，用於模擬網絡故障
- **測試框架**: `tester1/` 中的綜合測試套件，具有故障注入功能
- **持久化**: `labgob/` 中的狀態持久化工具

### 實驗依賴關係
實驗之間相互依賴：
1. **實驗 1**: MapReduce (mr/)
2. **實驗 2**: 鍵值服務器 (kvsrv1/)
3. **實驗 3**: Raft (raft1/)
4. **實驗 4**: 容錯鍵值服務 (kvraft1/)
5. **實驗 5**: 分片鍵值服務 (shardkv1/)

### 測試結構
- 每個實驗都有自己的測試文件 (`*_test.go`)
- 測試使用自定義 `tester1/` 框架來模擬網絡分區和故障
- 可靠測試假設無網絡故障；不可靠測試包含故障

### 模組結構
- Go 模組名稱: `6.5840`
- 外部依賴: `github.com/anishathalye/porcupine` 用於線性化檢查