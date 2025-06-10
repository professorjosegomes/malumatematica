# malumatematica
Jogo da Malu - Matemática
!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Malu e os Animais</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @keyframes walk {
            0% { transform: translateX(0) scaleX(1); }
            50% { transform: translateX(10px) scaleX(1); }
            100% { transform: translateX(0) scaleX(1); }
        }
        
        @keyframes bounce {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-20px); }
        }
        
        @keyframes shake {
            0% { transform: translateX(0); }
            25% { transform: translateX(5px); }
            50% { transform: translateX(-5px); }
            75% { transform: translateX(5px); }
            100% { transform: translateX(0); }
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: scale(0.8); }
            to { opacity: 1; transform: scale(1); }
        }
        
        @keyframes confetti {
            0% { transform: translateY(0) rotate(0deg); opacity: 1; }
            100% { transform: translateY(100vh) rotate(360deg); opacity: 0; }
        }
        
        .animate-walk {
            animation: walk 1s infinite;
        }
        
        .animate-bounce {
            animation: bounce 0.5s infinite;
        }
        
        .animate-shake {
            animation: shake 0.5s;
        }
        
        .animate-fadein {
            animation: fadeIn 0.5s;
        }
        
        .confetti {
            position: absolute;
            width: 10px;
            height: 10px;
            background-color: #f00;
            animation: confetti 3s linear forwards;
        }
        
        .cell[data-type="start"] {
            background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="50" r="40" fill="%23f59e0b"/><text x="50" y="55" font-family="Arial" font-size="40" text-anchor="middle" fill="white">🚪</text></svg>');
        }
        
        .cell[data-type="animal"] {
            background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="50" r="40" fill="%2383cc16"/><text x="50" y="55" font-family="Arial" font-size="40" text-anchor="middle" fill="white">🐇</text></svg>');
        }
        
        .cell[data-type="end"] {
            background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="50" r="40" fill="%23ef4444"/><text x="50" y="55" font-family="Arial" font-size="40" text-anchor="middle" fill="white">🏁</text></svg>');
        }
        
        .cell[data-type="math"] {
            background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="50" r="40" fill="%233b82f6"/><text x="50" y="55" font-family="Arial" font-size="40" text-anchor="middle" fill="white">?</text></svg>');
        }
        
        .cell[data-player="true"] {
            box-shadow: 0 0 20px 5px rgba(255, 255, 0, 0.8);
        }
        
        .animal-rescued {
            opacity: 0.5;
            filter: grayscale(1);
        }
    </style>
