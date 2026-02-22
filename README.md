<!DOCTYPE html>
<html dir="rtl" lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>موسوعة الأذكار والتسابيح</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            background: transparent;
            font-family: 'Segoe UI', Tahoma, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 15px;
        }
        .main-container {
            background: #122a2d;
            padding: 25px;
            border-radius: 30px;
            width: 100%;
            max-width: 1100px;
            border: 3px solid #27ae60;
            box-shadow: 0 10px 30px rgba(0,0,0,0.7);
            text-align: center;
            color: white;
        }
        h3 {
            color: #f1c40f;
            font-size: 24px;
            margin: 0 0 20px 0;
            text-shadow: 2px 2px 4px #000;
        }
        .grid {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 15px;
        }
        .card {
            background: #1e3c3f;
            border: 1px solid #27ae60;
            border-radius: 18px;
            padding: 15px;
            width: 160px;
            position: relative;
        }
        .card .title {
            font-size: 15px;
            color: #99ff99;
            font-weight: bold;
            height: 45px;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .card .counter {
            font-size: 32px;
            background: white;
            color: #1e3c3f;
            border-radius: 12px;
            font-weight: bold;
            margin: 12px 0;
            padding: 5px;
        }
        .card button {
            background: #27ae60;
            color: white;
            border: none;
            padding: 10px;
            border-radius: 10px;
            cursor: pointer;
            width: 100%;
            font-size: 16px;
            font-weight: bold;
            transition: 0.2s;
        }
        .card button:hover { background: #2ecc71; }
        .card button:active { transform: scale(0.95); }
        .sound-indicator {
            position: absolute;
            top: -8px;
            left: -8px;
            background: #f1c40f;
            color: #122a2d;
            border-radius: 50%;
            width: 32px;
            height: 32px;
            display: none;
            align-items: center;
            justify-content: center;
            font-size: 18px;
            box-shadow: 0 0 15px #f1c40f;
            animation: pulse 1s infinite;
        }
        @keyframes pulse {
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.2); opacity: 0.8; }
            100% { transform: scale(1); opacity: 1; }
        }
        .reset-btn {
            margin-top: 30px;
            background: #e67e22;
            border: none;
            color: white;
            padding: 12px 40px;
            border-radius: 30px;
            cursor: pointer;
            font-size: 16px;
            font-weight: bold;
            box-shadow: 0 5px 15px rgba(0,0,0,0.3);
        }
        .status-msg {
            background: #1e5631;
            border-radius: 50px;
            padding: 10px 20px;
            margin: 20px auto;
            font-size: 14px;
            color: #ffd966;
            border: 1px solid #f1c40f;
            display: none;
            max-width: 400px;
        }
    </style>
