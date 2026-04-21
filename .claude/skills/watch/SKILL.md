---
description: "Play a downloaded video with Chinese subtitles using mpv"
---

# /watch — 本地播放影片 + 中文字幕

用 mpv 開啟影片。字幕與影片同目錄同名，mpv 會自動載入。

## 用法

```
/watch <video_title_base>
```

例：`/watch "How Black Holes Actually Work"`（不含副檔名）

## 步驟

1. 確認 `videos/<name>.mp4` 存在，否則提示先執行 `/download`
2. 字幕選擇（依優先序）：
   - `videos/<name>.zh-tw.srt` 存在 → 指定載入中文字幕
   - `videos/<name>.en.srt` 存在 → fallback 英文字幕，提示可執行 `/translate` 取得中文版
   - 兩個都不存在 → 提示先執行 `/transcribe`，不開啟播放器
3. 開啟 mpv（字幕與影片同目錄，mpv 預設會自動偵測並載入）

## 指令

```bash
NAME="<name>"

if [ -f "videos/${NAME}.zh-tw.srt" ]; then
  mpv "videos/${NAME}.mp4" \
    --sub-file="videos/${NAME}.zh-tw.srt"

elif [ -f "videos/${NAME}.en.srt" ]; then
  echo "⚠️  找不到中文字幕，以英文字幕播放。可執行 /translate 取得中文版。"
  mpv "videos/${NAME}.mp4" \
    --sub-file="videos/${NAME}.en.srt"

else
  echo "❌ 找不到任何字幕，請先執行 /transcribe"
  exit 1
fi
```

## mpv 快捷鍵

| 按鍵 | 功能 |
|------|------|
| `v` | 開關字幕 |
| `Space` | 暫停 |
| `←` / `→` | 快退 / 快進 5 秒 |
| `[` / `]` | 調整播放速度 |

## 完成條件

- mpv 視窗正常開啟並顯示中文字幕
