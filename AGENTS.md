# AGENTS.md

> **歸屬聲明**：Herdr（`herdr-core`、`herdr-app`）為 [herdrdev](https://github.com/herdrdev) 所有。本專案（`herdr-core-skills`）為獨立開源專案，僅基於 `herdr-core`／`herdr-app` 指令建立 Agent 技能、規格書與操作規則，非官方專案。

本文件分為兩部分：
- **Part A**：Agent 操作 `herdr-core`／`herdr-app` 時必須遵守的執行規則。
- **Part B**：維護與貢獻 `herdr-core-skills` 儲存庫的開發規則。

---

## Part A：操作 `herdr-core` 與 `herdr-app` 的執行規則

### A1. 環境探索與事實確認（唯讀優先）
1. 先確認本機實際可用的可執行檔（`hc`、`herdr-core`、`herdr-app`）與版本；不硬編安裝路徑。若 PATH 上僅有 `herdr-core`，先以 `--help` 確認為新版 CLI 再使用。
2. 首次操作 core 讀 `<core-cli> --skill`；首次操作 GUI 讀 `herdr-app --skill`。以已安裝版本的 `--help`／`commands`／`schema` 為最終依據，不套用舊版 `herdr` TUI 或 `sheprd` 指令。
3. 區分 `$HC_TERMINAL_ID`（PowerShell：`$env:HC_TERMINAL_ID`）與 GUI pane ID；沒有該環境變數時，不假設處於 core 管理的代理環境。
4. 查詢 GUI 狀態只用 `herdr-app ls`、`herdr-app status`、`herdr-app commands`。**禁止執行無參數 `herdr-app`**（會開啟或拉起視窗）。

### A2. 安全邊界（不可違反）
- 未經使用者明確要求，**不停止或重啟**使用者的 core 或 app（包含 `hc server restart --fresh`、`app.reset`、`layout.reset`）。
- 不在研究或診斷過程中自行安裝、更新工具，或修改 `config.toml`、`settings.json`。
- 遇 `blocked` 或 trust／permission 提示時，先讀畫面並依現有授權回報或處置；**禁止**使用 `hc send --force` 或自動送出按鍵代為批准。
- **禁止**將 `confirmed` 參數視為已獲授權；執行 `space.close`、`tab.close`、`pane.close`、`terminal.kill` 前須確認目標與結束終端的副作用。
- **禁止**讀取與當前任務無關的使用者終端；只清理本次任務自己建立的 worker。
- **禁止**在使用者的真實顯示器上執行滑鼠／鍵盤自動化。
- 若需測試 core 啟停或恢復，一律使用獨立 `HC_SESSION` 與隔離設定／資料目錄。

### A3. Worker 協調與等待紀律
- 背景工作用 `hc run --bg`，不加 `herdr.space`／`herdr.tab` labels；慢指令卸載用 `hc run --bg --rm --no-notify`（需讀輸出時勿加 `--rm`）。
- 啟動互動式代理前先查 `hc templates`（Codex 須含 `--no-daemon`）；首個任務放在 launch 並檢查 `first_message.sent`。長輸出請子代理寫檔並回報路徑，不讓子代理再建立子代理。
- 等待狀態一律用 `hc wait`（預設 180s），禁止以 `sleep` 或高頻 `ls`/`read` 輪詢。`timeout`（或前景退出碼 124）不等於失敗，沿用原位址續等。
- 狀態以 OSC 7501 為準，不從畫面文字猜測（留意 Windows PowerShell 返回 shell 時 OSC 7501 狀態目前不會自動清除）。

### A4. GUI 定址與 Mod 開發紀律
- Agent 呼叫 `herdr-app do` 絕不會開啟缺參數對話框，必須傳齊必要參數並檢查 JSON 回傳與退出碼（失敗回 `{"error": "..."}`、exit 1；2 秒內非零退出會彈出 toast）。
- 一律使用已查得的明確字串 ID，避免省略或 `null` 作用到目前焦點。
- 修改 mod／layout 前先讀 app home 生成的 `AGENTS.md` 與 `.d.ts`（勿編輯它們）；內建項目先 `mod.clone`／`layout.clone`。
- `init()` 中以 `checking()` 保護會產生副作用的呼叫；`watchFolder(dir)` 須 `.cancel()`；`View.close(cx)` 遵守 250ms 預算；第三方 mod 須先審查再載入。

### A5. 驗證與回報
- Core 任務須核對實際輸出、退出碼與交付物；`done` 或 launch 成功不代表需求完成。
- GUI 變更須核對 `herdr-app ls`／`status`；mod 變更須依序通過 `herdr-app check`、`herdr-app status`、`herdr-app shot` 並實際檢視截圖。
- 回報時列出完成事項、涉及的 terminal／pane／space ID、驗證結果與未驗證範圍或阻塞原因。

---

## Part B：`herdr-core-skills` 儲存庫維護規則

### B1. 檔案分工與風格
- `herdr-core/SKILL.md` 保持精簡（約 40–50 行），僅放入口指引、路由表、核心原則與歸屬聲明；細節放入 `herdr-core/references/*.md`。
- `rules/herdr-core.md` 必須保持獨立、精簡，僅包含操作 `herdr-core`／`herdr-app` 的規則，**不可混入本儲存庫的維護規則**。
- 繁體中文撰寫說明，所有 CLI 名稱、子命令、旗標、環境變數、設定鍵與 JSON 欄位保留精確英文原文。
- 不得臆造未經本機 binary 或官方文件驗證的指令與參數。

### B2. 維護期間的本機環境保護
- 檢查本機已安裝的 `hc`／`herdr-core`／`herdr-app` 時，**僅執行唯讀資訊命令**（`--version`、`--help`、`--skill`、`schema`、`commands`）。
- 不得為了撰寫或驗證文件而啟動、停止、重啟或修改使用者的 `herdr-core`、`herdr-app` 視窗或設定檔。

### B3. 版本漂移更新檢查清單
當上游 `herdr-core` 或 `herdr-app` 發布新版本時：
1. 以唯讀方式比對新舊版本的 `--version`、`--help`、`--skill`、`schema` 與 `herdr-app commands`。
2. 同步更新 `herdr-core/SKILL.md`、`herdr-core/references/core-workflows.md`、`herdr-core/references/app-workflows.md`、`SPEC.md`、`AGENTS.md` 與 `rules/herdr-core.md`。
3. 更新 `herdr-core/references/official-sources.md` 與 `README.md` 中的查核日期與已驗證版本號。
4. 檢查所有相對 Markdown 連結皆有效，且無殘留暫存檔（如 `herdr-shot.png`）。
