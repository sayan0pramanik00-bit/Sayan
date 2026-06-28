<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultimate Study Suite</title>
    <style>
        :root {
            --bg-color: #f4f7f6;
            --card-bg: #ffffff;
            --primary-color: #4a90e2;
            --accent-color: #2ecc71;
            --text-color: #333333;
            --text-light: #7f8c8d;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: '-apple-system', BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
        }

        body {
            background-color: var(--bg-color);
            color: var(--text-color);
            display: flex;
            flex-direction: column;
            height: 100vh;
            overflow: hidden;
        }

        /* Header */
        header {
            background-color: var(--primary-color);
            color: white;
            text-align: center;
            padding: 15px;
            font-size: 1.2rem;
            font-weight: bold;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }

        /* Main Content App Container */
        main {
            flex: 1;
            overflow-y: auto;
            padding: 20px;
            padding-bottom: 80px; /* Space for bottom nav */
        }

        .tab-content {
            display: none;
        }

        .tab-content.active {
            display: block;
        }

        /* Component: Timer */
        .timer-display {
            font-size: 4rem;
            font-weight: bold;
            text-align: center;
            margin: 40px 0;
            color: var(--primary-color);
        }

        .button-group {
            display: flex;
            justify-content: center;
            gap: 10px;
        }

        button {
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 8px;
            font-size: 1rem;
            cursor: pointer;
            transition: background 0.2s;
        }

        button:active {
            transform: scale(0.98);
        }

        button.secondary {
            background-color: var(--text-light);
        }

        button.success {
            background-color: var(--accent-color);
        }

        /* Component: Inputs & Forms */
        .input-group {
            background: var(--card-bg);
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
            margin-bottom: 20px;
        }

        input, select {
            width: 100%;
            padding: 10px;
            margin: 8px 0;
            border: 1px solid #ddd;
            border-radius: 6px;
            font-size: 1rem;
        }

        /* Component: Flashcards */
        .flashcard {
            background: var(--card-bg);
            min-height: 150px;
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
            padding: 20px;
            margin-bottom: 15px;
            font-size: 1.2rem;
            cursor: pointer;
            position: relative;
            border-left: 5px solid var(--primary-color);
        }

        .delete-btn {
            position: absolute;
            top: 10px;
            right: 10px;
            background: #e74c3c;
            padding: 4px 8px;
            font-size: 0.8rem;
        }

        /* Component: Checklist */
        .list-item {
            background: var(--card-bg);
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 10px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
        }

        .list-item.completed span {
            text-decoration: line-through;
            color: var(--text-light);
        }

        .checkbox {
            width: 24px;
            height: 24px;
            margin-right: 15px;
            cursor: pointer;
        }

        /* Bottom Navigation Bar */
        nav {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            height: 65px;
            background-color: var(--card-bg);
            display: flex;
            border-top: 1px solid #ddd;
            box-shadow: 0 -2px 5px rgba(0,0,0,0.05);
        }

        .nav-btn {
            flex: 1;
            background: none;
            color: var(--text-light);
            border: none;
            border-radius: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            font-size: 0.9rem;
        }

        .nav-btn.active {
            color: var(--primary-color);
            font-weight: bold;
        }
    </style>