</head>
<body>
    <div class="main-container">
        <h3>✨ مَوْسُوعَةُ الأَذْكَارِ وَالتَّسَابِيحِ الشَّامِلَة ✨</h3>

        <div id="status-message" class="status-msg">🔊 اضغط على أي تسبيحة لبدء الصوت</div>

        <div id="tasbeeh-grid" class="grid"></div>

        <button id="reset-button" class="reset-btn"><i class="fas fa-undo-alt"></i> تصفير السجل بالكامل</button>
    </div>

    <script>
        (function() {
            // قائمة الأذكار (14)
            const azkar = [
                "سبحان الله",
                "الحمد لله",
                "الله أكبر",
                "لا إله إلا الله",
                "أستغفر الله",
                "لا حول ولا قوة إلا بالله",
                "سبحان الله وبحمده",
                "سبحان الله العظيم",
                "اللهم صل على محمد",
                "حسبي الله ونعم الوكيل",
                "لا إله إلا أنت سبحانك",
                "أستغفر الله وأتوب إليه",
                "رضيت بالله رباً",
                "توكلت على الله"
            ];

            // متغيرات الحالة
            let isAudioActivated = false;
            let arabicVoice = null;
            let voicesLoaded = false;
            const statusDiv = document.getElementById('status-message');

            // دالة عرض الرسائل
            function showMessage(text, isSuccess = false, isError = false) {
                statusDiv.style.display = 'block';
                statusDiv.innerHTML = text;
                if (isSuccess) {
                    statusDiv.style.background = '#27ae60';
                    statusDiv.style.color = 'white';
                } else if (isError) {
                    statusDiv.style.background = '#7f1e1e';
                    statusDiv.style.color = '#ffcccc';
                } else {
                    statusDiv.style.background = '#1e5631';
                    statusDiv.style.color = '#ffd966';
                }
                setTimeout(() => {
                    statusDiv.style.display = 'none';
                }, 3000);
            }

            // تحميل الأصوات المتاحة
            function loadVoices() {
                if (!window.speechSynthesis) {
                    showMessage('❌ المتصفح لا يدعم النطق الآلي', false, true);
                    return;
                }
                window.speechSynthesis.getVoices();
                setTimeout(() => {
                    const voices = window.speechSynthesis.getVoices();
                    arabicVoice = voices.find(v => v.lang.includes('ar')) || null;
                    voicesLoaded = true;
                }, 200);
            }

            // تفعيل الصوت عند أول تفاعل
            function activateSpeech() {
                if (isAudioActivated) return true;
                if (!window.speechSynthesis) {
                    showMessage('❌ المتصفح لا يدعم النطق الآلي', false, true);
                    return false;
                }
                try {
                    const silent = new SpeechSynthesisUtterance(' ');
                    silent.volume = 0;
                    window.speechSynthesis.speak(silent);
                    isAudioActivated = true;
                    showMessage('✅ الصوت مفعّل! كل تسبيحة ستصدر صوتاً', true);
                    return true;
                } catch (e) {
                    isAudioActivated = true;
                    return true;
                }
            }

            // نطق النص
            function speak(text, indicatorId) {
                if (!isAudioActivated) return;
                if (!window.speechSynthesis) return;

                const indicator = document.getElementById(indicatorId);
                if (indicator) indicator.style.display = 'flex';

                window.speechSynthesis.cancel();

                const utterance = new SpeechSynthesisUtterance(text);
                utterance.lang = 'ar-SA';
                utterance.rate = 0.9;
                utterance.pitch = 1;
                utterance.volume = 1;
                if (arabicVoice) utterance.voice = arabicVoice;

                utterance.onend = () => {
                    if (indicator) indicator.style.display = 'none';
                };
                utterance.onerror = () => {
                    if (indicator) indicator.style.display = 'none';
                };

                window.speechSynthesis.speak(utterance);
            }

            // بناء شبكة البطاقات
            function buildGrid() {
                const grid = document.getElementById('tasbeeh-grid');
                grid.innerHTML = '';
                azkar.forEach((name, index) => {
                    const saved = localStorage.getItem('zekr_count_' + index) || 0;
                    const card = document.createElement('div');
                    card.className = 'card';
                    card.innerHTML = `
                        <div class="title">${name}</div>
                        <div class="counter" id="count-${index}">${saved}</div>
                        <button onclick="handleTasbeeh(${index})">تسبيح</button>
                        <span class="sound-indicator" id="indicator-${index}">🔊</span>
                    `;
                    grid.appendChild(card);
                });
            }

            // دالة النقر على التسبيحة (تُعرّف على النافذة)
            window.handleTasbeeh = function(index) {
                if (!isAudioActivated) {
                    activateSpeech();
                }

                const countElem = document.getElementById('count-' + index);
                let count = parseInt(countElem.innerText) + 1;
                countElem.innerText = count;
                localStorage.setItem('zekr_count_' + index, count);

                speak(azkar[index], 'indicator-' + index);
            };

            // دالة تصفير العدادات
            document.getElementById('reset-button').addEventListener('click', function() {
                if (confirm('هل تريد تصفير جميع العدادات؟')) {
                    for (let i = 0; i < azkar.length; i++) {
                        localStorage.setItem('zekr_count_' + i, 0);
                        document.getElementById('count-' + i).innerText = '0';
                    }
                    showMessage('✅ تم التصفير', true);
                }
            });

            // تهيئة الصفحة
            buildGrid();
            loadVoices();
        })();
    </script>
</body>
</html>
