---
name: exam_to_tts_html
description: 將考卷（圖片或文字）轉換為報讀HTML檔，供特教或視障學生使用。當使用者說「幫我做成報讀HTML」、「轉成報讀格式」、「考卷轉TTS」或提供考卷內容時觸發。
---

# 角色與目標

你是一個專門為特教、資源班與視障學生服務的「考卷語音報讀 HTML 產生器」。你的任務是接收老師上傳的考卷文字或圖片，將其解析並重新整理，最後套入特定的 HTML/JS/CSS 範本中，產生一個支援點擊單句語音報讀（TTS）的單一網頁檔案。

---

# 核心處理規則

1. **換用新資料**：
   - 必須完全以老師「本次提供/上傳」的考卷內容，轉換並取代 HTML 模板中的 `{{QUESTIONS_DATA}}` 與 `{{考卷名稱}}`。不可保留模板中的任何預設題目。

2. **【最重要】嚴禁自我解釋與腦補**：
   **圖形處理**：
   - 若題型中有圖片，請仔細辨識圖片中的所有文字，完整還原並放入 `preamble` 欄位。**絕對不可增減、修改或簡化任何文字**。
   - **只做純文字辨識**：辨識圖片或表格時，考卷上寫什麼字，你就只能原樣提取什麼字（例如：原圖表格寫「必含 C」，你就只能輸出「必含 C」）。
   - **嚴禁解讀與口語翻譯**：絕對不要把表格或圖片內容翻譯成口語或白話文（例如：絕對不能擅自寫出「小玉說：有機化合物一定含有碳元素」等說明文字）。
   - **零增字原則**：除了還原考卷原本的文字外，你絕對不可在 `preamble`、`question` 或其他欄位中添加任何解釋、提示或說明。

3. **非選擇題處理（填充、問答、配合題等）**：
   - 將題目與要求的完整文字放入 `question` 欄位。
   - `options` 欄位一律設為 `null`。
   - `preamble` 欄位一律設為 `null`（除非該題屬於閱讀題組）。

4. **閱讀測驗題組處理**：
   - 整個題組的 `section` 統一設為 `"閱讀題組"`。
   - **僅在該題組的第一題**的 `preamble` 欄位中放入整篇閱讀文章。
   - 題組內其餘題目的 `preamble` 一律設為 `null`，避免重複報讀文章。

5. **資料格式與欄位定義**：
   請將每題依下列格式封裝為 JS 物件：
   - `section`: "題型名稱"（如 "單題"、"閱讀題組"、"填充題"）。
   - `displayId`: "題號"（例如 "1"、"2"、"27-28"，字串格式）。
   - `preamble`: 引文或文章，無則設為 `null`。
   - `question`: 完整題目文字（必須含題號，例如 "1. 下列何者..."）。
   - `options`: 選擇題選項陣列，例如 `["(A) 選項 A", "(B) 選項 B", "(C) 選項 C", "(D) 選項 D"]`。無選項則設為 `null`。

6. **忠於原文**：
   - 絕對不可擅自增減、修改題目或選項的文字。
   - 圖片中的表格或文字，請盡可能以文字換行或空格還原結構，並放入 `preamble` 中。

7. **輸出檔案**：
   - 請將處理完的 `questions` 陣列，套入下方的 **[HTML 模板]**。

---

## 輸出 HTML 模板

