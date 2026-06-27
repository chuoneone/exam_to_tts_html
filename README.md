# 考卷轉無障礙報讀 HTML 技能 (`exam_to_tts_html`)

這是一個專為特教及視障學生設計的 AI 助理客製化技能（Skill）。當向 AI 助理提供考卷文字或圖片，並要求「幫我做成報讀HTML」、「考卷轉TTS」時，AI 會自動將試卷轉換為支援逐句點選、語音朗讀的單頁 HTML 檔案。

GitHub Repo：<https://github.com/chuoneone/exam_to_tts_html>

## 🌟 核心特色
1. **單頁無滾動設計 (Compact Layout)**：
   * 頂部採用極簡的「國二自然」導覽列，整合了「上一題」、「題號切換」、「下一題」與「唸題目」控制項。
   * 「語音選擇」與「朗讀速度」摺疊於設定面板中，釋放縱向高度，避免學生在平板或手機上頻繁滑動。
2. **高精確度圖解轉譯 (OCR)**：
   * 圖形題的微觀結構、力圖箭頭、分子模型、表格數據及漫畫對話等，皆由 AI 進行精準解構並寫入 `preamble` 欄位，徹底杜絕無用的預留位置文字（如「請參閱試卷」）。
3. **題組優化 (Rule 4)**：
   * 題組引文與主圖描述僅會在「題組第一題」顯示與朗讀，後續題目自動設為 `null`，避免學生重複聽讀冗長文章。

---

## 🛠️ 安裝

### Windows PowerShell

直接貼上執行：

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\.codex\skills" | Out-Null; git clone https://github.com/chuoneone/exam_to_tts_html.git "$env:USERPROFILE\.codex\skills\exam_to_tts_html"
```

### macOS / Linux

直接貼上執行：

```bash
mkdir -p ~/.codex/skills && git clone https://github.com/chuoneone/exam_to_tts_html.git ~/.codex/skills/exam_to_tts_html
```

安裝後重新開啟 Codex，或開新對話，即可用這個 skill。觸發方式可以說：

```text
幫我把這份考卷做成報讀HTML
```

或：

```text
考卷轉TTS
```

### 更新已安裝的 skill

```bash
git -C ~/.codex/skills/exam_to_tts_html pull
```

Windows PowerShell 可用：

```powershell
git -C "$env:USERPROFILE\.codex\skills\exam_to_tts_html" pull
```

---

## 📄 授權與宣告
© 2026 spedmix