</head>
<body>

    <header id="app-title">Study Timer</header>

    <main>
        <section id="timer-tab" class="tab-content active">
            <div class="timer-display" id="time-display">25:00</div>
            <div class="button-group">
                <button id="start-btn" class="success" onclick="toggleTimer()">Start</button>
                <button class="secondary" onclick="resetTimer()">Reset</button>
            </div>
        </section>

        <section id="flashcards-tab" class="tab-content">
            <div class="input-group">
                <input type="text" id="card-front" placeholder="Front (Question)">
                <input type="text" id="card-back" placeholder="Back (Answer)">
                <button style="width: 100%;" onclick="addFlashcard()">Add Flashcard</button>
            </div>
            <div id="flashcards-container"></div>
        </section>

        <section id="checklist-tab" class="tab-content">
            <div class="input-group">
                <input type="text" id="todo-input" placeholder="Enter subject task...">
                <button style="width: 100%;" onclick="addTodo()">Add Task</button>
            </div>
            <div id="checklist-container"></div>
        </section>
    </main>

    <nav>
        <button class="nav-btn active" onclick="switchTab('timer', this)">⏱️<br>Timer</button>
        <button class="nav-btn" onclick="switchTab('flashcards', this)">📇<br>Cards</button>
        <button class="nav-btn" onclick="switchTab('checklist', this)">✅<br>Tasks</button>
    </nav>

    <script>
        // --- STATE MANAGEMENT & LOCALSTORAGE ---
        let flashcards = JSON.parse(localStorage.getItem('cards')) || [];
        let todos = JSON.parse(localStorage.getItem('todos')) || [];

        // --- TAB SYSTEM ---
        function switchTab(tabName, buttonElement) {
            document.querySelectorAll('.tab-content').forEach(tab => tab.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach(btn => btn.classList.remove('active'));
            
            document.getElementById(`${tabName}-tab`).classList.add('active');
            buttonElement.classList.add('active');
            
            // Update Header Text
            const titles = { timer: 'Study Timer', flashcards: 'Flashcards', checklist: 'Subject Checklist' };
            document.getElementById('app-title').innerText = titles[tabName];
        }

        // --- TIMER LOGIC ---
        let timerInterval = null;
        let timeLeft = 25 * 60; // 25 minutes in seconds

        function updateTimerDisplay() {
            const minutes = Math.floor(timeLeft / 60);
            const seconds = timeLeft % 60;
            document.getElementById('time-display').innerText = 
                `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
        }

        function toggleTimer() {
            const startBtn = document.getElementById('start-btn');
            if (timerInterval) {
                clearInterval(timerInterval);
                timerInterval = null;
                startBtn.innerText = "Start";
                startBtn.className = "success";
            } else {
                startBtn.innerText = "Pause";
                startBtn.className = "secondary";
                timerInterval = setInterval(() => {
                    if (timeLeft > 0) {
                        timeLeft--;
                        updateTimerDisplay();
                    } else {
                        clearInterval(timerInterval);
                        alert("Time's up! Take a break.");
                        resetTimer();
                    }
                }, 1000);
            }
        }

        function resetTimer() {
            clearInterval(timerInterval);
            timerInterval = null;
            timeLeft = 25 * 60;
            updateTimerDisplay();
            const startBtn = document.getElementById('start-btn');
            startBtn.innerText = "Start";
            startBtn.className = "success";
        }

        // --- FLASHCARDS LOGIC ---
        function saveCards() { localStorage.setItem('cards', JSON.stringify(flashcards)); renderCards(); }

        function addFlashcard() {
            const front = document.getElementById('card-front').value.trim();
            const back = document.getElementById('card-back').value.trim();
            if (!front || !back) return alert("Fill out both sides!");
            
            flashcards.push({ front, back, isFlipped: false });
            document.getElementById('card-front').value = '';
            document.getElementById('card-back').value = '';
            saveCards();
        }

        function toggleFlip(index) {
            flashcards[index].isFlipped = !flashcards[index].isFlipped;
            renderCards();
        }

        function deleteCard(index, event) {
            event.stopPropagation(); // Prevents flipping when clicking delete
            flashcards.splice(index, 1);
            saveCards();
        }

        function renderCards() {
            const container = document.getElementById('flashcards-container');
            container.innerHTML = '';
            flashcards.forEach((card, index) => {
                const cardEl = document.createElement('div');
                cardEl.className = 'flashcard';
                cardEl.innerText = card.isFlipped ? card.back : card.front;
                cardEl.onclick = () => toggleFlip(index);
                
                const delBtn = document.createElement('button');
                delBtn.className = 'delete-btn';
                delBtn.innerText = 'X';
                delBtn.onclick = (e) => deleteCard(index, e);
                
                cardEl.appendChild(delBtn);
                container.appendChild(cardEl);
            });
        }

        // --- CHECKLIST LOGIC ---
        function saveTodos() { localStorage.setItem('todos', JSON.stringify(todos)); renderTodos(); }

        function addTodo() {
            const text = document.getElementById('todo-input').value.trim();
            if (!text) return;
            todos.push({ text, completed: false });
            document.getElementById('todo-input').value = '';
            saveTodos();
        }

        function toggleTodo(index) {
            todos[index].completed = !todos[index].completed;
            saveTodos();
        }

        function deleteTodo(index) {
            todos.splice(index, 1);
            saveTodos();
        }

        function renderTodos() {
            const container = document.getElementById('checklist-container');
            container.innerHTML = '';
            todos.forEach((todo, index) => {
                const item = document.createElement('div');
                item.className = `list-item ${todo.completed ? 'completed' : ''}`;
                
                const leftDiv = document.createElement('div');
                leftDiv.style.display = 'flex';
                leftDiv.style.alignItems = 'center';
                
                const checkbox = document.createElement('input');
                checkbox.type = 'checkbox';
                checkbox.className = 'checkbox';
                checkbox.checked = todo.completed;
                checkbox.onclick = () => toggleTodo(index);
                
                const textSpan = document.createElement('span');
                textSpan.innerText = todo.text;
                
                leftDiv.appendChild(checkbox);
                leftDiv.appendChild(textSpan);
                
                const delBtn = document.createElement('button');
                delBtn.className = 'delete-btn';
                delBtn.style.position = 'static';
                delBtn.innerText = 'Delete';
                delBtn.onclick = () => deleteTodo(index);
                
                item.appendChild(leftDiv);
                item.appendChild(delBtn);
                container.appendChild(item);
            });
        }

        // Initial render on boot
        renderCards();
        renderTodos();
    </script>
</body>
</html>