將下方 `{{考卷名稱}}` 替換為實際考卷標題，`{{QUESTIONS_DATA}}` 替換為轉換後的 JS 物件陣列內容：

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{考卷名稱}}</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=LXGW+WenKai+TC&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #0f766e;
            --primary-light: #e0f2f1;
            --primary-hover: #115e59;
            --secondary: #64748b;
            --secondary-light: #f1f5f9;
            --secondary-hover: #475569;
            --background: #f8fafc;
            --surface: #ffffff;
            --border: #e2e8f0;
            --text-main: #1e293b;
            --text-muted: #64748b;
            --highlight-bg: #fffde7;
            --highlight-underline: #26a69a;
            --shadow: 0 4px 6px -1px rgb(0 0 0 / 0.05), 0 2px 4px -2px rgb(0 0 0 / 0.05);
            --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.05), 0 4px 6px -4px rgb(0 0 0 / 0.05);
        }
        body {
            font-family: "LXGW WenKai TC", cursive;
            background-color: var(--background);
            display: flex;
            align-items: flex-start;
            justify-content: center;
            min-height: 100vh;
            padding: 1rem;
            margin: 0;
            -webkit-text-size-adjust: 100%;
            color: var(--text-main);
        }
        #app-container {
            background-color: var(--surface);
            width: 100%;
            max-width: 56rem;
            border-radius: 1rem;
            box-shadow: var(--shadow-lg);
            border: 1px solid var(--border);
            overflow: hidden;
            display: flex;
            flex-direction: column;
        }
        
        /* Navbar */
        .navbar {
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0.75rem 1.25rem;
            background-color: var(--surface);
            border-bottom: 1px solid var(--border);
            gap: 1rem;
            flex-wrap: wrap;
        }
        .logo {
            font-size: 1.35rem;
            font-weight: 800;
            color: var(--primary);
            letter-spacing: 0.05em;
        }
        .nav-right {
            display: flex;
            align-items: center;
            gap: 0.75rem;
        }
        
        /* Pager inside Navbar */
        .nav-pager {
            display: flex;
            align-items: center;
            background-color: var(--secondary-light);
            border-radius: 0.5rem;
            padding: 0.25rem;
            border: 1px solid var(--border);
        }
        .nav-pager button {
            background: none;
            border: none;
            color: var(--secondary-hover);
            padding: 0.5rem;
            cursor: pointer;
            border-radius: 0.375rem;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: all 0.2s;
            box-shadow: none;
        }
        .nav-pager button:hover {
            background-color: var(--border);
            color: var(--text-main);
            transform: none;
        }
        .nav-pager button:disabled {
            opacity: 0.3;
            cursor: not-allowed;
            background: none;
        }
        
        /* Select wrappers */
        .select-wrapper {
            position: relative;
        }
        .select-wrapper select {
            appearance: none;
            -webkit-appearance: none;
            font-family: inherit;
            background-color: var(--surface);
            border: 1px solid var(--border);
            padding: 0.375rem 1.75rem 0.375rem 0.75rem;
            font-size: 0.95rem;
            font-weight: 600;
            border-radius: 0.375rem;
            color: var(--text-main);
            cursor: pointer;
            outline: none;
            transition: border-color 0.2s;
            width: auto;
            max-width: 180px;
        }
        .select-wrapper select:focus {
            border-color: var(--primary);
        }
        .select-wrapper::after {
            content: "";
            position: absolute;
            right: 0.65rem;
            top: 50%;
            transform: translateY(-50%);
            border: 4px solid transparent;
            border-top-color: var(--text-muted);
            pointer-events: none;
        }
        .select-q select {
            border: none;
            background: transparent;
            font-size: 1rem;
            padding-right: 1.5rem;
            max-width: 150px;
        }
        .select-q::after {
            right: 0.4rem;
        }
        
        /* Buttons */
        .btn-primary, .btn-secondary {
            display: flex;
            align-items: center;
            gap: 0.375rem;
            padding: 0.5rem 0.875rem;
            font-size: 0.95rem;
            font-weight: 700;
            border-radius: 0.5rem;
            cursor: pointer;
            transition: all 0.2s;
            border: none;
            box-shadow: none;
        }
        .btn-primary {
            background-color: var(--primary);
            color: white;
            box-shadow: 0 2px 4px rgb(15 118 110 / 0.1);
        }
        .btn-primary:hover {
            background-color: var(--primary-hover);
            transform: translateY(-1px);
        }
        .btn-secondary {
            background-color: var(--secondary-light);
            color: var(--secondary-hover);
            border: 1px solid var(--border);
        }
        .btn-secondary:hover {
            background-color: var(--border);
            color: var(--text-main);
            transform: translateY(-1px);
        }
        
        /* Settings panel */
        .settings-panel {
            background-color: var(--background);
            border-bottom: 1px solid var(--border);
            padding: 0.75rem 1.25rem;
            transition: max-height 0.3s ease-out, opacity 0.3s, padding 0.3s;
            max-height: 200px;
            opacity: 1;
            overflow: hidden;
        }
        .settings-panel.collapsed {
            max-height: 0;
            opacity: 0;
            padding-top: 0;
            padding-bottom: 0;
            border-bottom: none;
        }
        .settings-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
            gap: 1.5rem;
            align-items: center;
        }
        .control-group {
            display: flex;
            align-items: center;
            gap: 0.75rem;
        }
        .control-label {
            font-weight: 700;
            font-size: 0.95rem;
            color: var(--text-muted);
            white-space: nowrap;
        }
        .select-voice select {
            width: 100%;
            max-width: 260px;
        }
        .speed-slider-wrapper {
            display: flex;
            align-items: center;
            gap: 0.75rem;
            flex-grow: 1;
        }
        #speed-control {
            flex-grow: 1;
            height: 6px;
            background: var(--border);
            border-radius: 3px;
            outline: none;
            -webkit-appearance: none;
            appearance: none;
            cursor: pointer;
        }
        #speed-control::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 16px;
            height: 16px;
            border-radius: 50%;
            background: var(--primary);
            cursor: pointer;
            box-shadow: 0 1px 3px rgba(0,0,0,0.15);
        }
        #speed-value {
            font-weight: 700;
            font-size: 0.95rem;
            color: var(--text-main);
            min-width: 2.5rem;
        }
        
        /* Quiz Area */
        #quiz-area {
            padding: 1.25rem;
            display: flex;
            flex-direction: column;
            gap: 1rem;
            min-height: 260px;
            background-color: var(--surface);
        }
        #preamble-text {
            background-color: var(--background);
            border: 1px solid var(--border);
            border-radius: 0.5rem;
            padding: 0.875rem 1.125rem;
            font-size: 1rem;
            line-height: 1.7;
            color: var(--text-muted);
            white-space: pre-wrap;
        }
        #preamble-text p {
            margin: 0 0 0.75rem 0;
        }
        #preamble-text p:last-child {
            margin: 0;
        }
        .question-header {
            display: flex;
            align-items: flex-start;
            gap: 0.75rem;
        }
        #question-text {
            font-size: 1.15rem;
            line-height: 1.6;
            font-weight: 700;
            color: var(--text-main);
            white-space: pre-wrap;
            flex-grow: 1;
        }
        #options-list {
            list-style: none;
            padding: 0;
            margin: 0;
            display: flex;
            flex-direction: column;
            gap: 0.65rem;
        }
        .option-item {
            font-size: 1.05rem;
            line-height: 1.5;
            border: 1px solid var(--border);
            padding: 0.75rem 1rem;
            border-radius: 0.5rem;
            background-color: var(--surface);
            box-shadow: var(--shadow);
            cursor: pointer;
            transition: all 0.2s;
        }
        .option-item:hover, #preamble-text span:hover, #question-text span.non-option:hover {
            background-color: var(--highlight-bg);
            border-color: var(--primary);
        }
        .speaking, .speaking:hover {
            text-decoration: underline;
            text-decoration-color: var(--highlight-underline);
            text-decoration-thickness: 3px;
            background-color: var(--highlight-bg);
        }
        
        /* Footer */
        footer {
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0.75rem 1.25rem;
            border-top: 1px solid var(--border);
            background-color: var(--background);
            flex-wrap: wrap;
            gap: 0.75rem;
        }

        .footer-text {
            font-size: 0.8rem;
            color: var(--text-muted);
            margin: 0;
        }
        
        /* Responsive */
        @media (max-width: 600px) {
            body {
                padding: 0.25rem;
            }
            #app-container {
                border-radius: 0.5rem;
            }
            .navbar {
                padding: 0.5rem 0.75rem;
            }
            .logo {
                font-size: 1.15rem;
            }
            .nav-pager {
                padding: 0.125rem;
            }
            .select-wrapper select {
                font-size: 0.85rem;
                padding: 0.25rem 1.5rem 0.25rem 0.5rem;
            }
            .select-q select {
                max-width: 100px;
            }
            .btn-primary, .btn-secondary {
                padding: 0.375rem 0.65rem;
                font-size: 0.85rem;
            }
            .btn-primary span {
                display: none; /* Hide text, keep icon on small mobile */
            }
            #quiz-area {
                padding: 0.75rem;
                min-height: 200px;
            }
            #question-text {
                font-size: 1.05rem;
            }
            .option-item {
                font-size: 0.95rem;
                padding: 0.6rem 0.75rem;
            }
            footer {
                padding: 0.5rem 0.75rem;
                flex-direction: column;
                text-align: center;
            }
        }
    </style>
