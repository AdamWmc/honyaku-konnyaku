---
description: "Check and install all required tools for the Cleo pipeline"
---

# /setup — 環境初始化

檢查所有依賴工具是否已安裝，未安裝的自動用 brew / pip 安裝。**第一次使用前必須執行。**

## 用法

```
/setup
```

## 執行步驟

依序檢查以下工具，每個都印出 ✓ 已安裝 或 ✗ 安裝中：

```bash
# 1. yt-dlp
if command -v yt-dlp &>/dev/null; then
  echo "✓ yt-dlp $(yt-dlp --version)"
else
  echo "✗ 安裝 yt-dlp..."
  brew install yt-dlp
fi

# 2. ffmpeg
if command -v ffmpeg &>/dev/null; then
  echo "✓ ffmpeg $(ffmpeg -version 2>&1 | head -1)"
else
  echo "✗ 安裝 ffmpeg..."
  brew install ffmpeg
fi

# 3. whisper
if python3 -c "import whisper" &>/dev/null; then
  echo "✓ openai-whisper"
else
  echo "✗ 安裝 openai-whisper..."
  pip install openai-whisper
fi

# 4. mpv
if command -v mpv &>/dev/null; then
  echo "✓ mpv $(mpv --version | head -1)"
else
  echo "✗ 安裝 mpv..."
  brew install mpv
fi

# 5. IINA
if [ -d "/Applications/IINA.app" ]; then
  echo "✓ IINA 已安裝"
else
  echo "✗ 安裝 IINA（macOS GUI 播放器）..."
  brew install --cask iina
fi

# 6. 建立工作目錄
mkdir -p videos audio
echo "✓ 目錄結構建立完成（videos/ audio/）"

# 6. 預下載 Whisper medium 模型（約 1.5GB，首次需要時間）
echo "正在下載 Whisper medium 模型（約 1.5GB），請耐心等候..."
python3 -c "import whisper; whisper.load_model('medium')"
echo "✓ Whisper medium 模型就緒"

echo ""
echo "✅ 環境就緒，可以開始使用："
echo "   /process <YouTube URL>"
```

## 完成條件

- 五個工具全部可執行（yt-dlp、ffmpeg、whisper、mpv、IINA）
- Whisper medium 模型已下載
- `videos/` `audio/` 目錄存在
- 最後印出：「✅ 環境就緒，可以開始使用 /process <YouTube URL>」
