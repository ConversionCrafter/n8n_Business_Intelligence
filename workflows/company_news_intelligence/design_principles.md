# 應用 Nate Herk 設計原則的 n8n 工作流設計

本文檔說明我們如何將 Nate Herk 的 AI 工作流設計原則應用於 Company News Intelligence Workflow 的設計中。

## Nate Herk 的核心設計原則回顧

1. **確定性 vs 非確定性流程**：在確定性流程中使用線性工作流，僅在真正需要決策的地方使用 AI Agent
2. **線性工作流的四大優勢**：
   - 可靠性與一致性
   - 成本效率
   - 易於除錯和維護
   - 可擴展性
3. **RAG 實現不必綁定 Agent**：向量搜索可以作為普通節點實現

## 我們的實施方式

### 1. 混合架構設計

我們的工作流採用了"混合式架構"，明確區分了需要 AI 決策的環節和可以線性處理的環節：

#### AI Agent 節點（非確定性流程）
- **Topic Identification (AI Agent)**：需要複雜的業務判斷來識別重要營運主題
- **Topic Analysis**：需要深度理解和提取營運洞見
- **Generate Final Report (AI Agent)**：需要綜合分析能力生成執行層級報告

#### 線性工作流節點（確定性流程）
- **Input Preparation**：參數驗證和預處理
- **Initial Search**：搜索請求構建和執行
- **Result Filtering**：基於明確規則的過濾和評分
- **Topic Processing**：AI 輸出的結構化和處理
- **Split by Topics**：工作流分流
- **Deep Topic Search**：構建和執行深度搜索
- **Deep Result Processing**：過濾和處理深度搜索結果
- **Format Topic Result**：標準化輸出格式
- **Aggregate Topic Results**：結果合併
- **Organize Results**：最終數據結構整理
- **Format Final Output**：輸出格式化
- **Webhook Response**：返回HTTP響應

### 2. 提高可靠性與一致性

- **明確的數據流**：每個節點都有明確定義的輸入和輸出格式
- **健壯的錯誤處理**：每個處理節點都包含錯誤檢查和異常處理
- **標準化數據結構**：使用統一的數據格式在節點間傳遞
- **參數驗證**：在流程最開始進行完整的參數驗證

### 3. 優化成本效率

- **減少 LLM API 調用**：僅在三個關鍵決策點使用 OpenAI API
- **預處理和過濾**：在調用 AI 前進行數據過濾和整理，減少處理的數據量
- **並行處理**：對多個主題同時進行分析，減少總處理時間
- **結果緩存機制**：在 Result Filtering 節點實現基本的結果評分和過濾

### 4. 提高可維護性

- **模塊化設計**：工作流分為清晰的階段，每個階段有明確職責
- **詳細文檔**：每個節點都有完整的目的、輸入、輸出和設計考量文檔
- **命名規範**：使用清晰、一致的命名約定
- **節點獨立性**：節點設計為獨立單元，便於單獨測試和更換

### 5. 確保可擴展性

- **參數化設計**：關鍵參數（如公司、時間範圍、研究深度）可由用戶控制
- **可替換組件**：搜索服務可以輕鬆替換為其他提供商
- **水平擴展**：可以輕鬆添加更多主題分析或數據源
- **垂直擴展**：可以增加分析深度或添加新的分析維度

## 具體優化示例

### 1. 使用線性節點處理搜索結果過濾

傳統 RAG 方法可能會讓 AI 決定哪些結果相關。我們的方法是：

```javascript
// 在 Result Filtering 節點中使用確定性算法
const scoredResults = filteredResults.map(result => {
  // Basic scoring algorithm
  let score = 0;
  
  // Title relevance (higher weight)
  if (result.title?.toLowerCase().includes(companyLower)) {
    score += 5;
  }
  
  // Content relevance
  if (result.content?.toLowerCase().includes(companyLower)) {
    score += 3;
  }
  
  // 更多確定性評分規則...
});
```

這比讓 AI 評估相關性更一致、更高效。

### 2. 主題分割和並行處理

我們使用 n8n 的 Split 節點替代讓 AI 決定如何處理多個主題：

```
Topic Processing → Split by Topics → Deep Topic Search (並行執行)
```

這實現了確定性的並行處理，比讓 AI 管理複雜的多主題處理更可靠。

### 3. 標準化 AI 輸出處理

在每個 AI 節點後，我們添加了專門的函數節點處理輸出：

```javascript
// 在 Format Topic Result 節點中處理 AI 輸出
try {
  // Handle both string and object responses
  if (typeof $input.json.response === 'string') {
    analysis = JSON.parse($input.json.response);
  } else {
    analysis = $input.json.response;
  }
} catch (error) {
  throw new Error(`Failed to parse analysis: ${error.message}`);
}
```

這確保了即使 AI 回應格式有變化，後續節點也能接收到一致的數據格式。

## 與傳統 Agent 設計的對比

如果使用傳統的全 Agent 工作流設計，結構可能是：

```
Webhook → AI Agent (決定如何搜索) → Search Tool → 
AI Agent (分析結果和決定主題) → Search Tool (針對每個主題) → 
AI Agent (分析深度結果) → AI Agent (生成報告)
```

這種方法的問題：
1. **更多 LLM API 調用**：每次決策都需要調用 LLM
2. **隱含的流程控制**：流程邏輯隱藏在 AI 提示詞中，難以調試
3. **不一致性風險**：同樣的輸入可能產生不同的決策路徑
4. **擴展性挑戰**：添加新功能需要重寫複雜的系統提示詞

## 結論

通過應用 Nate Herk 的設計原則，我們創建了一個既利用 AI 強大分析能力，又保持高可靠性和效率的混合工作流。關鍵是識別哪些部分真正需要 AI 的判斷力，哪些部分可以使用線性、確定性的處理邏輯。

這種方法不僅提高了系統的可靠性和效率，還降低了 API 成本，並使工作流更容易維護和擴展。對於生產環境中的 AI 系統，這種混合架構提供了最佳的平衡點。