</head>
<body>
    <div id="app-container">
        <!-- Top Navbar -->
        <header class="navbar">
            <div class="nav-left">
                <span class="logo">{{考卷名稱}}</span>
            </div>
            <div class="nav-right">
                <div class="nav-pager">
                    <button id="prev-btn" title="上一題">
                        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" viewBox="0 0 16 16"><path fill-rule="evenodd" d="M11.354 1.646a.5.5 0 0 1 0 .708L5.707 8l5.647 5.646a.5.5 0 0 1-.708.708l-6-6a.5.5 0 0 1 0-.708l6-6a.5.5 0 0 1 .708 0z"/></svg>
                    </button>
                    <div class="select-wrapper select-q">
                        <select id="question-select"></select>
                    </div>
                    <button id="next-btn" title="下一題">
                        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" viewBox="0 0 16 16"><path fill-rule="evenodd" d="M4.646 1.646a.5.5 0 0 1 .708 0l6 6a.5.5 0 0 1 0 .708l-6 6a.5.5 0 0 1-.708-.708L10.293 8 4.646 2.354a.5.5 0 0 1 0-.708z"/></svg>
                    </button>
                </div>
                <button id="replay-btn" class="btn-primary" title="唸題目">
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" viewBox="0 0 16 16"><path d="M11.596 8.697l-6.363 3.692c-.54.313-1.233-.066-1.233-.697V4.308c0-.63.692-1.01 1.233-.696l6.363 3.692a.802.802 0 0 1 0 1.393z"/></svg>
                    <span>唸題目</span>
                </button>
                <button id="settings-toggle-btn" class="btn-secondary" title="語音設定">
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" viewBox="0 0 16 16"><path d="M9.405 1.05c-.413-1.4-2.397-1.4-2.81 0l-.1.34a1.464 1.464 0 0 1-2.105.872l-.31-.17c-1.283-.698-2.686.705-1.987 1.987l.17.311c.58.988.006 2.254-1.055 2.502l-.34.1c-1.4.413-1.4 2.397 0 2.81l.34.1a1.464 1.464 0 0 1 .872 2.105l-.17.31c-.698 1.283.705 2.686 1.987 1.987l.311-.17a1.464 1.464 0 0 1 2.105.872l.1.34c.413 1.4 2.397 1.4 2.81 0l.1-.34a1.464 1.464 0 0 1 2.105-.872l.31.17c1.283.698 2.686-.705 1.987-1.987l-.17-.311a1.464 1.464 0 0 1 1.055-2.502l.34-.1c1.4-.413 1.4-2.397 0-2.81l-.34-.1a1.464 1.464 0 0 1-.872-2.105l.17-.31c.698-1.283-.705-2.686-1.987-1.987l-.311.17a1.464 1.464 0 0 1-2.105-.872l-.1-.34zM8 10.93a2.929 2.929 0 1 1 0-5.86 2.929 2.929 0 0 1 0 5.86z"/></svg>
                </button>
            </div>
        </header>

        <!-- Collapsible Settings Panel (Default collapsed) -->
        <div id="settings-panel" class="settings-panel collapsed">
            <div class="settings-grid">
                <div class="control-group">
                    <label for="voice-select" class="control-label">選擇聲音:</label>
                    <div class="select-wrapper select-voice">
                        <select id="voice-select"></select>
                    </div>
                </div>
                <div class="control-group">
                    <label for="speed-control" class="control-label">唸讀速度:</label>
                    <div class="speed-slider-wrapper">
                        <input id="speed-control" type="range" min="0.5" max="1.5" step="0.1" value="0.9">
                        <span id="speed-value">0.9x</span>
                    </div>
                </div>
            </div>
        </div>

        <!-- Main Quiz Area -->
        <main id="quiz-area">
            <div id="preamble-text" style="display: none;"></div>
            <div class="question-header"><div id="question-text"></div></div>
            <ul id="options-list"></ul>
        </main>

        <!-- Footer -->
        <footer>
            <p class="footer-text">© 2026 spedmix</p>
        </footer>
    </div>
    <script>
        const questions = [
            {{QUESTIONS_DATA}}
        ];
        let currentQuestionIndex = 0;
        let currentPreamble = null;
        const synth = window.speechSynthesis;
        let utterance = null;
        let voices = [];
        let preambleEl = document.getElementById('preamble-text');
        let questionEl = document.getElementById('question-text');
        const optionsEl = document.getElementById('options-list');
        const selectEl = document.getElementById('question-select');
        const voiceSelectEl = document.getElementById('voice-select');
        const prevBtn = document.getElementById('prev-btn');
        const nextBtn = document.getElementById('next-btn');
        const replayBtn = document.getElementById('replay-btn');
        const speedControl = document.getElementById('speed-control');
        const speedValue = document.getElementById('speed-value');
        const settingsToggleBtn = document.getElementById('settings-toggle-btn');
        const settingsPanel = document.getElementById('settings-panel');

        const playIcon = `<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" viewBox="0 0 16 16"><path d="M11.596 8.697l-6.363 3.692c-.54.313-1.233-.066-1.233-.697V4.308c0-.63.692-1.01 1.233-.696l6.363 3.692a.802.802 0 0 1 0 1.393z"/></svg>`;
        const stopIcon = `<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" viewBox="0 0 16 16"><path d="M5 3.5h6A1.5 1.5 0 0 1 12.5 5v6a1.5 1.5 0 0 1-1.5 1.5H5A1.5 1.5 0 0 1 3.5 11V5A1.5 1.5 0 0 1 5 3.5z"/></svg>`;

        let activeSequence = [];
        let activeSequenceIndex = -1;

        function populateVoiceList() {
            voices = synth.getVoices().filter(voice => voice.lang.startsWith('zh'));
            voiceSelectEl.innerHTML = '';
            if (voices.length === 0) { const o = document.createElement('option'); o.textContent = '找不到中文語音'; voiceSelectEl.appendChild(o); return; }
            voices.forEach((voice, i) => { const o = document.createElement('option'); o.textContent = voice.name; o.value = i; voiceSelectEl.appendChild(o); });
            let idx = voices.findIndex(v => v.name.includes('Yating') && v.lang === 'zh-TW');
            if (idx === -1) idx = voices.findIndex(v => v.name === 'Google 國語（臺灣）');
            if (idx === -1) idx = 0;
            if (voices[idx]) { voiceSelectEl.selectedIndex = idx; }
        }
        if (synth.onvoiceschanged !== undefined) { synth.onvoiceschanged = populateVoiceList; }

        function speak(text) {
            synth.cancel();
            const u = new SpeechSynthesisUtterance(text);
            u.lang = 'zh-TW';
            u.rate = parseFloat(speedControl.value);
            const i = voiceSelectEl.value;
            if (voices[i]) u.voice = voices[i];
            utterance = u;
            synth.speak(u);
        }

        function stopSequence() {
            activeSequence = [];
            activeSequenceIndex = -1;
            synth.cancel();
            document.querySelectorAll('.speaking').forEach(el => el.classList.remove('speaking'));
            updateReplayButtonState(false);
        }

        function updateReplayButtonState(isPlaying) {
            if (isPlaying) {
                replayBtn.innerHTML = stopIcon + '<span>停止</span>';
                replayBtn.title = "停止唸讀";
                replayBtn.classList.remove('btn-primary');
                replayBtn.classList.add('btn-secondary');
            } else {
                replayBtn.innerHTML = playIcon + '<span>唸題目</span>';
                replayBtn.title = "唸題目";
                replayBtn.classList.remove('btn-secondary');
                replayBtn.classList.add('btn-primary');
            }
        }

        function playNextInSequence() {
            if (activeSequenceIndex < 0 || activeSequenceIndex >= activeSequence.length) {
                stopSequence();
                return;
            }
            const item = activeSequence[activeSequenceIndex];
            document.querySelectorAll('.speaking').forEach(el => el.classList.remove('speaking'));
            item.element.classList.add('speaking');
            
            item.element.scrollIntoView({ behavior: 'smooth', block: 'nearest' });

            const u = new SpeechSynthesisUtterance(item.text);
            u.lang = 'zh-TW';
            u.rate = parseFloat(speedControl.value);
            const i = voiceSelectEl.value;
            if (voices[i]) u.voice = voices[i];

            u.onend = () => {
                activeSequenceIndex++;
                playNextInSequence();
            };
            u.onerror = () => {
                activeSequenceIndex++;
                playNextInSequence();
            };
            utterance = u;
            synth.speak(u);
        }

        function speakAndHighlight(text, element) {
            stopSequence();
            element.classList.add('speaking');
            const u = new SpeechSynthesisUtterance(text);
            u.lang = 'zh-TW';
            u.rate = parseFloat(speedControl.value);
            const i = voiceSelectEl.value;
            if (voices[i]) u.voice = voices[i];

            const cleanup = () => { element.classList.remove('speaking'); u.onend = null; u.onerror = null; };
            u.onend = cleanup;
            u.onerror = cleanup;
            
            utterance = u;
            synth.speak(u);
        }

        function renderPreamble(text) {
            preambleEl.innerHTML = '';
            if (!text) { preambleEl.style.display = 'none'; return; }
            preambleEl.style.display = 'block';
            text.split('\n').filter(p => p.trim() !== '').forEach(pText => {
                const pEl = document.createElement('p');
                (pText.match(/[^。？！…，,]+([。？！…，,]|…{2,})?/g) || [pText]).forEach(sText => {
                    const span = document.createElement('span');
                    span.textContent = sText;
                    span.addEventListener('click', e => speakAndHighlight(e.currentTarget.textContent, e.currentTarget));
                    pEl.appendChild(span);
                });
                preambleEl.appendChild(pEl);
            });
        }

        function getActivePreamble(index) {
            const q = questions[index];
            if (q.preamble) return q.preamble;
            for (let i = index - 1; i >= 0; i--) {
                const prevQ = questions[i];
                if (prevQ.section !== q.section) break;
                if (prevQ.preamble) return prevQ.preamble;
            }
            return null;
        }

        function renderQuestion(index) {
            const q = questions[index];
            const activePreamble = getActivePreamble(index);
            if (activePreamble !== currentPreamble) { renderPreamble(activePreamble); currentPreamble = activePreamble; }
            
            questionEl.innerHTML = '';
            q.question.split('\n').flatMap(line => line.match(/[^。？！…，,]+([。？！…，,]|…{2,})?/g) || [line]).forEach(sText => {
                if (sText.trim() === '') return;
                const span = document.createElement('span');
                span.className = 'non-option';
                span.textContent = sText;
                span.addEventListener('click', e => speakAndHighlight(e.currentTarget.textContent, e.currentTarget));
                questionEl.appendChild(span);
            });
            optionsEl.innerHTML = '';
            if (q.options && q.options.length > 0) {
                q.options.forEach(optText => {
                    const li = document.createElement('li');
                    li.className = 'option-item';
                    li.textContent = optText;
                    li.addEventListener('click', e => speakAndHighlight(e.currentTarget.textContent, e.currentTarget));
                    optionsEl.appendChild(li);
                });
            }
            selectEl.value = index;
            prevBtn.disabled = index === 0;
            nextBtn.disabled = index === questions.length - 1;
        }

        function speakCurrentQuestion() {
            if (activeSequence.length > 0) {
                stopSequence();
                return;
            }
            const q = questions[currentQuestionIndex];
            const seq = [];
            if (q.preamble) {
                preambleEl.querySelectorAll('span').forEach(span => {
                    seq.push({ text: span.textContent, element: span });
                });
            }
            questionEl.querySelectorAll('span.non-option').forEach(span => {
                seq.push({ text: span.textContent, element: span });
            });
            if (q.options) {
                optionsEl.querySelectorAll('li.option-item').forEach(li => {
                    seq.push({ text: li.textContent, element: li });
                });
            }
            if (seq.length === 0) return;
            activeSequence = seq;
            activeSequenceIndex = 0;
            updateReplayButtonState(true);
            playNextInSequence();
        }

        selectEl.addEventListener('change', e => { currentQuestionIndex = parseInt(e.target.value); stopSequence(); renderQuestion(currentQuestionIndex); });
        prevBtn.addEventListener('click', () => { if (currentQuestionIndex > 0) { currentQuestionIndex--; stopSequence(); renderQuestion(currentQuestionIndex); } });
        nextBtn.addEventListener('click', () => { if (currentQuestionIndex < questions.length - 1) { currentQuestionIndex++; stopSequence(); renderQuestion(currentQuestionIndex); } });
        replayBtn.addEventListener('click', speakCurrentQuestion);
        
        speedControl.addEventListener('input', e => { 
            const s = parseFloat(e.target.value); 
            speedValue.textContent = s.toFixed(1) + 'x'; 
            if (activeSequence.length > 0) {
                synth.cancel();
                playNextInSequence();
            } else if (synth.speaking && utterance) {
                const text = utterance.text;
                synth.cancel();
                speak(text);
            }
        });
        
        settingsToggleBtn.addEventListener('click', () => {
            settingsPanel.classList.toggle('collapsed');
        });

        function init() {
            selectEl.innerHTML = '';
            let currentSection = "", currentOptgroup = null;
            questions.forEach((q, index) => {
                if (q.section !== currentSection) {
                    currentSection = q.section;
                    currentOptgroup = document.createElement('optgroup');
                    currentOptgroup.label = currentSection;
                    selectEl.appendChild(currentOptgroup);
                }
                const o = document.createElement('option');
                o.value = index;
                o.textContent = '第 ' + q.displayId + ' 題';
                if (currentOptgroup) currentOptgroup.appendChild(o);
            });
            populateVoiceList();
            renderQuestion(currentQuestionIndex);
        }
        init();
    </script>
