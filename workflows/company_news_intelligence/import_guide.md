# 工作流導入指南

此文檔說明如何將本地開發的 Company News Intelligence Workflow 導入 n8n。由於 GitHub 有文件大小限制，我們不直接上傳所有節點文件，而是提供結構描述和導入方法。

## 工作流節點結構

工作流包含以下 16 個節點，按執行順序排列：

1. **Webhook** (node01_webhook.json)
   - 提供 HTTP 入口點接收請求
   - 路徑: /company-news

2. **Input Preparation** (node02_input_preparation.json)
   - 驗證和處理輸入參數
   - 實現: Function 節點

3. **Initial Search** (node03_initial_search.json)
   - 執行初始搜索
   - 實現: HTTP Request 節點 (Tavily API)

4. **Result Filtering** (node04_result_filtering.json)
   - 過濾和評分搜索結果
   - 實現: Function 節點

5. **Topic Identification (AI Agent)** (node05_topic_identification.json)
   - 識別關鍵營運主題
   - 實現: OpenAI 節點 (GPT-4o)

6. **Topic Processing** (node06_topic_processing.json)
   - 處理 AI 識別的主題
   - 實現: Function 節點

7. **Split by Topics** (node07_split_by_topics.json)
   - 將工作流分為多個主題並行處理
   - 實現: Split In Batches 節點

8. **Deep Topic Search** (node08_deep_topic_search.json)
   - 對每個主題進行深度搜索
   - 實現: HTTP Request 節點 (Tavily API)

9. **Deep Result Processing** (node09_deep_result_processing.json)
   - 處理深度搜索結果
   - 實現: Function 節點

10. **Topic Analysis** (node10_topic_analysis.json)
    - 分析每個主題的深度信息
    - 實現: OpenAI 節點 (GPT-4o)

11. **Format Topic Result** (node11_format_topic_result.json)
    - 格式化主題分析結果
    - 實現: Function 節點

12. **Aggregate Topic Results** (node12_aggregate_topic_results.json)
    - 合併所有主題分析結果
    - 實現: Merge 節點

13. **Organize Results** (node13_organize_results.json)
    - 整理和結構化結果
    - 實現: Function 節點

14. **Generate Final Report (AI Agent)** (node14_generate_report.json)
    - 生成綜合分析報告
    - 實現: OpenAI 節點 (GPT-4o)

15. **Format Final Output** (node15_format_output.json)
    - 格式化最終輸出
    - 實現: Function 節點

16. **Webhook Response** (node16_webhook_response.json)
    - 返回 HTTP 響應
    - 實現: Respond to Webhook 節點

## 導入指南

### 方法一：從本地文件導入

1. 從本倉庫的 `resources` 目錄下載完整的節點定義文件
2. 創建本地目錄 `/path/to/nodes`
3. 將所有節點文件放入此目錄
4. 執行以下導入腳本：

```bash
#!/bin/bash
# 創建新工作流
WORKFLOW_ID=$(curl -s -X POST http://localhost:5678/rest/workflows \
  -H "X-N8N-API-KEY: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Company News Intelligence Workflow", "active": false}' | grep -o '"id":"[^"]*"' | grep -m 1 -o '[^"]*$')

echo "Created workflow with ID: $WORKFLOW_ID"

# 添加節點
NODES_DIR="/path/to/nodes"
for i in {1..16}; do
  NODE_FILE=$(printf "$NODES_DIR/node%02d_*.json" $i)
  echo "Adding node from $NODE_FILE"
  curl -s -X POST "http://localhost:5678/rest/workflows/$WORKFLOW_ID/nodes" \
    -H "X-N8N-API-KEY: YOUR_API_KEY" \
    -H "Content-Type: application/json" \
    -d @"$NODE_FILE"
done

# 添加連接
echo "Adding connections"
curl -s -X PATCH "http://localhost:5678/rest/workflows/$WORKFLOW_ID" \
  -H "X-N8N-API-KEY: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d @"$NODES_DIR/connections.json"

echo "Workflow import completed"
```

### 方法二：從頭創建

如果您想從頭創建工作流，可以參考以下步驟：

1. 在 n8n 界面創建新工作流
2. 參考 `metadata.json` 和 README 文檔中的結構說明添加節點
3. 每個節點的具體實現可參考相應的文檔說明
4. 按照 `connections.json` 中的定義連接節點

## 必要配置

1. **API 憑證**:
   - 需要配置 Tavily API 密鑰用於網絡搜索
   - 需要配置 OpenAI API 密鑰，且需要訪問 GPT-4o 模型

2. **節點參數調整**:
   - 對於 Initial Search 和 Deep Topic Search 節點，您可能需要根據實際情況調整搜索參數
   - 對於 OpenAI 節點，可以根據需求調整溫度和其他生成參數

## 測試工作流

導入完成後，可以使用以下 curl 命令測試工作流：

```bash
curl -X POST http://localhost:5678/webhook/company-news \
  -H "Content-Type: application/json" \
  -d '{
    "company": "Apple",
    "days": 30,
    "maxDepth": 2
  }'
```

## 故障排除

如果導入過程中遇到問題，請檢查：

1. n8n API 密鑰是否正確
2. 所有節點文件是否完整
3. 節點 ID 在 connections.json 中是否對應正確
4. 是否有所需的 API 密鑰配置

如需更多幫助，請參考 n8n 文檔或提交 issue。