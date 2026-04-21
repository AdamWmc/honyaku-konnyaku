---
description: "Full pipeline: download → (transcribe if needed) → translate → proofread → watch in one command"
---

# /process — 一鍵完整流程

輸入 YouTube URL，自動執行完整 pipeline 並在完成後播放。
若影片本身有 YouTube 英文字幕，自動跳過 Whisper 轉錄，節省大量時間。

## 用法

```
/process <YouTube URL>
```

例：`/process https://www.youtube.com/watch?v=BKIkTKqWMfg`

## 執行流程

```
[1] 下載影片（取得影片標題作為 base name）
      │
      ├─ YouTube 有英文字幕？
      │       ├─ YES → videos/<title>.en.srt 已就緒
      │       └─ NO  → 分離音軌
      │                    │
      │               [2] Whisper 轉錄 → videos/<title>.en.srt
      │
[3] 翻譯（sub-agent）→ videos/<title>.zh-tw.srt
      │
[4] 校稿（sub-agent）→ videos/<title>.zh-tw.srt（覆寫）
      │
✅ 完成（用戶自行點擊播放）
```

### 詳細步驟

1. **解析 base name**：
   - 用 `yt-dlp --print filename -o "videos/%(title)s.%(ext)s" --no-download <URL>` 取得實際輸出檔名
   - 去掉副檔名，得到 `<title>` base name，後續所有步驟都用這個
   - **記錄整體開始時間**（用於最終總耗時）
2. **下載**：
   - **記錄步驟開始時間**
   - 若 `videos/<title>.mp4` 已存在 → skip
   - 下載影片到 `videos/<title>.mp4`，同時嘗試抓 YouTube 英文字幕到 `videos/<title>.en.srt`
   - **記錄步驟結束時間**
3. **轉錄**（條件執行）：
   - **記錄步驟開始時間**
   - 若 `videos/<title>.en.srt` 已存在（YouTube 字幕或先前已轉錄）→ **skip，顯示「使用現有字幕」**
   - 若不存在 → 執行 Whisper 轉錄（預估時間依影片長度）
   - **記錄步驟結束時間**
4. **翻譯**：
   - **記錄步驟開始時間**
   - 若 `videos/<title>.zh-tw.srt` 已存在 → skip
   - 否則 spawn sub-agent 翻譯，參考 `/translate` skill 的 Sub-agent Prompt
   - sub-agent 結果的 `<usage>` 中有 `duration_ms`，直接用這個作為翻譯耗時
   - **記錄步驟結束時間**
5. **校稿**：
   - **記錄步驟開始時間**
   - spawn sub-agent 執行術語一致性 + 台灣用語 + 語感修正，覆寫 `videos/<title>.zh-tw.srt`
   - 參考 `/proofread` skill 的 Sub-agent Prompt
   - sub-agent 結果的 `<usage>` 中有 `duration_ms`，直接用這個作為校稿耗時
   - **記錄步驟結束時間**
6. **完成**：回報輸出路徑、各步驟耗時、總耗時，不自動播放

## 進度顯示格式

```
[1/4] 下載影片... "How Black Holes Actually Work"
      ✓ 找到 YouTube 英文字幕，跳過轉錄        42s
[2/4] 轉錄音軌... 已跳過                        0s
[3/4] 翻譯字幕...（sub-agent 執行中）          5m 13s
[4/4] 校稿字幕...（sub-agent 執行中）          2m 23s

✅ 完成 → videos/How Black Holes Actually Work.mp4
   總耗時：8m 18s

📊 Token 使用摘要（估算，模型：claude-sonnet-4-6）
   翻譯：~37,768 tokens ≈ $0.18
   校稿：~52,147 tokens ≈ $0.24
   合計：~89,915 tokens ≈ $0.42
   ℹ️  精確費用請用 /cost 查看
```

或（無現有字幕）：
```
[1/4] 下載影片... "How Black Holes Actually Work"
      ✗ 無 YouTube 字幕，需 Whisper 轉錄        38s
[2/4] 轉錄音軌...（Whisper 執行中）            12m 04s
[3/4] 翻譯字幕...（sub-agent 執行中）          5m 13s
[4/4] 校稿字幕...（sub-agent 執行中）          2m 23s

✅ 完成 → videos/How Black Holes Actually Work.mp4
   總耗時：20m 18s

📊 Token 使用摘要（估算，模型：claude-sonnet-4-6）
   翻譯：~N tokens ≈ $X.XX
   校稿：~N tokens ≈ $X.XX
   合計：~N tokens ≈ $X.XX
   ℹ️  精確費用請用 /cost 查看
```

### 耗時顯示規則
- 格式：`Xm Ys`（有分鐘時）或 `Xs`（純秒數）
- 下載、轉錄：從 Bash 指令開始到結束計算
- 翻譯、校稿：優先使用 sub-agent `<usage>` 的 `duration_ms` 欄位換算（÷ 1000）
- skip 的步驟顯示 `0s`
- 總耗時顯示在 ✅ 完成那行下方

## Token 估算方式

每個 sub-agent 結果的 `<usage>` 區塊會包含 `total_tokens`。用以下混合均價估算：
- **Sonnet 4.6**：~$5/M tokens（input $3 + output $15 的加權混合，翻譯任務 input 遠多於 output）
- 估算公式：`total_tokens ÷ 1,000,000 × 5`
- 顯示到小數點後兩位，前面加 `~` 表示估算值

## 失敗處理

每步完成後驗證輸出檔案存在。失敗時停止並提示用戶可單步執行對應 skill 進行 debug。

## 完成條件

- `videos/<title>.zh-tw.srt` 存在且已校稿
- 回報檔案路徑給用戶，由用戶自行點擊播放
