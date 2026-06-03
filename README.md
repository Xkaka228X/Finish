<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Мониторинг недель</title>
    <style>
        :root {
            --bg-color: #0f172a;
            --card-bg: #1e293b;
            --text-main: #f8fafc;
            --text-muted: #94a3b8;
            --accent: #38bdf8;
            --error: #ef4444;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-main);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            background-color: var(--card-bg);
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.3), 0 8px 10px -6px rgba(0, 0, 0, 0.3);
            width: 100%;
            max-width: 400px;
            text-align: center;
            border: 1px solid rgba(255, 255, 255, 0.05);
        }

        h1 {
            font-size: 1.2rem;
            text-transform: uppercase;
            letter-spacing: 0.05em;
            color: var(--text-muted);
            margin-bottom: 8px;
        }

        .week-number {
            font-size: 2.5rem;
            font-weight: 800;
            color: var(--accent);
            margin-bottom: 24px;
        }

        .code-label {
            font-size: 0.9rem;
            color: var(--text-muted);
            margin-bottom: 6px;
        }

        .code-value {
            font-family: "Courier New", Courier, monospace;
            font-size: 3.5rem;
            font-weight: 700;
            letter-spacing: 2px;
            color: var(--text-main);
            min-height: 70px;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .loader {
            border: 4px solid rgba(255, 255, 255, 0.1);
            border-left-color: var(--accent);
            border-radius: 50%;
            width: 36px;
            height: 36px;
            animation: spin 1s linear infinite;
            margin: 16px auto;
        }

        .error-message {
            color: var(--error);
            font-size: 0.95rem;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body>

    <div class="container">
        <h1 id="title-display">Текущая неделя</h1>
        <div class="week-number" id="week-display">-</div>
        
        <hr style="border: 0; border-top: 1px solid rgba(255,255,255,0.1); margin-bottom: 20px;">
        
        <div class="code-label">Код безопасности</div>
        <div class="code-value" id="code-display">
            <div class="loader"></div>
        </div>
    </div>

    <script>
        (() => {
            const DATA_URL = 'https://raw.githubusercontent.com/Xkaka228X/Obsalutno-secretno/refs/heads/main/README.md';

            function getWeekNumber(date) {
                const target = new Date(date.valueOf());
                const dayNumber = (date.getDay() + 6) % 7;
                target.setDate(target.getDate() - dayNumber + 3);
                const firstThursday = target.valueOf();
                target.setMonth(0, 1);
                if (target.getDay() !== 4) {
                    target.setMonth(0, 1 + ((4 - target.getDay()) + 7) % 7);
                }
                return 1 + Math.ceil((firstThursday - target) / 604800000);
            }

            async function init() {
                const weekDisplay = document.getElementById('week-display');
                const codeDisplay = document.getElementById('code-display');
                
                const currentWeek = getWeekNumber(new Date());
                weekDisplay.textContent = currentWeek;

                try {
                    const response = await fetch(DATA_URL);
                    if (!response.ok) throw new Error();
                    
                    const text = await response.text();
                    const lines = text.split('\n');
                    
                    let foundCode = null;
                    const regex = new RegExp(`^${currentWeek}\\s*\\((\\d+)\\)`);

                    for (let line of lines) {
                        const match = line.trim().match(regex);
                        if (match) {
                            foundCode = match[1];
                            break;
                        }
                    }

                    if (foundCode) {
                        codeDisplay.textContent = foundCode;
                    } else {
                        codeDisplay.innerHTML = '<span class="error-message">Не найден</span>';
                    }

                } catch (error) {
                    codeDisplay.innerHTML = '<span class="error-message">Ошибка загрузки</span>';
                }
            }

            document.addEventListener('DOMContentLoaded', init);
        })();
    </script>
</body>
</html>
