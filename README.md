<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>中文点读课 - 这是什么颜色</title>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #f5f7fb;
            --card: #ffffff;
            --text-dark: #2c3e50;
            --btn-next: #ff7043;
            --blue-q: #0026e6; /* 问句的标准蓝色 */
            --blue-q-en: #4a90e2; /* 英文问句颜色 */
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        body {
            margin: 0; padding: 15px;
            background: var(--bg);
            font-family: 'Noto Sans SC', sans-serif;
            display: flex; flex-direction: column; align-items: center;
            justify-content: center; min-height: 100vh;
        }

        /* 启动遮罩层 */
        #shield {
            position: fixed; inset: 0; background: linear-gradient(135deg, #0026e6, #4a90e2);
            z-index: 1000; color: white; display: flex;
            flex-direction: column; align-items: center; justify-content: center;
            cursor: pointer; text-align: center; padding: 20px;
        }

        .progress { font-size: 1.2rem; color: #7f8c8d; font-weight: bold; margin-bottom: 10px; }

        /* 主体卡片 */
        .card {
            background: var(--card); border-radius: 25px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.06);
            width: 100%; max-width: 550px;
            padding: 30px; text-align: left;
            position: relative; display: flex; flex-direction: column;
            min-height: 480px; justify-content: space-between;
        }

        /* 内容布局 */
        .main-content {
            display: flex; width: 100%; align-items: center; gap: 20px; flex-grow: 1;
        }

        .text-section { flex: 1; display: flex; flex-direction: column; justify-content: center; }
        
        /* 准确的颜色色块展示区域 */
        .color-box-section {
            width: 140px; height: 140px; display: flex; align-items: center; justify-content: center;
        }
        
        /* 动态圆角色块，保证颜色100%绝对准确 */
        .color-block {
            width: 100%; height: 100%; border-radius: 24px;
            box-shadow: inset 0 0 0 1px rgba(0,0,0,0.1), 0 8px 20px rgba(0,0,0,0.08);
            transition: background-color 0.3s ease;
        }

        .clickable-area { cursor: pointer; transition: transform 0.1s; user-select: none; }
        .clickable-area:active { transform: scale(0.98); opacity: 0.8; }

        /* 问句样式 */
        .question-box { margin-bottom: 25px; }
        .q-zh { font-size: 2.1rem; color: var(--blue-q); font-weight: bold; margin: 0 0 4px 0; }
        .q-en { font-size: 1.3rem; color: var(--blue-q-en); font-weight: normal; margin: 0; }

        /* 答句样式 */
        .answer-box {
            display: none; /* 初始隐藏 */
            border-top: 2px dashed #f0f3f6; padding-top: 20px;
        }
        .a-zh { font-size: 2.2rem; font-weight: bold; margin: 0 0 4px 0; }
        .a-en { font-size: 1.4rem; font-weight: bold; margin: 0; }

        /* 底部控制栏 */
        .nav-controls { width: 100%; display: flex; gap: 20px; margin-top: 25px; }
        .nav-btn {
            flex: 1; padding: 15px; border-radius: 50px; border: none;
            background: var(--text-dark); color: white; font-size: 1.1rem;
            font-weight: bold; cursor: pointer; text-align: center;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
        }
        .nav-btn:active { opacity: 0.9; }

        .audio-tip { font-size: 0.9rem; color: #cbd5e1; margin-left: 8px; vertical-align: middle; }
    </style>
</head>
<body>

    <div id="shield" onclick="boot()">
        <div style="font-size: 5.5rem; margin-bottom: 10px;">🎨</div>
        <h2>点击开始“这是什么颜色”点读练习</h2>
        <p style="font-size: 1.1rem; opacity: 0.9;">点击句子可以随时重复发声哦</p>
    </div>

    <div class="progress" id="prog-info">1 / 11</div>
    
    <div class="card">
        <div class="main-content">
            <div class="text-section">
                <div class="question-box clickable-area" onclick="clickQuestion()">
                    <p class="q-zh" id="q-zh-text"></p>
                    <p class="q-en" id="q-en-text"></p>
                </div>
                
                <div class="answer-box clickable-area" id="ans-section" onclick="clickAnswer()">
                    <p class="a-zh" id="a-zh-text"></p>
                    <p class="a-en" id="a-en-text"></p>
                </div>
            </div>
            
            <div class="color-box-section">
                <div class="color-block" id="color-block"></div>
            </div>
        </div>

        <div class="nav-controls">
            <button class="nav-btn" onclick="prev()">上一个</button>
            <button class="nav-btn" style="background: var(--btn-next);" onclick="next()">下一个</button>
        </div>
    </div>

    <script>
        // 严格校准的 11 种颜色双语及标准 HEX 色值数据库
        const colorData = [
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是红色", aEn: "This is red.", hex: "#FF0000" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是橙色", aEn: "This is orange.", hex: "#FF7F00" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是黄色", aEn: "This is yellow.", hex: "#FFFF00" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是绿色", aEn: "This is green.", hex: "#008000" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是蓝色", aEn: "This is blue.", hex: "#0000FF" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是紫色", aEn: "This is purple.", hex: "#800080" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是粉色", aEn: "This is pink.", hex: "#FFC0CB" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是白色", aEn: "This is white.", hex: "#FFFFFF" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是黑色", aEn: "This is black.", hex: "#000000" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是灰色", aEn: "This is grey.", hex: "#808080" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是棕色", aEn: "This is brown.", hex: "#8B4513" }
        ];

        let index = 0;
        const synth = window.speechSynthesis;

        function boot() {
            synth.speak(new SpeechSynthesisUtterance("")); // 解锁移动端音频
            document.getElementById('shield').style.display = 'none';
            render();
        }

        function render() {
            const item = colorData[index];
            
            // 更新进度与色块颜色
            document.getElementById('prog-info').textContent = `${index + 1} / ${colorData.length}`;
            document.getElementById('color-block').style.backgroundColor = item.hex;
            
            // 渲染问句
            document.getElementById('q-zh-text').innerHTML = item.qZh + '<span class="audio-tip">🔊</span>';
            document.getElementById('q-en-text').textContent = item.qEn;
            
            // 渲染答句并调整对应字体颜色
            const azhNode = document.getElementById('a-zh-text');
            azhNode.innerHTML = item.aZh + '<span class="audio-tip">🔊</span>';
            
            const aenNode = document.getElementById('a-en-text');
            aenNode.textContent = item.aEn;

            // 特殊处理：如果答句是白色，为了防止在白底上看不清，字体使用稍微深一点的灰色，其他均完美对齐标准色
            const displayColor = item.hex === "#FFFFFF" ? "#b0b0b0" : item.hex;
            azhNode.style.color = displayColor;
            aenNode.style.color = displayColor;

            // 初始化：隐藏回答部分
            document.getElementById('ans-section').style.display = 'none';

            // 自动标准朗读蓝色的提问
            speakDual(item.qZh, item.qEn);
        }

        // 点击提问：发声，并在1.2秒后自动展开并朗读下方的回答
        function clickQuestion() {
            const item = colorData[index];
            speakDual(item.qZh, item.qEn);
            
            setTimeout(() => {
                document.getElementById('ans-section').style.display = 'block';
                speakDual(item.aZh, item.aEn);
            }, 1200);
        }

        // 点击答句：独立重复点读
        function clickAnswer() {
            const item = colorData[index];
            speakDual(item.aZh, item.aEn);
        }

        // 双语连续标准朗读引擎
        function speakDual(zhText, enText) {
            synth.cancel(); // 切断当前音频，防重叠

            // 1. 朗读中文（慢速清晰）
            const msgZh = new SpeechSynthesisUtterance(zhText);
            msgZh.lang = 'zh-CN';
            msgZh.rate = 0.8; 
            
            const voices = synth.getVoices();
            const zhVoice = voices.find(v => v.name.includes('Xiaoxiao') || v.lang === 'zh-CN');
            if (zhVoice) msgZh.voice = zhVoice;

            // 2. 朗读英文
            const msgEn = new SpeechSynthesisUtterance(enText);
            msgEn.lang = 'en-US';
            msgEn.rate = 0.85;
            const enVoice = voices.find(v => v.lang.includes('en-US'));
            if (enVoice) msgEn.voice = enVoice;

            // 队列播放
            synth.speak(msgZh);
            synth.speak(msgEn);
        }

        function next() {
            index = (index + 1) % colorData.length;
            render();
        }

        function prev() {
            index = (index - 1 + colorData.length) % colorData.length;
            render();
        }

        window.speechSynthesis.onvoiceschanged = () => synth.getVoices();
    </script>
</body>
</html>
