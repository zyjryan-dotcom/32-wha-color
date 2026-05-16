<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>儿童中文点读课 - 这是什么颜色</title>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #f5f7fb;
            --card: #ffffff;
            --text-dark: #2c3e50;
            --btn-next: #ff7043;
            --blue-q: #0026e6; /* 还原截图中问题的标准深蓝色 */
            --blue-q-en: #4a90e2; /* 英文问题的浅蓝色 */
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        body {
            margin: 0; padding: 15px;
            background: var(--bg);
            font-family: 'Noto Sans SC', sans-serif;
            display: flex; flex-direction: column; align-items: center;
            justify-content: center; min-height: 100vh;
        }

        /* 开始启动遮罩层 */
        #shield {
            position: fixed; inset: 0; background: linear-gradient(135deg, #0026e6, #4a90e2);
            z-index: 1000; color: white; display: flex;
            flex-direction: column; align-items: center; justify-content: center;
            cursor: pointer; text-align: center; padding: 20px;
        }

        .progress { font-size: 1.2rem; color: #7f8c8d; font-weight: bold; margin-bottom: 10px; }

        /* 主体卡片排版 */
        .card {
            background: var(--card); border-radius: 25px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.06);
            width: 100%; max-width: 550px;
            padding: 30px; text-align: left;
            position: relative; display: flex; flex-direction: column;
            min-height: 480px; justify-content: space-between;
        }

        /* 上半部分内容区域：左边文本，右边插图 */
        .main-content {
            display: flex; width: 100%; align-items: center; gap: 15px; flex-grow: 1;
        }

        .text-section { flex: 1; display: flex; flex-direction: column; justify-content: center; }
        .image-section { width: 160px; height: 160px; display: flex; align-items: center; justify-content: center; }
        
        .display-emoji {
            font-size: 110px; line-height: 1; user-select: none;
            filter: drop-shadow(0 5px 10px rgba(0,0,0,0.1));
        }

        .clickable-area { cursor: pointer; transition: transform 0.1s; user-select: none; }
        .clickable-area:active { transform: scale(0.98); opacity: 0.8; }

        /* 问句区域样式 */
        .question-box { margin-bottom: 25px; }
        .q-zh { font-size: 2.1rem; color: var(--blue-q); font-weight: bold; margin: 0 0 4px 0; }
        .q-en { font-size: 1.3rem; color: var(--blue-q-en); font-weight: normal; margin: 0; }

        /* 答句区域样式 */
        .answer-box {
            display: none; /* 初始隐藏 */
            border-top: 2px dashed #f0f3f6; padding-top: 20px;
        }
        .a-zh { font-size: 2.2rem; font-weight: bold; margin: 0 0 4px 0; }
        .a-en { font-size: 1.4rem; font-weight: bold; margin: 0; }

        /* 底部导航控制栏 */
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
        <p style="font-size: 1.1rem; opacity: 0.9;">点击任何句子都可以单独发声哦！</p>
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
            
            <div class="image-section">
                <div class="display-emoji" id="display-emoji">❤️</div>
            </div>
        </div>

        <div class="nav-controls">
            <button class="nav-btn" onclick="prev()">上一个</button>
            <button class="nav-btn" style="background: var(--btn-next);" onclick="next()">下一个</button>
        </div>
    </div>

    <script>
        // 严格对照最后两张截图录入的完整 11 种颜色双语数据库
        const colorData = [
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是红色", aEn: "This is red.", color: "#e63946", emoji: "❤️" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是橙色", aEn: "This is orange.", color: "#f4a261", emoji: "👒" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是黄色", aEn: "This is yellow.", color: "#e9c46a", emoji: "🥭" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是绿色", aEn: "This is green.", color: "#2a9d8f", emoji: "🎋" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是蓝色", aEn: "This is blue.", color: "#0026e6", emoji: "🍎" }, // 截图里蓝色配图是蓝苹果
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是紫色", aEn: "This is purple.", color: "#8338ec", emoji: "👗" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是粉色", aEn: "This is pink.", color: "#ff007f", emoji: "🌸" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是白色", aEn: "This is white.", color: "#b0c4de", emoji: "🐱" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是黑色", aEn: "This is black.", color: "#000000", emoji: "🐕" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是灰色", aEn: "This is grey.", color: "#7f8c8d", emoji: "🐇" },
            { qZh: "这是什么颜色？", qEn: "What color is this?", aZh: "这是棕色", aEn: "This is brown.", color: "#8b4513", emoji: "🟫" }
        ];

        let index = 0;
        const synth = window.speechSynthesis;

        function boot() {
            synth.speak(new SpeechSynthesisUtterance("")); // 解锁音频流
            document.getElementById('shield').style.display = 'none';
            render();
        }

        function render() {
            const item = colorData[index];
            
            // 更新进度与配图
            document.getElementById('prog-info').textContent = `${index + 1} / ${colorData.length}`;
            document.getElementById('display-emoji').textContent = item.emoji;
            
            // 填充双语文本内容加小喇叭
            document.getElementById('q-zh-text').innerHTML = item.qZh + '<span class="audio-tip">🔊</span>';
            document.getElementById('q-en-text').textContent = item.qEn;
            
            const azhNode = document.getElementById('a-zh-text');
            azhNode.innerHTML = item.aZh + '<span class="audio-tip">🔊</span>';
            azhNode.style.color = item.color; // 完美还原截图中回答字体的对应颜色
            
            const aenNode = document.getElementById('a-en-text');
            aenNode.textContent = item.aEn;
            aenNode.style.color = item.color; // 英文回答同步对应颜色

            // 切换页面时，默认隐藏下方的回答框
            document.getElementById('ans-section').style.display = 'none';

            // 默认自动朗读蓝色的提问（先中后英）
            speakDual(item.qZh, item.qEn);
        }

        // 点击蓝色提问：发声，并在1.2秒后自动展开并朗读下方的回答
        function clickQuestion() {
            const item = colorData[index];
            speakDual(item.qZh, item.qEn);
            
            setTimeout(() => {
                document.getElementById('ans-section').style.display = 'block';
                speakDual(item.aZh, item.aEn);
            }, 1200);
        }

        // 点击红色/彩色答句：允许学生独立重复点读
        function clickAnswer() {
            const item = colorData[index];
            speakDual(item.aZh, item.aEn);
        }

        // 双语连续标准朗读引擎
        function speakDual(zhText, enText) {
            synth.cancel(); // 强行切断当前发声，防止连续点击时声音重叠

            // 1. 朗读中文（针对儿童优化语速）
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
