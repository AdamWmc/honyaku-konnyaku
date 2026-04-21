---
description: "Proofread zh-tw SRT for terminology consistency and natural phrasing via sub-agent"
---

# /proofread — 中文字幕校稿

**架構：spawn sub-agent 執行校稿，主 session context 保持乾淨。**

## 用法

```
/proofread <video_id>
```

## 主 session 職責

1. 確認 `videos/<name>.zh-tw.srt` 存在，否則提示先執行 `/translate`
2. 將下方 Sub-agent Prompt 中所有的 `<id>` 替換為實際 video ID，再傳入
3. Spawn sub-agent（Agent tool，subagent_type: general-purpose）並傳入替換後的完整 prompt
4. 等待完成後確認：`videos/<name>.zh-tw.srt` 段數與 `.en.srt` 一致（校稿覆寫同一檔案）
5. 回報修改摘要給用戶

---

## Sub-agent Prompt（完整版，逐字傳入）

```
任務：校稿 videos/<name>.zh-tw.srt，修正術語一致性與語感，覆寫回同一檔案。

## 校稿重點（按優先順序）

### 1. 術語一致性
掃描全文，找出同一英文術語被翻譯成不同中文的情況，統一為最自然的版本。
常見例子：
- "event horizon" 全部統一為「事件視界」
- "black hole" 全部統一為「黑洞」
- "singularity" 全部統一為「奇點」
- 其他技術名詞若前後不一致，以首次出現的翻譯為準，或選更自然的版本

### 2. 台灣用語一致性
確認全文使用台灣繁體中文慣用詞，發現大陸用語時統一替換：
- 軟件→軟體、硬件→硬體、網絡→網路
- 文件（檔案意義時）→檔案、文件夾→資料夾、應用程序→應用程式
- 內存→記憶體、服務器→伺服器

### 3. 語感自然度
修正明顯生硬的機器翻譯語感：
- 去除不必要的「的」「了」堆疊
- 口語化科技 YouTube 字幕風格（不是學術論文）
- 主詞省略若中文語境下自然則省略

### 4. 跨 entry 斷句修正（滑動視窗）
SRT 切割點基於時間軸，不是語義邊界，常造成中文在奇怪的地方斷開。
處理方式：同時讀取前後各 1-2 條 entry，理解完整語義後，在**不移動時間戳、不合併 entry** 的前提下，讓每條 entry 讀起來更完整自然。

常用技巧：
- 補主詞讓片段獨立成立：「視界」→「事件視界那一側」
- 用破折號或省略號暗示語句延續：「當你跨越事件視界——」
- 調整斷點措辭，讓語意在時間點上自然收尾

範例：
```
❌ 修改前
Entry 47: 當你穿越事件
Entry 48: 視界，一切都改變了。

✅ 修改後
Entry 47: 當你跨越事件視界的瞬間——
Entry 48: 一切都將改變。
```

### 5. 長度控制
每段文字不超過 25 字（含標點）。若超過，在語意自然處加換行（只限一個換行）。

## 格式規則（不可違反）

1. 序號：完全不動
2. 時間戳：完全不動，格式 `HH:MM:SS,mmm --> HH:MM:SS,mmm`
3. 段數：輸出段數必須與輸入完全一致，禁止合併或拆分
4. 空行：每段之間保留一個空行

## 執行步驟

1. 用 Read tool 讀取 videos/<name>.zh-tw.srt 與 videos/<name>.en.srt（對照原文用）
2. 第一遍掃描：建立術語表（找出所有技術名詞的翻譯版本）
3. 第二遍校稿：套用術語表 + 修正語感 + 滑動視窗修正跨 entry 斷句
   - 遇到讀不通順的 entry，對照 .en.srt 同序號原文，理解語義後再改
4. 用 Write tool 將完整結果覆寫回 videos/<name>.zh-tw.srt
5. 回報：總段數、修改了幾段、主要術語統一項目列表、跨 entry 修正了幾處

完成條件：videos/<name>.zh-tw.srt 已覆寫，段數與原檔一致。
```
