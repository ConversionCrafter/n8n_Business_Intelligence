# 工作流程圖

下面是 Company News Intelligence Workflow 的完整流程圖，展示了各節點之間的關係和數據流向。

```mermaid
graph TD
    Start[開始] --> A[Webhook]
    A --> B[Input Preparation]
    B --> C[Initial Search]
    C --> D[Result Filtering]
    D --> E[Topic Identification<br/>AI Agent]
    E --> F[Topic Processing]
    F --> G[Split by Topics]
    
    %% 平行處理分支
    G --> H1[Deep Topic Search<br/>主題 1]
    G --> H2[Deep Topic Search<br/>主題 2]
    G --> H3[Deep Topic Search<br/>主題 3]
    G --> Hn[Deep Topic Search<br/>主題 n]
    
    H1 --> I1[Deep Result Processing]
    H2 --> I2[Deep Result Processing]
    H3 --> I3[Deep Result Processing]
    Hn --> In[Deep Result Processing]
    
    I1 --> J1[Topic Analysis<br/>AI Agent]
    I2 --> J2[Topic Analysis<br/>AI Agent]
    I3 --> J3[Topic Analysis<br/>AI Agent]
    In --> Jn[Topic Analysis<br/>AI Agent]
    
    J1 --> K1[Format Topic Result]
    J2 --> K2[Format Topic Result]
    J3 --> K3[Format Topic Result]
    Jn --> Kn[Format Topic Result]
    
    %% 合併平行分支
    K1 --> L[Aggregate Topic Results]
    K2 --> L
    K3 --> L
    Kn --> L
    
    L --> M[Organize Results]
    M --> N[Generate Final Report<br/>AI Agent]
    N --> O[Format Final Output]
    O --> P[Webhook Response]
    P --> End[結束]
    
    %% 節點類型標記
    classDef webhook fill:#f9d,stroke:#333,stroke-width:2px;
    classDef function fill:#bbf,stroke:#33f,stroke-width:1px;
    classDef http fill:#bfb,stroke:#3f3,stroke-width:1px;
    classDef ai fill:#fbf,stroke:#f3f,stroke-width:2px;
    classDef split fill:#fdb,stroke:#f60,stroke-width:1px;
    classDef merge fill:#bdf,stroke:#06f,stroke-width:1px;
    
    class A,P webhook;
    class B,D,F,I1,I2,I3,In,K1,K2,K3,Kn,M,O function;
    class C,H1,H2,H3,Hn http;
    class E,J1,J2,J3,Jn,N ai;
    class G split;
    class L merge;
```

## 工作流階段說明

工作流分為五個主要階段：

### 1. 輸入處理

```
Webhook → Input Preparation
```

這個階段接收 HTTP 請求，驗證和處理輸入參數，準備後續搜索。

### 2. 初步搜索

```
Initial Search → Result Filtering
```

執行初步的網絡搜索，並對結果進行過濾和評分，確保只保留相關性高的內容。

### 3. 主題識別

```
Topic Identification → Topic Processing
```

使用 AI 分析搜索結果，識別出關鍵的營運主題，並處理這些主題以準備深度研究。

### 4. 深度研究

```
Split by Topics → Deep Topic Search → Deep Result Processing → Topic Analysis → Format Topic Result
```

這個階段將工作流分割為多個平行分支，每個分支專注於一個特定主題的深入研究。對每個主題：
1. 執行專門的深度搜索
2. 處理和過濾這些深度搜索結果
3. 使用 AI 分析提取營運洞見
4. 格式化分析結果

### 5. 情報整合

```
Aggregate Topic Results → Organize Results → Generate Final Report → Format Final Output → Webhook Response
```

這個階段將所有主題的研究結果合併起來，進行整體分析，生成綜合報告，並返回給客戶端。

## 節點類型說明

工作流使用以下類型的節點：

- **Webhook 節點** (紫紅色): 處理 HTTP 請求和響應
- **Function 節點** (藍色): 執行 JavaScript 代碼進行數據處理
- **HTTP Request 節點** (綠色): 調用外部 API 進行搜索
- **AI 節點** (紫色): 使用 OpenAI 進行複雜分析和決策
- **Split/Merge 節點** (橙/淺藍色): 處理工作流的分流和合流

## 混合架構說明

工作流採用混合架構，結合線性處理和 AI 決策：

- **線性確定性流程** (藍色和綠色節點): 用於數據處理、過濾、格式化等確定性任務
- **AI 決策流程** (紫色節點): 用於需要複雜決策和分析的任務

這種混合架構確保了工作流的高效性和靈活性，適用於複雜的情報分析任務。