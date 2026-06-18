---
name: exam_to_tts_html
description: 將考卷（圖片或文字）轉換為報讀HTML檔，供特教或視障學生使用。當使用者說「幫我做成報讀HTML」、「轉成報讀格式」、「考卷轉TTS」或提供考卷內容時觸發。
---

# 考卷轉報讀 HTML 技能

## 目的
將使用者提供的考卷內容（文字或圖片）轉換成標準的報讀 HTML 格式，供特教或視障學生使用。

---

## 資料結構

每一題轉成 JavaScript 物件，格式如下：

```js
{
  section: "題型名稱",   // 例："單題"、"閱讀題組"、"填充題"
  displayId: "1",        // 題號（字串）
  preamble: "...",       // 引文/題組文章/圖形文字，無則設 null
  question: "1. 題目文字", // 完整題目文字（含題號）
  options: ["(A)...", "(B)...", "(C)...", "(D)..."]  // 無選項則設 null
}
```

---

## 處理規則

### 規則 1：換用新資料
- 把考卷所有題目照規則轉換成 `questions` 陣列，取代原有示範資料。
- `<h1>` 標題改為考卷名稱（例：「114年國中教育會考社會科」）。

### 規則 2：若題型中有圖形
- 辨識圖片中的文字，原文放入 `preamble` 欄位。
- **不可增減任何文字**，完整還原圖中所有文字。
- 選項若在圖中，照樣辨識並放入 `options`。

### 規則 3：非選擇題（填充、問答、配合題等）
- 將全部文字放入 `question` 欄位。
- `options` 設為 `null`。
- `preamble` 設為 `null`（除非有題組文章）。

### 規則 4：閱讀測驗題組
- **僅題組首題**放入文章於 `preamble`。
- 題組內其餘題目的 `preamble` 設為 `null`。
- 題組的 `section` 統一設為 `"閱讀題組"`。

---

## 輸出 HTML 模板

輸出完整的獨立 HTML 檔，包含全部 CSS 與 JS，不依賴外部資源（字型除外）。

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
        let currentPreamble = '';
        const synth = window.speechSynthesis;
        let utterance = new SpeechSynthesisUtterance();
        utterance.lang = 'zh-TW';
        utterance.rate = 0.9;
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

        function populateVoiceList() {
            voices = synth.getVoices().filter(voice => voice.lang.startsWith('zh'));
            voiceSelectEl.innerHTML = '';
            if (voices.length === 0) { const o = document.createElement('option'); o.textContent = '找不到中文語音'; voiceSelectEl.appendChild(o); return; }
            voices.forEach((voice, i) => { const o = document.createElement('option'); o.textContent = voice.name; o.value = i; voiceSelectEl.appendChild(o); });
            let idx = voices.findIndex(v => v.name.includes('Yating') && v.lang === 'zh-TW');
            if (idx === -1) idx = voices.findIndex(v => v.name === 'Google 國語（臺灣）');
            if (idx === -1) idx = 0;
            if (voices[idx]) { utterance.voice = voices[idx]; voiceSelectEl.selectedIndex = idx; }
        }
        if (synth.onvoiceschanged !== undefined) { synth.onvoiceschanged = populateVoiceList; }

        function speak(text) {
            synth.cancel();
            utterance.text = text;
            utterance.rate = parseFloat(speedControl.value);
            const i = voiceSelectEl.value;
            if (voices[i]) utterance.voice = voices[i];
            synth.speak(utterance);
        }
        function speakAndHighlight(text, element) {
            document.querySelectorAll('.speaking').forEach(el => el.classList.remove('speaking'));
            element.classList.add('speaking');
            const cleanup = () => { element.classList.remove('speaking'); utterance.onend = null; utterance.onerror = null; };
            utterance.onend = cleanup;
            utterance.onerror = cleanup;
            speak(text);
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
        function renderQuestion(index) {
            const q = questions[index];
            if (q.preamble !== currentPreamble) { renderPreamble(q.preamble); currentPreamble = q.preamble; }
            else if (!q.preamble && currentPreamble) { renderPreamble(null); currentPreamble = null; }
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
            synth.cancel();
            document.querySelectorAll('.speaking').forEach(el => el.classList.remove('speaking'));
            const q = questions[currentQuestionIndex];
            let text = '';
            if (q.preamble && q.preamble !== currentPreamble) text += q.preamble + '。';
            text += q.question;
            if (q.options) text += '。' + q.options.join('。');
            speak(text);
        }
        selectEl.addEventListener('change', e => { currentQuestionIndex = parseInt(e.target.value); synth.cancel(); renderQuestion(currentQuestionIndex); });
        prevBtn.addEventListener('click', () => { if (currentQuestionIndex > 0) { currentQuestionIndex--; synth.cancel(); renderQuestion(currentQuestionIndex); } });
        nextBtn.addEventListener('click', () => { if (currentQuestionIndex < questions.length - 1) { currentQuestionIndex++; synth.cancel(); renderQuestion(currentQuestionIndex); } });
        replayBtn.addEventListener('click', speakCurrentQuestion);
        speedControl.addEventListener('input', e => { const s = parseFloat(e.target.value); utterance.rate = s; speedValue.textContent = s.toFixed(1) + 'x'; if (synth.speaking) speak(utterance.text); });
        
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