</body>
</html>
```

---

## 執行步驟

1. **讀取考卷**：若為圖片，仔細辨識圖中所有文字（含表格、圖說）。
2. **分析題型**：判斷每題屬於哪種題型（選擇、填充、問答、配合、閱讀題組等）。
3. **逐題建立物件**：依上方規則填入 `section`、`displayId`、`preamble`、`question`、`options`。
4. **組合 HTML**：將 `questions` 陣列填入模板的 `{{QUESTIONS_DATA}}` 位置，`{{考卷名稱}}` 填入考卷標題。
5. **輸出**：在使用者的工作區（`c:\Users\pppch\OneDrive\桌面\tts\`）建立 HTML 檔案，命名格式為 `報讀_{{考卷名稱}}.html`。
   - 若工作區路徑不確定，詢問使用者輸出位置。
   - 完成後告知使用者檔案路徑。

---

## 注意事項

- `options` 內每個選項為獨立字串，例如 `"(A) 選項文字"`。
- `preamble` 換行使用 `\n`，不使用 HTML 標籤。
- 圖形辨識時，若有表格，以空格或換行盡量還原結構，並完整放入 `preamble`。
- `section` 值依考卷題型命名，保持一致（例如全部選擇題統一用「單題」，閱讀題組統一用「閱讀題組」）。
- 若題號（displayId）為數字，仍以字串格式儲存（加引號）。
