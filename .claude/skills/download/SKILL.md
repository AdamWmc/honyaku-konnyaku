---
description: "Download a YouTube video, extract audio, and grab existing English subtitles if available"
---

# /download — 下載 YouTube 影片

下載影片、分離音軌，並嘗試直接抓取 YouTube 現有英文字幕。
若成功抓到字幕，後續可跳過 `/transcribe` 直接執行 `/translate`。

## 用法

```
/download <YouTube URL>
```

## 步驟

1. 用 `--print filename` 預先取得 yt-dlp 實際會用的輸出檔名（即影片標題），不發多餘 request
2. 若 `videos/<title>.mp4` 已存在 → skip，告知用戶並輸出 base name 結束
3. 建立 `videos/` `audio/` 目錄
4. `yt-dlp` 下載最佳畫質 MP4，輸出到 `videos/<title>.mp4`
5. 同步嘗試抓 YouTube 現有英文字幕，轉 SRT，輸出到 `videos/<title>.en.srt`
6. 若字幕存在 → 回報「已取得 YouTube 字幕，可跳過 /transcribe」
7. 若字幕不存在 → `ffmpeg` 分離音軌到 `audio/<title>.wav`，回報「需執行 /transcribe」
8. **輸出 base name**（不含副檔名）供後續 skill 使用

## 指令

```bash
# 預先取得實際輸出檔名（不下載）
BASE=$(yt-dlp --print filename \
  -o "videos/%(title)s.%(ext)s" \
  --no-download \
  "$URL" | head -1 | xargs basename -s .mp4)

# 冪等：已存在則 skip
if [ -f "videos/${BASE}.mp4" ]; then
  echo "已存在 videos/${BASE}.mp4，skip。"
  echo "BASE_NAME=${BASE}"
  exit 0
fi

mkdir -p videos audio

# 下載影片 + 同時抓字幕（一次 network request）
yt-dlp -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]" \
  --merge-output-format mp4 \
  --write-subs --write-auto-subs \
  --sub-langs en \
  --convert-subs srt \
  --sub-format srt \
  -o "videos/%(title)s.%(ext)s" \
  "$URL"

if [ -f "videos/${BASE}.en.srt" ]; then
  echo "✓ 取得 YouTube 英文字幕 → videos/${BASE}.en.srt"
  echo "→ 可直接執行 /translate，跳過 /transcribe"
else
  echo "✗ 無現有字幕，分離音軌供 Whisper 使用..."
  ffmpeg -i "videos/${BASE}.mp4" \
    -ar 16000 -ac 1 -c:a pcm_s16le \
    "audio/${BASE}.wav"
  echo "✓ audio/${BASE}.wav 已就緒，請執行 /transcribe"
fi

echo "BASE_NAME=${BASE}"
```

## 完成條件

- `videos/<title>.mp4` 存在且大小 > 0
- 以下擇一：
  - `videos/<title>.en.srt` 存在（YouTube 字幕，快速路徑）
  - `audio/<title>.wav` 存在（Whisper fallback 路徑）
- 輸出 `BASE_NAME=<title>` 供後續 skill 使用
