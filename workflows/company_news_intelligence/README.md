# 公司新聞智能分析工作流

## 概述

這個 n8n 工作流能夠自動分析公司的重大營運新聞，識別關鍵主題，進行深度研究，並生成詳細的智能報告。工作流採用混合式架構，結合線性處理流程和 AI 代理，實現高效、可靠的情報分析。

## 架構特點

本工作流基於 Nate Herk 的 AI 工作流設計原則建構，具有以下特點：

1. **混合式架構**：在確定性流程中使用線性工作流，在需要複雜決策的地方使用 AI Agent
2. **專注於營運影響**：分析重點放在公司的實際營運表現，而非一般新聞
3. **多層次過濾**：結果經過多重過濾和評分，確保資訊質量
4. **平行處理**：對多個主題同時進行深度研究，提高效率
5. **結構化輸出**：生成格式統一、重點明確的執行層級報告

## 工作流程

1. **輸入處理**：接收公司名稱和時間範圍
2. **初步搜索**：獲取公司相關新聞並進行相關性過濾
3. **主題識別**：AI 識別需要深入研究的關鍵營運主題
4. **深度研究**：對每個主題進行專門的深度搜索和分析
5. **情報整合**：將所有主題分析整合為全面的情報報告

## 使用方法

### 導入工作流

1. 在 n8n 中創建新的工作流
2. 使用以下命令從文件系統導入節點：

```bash
# 首先創建工作流
curl -X POST http://localhost:5678/rest/workflows \
  -H "X-N8N-API-KEY: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Company News Intelligence Workflow", "active": false}'

# 獲取工作流 ID
WORKFLOW_ID=$(curl -s http://localhost:5678/rest/workflows \
  -H "X-N8N-API-KEY: YOUR_API_KEY" | grep -o '"id":"[^"]*"' | grep -m 1 -o '[^"]*$')

# 然後逐個添加節點
for i in {1..16}; do
  NODE_FILE=$(printf "./node%02d_*.json" $i)
  NODE_DATA=$(cat $NODE_FILE)
  curl -X POST http://localhost:5678/rest/workflows/$WORKFLOW_ID/nodes \
    -H "X-N8N-API-KEY: YOUR_API_KEY" \
    -H "Content-Type: application/json" \
    -d "$NODE_DATA"
done

# 最後添加連接
CONNECTIONS=$(cat ./connections.json)
curl -X PATCH http://localhost:5678/rest/workflows/$WORKFLOW_ID \
  -H "X-N8N-API-KEY: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$CONNECTIONS"
```

### 調用工作流

向 Webhook 節點發送 POST 請求：

```bash
curl -X POST http://localhost:5678/webhook/company-news \
  -H "Content-Type: application/json" \
  -d '{
    "company": "Apple",
    "days": 30,
    "maxDepth": 2
  }'
```

## 參數說明

- **company**：要研究的公司名稱（必填）
- **days**：向後查詢的天數（必填）
- **maxDepth**：研究深度 1-3（可選，預設為 2）

## 輸出格式

```json
{
  "report": "完整的分析報告（Markdown 格式）",
  "metadata": {
    "company": "公司名稱",
    "timeframe": "分析時間範圍",
    "analysisDate": "分析完成時間",
    "topicCount": "分析的主題數量"
  }
}
```

## API 需求

- **Tavily API**：用於網絡搜尋
- **OpenAI API**：用於 AI 分析（需要 gpt-4o 訪問權限）

## 優化建議

1. 添加緩存機制避免重複搜索
2. 實施錯誤重試邏輯處理 API 失敗
3. 根據公司行業添加更多特定領域的過濾邏輯
4. 擴展支持文件上傳（如年報、新聞稿）作為額外信息源

## 設計理念

參閱 [design_principles.md](./design_principles.md) 了解本工作流如何應用 Nate Herk 的 AI 工作流設計原則。

## 使用案例

查看 [use_case.md](./use_case.md) 獲取實際應用案例和示例報告。

## 版本

v1.0.0 - 2025年3月28日初始版本