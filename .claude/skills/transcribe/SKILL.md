---
description: "Transcribe a downloaded video's audio to English SRT using Whisper"
---

# /transcribe — 英文語音轉文字（Fallback）

> **通常不需要手動執行。** `/download` 會優先抓 YouTube 現有字幕；只有在影片沒有英文字幕時，才需要跑這個步驟。

用本地 Whisper 將音軌轉錄為帶時間戳的英文 SRT。**不需要任何 token。**

## 用法

```
/transcribe <video_title_base>
```

例：`/transcribe "How Black Holes Actually Work"`（不含副檔名）

## 步驟

1. 確認 `audio/<name>.wav` 存在，否則提示先執行 `/download`
2. 若 `videos/<name>.en.srt` 已存在 → skip，告知用戶並結束
3. 根據影片時長預估 Whisper 執行時間並提示用戶
4. 執行 Whisper（medium 模型），輸出 SRT 到暫存目錄
5. 將輸出移到 `videos/<name>.en.srt`
6. 回報字幕段數與總時長

## 指令

```bash
NAME="<name>"

# 冪等：已存在則 skip
if [ -f "videos/${NAME}.en.srt" ]; then
  echo "已存在 videos/${NAME}.en.srt，skip。如需重新轉錄請先刪除該檔案。"
  exit 0
fi

# 執行時間預估（先取得音檔時長）
DURATION=$(ffprobe -v error -show_entries format=duration \
  -of default=noprint_wrappers=1:nokey=1 "audio/${NAME}.wav")
MINUTES=$(echo "$DURATION / 60" | bc)
echo "影片長度約 ${MINUTES} 分鐘，Whisper medium 預估需要 $((MINUTES / 3 + 1))–$((MINUTES / 2 + 1)) 分鐘，請耐心等候..."

# 轉錄到暫存目錄，再移到 videos/
mkdir -p /tmp/whisper_out
whisper "audio/${NAME}.wav" \
  --model medium \
  --language English \
  --output_format srt \
  --output_dir /tmp/whisper_out/

mv "/tmp/whisper_out/${NAME}.srt" "videos/${NAME}.en.srt"
```

## 模型選擇

| 模型 | 速度 | 精準度 | 建議 |
|------|------|--------|------|
| base | 快 | 普通 | 測試用 |
| medium | 中 | 好 | **預設** |
| large-v3 | 慢 | 最佳 | 重要影片 |

## 完成條件

- `videos/<name>.en.srt` 存在
- 時間戳格式正常（`00:00:00,000 --> 00:00:05,000`）
