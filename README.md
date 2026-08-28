# AgentCore 課程摘要卡

課程「Building Agentic AI with Amazon Bedrock AgentCore」(200-MLAGAC-10-EN)全部八個模組(M00–M07)的視覺化重點摘要,每個模組一張單頁 HTML 摘要卡,並以 `index.html` 作為索引首頁。

## 使用方式

1. 開啟 `index.html`(直接雙擊或用瀏覽器開啟)。
2. 點選任一模組卡片,會在新的瀏覽器視窗 / 分頁開啟該模組完整的摘要卡。
3. 所有頁面皆支援亮色 / 暗色主題,會依系統設定自動切換。

## 檔案列表

| 檔案 | 模組 | 內容重點 |
| --- | --- | --- |
| `index.html` | — | 八個模組的索引首頁 |
| `m00-course-intro.html` | M00 Introduction | 課程目標與議程、課堂規範與虛擬教室功能、先備知識、學員自我介紹、課程資源與實驗環境存取 |
| `m01-foundations.html` | M01 Foundations of Agentic AI Patterns | 傳統 AI vs. Agentic AI、Agent 五大核心元件、Prototype-to-Production 鴻溝、AgentCore 九大服務全覽、客服代理人範例 |
| `m02-runtime.html` | M02 AgentCore Runtime and Framework Integration | 開源框架比較(Strands、CrewAI、LangGraph、LlamaIndex)、Runtime 核心特色、Session 隔離、容器化 vs. 直接程式碼部署 |
| `m03-security-identity.html` | M03 Security and Identity Management | 入站 / 出站驗證、SigV4 / OAuth 2LO / OAuth 3LO 三種授權模式、AgentCore Identity 關鍵能力、企業安全考量 |
| `m04-tools-gateway.html` | M04 Tool Integration and AgentCore Gateway | 內建工具(Browser、Code Interpreter)、Model Context Protocol(MCP)、AgentCore Gateway 架構與驗證、AgentCore Policy(Cedar) |
| `m05-memory.html` | M05 Agentic Memory Implementation | 短期 / 長期記憶、五種記憶萃取策略、記憶生命週期、Namespace 細緻存取控制 |
| `m06-deployment-observability.html` | M06 Production Monitoring and Observability | Session/Trace/Span 階層、CloudWatch GenAI Observability 檢視層級、AgentCore Evaluations 三層級評估 |
| `m07-wrap-up.html` | M07 Course Wrap-Up | 七大模組回顧、課程目標檢核、AWS Skill Builder 學習資源、AWS 認證等級總覽、考照準備四步驟 |
| `README.md` | — | 本說明文件 |

## 設計說明

所有頁面共用同一套視覺系統,方便並排閱讀與日後擴充:

- **字體**:IBM Plex Sans Condensed(標題)、IBM Plex Sans(內文)、IBM Plex Mono(代碼 / 標籤)
- **色彩**:深藍(Navy)搭配 AWS 橘作為主色調,輔以青色(Teal)區分「記憶與洞察」類服務
- **版型**:深色 Hero 區塊呈現模組標題與學習目標,內文區塊以卡片 / 網格呈現各主題重點,底部深色 Footer 收錄知識驗證(Knowledge Check)與模組總結

## 內容來源

摘要內容整理自課程講師手冊簡報檔:

- `MLAGAC-10-EN-M00-CourseIntro_InstructorDeck.pptx`
- `MLAGAC-10-EN-M01-Foundations_InstructorDeck.pptx`
- `MLAGAC-10-EN-M02-Runtime_InstructorDeck.pptx`
- `MLAGAC-10-EN-M03-SecurityAndIdentity_InstructorDeck.pptx`
- `MLAGAC-10-EN-M04-ToolsAndGateway_InstructorDeck.pptx`
- `MLAGAC-10-EN-M05-Memory_InstructorDeck.pptx`
- `MLAGAC-10-EN-M06-DeploymentObservablity_InstructorDeck.pptx`
- `MLAGAC-10-EN-M07-WrapUp_InstructorDeck.pptx`
