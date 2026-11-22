# Herdr Core & App 操作規則（Agent Rule）

> 本規則適用於操作 [herdrdev](https://github.com/herdrdev) 的 `herdr-core`（`hc`）與 `herdr-app`。Herdr 為 `herdrdev` 所有；本規則由獨立開源專案 `herdr-core-skills` 提供，非官方文件。

## 1. 環境確認與介面辨識
- 操作前先確認實際可執行檔（`hc`、`herdr-core`、`herdr-app`）與版本；Windows 上常見僅有 `herdr-core.exe`，呼叫變數內執行檔用 `& $coreCli` 並立即檢查 `$LASTEXITCODE`。
- 首次操作 core 讀 `<core-cli> --skill`，操作 GUI 讀 `herdr-app --skill`；以已安裝版本的 `--help`／`commands`／`schema` 為準。
- 不要把舊版 `herdr` TUI（`herdr workspace/pane`）或 `sheprd`（`sheprd browser`、Luau mods）套用在這套介面。
- `$HC_TERMINAL_ID`（PowerShell：`$env:HC_TERMINAL_ID`）是 core terminal 位址，不是 GUI pane ID；未設定時不依賴隱含父子關係。
- 查 GUI 狀態用 `herdr-app ls`、`herdr-app status`、`herdr-app commands`；**禁止執行無參數 `herdr-app`**（會開啟或拉起視窗）。

## 2. 安全紅線
- 未經使用者明確要求，不停止或重啟使用者的 core／app（含 `hc server restart --fresh`、`app.reset`、`layout.reset`），也不自行安裝、更新或修改 `config.toml`／`settings.json`。
- 遇 `blocked` 或 trust／permission 提示時先讀畫面並回報；**禁止**用 `hc send --force` 或自動按鍵代為批准。
- `space.close`、`tab.close`、`pane.close` 與 `terminal.kill` 會結束終端（與 `window.close` 不同）；執行前確認目標，不得把 `confirmed` 當成已有授權。
- 不讀取與任務無關的使用者終端；只清理本次自己建立的 worker。
- 絕不在使用者的真實顯示器上執行滑鼠／鍵盤自動化。測試 core 啟停或恢復時，必須使用獨立 `HC_SESSION` 與隔離目錄。

## 3. Core 執行與等待紀律
- 背景工作用 `hc run --bg`，不加 `herdr.space`／`herdr.tab` labels；慢指令卸載用 `hc run --bg --rm --no-notify`（需讀輸出時勿加 `--rm`）。
- 啟動代理先查 `hc templates`（Codex 須含 `--no-daemon`），把首個任務放在 launch 並檢查 `first_message.sent`；長輸出請子代理寫檔並回報路徑，不讓子代理再建立代理。
- 用 `hc wait`（預設 180s）等待，禁止用 `sleep` 或高頻 `ls`/`read` 輪詢。`timeout`（或前景退出碼 124）不等於失敗，沿用原位址續等。
- 狀態以 OSC 7501 為準，不從畫面文字猜測（Windows PowerShell 返回 shell 時狀態目前不會自動清除）。

## 4. App 與 Mod 紀律
- Agent 呼叫 `herdr-app do` 絕不開缺參數對話框，必須傳齊必要參數並檢查退出碼與回傳 JSON（失敗為 `{"error": "..."}`、exit 1）。一律使用查得的明確字串 ID，避免作用到目前焦點。
- PowerShell 傳多欄位 JSON 用 `ConvertTo-Json -Compress`，複雜流程寫 `.ps1`。
- 開發 mod／layout 前先讀 app home 生成的 `AGENTS.md` 與 `.d.ts`（勿編輯）；內建項目先 `mod.clone`／`layout.clone`。第三方 mod 先審查再載入；`init()` 以 `checking()` 保護副作用；`watchFolder(dir)` 須 `.cancel()`；`View.close(cx)` 遵守 250ms 預算。

## 5. 驗證與回報
- Core 任務核對實際輸出、退出碼與交付物；GUI 變更核對 `ls`/`status`；mod 變更依序執行 `herdr-app check`、`herdr-app status`、`herdr-app shot` 並實際檢視圖片。
- 回報已完成項目、涉及的 terminal／pane／space ID、驗證結果與未驗證範圍或阻塞。
