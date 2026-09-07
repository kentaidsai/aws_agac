# AgentCore Lab — 從原型到生產：以 Amazon Bedrock AgentCore 打造客服智能體

本資料夾是《Building Agentic AI with Amazon Bedrock AgentCore》課程 Lab 1 的實作素材，示範如何用 **Strands Agents** 框架建立一個電子產品電商的客服智能體（Customer Support Agent），並逐步導入 Amazon Bedrock AgentCore 的三大能力：**Memory（記憶）**、**Gateway（工具閘道）** 與 **Identity（身分／OAuth 認證）**，讓一個本機原型演進為具備企業級記憶、工具整合與身分治理的智能體。

## 內容速覽

| 檔案 / 資料夾 | 說明 |
|---|---|
| `notebook.ipynb` | 主要教學筆記本（英文），依 Task 1.1 → 1.2 → 1.3 循序建置智能體 |
| `notebook_zh_cn.ipynb` | 同一份筆記本的簡體中文版本，內容與結構（34 個 cell）完全對應 |
| `lab_helpers/lab1_strands_agent.py` | 智能體的本地工具與 System Prompt 定義 |
| `lab_helpers/utils.py` | 建置與清理 AWS 資源的輔助函式（SSM、Secrets Manager、Cognito、IAM、AgentCore 各服務） |
| `scripts/` | 一組獨立可執行的 CLI 腳本，用於建立／刪除 AgentCore Gateway、Memory、憑證提供者、Runtime 等雲端資源 |
| `requirements.txt` | Python 套件相依清單 |

## 筆記本學習流程（notebook.ipynb / notebook_zh_cn.ipynb）

筆記本以「持續增強同一個智能體」的方式編排，共三個任務：

**Task 1.1　基礎設置與基本智能體建立**
- 安裝依賴、載入 AWS SDK / AgentCore / Strands 相關套件
- 檢視 `lab_helpers/lab1_strands_agent.py` 中以 `@tool` 定義的四個本地工具：
  - `get_product_info()`：取得產品技術規格
  - `get_return_policy()`：取得退換貨政策
  - `get_technical_support()`：查詢技術支援知識庫（透過 Bedrock Knowledge Base 檢索）
  - `web_search()`：以 DuckDuckGo（`ddgs`）進行網路搜尋
- 用 `BedrockModel`（`us.amazon.nova-pro-v1:0`）+ 上述工具建立最基礎的 `Agent`，並測試一則退貨政策詢問
- 說明此原型的三個限制：沒有持久記憶、只有本地工具、沒有身分管理 —— 作為後續兩個任務的動機

**Task 1.2　以 AgentCore Memory 強化記憶**
- 情境：客戶三週前聯繫過客服，若智能體沒有跨會話記憶，客戶得重複描述一次自己的狀況
- 建立/取得 AgentCore Memory 資源，設定三種記憶策略：語意事實擷取（fact extractor）、對話摘要（conversation summary）、使用者偏好（user preference）
- 灌入一段模擬的歷史客服對話（`customer_001`），讓 AgentCore Memory 自動萃取成長期客戶洞察
- 透過 Strands 的 Hook 機制（`CustomerSupportMemoryHooks`）在每次對話前自動撈取客戶情境、對話後自動寫回記憶，無需手動介入
- 測試記憶增強後的智能體是否能回答「我的筆電偏好是什麼？」這類需要跨會話記憶的問題

**Task 1.3　以 Gateway 與 Identity 擴展工具規模**
- 說明企業導入智能體常見痛點：工具/API 難以安全、可規模化地共用
- 介紹 AgentCore Gateway 如何以 **MCP（Model Context Protocol）** 統一暴露既有 Lambda 函式為智能體工具
- 說明 Gateway 的 OAuth2 / JWT 認證機制：Amazon Cognito 簽發 token、Gateway 驗證 token 是否來自允許的 client
- 依序建立 Gateway → 掛載 Lambda 目標（`check_warranty_status`、`web_search` 兩個工具，由同一支 Lambda 依工具名稱路由）→ 以 Cognito 憑證換取 OAuth token → 用 Strands 的 `MCPClient` 連上 Gateway
- 將本地工具（product info / return policy / technical support）與 Gateway 工具（warranty check / web search）合併，建立「本地 + 集中式」的混合工具架構
- 測試整合後的智能體：查保固、網路搜尋、以及「記憶 + Gateway」的複合情境
- 收尾提示：完成本筆記本後，回到 Lab 操作手冊，進入 Task 2 於 AgentCore 主控台檢視已建立的資源

兩份筆記本內容一致，`notebook_zh_cn.ipynb` 供中文學習者使用；程式碼儲存格完全相同，僅 Markdown 說明文字（含架構圖檔名 `_zh_cn` 版本）已翻譯。

## `lab_helpers/`：智能體工具與資源輔助函式

