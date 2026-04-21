# Honyaku Konnyaku — 影片字幕本地化 Pipeline

## 快速開始（第一次使用）

```
/setup
```

這會自動檢查並安裝所有依賴工具（yt-dlp、ffmpeg、whisper、mpv、IINA），並預下載 Whisper 語音模型（約 1.5GB，需要一點時間）。**只需要跑一次。**

完成後，往後每次使用只需要：

```
/process https://www.youtube.com/watch?v=BKIkTKqWMfg
```

---

## 目標

將網路影片（YouTube 等平台）自動轉成帶本地語言字幕的可播放版本，供個人收看。

## Pipeline 架構

```
YouTube URL
    │
    ▼
[/process]  一鍵完整流程（推薦）
    │
    ├── [/download]   yt-dlp 下載影片 + 音軌
    ├── [/transcribe] Whisper 轉錄英文 → .en.srt
    ├── [/translate]  Claude Code sub-agent 翻譯 → .zh-tw.srt
    └── [/watch]      mpv 播放影片 + 中文字幕
```

## 工作目錄結構

```
honyaku-konnyaku/
├── CLAUDE.md
├── .claude/
│   └── skills/
│       ├── setup/SKILL.md
│       ├── process/SKILL.md
│       ├── download/SKILL.md
│       ├── transcribe/SKILL.md
│       ├── translate/SKILL.md
│       ├── proofread/SKILL.md
│       └── watch/SKILL.md
├── videos/               # 影片 + 字幕全部同層 (.mp4 / .en.srt / .zh-tw.srt)
└── audio/                # 分離的音軌（Whisper 用）(.wav)
```

## 架構決策

- **翻譯由 sub-agent 執行**：`/translate` 透過 Agent tool spawn general-purpose sub-agent 來讀取和寫入字幕，避免大量 SRT 內容污染主 session context。主 session 只負責驗證結果。
- **翻譯不需要額外 API key**：直接使用當前 Claude Code session，零額外設定。

## 依賴工具

| 工具 | 用途 | 安裝 | Token |
|------|------|------|-------|
| yt-dlp | 下載 YouTube | `brew install yt-dlp` | 不需要 |
| ffmpeg | 音軌分離 | `brew install ffmpeg` | 不需要 |
| whisper | 語音轉錄 | `pip install openai-whisper` | 不需要 |
| mpv | 本地播放（CLI） | `brew install mpv` | 不需要 |
| IINA | 本地播放（GUI，推薦） | `brew install --cask iina` | 不需要 |

## 命名慣例

影片以 YouTube 原始標題命名（yt-dlp `%(title)s` 自動清理特殊字元），例如：
- `videos/How Black Holes Actually Work.mp4`
- `videos/How Black Holes Actually Work.en.srt`
- `videos/How Black Holes Actually Work.zh-tw.srt`
- `audio/How Black Holes Actually Work.wav`（僅 Whisper 路徑需要）

mp4 與 srt 同目錄同 base name → IINA / mpv 開啟 mp4 時自動偵測並載入字幕。

## Skill 使用方式

**單步 debug 用：**
```
/download   https://www.youtube.com/watch?v=BKIkTKqWMfg
/transcribe "How Black Holes Actually Work"
/translate  "How Black Holes Actually Work"
/proofread  "How Black Holes Actually Work"
/watch      "How Black Holes Actually Work"
```
