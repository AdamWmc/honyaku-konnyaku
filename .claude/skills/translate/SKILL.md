---
description: "Translate English SRT to Traditional Chinese via sub-agent to keep main context clean"
---

# /translate — 英文字幕翻譯成繁體中文

**架構：spawn sub-agent 執行翻譯，主 session context 保持乾淨。**

## 用法

```
/translate <video_id>
```

## 主 session 職責

1. 確認 `videos/<name>.en.srt` 存在，否則提示先執行 `/transcribe`
2. 若 `videos/<name>.zh-tw.srt` 已存在 → skip，告知用戶並結束（如需重新翻譯請先刪除該檔案）
3. 將下方 Sub-agent Prompt 中所有的 `<name>` 替換為實際影片標題 base name，再傳入
4. Spawn sub-agent（Agent tool，subagent_type: general-purpose）並傳入替換後的完整 prompt
5. 等待完成後確認：`videos/<name>.zh-tw.srt` 存在、段數與原檔一致
6. 回報給用戶

---

## Sub-agent Prompt（完整版，逐字傳入）

```
任務：將 videos/<name>.en.srt 翻譯成繁體中文，輸出到 videos/<name>.zh-tw.srt。

## SRT 格式規範

SRT 每個字幕段由三部分組成，段與段之間必須有一個空行：

  <序號>
  <開始時間> --> <結束時間>
  <字幕文字>

範例輸入：
  1
  00:00:01,000 --> 00:00:03,500
  This is a GPU running at full speed.

  2
  00:00:03,500 --> 00:00:06,000
  The transformer model requires massive compute.

範例輸出：
  1
  00:00:01,000 --> 00:00:03,500
  這是一顆 GPU 全速運行。

  2
  00:00:03,500 --> 00:00:06,000
  transformer 模型需要龐大的算力。

## 格式規則（不可違反）

1. 序號：完全不動，一字不改
2. 時間戳：完全不動，格式 `HH:MM:SS,mmm --> HH:MM:SS,mmm`
3. 段數：輸出段數必須與輸入完全一致，禁止合併或拆分段落
4. 空行：每段之間保留一個空行，不多不少
5. 不在文字行前後加任何額外空白或標點

## 翻譯規則

1. **台灣用語優先**：使用台灣繁體中文慣用詞，例如：
   - 軟體（不用「軟件」）、硬體（不用「硬件」）、網路（不用「網絡」）
   - 檔案（不用「文件」）、資料夾（不用「文件夾」）、應用程式（不用「應用程序」）
   - 記憶體（不用「內存」）、處理器（不用「處理機」）、伺服器（不用「服務器」）
2. 保留英文的專有名詞：GPU、CPU、TPU、transformer、attention、latency、throughput、token、inference、fine-tune、benchmark、CUDA、API、LLM、SSD、RAM 等技術詞
3. 每段文字不超過 25 個字（含標點），過長時可以在語意自然處換行（只限一個換行）
4. 音效標記直接保留或意譯：[Music] → [音樂]、[Applause] → [掌聲]、[Laughter] → [笑聲]
5. 若原文是空行或純音效，翻譯欄位維持對應內容（不留空）
6. 語氣自然、口語化，符合科技 YouTube 字幕風格

## 執行步驟

1. 用 Read tool 讀取 videos/<name>.en.srt
2. 逐段翻譯（在自己的 context 中處理，不需要分批呼叫外部 API）
3. 用 Write tool 將完整結果寫入 videos/<name>.zh-tw.srt
4. 驗證：用 Grep 確認輸出段數 = 輸入段數
5. 回報：總段數、處理時間、有無無法翻譯的段落

完成條件：videos/<name>.zh-tw.srt 存在，且段數與 .en.srt 完全一致。
```
