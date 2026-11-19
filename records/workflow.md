# 戀愛行銷教學簡報製作工作流程說明書 (Workflow Execution Guide)

本文件依據 Antigravity 專案環境與 Agent 工具能力，提出最簡潔、穩健且可實際落地之工作流程規範與可行性評估，嚴格遵守真實性核實與下游鏈式重跑規則。

---

## 一、實作環境與可行性評估

### 1. 工具能力盤點
- **檔案讀寫工具 (`read_file`, `write_file`, `edit_file`)**：精準讀取 Wiki 知識庫、輸出 Markdown 草稿、管理檢查點報告與生成單一 HTML 簡報檔。
- **終端檢驗工具 (`run_command` with Python)**：可在沙盒內快速執行客製化 Lint 腳本（例如核算各頁漢字數量、驗證 HTML 標籤閉合性、檢查 Grounding 標籤覆蓋率），無需安裝任何外部厚重套件。
- **人機協同互動 (`USER_INPUT`, visible chat)**：在關鍵決策點（大綱審定、最終畫面驗收）由使用者把關，落實 Human-in-the-Loop。

### 2. 避免複雜框架的工程設計
- 本流程**不引入 LangGraph、CrewAI 或其他需要 pip 安裝外部伺服器的厚重框架**。
- 採用標準的**「狀態檔案驅動模式（File-driven State Machine）」**：以 `slides/state.json` 記錄當前節點狀態、重試次數與歷史軌跡，以 `slides/checkpoints/` 留存各節點查驗報告，結構透明、可審計且完全支援期末成果反思。

---

## 二、六大核心機制問答與實作說明

### 1. 調整後的 Node 1：以讀取、查詢與補足既有資料為主
- **既有資產保護**：本專案已擁有完整的 `wiki/` 知識庫與 11 張證據卡（`slides/evidence-cards.md`）。
- **嚴禁覆蓋**：Node 1 的任務是檢索 Wiki 內容以檢視既有卡片是否完備，**嚴禁直接重新生成或全面覆蓋 `evidence-cards.md`**！
- **提案補正**：若在後續檢查中發現卡片內容有疏漏或偏頗，僅針對該卡片提出修改建議（Patch Proposal）並進行局部增補。

### 2. 調整後的 Node 2：拒絕循環自證，直球對決原始 PDF 頁碼
- **杜絕自我驗證**：不得僅以「證據卡」與「AI 自行生成的檢查摘要」互相佐證。
- **直溯原始文獻**：重要案例事實、法條、行政處分及法律主張，必須逐字核對至 `raw/` 目錄中 7 份原始 PDF 的具體頁碼。
- **三層次嚴格區分**：核查紀錄中必須嚴格標明「原文引句（`> `）」、「Wiki 整理」與「AI 分析」。
- **誠實揭露**：凡在原始 PDF 中無法找到確切文字或尚存疑義者，一律標註 `【⚠️ 待人工確認】`，絕不自行補齊結論。

### 3. 調整後的 Node 7：明確區分「原始碼檢查」與「瀏覽器畫面檢查」
- **原始碼檢查（Agent 自動執行）**：檢查 HTML5 語法、DOM 標籤閉合、無外部 CDN 相依、原生 JavaScript 翻頁無報錯。
- **實際畫面檢查（人類使用者執行）**：因 Agent 無法直接「觀看」使用者本機瀏覽器的實際渲染畫面，**Agent 嚴禁宣稱「視覺畫面檢查通過」**！
- **交付檢查清單**：Agent 會產出 `node7-inspection-report.md`，列出待人工檢查之項目（如投影片切換流暢度、高低解析度版面自適應、文字對比舒適度、有無視覺截斷），由使用者親自開啟瀏覽器驗收（Gate 7）。

### 4. 完整的反饋迴圈：下游鏈式重跑（Cascading Downstream Re-execution）
- **杜絕跳步直達**：任何節點在退回修正後，都必須依序重新經過所有受影響的下游節點與檢驗關卡！
- **典型退迴路徑示範**：
  - *情境*：Gate 7 人工檢查發現第 4 頁文字在手機螢幕上過於擁擠 ➔ 退回 **Node 4** 進行草稿文案精簡 ➔ 通過 **Gate 4** 結構檢查 ➔ 進入 **Node 5** 重新跑文字量 Lint ➔ 通過 **Gate 5** ➔ 進入 **Node 6** 重新渲染 HTML ➔ 通過 **Gate 6** 原始碼檢查 ➔ 進入 **Node 7** 重新提報使用者驗收 ➔ 通過 **Gate 7** 完成。