- **`lab1_strands_agent.py`**：定義 `SYSTEM_PROMPT`（電子產品客服人設）與四個 `@tool` 函式。其中 `get_return_policy()` / `get_product_info()` 使用寫死在程式中的模擬資料庫（模擬真實情境會查詢的政策/產品資料庫），`get_technical_support()` 則會從 SSM 取得 Knowledge Base ID 並呼叫 Strands 的 `retrieve` 工具做 RAG 檢索。
- **`utils.py`**：貫穿整個 Lab 使用的工具箱，可分為四類：
  - **設定存取**：`get_ssm_parameter` / `put_ssm_parameter` / `delete_ssm_parameter`、`read_config`（自動判斷 JSON/YAML）、`load_api_spec`
  - **身分/認證建置**：`setup_cognito_user_pool`（建立 User Pool、App Client、測試使用者並取得 Bearer Token）、`reauthenticate_user`、`get_cognito_client_secret`、Secrets Manager 存取（`save/get/delete_customer_support_secret`）
  - **IAM 角色建置**：`create_agentcore_runtime_execution_role`（建立 AgentCore Runtime 執行角色，含 ECR 拉取映像檔、CloudWatch Logs 權限與信任關係）、`delete_agentcore_runtime_execution_role`
  - **資源清理**：`cleanup_cognito_resources`、`agentcore_memory_cleanup`、`gateway_target_cleanup`、`runtime_resource_cleanup`（含刪除對應的 ECR repository）、`delete_observability_resources`（刪除 CloudWatch 日誌群組/串流）、`local_file_cleanup`（清除本地產生的 `Dockerfile`、`.bedrock_agentcore.yaml` 等部署殘留檔）

## `scripts/`：AgentCore 資源管理 CLI

每支腳本都可獨立以命令列執行（多數用 `click` 包裝成子命令），共用 `scripts/utils.py`（與 `lab_helpers/utils.py` 功能重疊的精簡版）：

- **`agentcore_agent_runtime.py`**：依名稱搜尋並刪除指定的 AgentCore Agent Runtime，支援 `--dry-run` 預覽
- **`agentcore_gateway.py`**（`create` / `delete`）：建立 Gateway（含 JWT 授權設定、Lambda 目標、把 Gateway ID/URL/ARN 等寫回 SSM）或刪除 Gateway 與其所有 target
- **`agentcore_memory.py`**（`create` / `delete`）：建立/刪除 AgentCore Memory 資源（含語意/摘要/使用者偏好三種策略），記憶體 ID 存放於 SSM
- **`cognito_credentials_provider.py`**（`create` / `delete` / `list`）：管理 AgentCore Identity 的 Cognito OAuth2 憑證提供者
- **`cleanup.sh`**：刪除課程用的 CloudFormation stacks（Infra + Cognito）與 S3 bucket，互動式確認後執行
- **`prereq.sh` / `prereq.ps1`**：前置作業腳本（Bash / PowerShell 兩版），建立 S3 bucket、打包並上傳 Lambda 程式碼與 layer、部署 `prerequisite/infrastructure.yaml` 與 `prerequisite/cognito.yaml` 兩份 CloudFormation 樣板
- **`list_ssm_parameters.sh`**：列出 `/app/customersupport` 命名空間下所有 SSM 參數（含解密值），方便除錯

> `prereq.sh` / `prereq.ps1` 會參照 `prerequisite/lambda/...`、`prerequisite/infrastructure.yaml`、`prerequisite/cognito.yaml`，但目前這個資料夾中沒有 `prerequisite/` 子目錄，代表這些前置資源檔案存放在課程的其他部分（尚未同步到此資料夾），執行前需先取得完整專案結構。同樣地，筆記本引用的 `images/` 架構示意圖資料夾在此目錄中也不存在。

## 依賴套件（`requirements.txt`）

`strands-agents`、`strands-agents-tools`、`boto3` / `botocore`、`bedrock-agentcore` 與 `bedrock-agentcore-starter-toolkit`、`aws-opentelemetry-distro`（可觀測性）、`ddgs`（網路搜尋）、`pyyaml`、`typing_extensions`。

## 建議操作順序

1. 安裝 `requirements.txt` 依賴，並確認已設定好可用的 AWS 憑證（預設 region 為 `us-east-1`）
2. 若尚未部署前置資源，先取得完整的 `prerequisite/` 目錄後執行 `scripts/prereq.sh`（或 `.ps1`）建立 S3、Lambda、Cognito、Infra 等基礎設施
3. 依序開啟並執行 `notebook.ipynb`（或中文版 `notebook_zh_cn.ipynb`），完成 Task 1.1 → 1.2 → 1.3
4. 練習/課程結束後，可用 `scripts/cleanup.sh`、`scripts/agentcore_*.py delete`、或 `lab_helpers/utils.py` 內的 `*_cleanup()` 系列函式回收所有雲端資源，避免產生額外費用
