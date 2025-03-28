# 貢獻指南

感謝您對 n8n 商業智能工作流集合的興趣！我們歡迎各種形式的貢獻，包括新工作流、功能改進、文檔更新和錯誤修復。

## 貢獻流程

1. Fork 這個倉庫
2. 創建您的功能分支 (`git checkout -b feature/amazing-feature`)
3. 提交您的更改 (`git commit -m '添加某個功能'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 提交 Pull Request

## 工作流貢獻指南

### 目錄結構

新工作流應遵循以下目錄結構：

```
workflows/
└── your_workflow_name/
    ├── README.md                # 工作流文檔
    ├── metadata.json            # 工作流元數據
    ├── node01_xxx.json          # 節點定義 
    ├── node02_xxx.json          # 按執行順序排序
    ├── connections.json         # 節點間連接
    └── docs/                    # 補充文檔（可選）
        ├── use_case.md          # 使用案例
        └── screenshots/         # 截圖
```

### 代碼風格

- 節點命名應遵循功能優先原則，如 `node01_webhook.json` 而非 `node01.json`
- 使用有意義的 ID 和變量名
- 每個節點應包含完整的文檔說明其目的和功能
- 添加適當的錯誤處理

### 文檔要求

每個工作流必須包含：

1. **基本README**：解釋工作流的目的、用法和配置
2. **元數據**：描述工作流的版本、作者、依賴和結構
3. **使用案例**：至少一個實際應用案例示例

## Pull Request 流程

1. 確保您的 PR 針對 `develop` 分支而非 `main`
2. 在 PR 描述中提供詳細說明
3. 請確保您的工作流在提交前已經過測試
4. 標記任何依賴或特殊需求

## 許可證

貢獻到此倉庫即表示您同意您的代碼將使用與此項目相同的 [MIT 許可證](../LICENSE)。
