# log.md - AI 輔助設計過程紀錄

## 專案
Homework 4 - AI Harness Systems Design and Analysis

## 選定情境
教育助理 AI Harness

---

## 迭代 1：初始框架定義

### Prompt / 討論重點
- 定義一個適合系統設計的實用 AI 應用情境。
- 確保重點放在 orchestration，而非模型訓練。

### 初始設計決策
1. 選定領域：高等教育場域的教育助理。
2. 選定核心任務：
   - 概念問答（concept Q&A）
   - 考前讀書計畫生成
   - 作業形成性回饋

### 為何採用此方向
- 具高度真實情境需求。
- 天然需要多步驟 workflow 與 tool calling。
- 容易建立可量化的 evaluation 指標。

---

## 迭代 2：架構精煉

### Prompt / 討論重點
- 建立包含 LLM + tools + memory 的整體架構。
- 釐清各元件責任邊界。

### 架構調整
1. 在 UI 與 LLM 之間加入專責 **Orchestrator**，負責 workflow 控制。
2. 將 memory 拆分為短期記憶與長期記憶。
3. 新增 evaluation logger 作為獨立模組。

### 發現問題
- 早期架構將 LLM 視為直接執行者，導致控制邊界不清。

### 修正方式
- 重新定義 LLM 為 system controller，負責規劃與推理。
- 將 retry、fallback、branching 邏輯移至 orchestrator。

---

## 迭代 3：工具集合與函式契約

### Prompt / 討論重點
- 定義至少 3 個具清楚 input/output 的工具。

### 工具演進
1. `retrieve_course_knowledge`
2. `analyze_student_progress`
3. `generate_study_schedule`
4. `assignment_rubric_checker`（為補齊作業回饋 workflow 而新增）

### 設計決策
- 使用 schema 約束輸入與輸出，降低語意歧義。
- 統一 error code 與 timeout 行為。

### 發現問題
- 缺乏型別化輸出契約時，tool chaining 會產生欄位不一致問題。

### 修正方式
- 為每個工具加入嚴格 output schema，並在 LLM synthesis 前加入 normalization layer。

---

## 迭代 4：Workflow 與 Orchestration 邏輯

### Prompt / 討論重點
- 建立多步驟 agent workflows 與關鍵決策點。

### Workflow 設計決策
1. 概念問答流程：先 retrieval，再 synthesis。
2. 考前規劃流程：先學習進度分析，再產生讀書計畫。
3. 作業回饋流程：先 rubric 檢查，若落差過大則先澄清需求。

### Orchestration 策略
- 以 rule-based 方式對低信心/高風險回覆強制 retrieval。
- 依意圖類型進行條件分支。
- Fallback 路徑：retry -> alternative tool -> 詢問澄清問題。

### 發現問題
- 單一路徑的線性 workflow 無法處理使用者缺漏條件。

### 修正方式
- 在 tool execution 前插入 data-gathering/clarification 狀態。

---

## 迭代 5：Evaluation 設計

### Prompt / 討論重點
- 定義可衡量系統有效性的 evaluation 方法。

### Evaluation 決策
1. 離線指標：grounding score、rubric alignment、workflow success。
2. 線上指標：user satisfaction、completion time、re-ask rate。
3. 建立整合多面向的複合品質指標。

### 發現問題
- 初版指標過度偏重回覆品質，未涵蓋 workflow 可靠性。

### 修正方式
- 新增 tool success ratio 與 workflow completion rate。

---

## 最終設計摘要

1. 系統被設計為可控的 AI harness，而非單純聊天機器人。
2. LLM 負責意圖推理與回覆生成。
3. Orchestrator 負責執行策略、分支、重試與狀態轉移。
4. 四個工具共同支援檢索、診斷、規劃與 rubric 回饋。
5. Memory 在隱私前提下支援個人化。
6. Evaluation 同時涵蓋品質面與運維面指標。

---

## 反思

### 成效良好的部分
- 將推理（LLM）與控制（orchestrator）分離後，可解釋性明顯提升。
- 明確的工具契約使 workflow 更穩定且可稽核。

### 可再改進之處
- 增加長期實驗（A/B testing）驗證 orchestration 策略。
- 擴充面向教師端的學習分析儀表板。

### 關鍵學習
AI Harness 的設計品質，往往更取決於 orchestration、工具契約與 evaluation 迴圈，而非模型規模本身。