### 5. 整理後的 Node 與 Gate 編號（1:1 精準對齊）
- **Node 1** ➔ **Gate 1: 內容涵蓋檢查 (`GATE_1_COVERAGE_CHECK`)**
- **Node 2** ➔ **Gate 2: 原始文獻核實閘門 (`GATE_2_SOURCE_VERIFICATION`)**
- **Node 3** ➔ **Gate 3: 大綱人工確認閘門 (`GATE_3_HUMAN_OUTLINE_APPROVAL`)** *(當前專案停駐點)*
- **Node 4** ➔ **Gate 4: 草稿結構檢查閘門 (`GATE_4_DRAFT_STRUCTURE_CHECK`)**
- **Node 5** ➔ **Gate 5: 法律文案與文字負載 Lint 閘門 (`GATE_5_LEGAL_AND_TEXT_LINT`)**
- **Node 6** ➔ **Gate 6: 原始碼完整性檢查閘門 (`GATE_6_CODE_INTEGRITY_CHECK`)**
- **Node 7** ➔ **Gate 7: 人工最終視覺驗收閘門 (`GATE_7_HUMAN_FINAL_APPROVAL`)**

### 6. 保留實際執行紀錄（真實可信、供期末作業反思）
- **即時記錄**：每次執行、檢查、失敗、退回與修正，均即時以 ISO 8601 時間戳記寫入 `slides/state.json` 之 `history` 陣列。
- **嚴禁事後虛構**：紀錄必須完全忠實呈現真實執行過程中的錯誤與修正，供期末報告做真實成果與反思分析之用。

---

## 三、預計產生的檔案清單與目錄結構

```
slides/
├── graph.md                  # 流程圖、7大節點規格與鏈式退回規則定義書（已更新）
├── workflow.md               # 本執行指南與機制說明（已更新）
├── state.json                # 狀態機即時追蹤檔（記錄節點、重試次數與真實歷史軌跡）
├── evidence-cards.md         # 11 張核心證據卡（既有資產，不予覆蓋）
├── outline.md                # 10 頁教學大綱規劃（既有資產，不予覆蓋）
├── draft.md                  # 10 頁每頁文案、情境對話與排版草稿（待 Node 4 產出）
├── presentation.html         # 最終成果：獨立單一離線可用之 HTML 簡報（待 Node 6 產出）
└── checkpoints/              # 各節點檢驗歷史報告存檔目錄
    ├── node2-evidence-check.md   # 原始 PDF 具體頁碼核查報告
    ├── node5-lint-report.md      # 草稿法律真實性與文字負載 Lint 報告
    └── node7-inspection-report.md# 原始碼檢查報告與待人工檢查清單
```

---

## 四、實際執行步驟指南

1. **Step 1（當前位置）**：停留在 **Gate 3（大綱人工確認閘門）**，等待使用者審閱並確認 10 頁教學大綱與本流程規範。
2. **Step 2**：在獲得使用者明確核可後，狀態機推進至 **Node 4**，產出完整的 10 頁白話文案與情境草稿 `slides/draft.md`，並通過 **Gate 4** 結構檢驗。
3. **Step 3**：推進至 **Node 5**，執行真實性約束與文字負載 Lint，產出 `node5-lint-report.md`。若有違規依 Loop 目標局部退回修改，並重跑下游檢驗，直到 **Gate 5** 通過。
4. **Step 4**：推進至 **Node 6**，將核可草稿編譯為單一純 HTML/CSS/JS 簡報 `slides/presentation.html`，並通過 **Gate 6** 程式碼檢查。
5. **Step 5**：推進至 **Node 7**，執行原始碼靜態分析，產出待人工檢查清單 `node7-inspection-report.md`，呈請使用者以瀏覽器開啟實際驗收，通過 **Gate 7** 全案交付。