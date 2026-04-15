# Drag-the-Food-to-its-Container
Game about dragging the food to their designated container
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Food Container Challenge</title>
    <style>
        :root {
            --team-a-color: #ff4757;
            --team-b-color: #2e86de;
            --bg-color: #f1f2f6;
            --container-bg: rgba(255, 255, 255, 0.8);
        }

        * {
            box-sizing: border-box;
            user-select: none;
            -webkit-user-drag: none;
            touch-action: none;
        }

        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            font-family: 'Comic Sans MS', 'Chalkboard SE', sans-serif;
            background: #ffeaa7; /* Cafe/Kitchen vibe */
        }

        #game-container {
            display: flex;
            flex-direction: column;
            width: 100vw;
            height: 100vh;
        }

        /* --- Header Section --- */
        header {
            height: 15%;
            background: #fdcb6e;
            display: flex;
            justify-content: space-around;
            align-items: center;
            border-bottom: 5px solid #e17055;
            z-index: 10;
        }

        .timer-box {
            font-size: 3rem;
            font-weight: bold;
            background: white;
            padding: 10px 30px;
            border-radius: 50px;
            border: 4px solid #333;
        }

        .game-title { font-size: 2.5rem; color: #d63031; text-transform: uppercase; }

        /* --- Main Play Area --- */
        main {
            display: flex;
            flex: 1;
            position: relative;
        }

        .team-section {
            flex: 1;
            position: relative;
            overflow: hidden;
            border: 4px solid #333;
            margin: 10px;
            border-radius: 20px;
            background: rgba(255, 255, 255, 0.3);
            display: flex;
            flex-direction: column;
        }

        #team-a { border-color: var(--team-a-color); }
        #team-b { border-color: var(--team-b-color); }

        .score-board {
            font-size: 2rem;
            padding: 10px;
            text-align: center;
            font-weight: bold;
            color: white;
        }
        .a-bg { background: var(--team-a-color); }
        .b-bg { background: var(--team-b-color); }

        /* --- Draggable Items --- */
        .food-item {
            position: absolute;
            width: 100px;
            height: 100px;
            background: white;
            border-radius: 15px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            cursor: grab;
            box-shadow: 0 6px 10px rgba(0,0,0,0.2);
            z-index: 5;
            font-weight: bold;
            text-align: center;
            font-size: 1.2rem;
            border: 2px solid #333;
        }

        .food-item span { font-size: 2.5rem; }

        /* --- Target Containers --- */
        .drop-zone-container {
            height: 180px;
            background: rgba(0,0,0,0.05);
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 10px;
            margin-top: auto;
            border-top: 2px dashed #999;
        }

        .target-box {
            width: 110px;
            height: 140px;
            border: 3px solid #636e72;
            border-radius: 10px;
            background: var(--container-bg);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: 1.1rem;
            font-weight: bold;
            text-align: center;
        }

        .target-box.hover { transform: scale(1.1); background: #fff; border-color: gold; }

        /* --- Overlay --- */
        #overlay {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.8);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 100;
            color: white;
            display: none;
        }

        #overlay h1 { font-size: 5rem; margin-bottom: 20px; }
        .btn {
            padding: 20px 50px;
            font-size: 2rem;
            cursor: pointer;
            background: #55efc4;
            border: none;
            border-radius: 10px;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <div id="game-container">
        <header>
            <div class="game-title">🥪 Food Sorter</div>
            <div class="timer-box">Time: <span id="timer-val">60</span>s</div>
            <button class="btn" style="font-size: 1.2rem; padding: 10px 20px;" onclick="resetGame()">Restart</button>
        </header>

        <main>
            <div id="team-a" class="team-section">
                <div class="score-board a-bg">TEAM A: <span id="score-a">0</span></div>
                <div id="field-a" style="flex: 1; position: relative;"></div>
                <div class="drop-zone-container" id="drop-a"></div>
            </div>

            <div id="team-b" class="team-section">
                <div class="score-board b-bg">TEAM B: <span id="score-b">0</span></div>
                <div id="field-b" style="flex: 1; position: relative;"></div>
                <div class="drop-zone-container" id="drop-b"></div>
            </div>
        </main>
    </div>

    <div id="overlay">
        <h1 id="winner-text">TEAM A WINS!</h1>
        <button class="btn" onclick="resetGame()">Play Again</button>
    </div>

    <audio id="snd-correct" src="https://assets.mixkit.co/active_storage/sfx/2013/2013-preview.mp3"></audio>
    <audio id="snd-wrong" src="https://assets.mixkit.co/active_storage/sfx/2018/2018-preview.mp3"></audio>

    <script>
        const foodData = [
            { name: "Milk", emoji: "🥛", type: "bottle" },
            { name: "Rice", emoji: "🍚", type: "bowl" },
            { name: "Bread", emoji: "🍞", type: "loaf" },
            { name: "Soup", emoji: "🥣", type: "bowl" },
            { name: "Juice", emoji: "🧃", type: "carton" },
            { name: "Coffee", emoji: "☕", type: "cup" },
            { name: "Sugar", emoji: "🧂", type: "bag" },
            { name: "Ice Cream", emoji: "🍦", type: "cone" }
        ];

        const containers = [
            { id: "bottle", label: "Bottle" },
            { id: "bowl", label: "Bowl" },
            { id: "loaf", label: "Loaf" },
            { id: "carton", label: "Carton" },
            { id: "cup", label: "Cup" },
            { id: "bag", label: "Bag" },
            { id: "cone", label: "Cone" }
        ];

        let scores = { a: 0, b: 0 };
        let timeLeft = 60;
        let gameActive = true;
        let timerInterval;

        function init() {
            createGameSide('a');
            createGameSide('b');
            startTimer();
        }

        function createGameSide(team) {
            const field = document.getElementById(`field-${team}`);
            const dropArea = document.getElementById(`drop-${team}`);
            
            // Clear previous
            field.innerHTML = '';
            dropArea.innerHTML = '';

            // Create Containers
            containers.forEach(c => {
                const box = document.createElement('div');
                box.className = 'target-box';
                box.dataset.type = c.id;
                box.dataset.team = team;
                box.innerHTML = `<span>📥</span>${c.label}`;
                dropArea.appendChild(box);
            });

            // Create Food Items
            foodData.forEach((food, index) => {
                const item = document.createElement('div');
                item.className = 'food-item';
                item.id = `food-${team}-${index}`;
                item.dataset.type = food.type;
                item.dataset.team = team;
                item.innerHTML = `<span>${food.emoji}</span>${food.name}`;
                
                // Random position
                item.style.left = Math.random() * 60 + 10 + "%";
                item.style.top = Math.random() * 60 + 10 + "%";

                field.appendChild(item);
                makeDraggable(item);
            });
        }

        function makeDraggable(el) {
            let offsetX, offsetY;

            el.onpointerdown = function(e) {
                if (!gameActive) return;
                el.style.zIndex = 1000;
                const rect = el.getBoundingClientRect();
                offsetX = e.clientX - rect.left;
                offsetY = e.clientY - rect.top;

                document.onpointermove = function(e) {
                    el.style.left = (e.clientX - offsetX - el.parentElement.getBoundingClientRect().left) + 'px';
                    el.style.top = (e.clientY - offsetY - el.parentElement.getBoundingClientRect().top) + 'px';
                    
                    // Highlight potential drop zones
                    checkHover(el, e.clientX, e.clientY);
                };

                document.onpointerup = function(e) {
                    document.onpointermove = null;
                    document.onpointerup = null;
                    checkDrop(el, e.clientX, e.clientY);
                };
            };
        }

        function checkHover(el, x, y) {
            const targets = document.querySelectorAll(`.target-box[data-team="${el.dataset.team}"]`);
            targets.forEach(t => {
                const r = t.getBoundingClientRect();
                if (x > r.left && x < r.right && y > r.top && y < r.bottom) {
                    t.classList.add('hover');
                } else {
                    t.classList.remove('hover');
                }
            });
        }

        function checkDrop(el, x, y) {
            const targets = document.querySelectorAll(`.target-box[data-team="${el.dataset.team}"]`);
            let matched = false;

            targets.forEach(t => {
                const r = t.getBoundingClientRect();
                if (x > r.left && x < r.right && y > r.top && y < r.bottom) {
                    if (t.dataset.type === el.dataset.type) {
                        // Correct
                        el.style.display = 'none';
                        scores[el.dataset.team]++;
                        updateScore();
                        playSound('snd-correct');
                        matched = true;
                        checkWinStatus();
                    }
                }
                t.classList.remove('hover');
            });

            if (!matched) {
                // Return to random spot in field
                el.style.left = Math.random() * 60 + 10 + "%";
                el.style.top = Math.random() * 60 + 10 + "%";
                playSound('snd-wrong');
            }
        }

        function updateScore() {
            document.getElementById('score-a').innerText = scores.a;
            document.getElementById('score-b').innerText = scores.b;
        }

        function playSound(id) {
            const s = document.getElementById(id);
            s.currentTime = 0;
            s.play().catch(()=>{}); // Catch browser block
        }

        function startTimer() {
            clearInterval(timerInterval);
            timerInterval = setInterval(() => {
                timeLeft--;
                document.getElementById('timer-val').innerText = timeLeft;
                if (timeLeft <= 0) endGame();
            }, 1000);
        }

        function checkWinStatus() {
            const itemsPerTeam = foodData.length;
            if (scores.a === itemsPerTeam || scores.b === itemsPerTeam) {
                endGame();
            }
        }

        function endGame() {
            gameActive = false;
            clearInterval(timerInterval);
            const overlay = document.getElementById('overlay');
            const winText = document.getElementById('winner-text');
            overlay.style.display = 'flex';

            if (scores.a > scores.b) winText.innerText = "TEAM A WINS! 🎉";
            else if (scores.b > scores.a) winText.innerText = "TEAM B WINS! 🎉";
            else winText.innerText = "IT'S A TIE! 🤝";
        }

        function resetGame() {
            scores = { a: 0, b: 0 };
            timeLeft = 60;
            gameActive = true;
            updateScore();
            document.getElementById('timer-val').innerText = timeLeft;
            document.getElementById('overlay').style.display = 'none';
            init();
        }

        // Run on load
        window.onload = init;
    </script>
</body>
</html>