</head>
<body class="bg-green-100 min-h-screen flex flex-col items-center justify-center font-sans overflow-hidden">
    <div class="game-container w-full max-w-4xl mx-auto p-4">
        <!-- Game Header -->
        <div class="header bg-green-800 text-white rounded-t-lg p-4 shadow-lg">
            <div class="flex justify-between items-center">
                <h1 class="text-2xl md:text-3xl font-bold">Malu's Math Adventure</h1>
                <div class="stats flex gap-4 items-center">
                    <div class="rescue-count">
                        <span class="text-yellow-300">🐾</span> 
                        <span id="rescuedCount">0</span>/<span id="totalAnimals">0</span>
                    </div>
                    <div class="level">
                        <span class="text-yellow-300">✨</span> 
                        Nível <span id="currentLevel">1</span>
                    </div>
                </div>
            </div>
        </div>
        
        <!-- Game Board -->
        <div class="game-board bg-green-700 p-4 rounded-b-lg shadow-lg relative overflow-hidden">
            <!-- Maze Container -->
            <div class="maze-container p-4 bg-green-900 bg-opacity-30 rounded-lg relative">
                <!-- Maze Grid -->
                <div id="maze" class="grid grid-cols-5 gap-2"></div>
                
                <!-- Player Malu -->
                <div id="player" class="absolute transition-all duration-500 ease-in-out">
                    <div class="w-10 h-10 rounded-full bg-purple-500 flex items-center justify-center text-white text-xl animate-walk">
                        👧
                    </div>
                </div>
            </div>
            
            <!-- Forest Decorations -->
            <div class="absolute top-0 left-0 w-full h-full pointer-events-none opacity-50">
                <div class="tree absolute top-2 left-2 text-green-900 text-3xl">🌲</div>
                <div class="tree absolute top-2 right-2 text-green-900 text-3xl">🌲</div>
                <div class="tree absolute bottom-2 left-4 text-green-900 text-3xl">🌳</div>
                <div class="tree absolute bottom-2 right-4 text-green-900 text-3xl">🌳</div>
                <div class="tree absolute top-10 left-1/4 text-green-900 text-3xl">🌴</div>
                <div class="tree absolute top-20 right-1/4 text-green-900 text-3xl">🌴</div>
            </div>
        </div>
        
        <!-- Math Problem Panel -->
        <div id="mathPanel" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center hidden z-50">
            <div class="bg-white rounded-xl p-6 w-full max-w-md mx-4 shadow-2xl animate-fadein">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-xl font-bold text-blue-600">Resolva para ajudar Malu!</h2>
                    <button id="closeMathBtn" class="text-gray-500 hover:text-gray-700">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
                <div id="problem" class="text-4xl font-bold text-center my-5"></div>
                <div class="options grid grid-cols-3 gap-3 my-5">
                    <button class="option bg-blue-100 hover:bg-blue-200 text-blue-800 py-3 px-4 rounded-lg text-xl font-bold transition-all hover:scale-105"></button>
                    <button class="option bg-blue-100 hover:bg-blue-200 text-blue-800 py-3 px-4 rounded-lg text-xl font-bold transition-all hover:scale-105"></button>
                    <button class="option bg-blue-100 hover:bg-blue-200 text-blue-800 py-3 px-4 rounded-lg text-xl font-bold transition-all hover:scale-105"></button>
                </div>
                <div id="feedback" class="text-center text-xl font-bold my-3"></div>
            </div>
        </div>
        
        <!-- Game Over Panel -->
        <div id="gameOverPanel" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center hidden z-50">
            <div class="bg-white rounded-xl p-6 w-full max-w-md mx-4 shadow-2xl text-center animate-fadein">
                <div class="text-6xl mb-4">🎉</div>
                <h2 class="text-3xl font-bold text-green-600 mb-4">Parabéns!</h2>
                <p id="resultsText" class="text-lg mb-4">Você ajudou Malu a resgatar todos os animais!</p>
                <button id="nextLevelBtn" class="bg-green-500 hover:bg-green-600 text-white font-bold py-3 px-6 rounded-lg text-lg w-full transition-colors">
                    Próxima Aventura
                </button>
            </div>
        </div>
        
        <!-- Animals Rescued Panel -->
        <div id="animalsPanel" class="fixed bottom-4 right-4 bg-white bg-opacity-90 rounded-lg shadow-lg p-4 z-40">
            <h3 class="font-bold text-sm mb-2 text-center">Animais Resgatados</h3>
            <div id="rescuedAnimals" class="flex gap-2 flex-wrap justify-center"></div>
        </div>
    </div>
    
    <!-- Audio Elements -->
    <audio id="correctSound" src="https://assets.mixkit.co/sfx/preview/mixkit-correct-answer-tone-2870.mp3" preload="auto"></audio>
    <audio id="wrongSound" src="https://assets.mixkit.co/sfx/preview/mixkit-wrong-answer-fail-notification-946.mp3" preload="auto"></audio>
    <audio id="winSound" src="https://assets.mixkit.co/sfx/preview/mixkit-winning-chimes-2015.mp3" preload="auto"></audio>
    <audio id="forestSound" src="https://assets.mixkit.co/sfx/preview/mixkit-forest-ambience-2703.mp3" preload="auto" loop></audio>
    
    <script>
        // Game state
        const gameState = {
            currentLevel: 1,
            playerPosition: 0,
            rescuedAnimals: [],
            totalAnimals: 0,
            currentProblem: null,
            maze: []
        };
        
        // Available animals
        const animals = ["🐇", "🐿️", "🦉", "🦊", "🐒", "🦔", "🦆", "🦝", "🐢", "🦋", "🐝", "🐞"];
        
        // Sound effects
        const correctSound = document.getElementById('correctSound');
        const wrongSound = document.getElementById('wrongSound');
        const winSound = document.getElementById('winSound');
        const forestSound = document.getElementById('forestSound');
        
        // Elements
        const mazeElement = document.getElementById('maze');
        const playerElement = document.getElementById('player');
        const mathPanel = document.getElementById('mathPanel');
        const problemElement = document.getElementById('problem');
        const optionButtons = document.querySelectorAll('.option');
        const feedbackElement = document.getElementById('feedback');
        const rescuedCountElement = document.getElementById('rescuedCount');
        const totalAnimalsElement = document.getElementById('totalAnimals');
        const currentLevelElement = document.getElementById('currentLevel');
        const rescuedAnimalsElement = document.getElementById('rescuedAnimals');
        const gameOverPanel = document.getElementById('gameOverPanel');
        const resultsText = document.getElementById('resultsText');
        const nextLevelBtn = document.getElementById('nextLevelBtn');
        const closeMathBtn = document.getElementById('closeMathBtn');
        
        // Initialize game
        function initGame() {
            // Start forest sounds
            forestSound.volume = 0.3;
            forestSound.play().catch(e => console.log("Audio play error:", e));
            
            gameState.currentLevel = 1;
            loadLevel(gameState.currentLevel);
            updateUI();
        }
        
        // Load a level
        function loadLevel(level) {
            // Reset game state
            gameState.playerPosition = 0;
            gameState.rescuedAnimals = [];
            gameState.maze = [];
            
            // Level configuration (size increases with level)
            const mazeSize = Math.min(5 + Math.floor(level / 2), 15);
            gameState.totalAnimals = Math.min(3 + level, 8);
            
            // Generate maze path
            generateMaze(mazeSize);
            
            // Update UI
            renderMaze();
            positionPlayer();
            updateUI();
        }
        
        // Generate maze path
        function generateMaze(size) {
            // Simple linear path for this example (could be made more complex)
            for (let i = 0; i < size; i++) {
                if (i === 0) {
                    gameState.maze.push({ type: 'start', completed: false });
                } else if (i === size - 1) {
                    gameState.maze.push({ type: 'end', completed: false });
                } else {
                    // Alternate between animal and math cells, with higher chance of math cells
                    if (i % 3 === 0 && gameState.rescuedAnimals.length < gameState.totalAnimals) {
                        gameState.maze.push({ 
                            type: 'animal', 
                            animal: animals[i % animals.length],
                            completed: false 
                        });
                    } else {
                        gameState.maze.push({ 
                            type: 'math', 
                            completed: false,
                            problem: generateProblem(gameState.currentLevel)
                        });
                    }
                }
            }
        }
        
        // Generate a math problem
        function generateProblem(level) {
            // Problems get slightly harder as levels increase
            const maxNumber = Math.min(5 + level, 10);
            const isAddition = Math.random() > 0.5;
            
            if (isAddition) {
                const a = Math.floor(Math.random() * maxNumber) + 1;
                const b = Math.floor(Math.random() * maxNumber) + 1;
                const answer = a + b;
                return {
                    question: `${a} + ${b} = ?`,
                    answer: answer,
                    options: generateOptions(answer, maxNumber * 2)
                };
            } else {
                const a = Math.floor(Math.random() * maxNumber) + 2;
                const b = Math.floor(Math.random() * (a - 1)) + 1;
                const answer = a - b;
                return {
                    question: `${a} - ${b} = ?`,
                    answer: answer,
                    options: generateOptions(answer, maxNumber)
                };
            }
        }
        
        // Generate answer options
        function generateOptions(correct, max) {
            const options = [correct];
            
            while (options.length < 3) {
                let randomOption;
                do {
                    randomOption = correct + Math.floor(Math.random() * 5) - 2; // -2 to +2 from correct
                    randomOption = Math.max(1, Math.min(randomOption, max)); // Keep within 1-max range
                } while (options.includes(randomOption));
                
                options.push(randomOption);
            }
            
            return shuffleArray(options);
        }
        
        // Shuffle array
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }
        
        // Render the maze
        function renderMaze() {
            mazeElement.innerHTML = '';
            mazeElement.className = `grid gap-2`;
            
            // Calculate grid cols based on maze size (limited to screen size)
            const cols = Math.min(gameState.maze.length, 10);
            mazeElement.classList.add(`grid-cols-${cols}`);
            
            gameState.maze.forEach((cell, index) => {
                const cellElement = document.createElement('div');
                cellElement.className = `cell aspect-square rounded-lg bg-white bg-opacity-30 bg-contain bg-center bg-no-repeat relative transition-all`;
                cellElement.dataset.index = index;
                cellElement.dataset.type = cell.type;
                
                if (cell.type === 'animal' && !cell.completed) {
                    cellElement.innerHTML = `<div class="absolute inset-0 flex items-center justify-center text-3xl animate-bounce">${cell.animal}</div>`;
                }
                
                if (index === gameState.playerPosition) {
                    cellElement.dataset.player = true;
                }
                
                if (cell.completed) {
                    cellElement.classList.add('opacity-70');
                }
                
                mazeElement.appendChild(cellElement);
            });
        }
        
        // Position player
        function positionPlayer() {
            const mazeRect = mazeElement.getBoundingClientRect();
            const cols = Math.min(gameState.maze.length, 10);
            const cellSize = mazeRect.width / cols;
            
            const row = Math.floor(gameState.playerPosition / cols);
            const col = gameState.playerPosition % cols;
            
            playerElement.style.left = `${col * cellSize + cellSize/2 - 20}px`;
            playerElement.style.top = `${row * cellSize + cellSize/2 - 20}px`;
            
            // Update maze cells
            document.querySelectorAll('.cell[data-player="true"]').forEach(el => el.dataset.player = false);
            document.querySelector(`.cell[data-index="${gameState.playerPosition}"]`).dataset.player = true;
        }
        
        // Show math problem
        function showMathProblem(problem) {
            gameState.currentProblem = problem;
            problemElement.textContent = problem.question;
            
            optionButtons.forEach((btn, i) => {
                btn.textContent = problem.options[i];
                btn.onclick = () => checkAnswer(problem.options[i]);
            });
            
            feedbackElement.textContent = '';
            feedbackElement.className = 'text-center text-xl font-bold my-3';
            
            mathPanel.classList.remove('hidden');
        }
        
        // Check answer
        function checkAnswer(selectedAnswer) {
            if (selectedAnswer == gameState.currentProblem.answer) {
                // Correct answer
                correctSound.currentTime = 0;
                correctSound.play().catch(e => console.log(e));
                
                feedbackElement.textContent = 'Correto! 🎉';
                feedbackElement.className = 'text-center text-xl font-bold my-3 text-green-600';
                
                // Create confetti effect
                createConfetti();
                
                // Mark current cell as completed
                gameState.maze[gameState.playerPosition].completed = true;
                
                // Move player after a delay
                setTimeout(() => {
                    mathPanel.classList.add('hidden');
                    
                    // Check if we've reached the end
                    if (gameState.playerPosition === gameState.maze.length - 1) {
                        checkGameCompletion();
                    } else {
                        // Move to next cell
                        gameState.playerPosition++;
                        positionPlayer();
                        renderMaze();
                        
                        // Check what's in the new cell
                        checkCell();
                    }
                }, 1000);
            } else {
                // Wrong answer
                wrongSound.currentTime = 0;
                wrongSound.play().catch(e => console.log(e));
                
                feedbackElement.textContent = 'Tente novamente!';
                feedbackElement.className = 'text-center text-xl font-bold my-3 text-red-600 animate-shake';
                
                playerElement.classList.add('animate-shake');
                setTimeout(() => playerElement.classList.remove('animate-shake'), 500);
            }
        }
        
        // Check what's in the current cell
        function checkCell() {
            const currentCell = gameState.maze[gameState.playerPosition];
            
            if (!currentCell.completed) {
                switch (currentCell.type) {
                    case 'animal':
                        rescueAnimal();
                        break;
                    case 'math':
                        showMathProblem(currentCell.problem);
                        break;
                    case 'end':
                        checkGameCompletion();
                        break;
                }
            } else if (gameState.playerPosition < gameState.maze.length - 1) {
                // If cell is completed, automatically move to next
                setTimeout(() => {
                    gameState.playerPosition++;
                    positionPlayer();
                    renderMaze();
                    checkCell();
                }, 300);
            }
        }
        
        // Rescue animal
        function rescueAnimal() {
            const currentCell = gameState.maze[gameState.playerPosition];
            
            gameState.maze[gameState.playerPosition].completed = true;
            gameState.rescuedAnimals.push(currentCell.animal);
            
            // Create animal rescue effect
            const cellElement = document.querySelector(`.cell[data-index="${gameState.playerPosition}"]`);
            if (cellElement) {
                cellElement.classList.add('animal-rescued');
                
                if (cellElement.firstChild) {
                    cellElement.firstChild.classList.remove('animate-bounce');
                    cellElement.firstChild.classList.add('animate-fadein');
                    setTimeout(() => {
                        cellElement.firstChild.style.opacity = '0.5';
                    }, 500);
                }
            }
            
            correctSound.currentTime = 0;
            correctSound.play().catch(e => console.log(e));
            
            createConfetti();
            
            updateUI();
            
            // Move to next cell after brief delay or check game completion
            setTimeout(() => {
                if (gameState.playerPosition === gameState.maze.length - 1) {
                    checkGameCompletion();
                } else {
                    gameState.playerPosition++;
                    positionPlayer();
                    renderMaze();
                    checkCell();
                }
            }, 800);
        }
        
        // Check if game is completed
        function checkGameCompletion() {
            if (gameState.playerPosition === gameState.maze.length - 1) {
                // Check if all animals are rescued
                if (gameState.rescuedAnimals.length === gameState.totalAnimals) {
                    // Game completed successfully
                    winSound.currentTime = 0;
                    winSound.play().catch(e => console.log(e));
                    
                    createConfetti(50);
                    
                    resultsText.textContent = `Você ajudou Malu a resgatar todos os ${gameState.totalAnimals} animais!`;
                    gameOverPanel.classList.remove('hidden');
                } else {
                    // Not all animals rescued
                    nextLevelBtn.textContent = 'Tentar Novamente';
                    resultsText.textContent = `Você resgatou ${gameState.rescuedAnimals.length} de ${gameState.totalAnimals} animais. Tente novamente!`;
                    gameOverPanel.classList.remove('hidden');
                }
            }
        }
        
        // Update UI elements
        function updateUI() {
            rescuedCountElement.textContent = gameState.rescuedAnimals.length;
            totalAnimalsElement.textContent = gameState.totalAnimals;
            currentLevelElement.textContent = gameState.currentLevel;
            
            // Update rescued animals display
            rescuedAnimalsElement.innerHTML = '';
            animals.forEach(animal => {
                const icon = document.createElement('div');
                icon.className = `text-2xl ${gameState.rescuedAnimals.includes(animal) ? '' : 'opacity-20'}`;
                icon.textContent = animal;
                rescuedAnimalsElement.appendChild(icon);
            });
        }
        
        // Create confetti effect
        function createConfetti(count = 20) {
            const container = document.querySelector('.game-board');
            
            for (let i = 0; i < count; i++) {
                const confetti = document.createElement('div');
                confetti.className = 'confetti';
                confetti.style.left = `${Math.random() * 100}%`;
                confetti.style.backgroundColor = `hsl(${Math.random() * 360}, 100%, 50%)`;
                confetti.style.animationDuration = `${2 + Math.random() * 3}s`;
                container.appendChild(confetti);
                
                setTimeout(() => {
                    confetti.remove();
                }, 3000);
            }
        }
        
        // Event listeners
        closeMathBtn.addEventListener('click', () => {
            mathPanel.classList.add('hidden');
        });
        
        nextLevelBtn.addEventListener('click', () => {
            gameOverPanel.classList.add('hidden');
            gameState.currentLevel++;
            loadLevel(gameState.currentLevel);
        });
        
        // Start the game when a cell is clicked (for mobile)
        mazeElement.addEventListener('click', (e) => {
            const cell = e.target.closest('.cell');
            if (cell) {
                const index = parseInt(cell.dataset.index);
                if (index === gameState.playerPosition) {
                    checkCell();
                }
            }
        });
        
        // Keyboard support for accessibility
        document.addEventListener('keydown', (e) => {
            if (mathPanel.classList.contains('hidden')) {
                if (e.key === 'ArrowRight' && gameState.playerPosition < gameState.maze.length - 1) {
                    gameState.playerPosition++;
                    positionPlayer();
                    renderMaze();
                    checkCell();
                } else if (e.key === 'ArrowLeft' && gameState.playerPosition > 0) {
                    gameState.playerPosition--;
                    positionPlayer();
                    renderMaze();
                } else if (e.key === 'Enter' || e.key === ' ') {
                    checkCell();
                }
            }
        });
        
        // Initialize the game
        initGame();
    </script>
</body>
</html>
