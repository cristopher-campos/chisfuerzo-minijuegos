<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chisfuerzo Minijuegos de Áreas</title>
    <!-- Enlace a Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- MathJax para renderizar ecuaciones LaTeX (útil para las descripciones de los juegos y pistas) -->
    <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
    <script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
    <!-- Tone.js para efectos de sonido -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.min.js"></script>
    <style>
        /* Fuentes personalizadas para una mejor estética */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #dcfce7; /* Color de fondo verde suave (green-50) */
            color: #334155; /* Color de texto principal */
        }
        /* Estilos para las tarjetas de juego */
        .game-card {
            transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
            cursor: pointer;
        }
        .game-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
        }
        /* Estilos para los botones de navegación y acción del juego */
        .nav-button, .game-action-button, .level-button {
            background-image: linear-gradient(to right, #6366f1, #8b5cf6); /* Gradiente morado-azul */
            transition: background-color 0.3s ease;
        }
        .nav-button:hover, .game-action-button:hover, .level-button:hover {
            background-image: linear-gradient(to right, #4f46e5, #7c3aed);
        }
        .level-button.active {
            background-image: linear-gradient(to right, #4f46e5, #7c3aed);
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        }
        /* Estilos para los campos de entrada */
        .input-field {
            border: 1px solid #cbd5e1;
            padding: 0.75rem 1rem;
            border-radius: 0.5rem;
            outline: none;
            transition: border-color 0.2s ease-in-out;
        }
        .input-field:focus {
            border-color: #6366f1;
            box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.2);
        }
        /* Estilos para los mensajes de resultado */
        .correct-message {
            color: #22c55e; /* Verde */
        }
        .incorrect-message {
            color: #ef4444; /* Rojo */
        }
        /* Estilos para el SVG del juego */
        .game-svg-container {
            border: 2px solid #a78bfa; /* Borde morado claro */
            background-color: #ede9fe; /* Fondo morado muy claro */
            border-radius: 0.75rem;
            overflow: hidden; /* Asegura que la forma no se salga */
        }
        .shape-fill {
            /* Color de relleno de la forma - se asignará dinámicamente */
            stroke-width: 2;
        }
        .dimension-line {
            stroke: #ef4444; /* Rojo */
            stroke-width: 1.5;
            stroke-dasharray: 4,4;
        }
        .dimension-text {
            fill: #334155; /* Color de texto oscuro */
            font-size: 14px;
            font-weight: 600;
        }
        /* Estilos para los stickers */
        .sticker {
            position: absolute;
            font-size: 4rem; /* Tamaño grande para los emojis */
            opacity: 0.3; /* Ligeramente transparentes para no distraer */
            z-index: -1; /* Detrás del contenido principal */
        }
        /* Estilos para las opciones de los juegos de adivinanzas */
        .phrase-option-button, .work-option-button, .english-option-button, .verb-option-button, .general-knowledge-option-button {
            background-color: #f3f4f6; /* Gris claro */
            color: #334155;
            padding: 0.75rem 1.5rem;
            border-radius: 0.5rem;
            font-weight: 600;
            transition: background-color 0.2s ease-in-out, transform 0.1s ease-in-out;
            border: 1px solid #cbd5e1;
            width: 100%;
            text-align: center;
        }
        .phrase-option-button:hover, .work-option-button:hover, .english-option-button:hover, .verb-option-button:hover, .general-knowledge-option-button:hover {
            background-color: #e5e7eb;
            transform: translateY(-2px);
        }
        .phrase-option-button.correct, .work-option-button.correct, .english-option-button.correct, .verb-option-button.correct, .general-knowledge-option-button.correct {
            background-color: #d1fae5; /* Verde muy claro */
            border-color: #34d399; /* Verde */
        }
        .phrase-option-button.incorrect, .work-option-button.incorrect, .english-option-button.incorrect, .verb-option-button.incorrect, .general-knowledge-option-button.incorrect {
            background-color: #fee2e2; /* Rojo muy claro */
            border-color: #ef4444; /* Rojo */
        }
        /* Estilo específico para los botones de nivel */
        .phrase-level-button, .work-level-button, .english-level-button, .verb-level-button, .general-knowledge-level-button {
            color: #1d4ed8; /* Azul oscuro para el texto de los botones de nivel */
        }
    </style>
</head>
<body class="min-h-screen flex flex-col">

    <!-- Encabezado / Barra de Navegación -->
    <header class="bg-white shadow-md py-4 px-6 md:px-8">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-3xl font-bold text-indigo-700">Chisfuerzo <span class="text-purple-700">Minijuegos</span></h1>
            <nav>
                <ul class="flex space-x-6">
                    <li><a href="#" id="homeLink" class="text-gray-600 hover:text-indigo-700 font-semibold">Inicio</a></li>
                    <!-- Eliminado el botón "Acerca de" -->
                </ul>
            </nav>
        </div>
    </header>

    <!-- Sección Principal de Contenido -->
    <main class="flex-grow container mx-auto px-4 py-8 md:py-12 relative overflow-hidden" id="app">
        <!-- Stickers decorativos -->
        <div class="sticker top-10 left-5 transform rotate-12">📐</div>
        <div class="sticker bottom-5 right-10 transform -rotate-6">✨</div>
        <div class="sticker top-1/4 right-1/4 transform rotate-45">📏</div>
        <div class="sticker bottom-1/3 left-1/4 transform -rotate-12">🧠</div>
        <!-- El contenido de la aplicación se renderizará aquí -->
    </main>

    <!-- Pie de Página -->
    <footer class="bg-gray-800 text-white py-6 mt-auto">
        <div class="container mx-auto text-center">
            <p>&copy; 2024 Chisfuerzo Minijuegos. Todos los derechos reservados.</p>
        </div>
    </footer>

    <script type="module">
        // --- Funciones de utilidad globales ---
        // Función para generar un número aleatorio en un rango
        function getRandomInt(min, max) {
            return Math.floor(Math.random() * (max - min + 1)) + min;
        }

        // Función para mezclar un array (Fisher-Yates shuffle)
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        // --- Sonidos ---
        // Inicializar Tone.js para los sonidos
        const correctSound = new Tone.Synth().toDestination();
        const incorrectSound = new Tone.MembraneSynth().toDestination();

        // Función para reproducir sonido de acierto
        function playCorrectSound() {
            correctSound.triggerAttackRelease("C5", "8n"); // C5 por un octavo de nota
        }

        // Función para reproducir sonido de fallo
        function playIncorrectSound() {
            incorrectSound.triggerAttackRelease("F#2", "8n"); // F#2 por un octavo de nota
        }

        // --- Contenido de los Minijuegos ---
        const areaGamesContent = {
            'Adivina el Área': {
                category: 'Matematicas',
                icon: '❓',
                description: 'Adivina el área de figuras geométricas con dimensiones aleatorias. ¡Gana puntos y usa pistas!',
                gameHtml: `
                    <div id="levelSelection" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Selecciona un Nivel de Dificultad</h3>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-lg justify-center">
                            <button class="level-button text-white font-bold py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Basico">
                                Básico
                            </button>
                            <button class="level-button text-white font-bold py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Intermedio">
                                Intermedio
                            </button>
                            <button class="level-button text-white font-bold py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Avanzado">
                                Avanzado
                            </button>
                        </div>
                    </div>

                    <div id="gamePlay" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg mt-8 hidden">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Adivina el Área de la Figura</h3>
                        <p class="text-gray-600 text-center mb-4">Calcula el área de la figura mostrada y escribe tu respuesta. ¡Buena suerte!</p>

                        <div id="scoreDisplay" class="text-2xl font-bold text-purple-700 mb-4">Puntos: 0</div>

                        <div id="gameCanvas" class="game-svg-container w-[400px] h-[300px] flex items-center justify-center">
                            <svg id="shapeSvg" width="100%" height="100%" viewBox="0 0 300 200">
                                <!-- Elementos de la forma se dibujarán aquí dinámicamente -->
                            </svg>
                        </div>

                        <div class="w-full max-w-sm">
                            <label for="userAreaGuess" class="block text-gray-700 text-sm font-bold mb-2">Tu Área:</label>
                            <input type="number" id="userAreaGuess" class="input-field w-full text-center text-lg" placeholder="Introduce tu respuesta" min="0" step="0.01">
                        </div>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-sm">
                            <button id="checkAreaBtn" class="game-action-button text-white font-bold py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1">
                                Verificar
                            </button>
                            <button id="hintBtn" class="game-action-button text-white font-bold py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-purple-300 flex-1">
                                Pista (-20 Puntos)
                            </button>
                        </div>
                        <p id="feedbackMessage" class="text-xl font-bold mt-4"></p>
                        <p id="hintMessage" class="text-gray-700 text-center mt-2 hidden"></p>
                        <button id="nextChallengeBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md hidden">
                            Siguiente Desafío
                        </button>
                    </div>
                `,
                gameScript: function() {
                    const levelSelectionDiv = document.getElementById('levelSelection');
                    const gamePlayDiv = document.getElementById('gamePlay');
                    const levelButtons = document.querySelectorAll('.level-button');

                    const shapeSvg = document.getElementById('shapeSvg');
                    const userAreaGuessInput = document.getElementById('userAreaGuess');
                    const checkAreaBtn = document.getElementById('checkAreaBtn');
                    const feedbackMessage = document.getElementById('feedbackMessage');
                    const nextChallengeBtn = document.getElementById('nextChallengeBtn');
                    const scoreDisplay = document.getElementById('scoreDisplay');
                    const hintBtn = document.getElementById('hintBtn');
                    const hintMessage = document.getElementById('hintMessage');

                    let currentLevel = null;
                    let currentShapeType;
                    let currentDimensions;
                    let correctArea;
                    let score = 0;
                    let hintUsedThisChallenge = false;
                    const PI = Math.PI;

                    // Paletas de colores para las formas
                    const shapeColorPalettes = [
                        { fill: '#8b5cf6', stroke: '#6d28d9' }, // Purple
                        { fill: '#22c55e', stroke: '#16a34a' }, // Green
                        { fill: '#f97316', stroke: '#ea580c' }, // Orange
                        { fill: '#0ea5e9', stroke: '#0284c7' }, // Sky Blue
                        { fill: '#facc15', stroke: '#eab308' }  // Yellow
                    ];

                    // Definición de formas por nivel
                    const levelShapes = {
                        'Basico': ['square', 'rectangle', 'triangle'],
                        'Intermedio': ['rhombus', 'trapezoid'],
                        'Avanzado': ['circle', 'composite'] // 'composite' will be a simple fixed one for now
                    };

                    // Fórmulas para la pista
                    const formulas = {
                        'square': 'Área = $l \\times l = l^2$',
                        'rectangle': 'Área = $b \\times h$',
                        'triangle': 'Área = $\\frac{b \\times h}{2}$',
                        'rhombus': 'Área = $\\frac{d_1 \\times d_2}{2}$',
                        'trapezoid': 'Área = $\\frac{(B + b) \\times h}{2}$',
                        'circle': 'Área = $\\pi r^2$',
                        'composite': 'Área = Suma de áreas de figuras simples'
                    };

                    // Función para actualizar la visualización de la puntuación
                    function updateScoreDisplay() {
                        scoreDisplay.textContent = `Puntos: ${score}`;
                        if (score < 20 || hintUsedThisChallenge) {
                            hintBtn.disabled = true;
                            hintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                        } else {
                            hintBtn.disabled = false;
                            hintBtn.classList.remove('opacity-50', 'cursor-not-allowed');
                        }
                    }

                    // Función para limpiar el SVG antes de dibujar una nueva forma
                    function clearSvg() {
                        while (shapeSvg.firstChild) {
                            shapeSvg.removeChild(shapeSvg.firstChild);
                        }
                    }

                    // Función para dibujar la forma y sus dimensiones en el SVG
                    function drawShape(type, dimensions) {
                        clearSvg();

                        const svgWidth = 300;
                        const svgHeight = 200;
                        const padding = 30; // Aumentado el padding para más espacio para el texto

                        // Seleccionar una paleta de colores aleatoria
                        const colorPalette = shapeColorPalettes[getRandomInt(0, shapeColorPalettes.length - 1)];

                        let shapeElement;
                        // let dimTexts = []; // Array para almacenar los elementos de texto de las dimensiones

                        // Helper para escalar y centrar la forma
                        function scaleAndCenter(width, height) {
                            const scaleX = (svgWidth - 2 * padding) / width;
                            const scaleY = (svgHeight - 2 * padding) / height;
                            const scale = Math.min(scaleX, scaleY);

                            const scaledWidth = width * scale;
                            const scaledHeight = height * scale;

                            const offsetX = (svgWidth - scaledWidth) / 2;
                            const offsetY = (svgHeight - scaledHeight) / 2;
                            return { scaledWidth, scaledHeight, offsetX, offsetY };
                        }

                        if (type === 'square' || type === 'rectangle') {
                            const { side1, side2 } = dimensions;
                            const { scaledWidth, scaledHeight, offsetX, offsetY } = scaleAndCenter(side1, side2);

                            shapeElement = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
                            shapeElement.setAttribute('x', offsetX);
                            shapeElement.setAttribute('y', offsetY);
                            shapeElement.setAttribute('width', scaledWidth);
                            shapeElement.setAttribute('height', scaledHeight);
                            shapeElement.classList.add('shape-fill');
                            shapeElement.style.fill = colorPalette.fill;
                            shapeElement.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(shapeElement);

                            // Líneas y textos de dimensión
                            // Base
                            let line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX);
                            line.setAttribute('y1', offsetY + scaledHeight + 10);
                            line.setAttribute('x2', offsetX + scaledWidth);
                            line.setAttribute('y2', offsetY + scaledHeight + 10);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledWidth / 2);
                            text.setAttribute('y', offsetY + scaledHeight + 25);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = side1;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            // Altura
                            line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX - 10);
                            line.setAttribute('y1', offsetY);
                            line.setAttribute('x2', offsetX - 10);
                            line.setAttribute('y2', offsetY + scaledHeight);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            // Ajuste para evitar que el texto se corte
                            text.setAttribute('x', offsetX - 25); // Mover más a la izquierda
                            text.setAttribute('y', offsetY + scaledHeight / 2);
                            text.setAttribute('text-anchor', 'start'); // Alinear al inicio
                            text.textContent = side2;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                        } else if (type === 'triangle') {
                            const { base, height } = dimensions;
                            const { scaledWidth, scaledHeight, offsetX, offsetY } = scaleAndCenter(base, height);

                            shapeElement = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
                            const points = `${offsetX + scaledWidth / 2},${offsetY} ${offsetX},${offsetY + scaledHeight} ${offsetX + scaledWidth},${offsetY + scaledHeight}`;
                            shapeElement.setAttribute('points', points);
                            shapeElement.classList.add('shape-fill');
                            shapeElement.style.fill = colorPalette.fill;
                            shapeElement.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(shapeElement);

                            // Línea de altura
                            let heightLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            heightLine.setAttribute('x1', offsetX + scaledWidth / 2);
                            heightLine.setAttribute('y1', offsetY);
                            heightLine.setAttribute('x2', offsetX + scaledWidth / 2);
                            heightLine.setAttribute('y2', offsetY + scaledHeight);
                            heightLine.classList.add('dimension-line');
                            shapeSvg.appendChild(heightLine);

                            // Línea de base
                            let baseLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            baseLine.setAttribute('x1', offsetX);
                            baseLine.setAttribute('y1', offsetY + scaledHeight + 10);
                            baseLine.setAttribute('x2', offsetX + scaledWidth);
                            baseLine.setAttribute('y2', offsetY + scaledHeight + 10);
                            baseLine.classList.add('dimension-line');
                            shapeSvg.appendChild(baseLine);

                            // Textos de dimensión
                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledWidth / 2);
                            text.setAttribute('y', offsetY + scaledHeight + 25);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = base;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledWidth / 2 + 5);
                            text.setAttribute('y', offsetY + scaledHeight / 2);
                            text.setAttribute('text-anchor', 'start');
                            text.textContent = height;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                        } else if (type === 'rhombus') {
                            const { d1, d2 } = dimensions; // d1 = diagonal horizontal, d2 = diagonal vertical
                            const { scaledWidth, scaledHeight, offsetX, offsetY } = scaleAndCenter(d1, d2);

                            shapeElement = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
                            const points = `${offsetX + scaledWidth / 2},${offsetY} ${offsetX + scaledWidth},${offsetY + scaledHeight / 2} ${offsetX + scaledWidth / 2},${offsetY + scaledHeight} ${offsetX},${offsetY + scaledHeight / 2}`;
                            shapeElement.setAttribute('points', points);
                            shapeElement.classList.add('shape-fill');
                            shapeElement.style.fill = colorPalette.fill;
                            shapeElement.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(shapeElement);

                            // Líneas de diagonales
                            let line1 = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line1.setAttribute('x1', offsetX);
                            line1.setAttribute('y1', offsetY + scaledHeight / 2);
                            line1.setAttribute('x2', offsetX + scaledWidth);
                            line1.setAttribute('y2', offsetY + scaledHeight / 2);
                            line1.classList.add('dimension-line');
                            shapeSvg.appendChild(line1);

                            let line2 = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line2.setAttribute('x1', offsetX + scaledWidth / 2);
                            line2.setAttribute('y1', offsetY);
                            line2.setAttribute('x2', offsetX + scaledWidth / 2);
                            line2.setAttribute('y2', offsetY + scaledHeight);
                            line2.classList.add('dimension-line');
                            shapeSvg.appendChild(line2);

                            // Textos de diagonales
                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledWidth / 2);
                            text.setAttribute('y', offsetY + scaledHeight / 2 - 10);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = `d1: ${d1}`;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledWidth / 2 + 10);
                            text.setAttribute('y', offsetY + scaledHeight / 2 + 20);
                            text.setAttribute('text-anchor', 'start');
                            text.textContent = `d2: ${d2}`;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                        } else if (type === 'trapezoid') {
                            const { B, b, h } = dimensions; // B = base mayor, b = base menor, h = altura
                            // Escalar para que B sea la base mayor y b la menor, h la altura
                            const { scaledWidth: scaledB, scaledHeight, offsetX, offsetY } = scaleAndCenter(B, h);
                            const scaledb = (b / B) * scaledB; // Escalar la base menor proporcionalmente

                            shapeElement = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
                            // Puntos: (top-left), (top-right), (bottom-right), (bottom-left)
                            const points = `${offsetX + (scaledB - scaledb) / 2},${offsetY} ${offsetX + scaledB - (scaledB - scaledb) / 2},${offsetY} ${offsetX + scaledB},${offsetY + scaledHeight} ${offsetX},${offsetY + scaledHeight}`;
                            shapeElement.setAttribute('points', points);
                            shapeElement.classList.add('shape-fill');
                            shapeElement.style.fill = colorPalette.fill;
                            shapeElement.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(shapeElement);

                            // Línea de altura
                            let heightLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            heightLine.setAttribute('x1', offsetX + scaledB / 2);
                            heightLine.setAttribute('y1', offsetY);
                            heightLine.setAttribute('x2', offsetX + scaledB / 2);
                            heightLine.setAttribute('y2', offsetY + scaledHeight);
                            heightLine.classList.add('dimension-line');
                            shapeSvg.appendChild(heightLine);

                            // Línea de base mayor
                            let baseBLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            baseBLine.setAttribute('x1', offsetX);
                            baseBLine.setAttribute('y1', offsetY + scaledHeight + 10);
                            baseBLine.setAttribute('x2', offsetX + scaledB);
                            baseBLine.setAttribute('y2', offsetY + scaledHeight + 10);
                            baseBLine.classList.add('dimension-line');
                            shapeSvg.appendChild(baseBLine);

                            // Línea de base menor
                            let basebLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            basebLine.setAttribute('x1', offsetX + (scaledB - scaledb) / 2);
                            basebLine.setAttribute('y1', offsetY - 10);
                            basebLine.setAttribute('x2', offsetX + scaledB - (scaledB - scaledb) / 2);
                            basebLine.setAttribute('y2', offsetY - 10);
                            basebLine.classList.add('dimension-line');
                            shapeSvg.appendChild(basebLine);

                            // Textos de dimensión
                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledB / 2);
                            text.setAttribute('y', offsetY + scaledHeight + 25);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = `B: ${B}`;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledB / 2);
                            text.setAttribute('y', offsetY - 25);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = `b: ${b}`;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledB / 2 + 5);
                            text.setAttribute('y', offsetY + scaledHeight / 2);
                            text.setAttribute('text-anchor', 'start');
                            text.textContent = `h: ${h}`;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                        } else if (type === 'circle') {
                            const { radius } = dimensions;
                            const maxRadius = Math.min(svgWidth, svgHeight) / 2 - padding;

                            const scaledRadius = (radius / 15) * maxRadius;
                            const displayRadius = Math.max(20, scaledRadius);

                            shapeElement = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
                            shapeElement.setAttribute('cx', svgWidth / 2);
                            shapeElement.setAttribute('cy', svgHeight / 2);
                            shapeElement.setAttribute('r', displayRadius);
                            shapeElement.classList.add('shape-fill');
                            shapeElement.style.fill = colorPalette.fill;
                            shapeElement.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(shapeElement);

                            // Línea de radio
                            let radiusLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            radiusLine.setAttribute('x1', svgWidth / 2);
                            radiusLine.setAttribute('y1', svgHeight / 2);
                            radiusLine.setAttribute('x2', svgWidth / 2 + displayRadius);
                            radiusLine.setAttribute('y2', svgHeight / 2);
                            radiusLine.classList.add('dimension-line');
                            shapeSvg.appendChild(radiusLine);

                            // Texto de radio
                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', svgWidth / 2 + displayRadius / 2);
                            text.setAttribute('y', svgHeight / 2 - 5);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = radius;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                        } else if (type === 'composite') {
                            // Figura compuesta simple: Rectángulo con un triángulo encima
                            // Dimensiones fijas para simplificar el desafío
                            const rectBase = 20;
                            const rectHeight = 10;
                            const triHeight = 5;

                            const { scaledWidth: scaledRectBase, scaledHeight: scaledTotalHeight, offsetX, offsetY } = scaleAndCenter(rectBase, rectHeight + triHeight);

                            const rectScaledHeight = (rectHeight / (rectHeight + triHeight)) * scaledTotalHeight;
                            const triScaledHeight = (triHeight / (rectHeight + triHeight)) * scaledTotalHeight;

                            // Rectángulo
                            let rect = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
                            rect.setAttribute('x', offsetX);
                            rect.setAttribute('y', offsetY + triScaledHeight);
                            rect.setAttribute('width', scaledRectBase);
                            rect.setAttribute('height', rectScaledHeight);
                            rect.classList.add('shape-fill');
                            rect.style.fill = colorPalette.fill;
                            rect.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(rect);

                            // Triángulo encima
                            let triangle = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
                            const triPoints = `${offsetX + scaledRectBase / 2},${offsetY} ${offsetX},${offsetY + triScaledHeight} ${offsetX + scaledRectBase},${offsetY + triScaledHeight}`;
                            triangle.setAttribute('points', triPoints);
                            triangle.classList.add('shape-fill');
                            triangle.style.fill = colorPalette.fill;
                            triangle.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(triangle);

                            // Dimensiones del rectángulo
                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledRectBase / 2);
                            text.setAttribute('y', offsetY + triScaledHeight + rectScaledHeight + 15);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = rectBase;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            // Ajuste para la figura compuesta:
                            text.setAttribute('x', offsetX - 25); // Mover más a la izquierda
                            text.setAttribute('y', offsetY + triScaledHeight + rectScaledHeight / 2);
                            text.setAttribute('text-anchor', 'start'); // Alinear al inicio
                            text.textContent = rectHeight;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            // Dimensión de altura del triángulo
                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledRectBase / 2 + 5);
                            text.setAttribute('y', offsetY + triScaledHeight / 2);
                            text.setAttribute('text-anchor', 'start');
                            text.textContent = triHeight;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            // Líneas de dimensión
                            let line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX);
                            line.setAttribute('y1', offsetY + triScaledHeight + rectScaledHeight + 10);
                            line.setAttribute('x2', offsetX + scaledRectBase);
                            line.setAttribute('y2', offsetY + triScaledHeight + rectScaledHeight + 10);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX - 10);
                            line.setAttribute('y1', offsetY + triScaledHeight);
                            line.setAttribute('x2', offsetX - 10);
                            line.setAttribute('y2', offsetY + triScaledHeight + rectScaledHeight);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX + scaledRectBase / 2);
                            line.setAttribute('y1', offsetY);
                            line.setAttribute('x2', offsetX + scaledRectBase / 2);
                            line.setAttribute('y2', offsetY + triScaledHeight);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);
                        }
                    }

                    // Función para calcular el área según el tipo de forma
                    function calculateArea(type, dims) {
                        if (type === 'square') {
                            return dims.side1 * dims.side1;
                        } else if (type === 'rectangle') {
                            return dims.side1 * dims.side2;
                        } else if (type === 'triangle') {
                            return (dims.base * dims.height) / 2;
                        } else if (type === 'rhombus') {
                            return (dims.d1 * dims.d2) / 2;
                        } else if (type === 'trapezoid') {
                            return ((dims.B + dims.b) * dims.h) / 2;
                        } else if (type === 'circle') {
                            return PI * dims.radius * dims.radius;
                        } else if (type === 'composite') {
                            // Área de un rectángulo (20x10) + un triángulo (base 20, altura 5)
                            const rectArea = 20 * 10;
                            const triArea = (20 * 5) / 2;
                            return rectArea + triArea;
                        }
                        return 0;
                    }

                    // Función para mostrar la pista (fórmula)
                    function displayHint(type) {
                        hintMessage.innerHTML = formulas[type];
                        hintMessage.classList.remove('hidden');
                        if (typeof MathJax !== 'undefined') {
                            MathJax.typesetPromise(); // Renderizar la fórmula LaTeX
                        }
                    }

                    // Función para generar un nuevo desafío
                    function generateNewChallenge() {
                        const availableShapes = levelShapes[currentLevel];
                        currentShapeType = availableShapes[getRandomInt(0, availableShapes.length - 1)];

                        if (currentShapeType === 'square') {
                            // Rango más pequeño para que el cuadrado se vea más pequeño en el recuadro grande
                            const side = getRandomInt(3, 10);
                            currentDimensions = { side1: side, side2: side };
                        } else if (currentShapeType === 'rectangle') {
                            const side1 = getRandomInt(5, 18); // Ajustado el rango
                            let side2 = getRandomInt(3, 9);    // Ajustado el rango
                            while (side2 === side1) {
                                side2 = getRandomInt(3, 9);
                            }
                            currentDimensions = { side1: side1, side2: side2 };
                        } else if (currentShapeType === 'triangle') {
                            const base = getRandomInt(6, 18);
                            const height = getRandomInt(4, 12);
                            currentDimensions = { base: base, height: height };
                        } else if (currentShapeType === 'rhombus') {
                            const d1 = getRandomInt(10, 22);
                            let d2 = getRandomInt(5, 12);
                            while (d2 >= d1) { // d2 debe ser menor que d1
                                d2 = getRandomInt(5, 12);
                            }
                            currentDimensions = { d1: d1, d2: d2 };
                        } else if (currentShapeType === 'trapezoid') {
                            const B = getRandomInt(12, 22); // Base mayor
                            let b = getRandomInt(5, B - 5); // Base menor, al menos 5 unidades menor que B
                            const h = getRandomInt(4, 10); // Altura
                            currentDimensions = { B: B, b: b, h: h };
                        } else if (currentShapeType === 'circle') {
                            const radius = getRandomInt(3, 8); // Rango más pequeño para el círculo
                            currentDimensions = { radius: radius };
                        } else if (currentShapeType === 'composite') {
                            // Para el nivel avanzado, la figura compuesta es fija para un desafío específico
                            currentDimensions = {}; // No hay dimensiones aleatorias para esta figura fija
                        }

                        correctArea = calculateArea(currentShapeType, currentDimensions);

                        drawShape(currentShapeType, currentDimensions);

                        userAreaGuessInput.value = '';
                        feedbackMessage.textContent = '';
                        feedbackMessage.classList.remove('correct-message', 'incorrect-message');
                        hintMessage.classList.add('hidden');
                        checkAreaBtn.disabled = false;
                        checkAreaBtn.classList.remove('opacity-50', 'cursor-not-allowed');
                        nextChallengeBtn.classList.add('hidden');
                        hintUsedThisChallenge = false;
                        updateScoreDisplay();
                    }

                    // Event listener para verificar la respuesta
                    checkAreaBtn.addEventListener('click', () => {
                        const userAnswer = parseFloat(userAreaGuessInput.value);

                        if (isNaN(userAnswer)) {
                            feedbackMessage.textContent = 'Por favor, introduce un número válido.';
                            feedbackMessage.classList.add('incorrect-message');
                            return;
                        }

                        if (Math.abs(userAnswer - correctArea) < 0.02) {
                            playCorrectSound(); // Play sound on correct answer
                            feedbackMessage.textContent = `¡Correcto! El área es ${correctArea.toFixed(2)}.`;
                            feedbackMessage.classList.remove('incorrect-message');
                            feedbackMessage.classList.add('correct-message');
                            score += 10;
                            updateScoreDisplay();
                            checkAreaBtn.disabled = true;
                            checkAreaBtn.classList.add('opacity-50', 'cursor-not-allowed');
                            nextChallengeBtn.classList.remove('hidden');
                            hintBtn.disabled = true;
                            hintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                        } else {
                            playIncorrectSound(); // Play sound on incorrect answer
                            feedbackMessage.textContent = 'Incorrecto. Inténtalo de nuevo.';
                            feedbackMessage.classList.remove('correct-message');
                            feedbackMessage.classList.add('incorrect-message');
                        }
                    });

                    // Event listener para el botón de pista
                    hintBtn.addEventListener('click', () => {
                        if (score >= 20 && !hintUsedThisChallenge) {
                            score -= 20;
                            updateScoreDisplay();
                            displayHint(currentShapeType);
                            hintUsedThisChallenge = true;
                            hintBtn.disabled = true;
                            hintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                        } else if (hintUsedThisChallenge) {
                            feedbackMessage.textContent = 'Ya usaste la pista para este desafío.';
                            feedbackMessage.classList.add('incorrect-message');
                        } else {
                            feedbackMessage.textContent = 'No tienes suficientes puntos para una pista (necesitas 20).';
                            feedbackMessage.classList.add('incorrect-message');
                        }
                    });

                    // Event listener para el siguiente desafío
                    nextChallengeBtn.addEventListener('click', generateNewChallenge);

                    // Permitir presionar Enter para verificar
                    userAreaGuessInput.addEventListener('keypress', (event) => {
                        if (event.key === 'Enter' && !checkAreaBtn.disabled) {
                            checkAreaBtn.click();
                        }
                    });

                    // Lógica de selección de nivel
                    levelButtons.forEach(button => {
                        button.addEventListener('click', (event) => {
                            currentLevel = event.target.dataset.level;
                            levelSelectionDiv.classList.add('hidden');
                            gamePlayDiv.classList.remove('hidden');
                            generateNewChallenge(); // Iniciar el primer desafío del nivel
                            updateScoreDisplay(); // Asegurarse de que la puntuación inicial sea visible
                        });
                    });

                    // Ocultar el juego al inicio, mostrar solo la selección de nivel
                    gamePlayDiv.classList.add('hidden');
                    levelSelectionDiv.classList.remove('hidden');
                }
            },
            'Adivina el Perímetro': {
                category: 'Matematicas',
                icon: '📏',
                description: 'Calcula el perímetro de figuras geométricas con dimensiones aleatorias. ¡Gana puntos y usa pistas!',
                gameHtml: `
                    <div id="perimeterLevelSelection" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Selecciona un Nivel de Dificultad</h3>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-lg justify-center">
                            <button class="level-button text-white font-bold py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Basico">
                                Básico
                            </button>
                            <button class="level-button text-white font-bold py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Intermedio">
                                Intermedio
                            </button>
                            <button class="level-button text-white font-bold py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Avanzado">
                                Avanzado
                            </button>
                        </div>
                    </div>

                    <div id="perimeterGamePlay" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg mt-8 hidden">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Adivina el Perímetro de la Figura</h3>
                        <p class="text-gray-600 text-center mb-4">Calcula el perímetro de la figura mostrada y escribe tu respuesta. ¡Buena suerte!</p>

                        <div id="perimeterScoreDisplay" class="text-2xl font-bold text-purple-700 mb-4">Puntos: 0</div>

                        <div id="perimeterGameCanvas" class="game-svg-container w-[400px] h-[300px] flex items-center justify-center">
                            <svg id="perimeterShapeSvg" width="100%" height="100%" viewBox="0 0 300 200">
                                <!-- Elementos de la forma se dibujarán aquí dinámicamente -->
                            </svg>
                        </div>

                        <div class="w-full max-w-sm">
                            <label for="userPerimeterGuess" class="block text-gray-700 text-sm font-bold mb-2">Tu Perímetro:</label>
                            <input type="number" id="userPerimeterGuess" class="input-field w-full text-center text-lg" placeholder="Introduce tu respuesta" min="0" step="0.01">
                        </div>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-sm">
                            <button id="checkPerimeterBtn" class="game-action-button text-white font-bold py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1">
                                Verificar
                            </button>
                            <button id="perimeterHintBtn" class="game-action-button text-white font-bold py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-purple-300 flex-1">
                                Pista (-20 Puntos)
                            </button>
                        </div>
                        <p id="perimeterFeedbackMessage" class="text-xl font-bold mt-4"></p>
                        <p id="perimeterHintMessage" class="text-gray-700 text-center mt-2 hidden"></p>
                        <button id="nextPerimeterChallengeBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md hidden">
                            Siguiente Desafío
                        </button>
                    </div>
                `,
                gameScript: function() {
                    const levelSelectionDiv = document.getElementById('perimeterLevelSelection');
                    const gamePlayDiv = document.getElementById('perimeterGamePlay');
                    const levelButtons = document.querySelectorAll('#perimeterLevelSelection .level-button'); // Select specifically for perimeter

                    const shapeSvg = document.getElementById('perimeterShapeSvg');
                    const userPerimeterGuessInput = document.getElementById('userPerimeterGuess');
                    const checkPerimeterBtn = document.getElementById('checkPerimeterBtn');
                    const feedbackMessage = document.getElementById('perimeterFeedbackMessage');
                    const nextChallengeBtn = document.getElementById('nextPerimeterChallengeBtn');
                    const scoreDisplay = document.getElementById('perimeterScoreDisplay');
                    const hintBtn = document.getElementById('perimeterHintBtn');
                    const hintMessage = document.getElementById('perimeterHintMessage');

                    let currentLevel = null;
                    let currentShapeType;
                    let currentDimensions;
                    let correctPerimeter;
                    let score = 0;
                    let hintUsedThisChallenge = false;
                    const PI = Math.PI;

                    // Paletas de colores para las formas
                    const shapeColorPalettes = [
                        { fill: '#8b5cf6', stroke: '#6d28d9' }, // Purple
                        { fill: '#22c55e', stroke: '#16a34a' }, // Green
                        { fill: '#f97316', stroke: '#ea580c' }, // Orange
                        { fill: '#0ea5e9', stroke: '#0284c7' }, // Sky Blue
                        { fill: '#facc15', stroke: '#eab308' }  // Yellow
                    ];

                    // Definición de formas por nivel
                    const levelShapes = {
                        'Basico': ['square', 'rectangle', 'triangle'],
                        'Intermedio': ['rhombus', 'trapezoid'],
                        'Avanzado': ['circle', 'composite']
                    };

                    // Fórmulas para la pista (Perímetro)
                    const formulas = {
                        'square': 'Perímetro = $4 \\times l$',
                        'rectangle': 'Perímetro = $2 \\times (b + h)$',
                        'triangle': 'Perímetro = $l_1 + l_2 + l_3$', // Asumimos triángulos con lados a, b, c
                        'rhombus': 'Perímetro = $4 \\times l$', // Lado del rombo
                        'trapezoid': 'Perímetro = $B + b + l_1 + l_2$', // Bases y lados no paralelos
                        'circle': 'Perímetro (Circunferencia) = $2 \\times \\pi \\times r$',
                        'composite': 'Perímetro = Suma de los lados exteriores'
                    };

                    // Función para actualizar la visualización de la puntuación
                    function updateScoreDisplay() {
                        scoreDisplay.textContent = `Puntos: ${score}`;
                        if (score < 20 || hintUsedThisChallenge) {
                            hintBtn.disabled = true;
                            hintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                        } else {
                            hintBtn.disabled = false;
                            hintBtn.classList.remove('opacity-50', 'cursor-not-allowed');
                        }
                    }

                    // Función para limpiar el SVG antes de dibujar una nueva forma
                    function clearSvg() {
                        while (shapeSvg.firstChild) {
                            shapeSvg.removeChild(shapeSvg.firstChild);
                        }
                    }

                    // Función para dibujar la forma y sus dimensiones en el SVG
                    function drawShape(type, dimensions) {
                        clearSvg();

                        const svgWidth = 300;
                        const svgHeight = 200;
                        const padding = 30;

                        const colorPalette = shapeColorPalettes[getRandomInt(0, shapeColorPalettes.length - 1)];

                        let shapeElement;

                        function scaleAndCenter(width, height) {
                            const scaleX = (svgWidth - 2 * padding) / width;
                            const scaleY = (svgHeight - 2 * padding) / height;
                            const scale = Math.min(scaleX, scaleY);

                            const scaledWidth = width * scale;
                            const scaledHeight = height * scale;

                            const offsetX = (svgWidth - scaledWidth) / 2;
                            const offsetY = (svgHeight - scaledHeight) / 2;
                            return { scaledWidth, scaledHeight, offsetX, offsetY };
                        }

                        if (type === 'square' || type === 'rectangle') {
                            const { side1, side2 } = dimensions;
                            const { scaledWidth, scaledHeight, offsetX, offsetY } = scaleAndCenter(side1, side2);

                            shapeElement = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
                            shapeElement.setAttribute('x', offsetX);
                            shapeElement.setAttribute('y', offsetY);
                            shapeElement.setAttribute('width', scaledWidth);
                            shapeElement.setAttribute('height', scaledHeight);
                            shapeElement.classList.add('shape-fill');
                            shapeElement.style.fill = colorPalette.fill;
                            shapeElement.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(shapeElement);

                            // Líneas y textos de dimensión
                            let line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX);
                            line.setAttribute('y1', offsetY + scaledHeight + 10);
                            line.setAttribute('x2', offsetX + scaledWidth);
                            line.setAttribute('y2', offsetY + scaledHeight + 10);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledWidth / 2);
                            text.setAttribute('y', offsetY + scaledHeight + 25);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = side1;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX - 10);
                            line.setAttribute('y1', offsetY);
                            line.setAttribute('x2', offsetX - 10);
                            line.setAttribute('y2', offsetY + scaledHeight);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX - 25);
                            text.setAttribute('y', offsetY + scaledHeight / 2);
                            text.setAttribute('text-anchor', 'start');
                            text.textContent = side2;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                        } else if (type === 'triangle') {
                            const { base, height, side1, side2, side3 } = dimensions; // Now includes all three sides
                            const { scaledWidth, scaledHeight, offsetX, offsetY } = scaleAndCenter(base, height);

                            shapeElement = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
                            const points = `${offsetX + scaledWidth / 2},${offsetY} ${offsetX},${offsetY + scaledHeight} ${offsetX + scaledWidth},${offsetY + scaledHeight}`;
                            shapeElement.setAttribute('points', points);
                            shapeElement.classList.add('shape-fill');
                            shapeElement.style.fill = colorPalette.fill;
                            shapeElement.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(shapeElement);

                            // Display all three sides for perimeter
                            // Base
                            let line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX);
                            line.setAttribute('y1', offsetY + scaledHeight + 10);
                            line.setAttribute('x2', offsetX + scaledWidth);
                            line.setAttribute('y2', offsetY + scaledHeight + 10);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledWidth / 2);
                            text.setAttribute('y', offsetY + scaledHeight + 25);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = side1; // Use side1 for base
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            // Left side
                            const x1_left = offsetX;
                            const y1_left = offsetY + scaledHeight;
                            const x2_left = offsetX + scaledWidth / 2;
                            const y2_left = offsetY;
                            //const length_left = Math.sqrt(Math.pow(x2_left - x1_left, 2) + Math.pow(y2_left - y1_left, 2)); // Not used
                            const mid_x_left = (x1_left + x2_left) / 2;
                            const mid_y_left = (y1_left + y2_left) / 2;

                            line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', x1_left - 5);
                            line.setAttribute('y1', y1_left - 5);
                            line.setAttribute('x2', x2_left - 5);
                            line.setAttribute('y2', y2_left + 5);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', mid_x_left - 20);
                            text.setAttribute('y', mid_y_left - 5);
                            text.setAttribute('text-anchor', 'end');
                            text.textContent = side2;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            // Right side
                            const x1_right = offsetX + scaledWidth;
                            const y1_right = offsetY + scaledHeight;
                            const x2_right = offsetX + scaledWidth / 2;
                            const y2_right = offsetY;
                            //const length_right = Math.sqrt(Math.pow(x2_right - x1_right, 2) + Math.pow(y2_right - y1_right, 2)); // Not used
                            const mid_x_right = (x1_right + x2_right) / 2;
                            const mid_y_right = (y1_right + y2_right) / 2;

                            line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', x1_right + 5);
                            line.setAttribute('y1', y1_right - 5);
                            line.setAttribute('x2', x2_right + 5);
                            line.setAttribute('y2', y2_right + 5);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', mid_x_right + 20);
                            text.setAttribute('y', mid_y_right - 5);
                            text.setAttribute('text-anchor', 'start');
                            text.textContent = side3;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                        } else if (type === 'rhombus') {
                            const { side } = dimensions; // Rombo: todos los lados son iguales
                            const { scaledWidth, scaledHeight, offsetX, offsetY } = scaleAndCenter(side * 2, side * 2); // Aproximación para centrar

                            shapeElement = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
                            const points = `${offsetX + scaledWidth / 2},${offsetY} ${offsetX + scaledWidth},${offsetY + scaledHeight / 2} ${offsetX + scaledWidth / 2},${offsetY + scaledHeight} ${offsetX},${offsetY + scaledHeight / 2}`;
                            shapeElement.setAttribute('points', points);
                            shapeElement.classList.add('shape-fill');
                            shapeElement.style.fill = colorPalette.fill;
                            shapeElement.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(shapeElement);

                            // Mostrar un solo lado, ya que todos son iguales
                            let line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX + scaledWidth / 2);
                            line.setAttribute('y1', offsetY);
                            line.setAttribute('x2', offsetX + scaledWidth);
                            line.setAttribute('y2', offsetY + scaledHeight / 2);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledWidth * 0.75 + 5);
                            text.setAttribute('y', offsetY + scaledHeight * 0.25 - 5);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = side;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                        } else if (type === 'trapezoid') {
                            const { B, b, h } = dimensions; // B = base mayor, b = base menor, h = altura
                            // Escalar para que B sea la base mayor y b la menor, h la altura
                            const { scaledWidth: scaledB, scaledHeight, offsetX, offsetY } = scaleAndCenter(B, h);
                            const scaledb = (b / B) * scaledB; // Escalar la base menor proporcionalmente

                            shapeElement = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
                            // Puntos: (top-left), (top-right), (bottom-right), (bottom-left)
                            const points = `${offsetX + (scaledB - scaledb) / 2},${offsetY} ${offsetX + scaledB - (scaledB - scaledb) / 2},${offsetY} ${offsetX + scaledB},${offsetY + scaledHeight} ${offsetX},${offsetY + scaledHeight}`;
                            shapeElement.setAttribute('points', points);
                            shapeElement.classList.add('shape-fill');
                            shapeElement.style.fill = colorPalette.fill;
                            shapeElement.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(shapeElement);

                            // Línea de altura
                            let heightLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            heightLine.setAttribute('x1', offsetX + scaledB / 2);
                            heightLine.setAttribute('y1', offsetY);
                            heightLine.setAttribute('x2', offsetX + scaledB / 2);
                            heightLine.setAttribute('y2', offsetY + scaledHeight);
                            heightLine.classList.add('dimension-line');
                            shapeSvg.appendChild(heightLine);

                            // Línea de base mayor
                            let baseBLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            baseBLine.setAttribute('x1', offsetX);
                            baseBLine.setAttribute('y1', offsetY + scaledHeight + 10);
                            baseBLine.setAttribute('x2', offsetX + scaledB);
                            baseBLine.setAttribute('y2', offsetY + scaledHeight + 10);
                            baseBLine.classList.add('dimension-line');
                            shapeSvg.appendChild(baseBLine);

                            // Línea de base menor
                            let basebLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            basebLine.setAttribute('x1', offsetX + (scaledB - scaledb) / 2);
                            basebLine.setAttribute('y1', offsetY - 10);
                            basebLine.setAttribute('x2', offsetX + scaledB - (scaledB - scaledb) / 2);
                            basebLine.setAttribute('y2', offsetY - 10);
                            basebLine.classList.add('dimension-line');
                            shapeSvg.appendChild(basebLine);

                            // Textos de dimensión
                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledB / 2);
                            text.setAttribute('y', offsetY + scaledHeight + 25);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = `B: ${B}`;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledB / 2);
                            text.setAttribute('y', offsetY - 25);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = `b: ${b}`;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledB / 2 + 5);
                            text.setAttribute('y', offsetY + scaledHeight / 2);
                            text.setAttribute('text-anchor', 'start');
                            text.textContent = `h: ${h}`;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                        } else if (type === 'circle') {
                            const { radius } = dimensions;
                            const maxRadius = Math.min(svgWidth, svgHeight) / 2 - padding;

                            const scaledRadius = (radius / 15) * maxRadius;
                            const displayRadius = Math.max(20, scaledRadius);

                            shapeElement = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
                            shapeElement.setAttribute('cx', svgWidth / 2);
                            shapeElement.setAttribute('cy', svgHeight / 2);
                            shapeElement.setAttribute('r', displayRadius);
                            shapeElement.classList.add('shape-fill');
                            shapeElement.style.fill = colorPalette.fill;
                            shapeElement.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(shapeElement);

                            // Línea de radio
                            let radiusLine = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            radiusLine.setAttribute('x1', svgWidth / 2);
                            radiusLine.setAttribute('y1', svgHeight / 2);
                            radiusLine.setAttribute('x2', svgWidth / 2 + displayRadius);
                            radiusLine.setAttribute('y2', svgHeight / 2);
                            radiusLine.classList.add('dimension-line');
                            shapeSvg.appendChild(radiusLine);

                            // Texto de radio
                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', svgWidth / 2 + displayRadius / 2);
                            text.setAttribute('y', svgHeight / 2 - 5);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = radius;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                        } else if (type === 'composite') {
                            // Figura compuesta simple: Rectángulo con un triángulo encima
                            // Dimensiones fijas para simplificar el desafío
                            const rectBase = 20;
                            const rectHeight = 10;
                            const triHeight = 5;

                            const { scaledWidth: scaledRectBase, scaledHeight: scaledTotalHeight, offsetX, offsetY } = scaleAndCenter(rectBase, rectHeight + triHeight);

                            const rectScaledHeight = (rectHeight / (rectHeight + triHeight)) * scaledTotalHeight;
                            const triScaledHeight = (triHeight / (rectHeight + triHeight)) * scaledTotalHeight;

                            // Rectángulo
                            let rect = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
                            rect.setAttribute('x', offsetX);
                            rect.setAttribute('y', offsetY + triScaledHeight);
                            rect.setAttribute('width', scaledRectBase);
                            rect.setAttribute('height', rectScaledHeight);
                            rect.classList.add('shape-fill');
                            rect.style.fill = colorPalette.fill;
                            rect.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(rect);

                            // Triángulo encima
                            let triangle = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
                            const triPoints = `${offsetX + scaledRectBase / 2},${offsetY} ${offsetX},${offsetY + triScaledHeight} ${offsetX + scaledRectBase},${offsetY + triScaledHeight}`;
                            triangle.setAttribute('points', triPoints);
                            triangle.classList.add('shape-fill');
                            triangle.style.fill = colorPalette.fill;
                            triangle.style.stroke = colorPalette.stroke;
                            shapeSvg.appendChild(triangle);

                            // Dimensiones del rectángulo
                            let text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledRectBase / 2);
                            text.setAttribute('y', offsetY + triScaledHeight + rectScaledHeight + 15);
                            text.setAttribute('text-anchor', 'middle');
                            text.textContent = rectBase;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            // Ajuste para la figura compuesta:
                            text.setAttribute('x', offsetX - 25); // Mover más a la izquierda
                            text.setAttribute('y', offsetY + triScaledHeight + rectScaledHeight / 2);
                            text.setAttribute('text-anchor', 'start'); // Alinear al inicio
                            text.textContent = rectHeight;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            // Dimensión de altura del triángulo
                            text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                            text.setAttribute('x', offsetX + scaledRectBase / 2 + 5);
                            text.setAttribute('y', offsetY + triScaledHeight / 2);
                            text.setAttribute('text-anchor', 'start');
                            text.textContent = triHeight;
                            text.classList.add('dimension-text');
                            shapeSvg.appendChild(text);

                            // Líneas de dimensión
                            let line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX);
                            line.setAttribute('y1', offsetY + triScaledHeight + rectScaledHeight + 10);
                            line.setAttribute('x2', offsetX + scaledRectBase);
                            line.setAttribute('y2', offsetY + triScaledHeight + rectScaledHeight + 10);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX - 10);
                            line.setAttribute('y1', offsetY + triScaledHeight);
                            line.setAttribute('x2', offsetX - 10);
                            line.setAttribute('y2', offsetY + triScaledHeight + rectScaledHeight);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);

                            line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                            line.setAttribute('x1', offsetX + scaledRectBase / 2);
                            line.setAttribute('y1', offsetY);
                            line.setAttribute('x2', offsetX + scaledRectBase / 2);
                            line.setAttribute('y2', offsetY + triScaledHeight);
                            line.classList.add('dimension-line');
                            shapeSvg.appendChild(line);
                        }
                    }

                    // Función para calcular el perímetro según el tipo de forma
                    function calculatePerimeter(type, dims) {
                        if (type === 'square') {
                            return 4 * dims.side1;
                        } else if (type === 'rectangle') {
                            return 2 * (dims.side1 + dims.side2);
                        } else if (type === 'triangle') {
                            // Para un triángulo, necesitamos los tres lados para el perímetro.
                            // Si solo tenemos base y altura (como en el juego de área),
                            // asumimos un triángulo isósceles para calcular los lados inclinados.
                            // O, si el nivel es avanzado, podríamos pedir los 3 lados.
                            // Por simplicidad, para el perímetro, los generaremos.
                            return dims.side1 + dims.side2 + dims.side3;
                        } else if (type === 'rhombus') {
                            return 4 * dims.side;
                        } else if (type === 'trapezoid') {
                            return dims.B + dims.b + dims.side1 + dims.side2;
                        } else if (type === 'circle') {
                            return 2 * PI * dims.radius;
                        } else if (type === 'composite') {
                            // Perímetro de la figura compuesta (rectángulo con triángulo encima)
                            // Lados exteriores: base del rectángulo + 2 * altura del rectángulo + 2 * lado inclinado del triángulo
                            const rectBase = 20;
                            const rectHeight = 10;
                            const triHeight = 5;
                            const triSide = Math.sqrt(Math.pow(rectBase / 2, 2) + Math.pow(triHeight, 2));
                            return rectBase + (2 * rectHeight) + (2 * triSide);
                        }
                        return 0;
                    }

                    // Función para mostrar la pista (fórmula)
                    function displayHint(type) {
                        hintMessage.innerHTML = formulas[type];
                        hintMessage.classList.remove('hidden');
                        if (typeof MathJax !== 'undefined') {
                            MathJax.typesetPromise(); // Renderizar la fórmula LaTeX
                        }
                    }

                    // Función para generar un nuevo desafío
                    function generateNewChallenge() {
                        const availableShapes = levelShapes[currentLevel];
                        currentShapeType = availableShapes[getRandomInt(0, availableShapes.length - 1)];

                        if (currentShapeType === 'square') {
                            const side = getRandomInt(3, 10);
                            currentDimensions = { side1: side, side2: side };
                        } else if (currentShapeType === 'rectangle') {
                            const side1 = getRandomInt(5, 18);
                            let side2 = getRandomInt(3, 9);
                            while (side2 === side1) {
                                side2 = getRandomInt(3, 9);
                            }
                            currentDimensions = { side1: side1, side2: side2 };
                        } else if (currentShapeType === 'triangle') {
                            // Para perímetro, necesitamos 3 lados. Generamos un triángulo escaleno o isósceles.
                            const side1 = getRandomInt(5, 15);
                            const side2 = getRandomInt(5, 15);
                            const side3 = getRandomInt(Math.abs(side1 - side2) + 1, side1 + side2 - 1); // Asegura que sea un triángulo válido
                            currentDimensions = { side1: side1, side2: side2, side3: side3, base: side1, height: getRandomInt(4,12) }; // base and height for drawing
                        } else if (currentShapeType === 'rhombus') {
                            const side = getRandomInt(5, 12);
                            currentDimensions = { side: side };
                        } else if (currentShapeType === 'trapezoid') {
                            const B = getRandomInt(12, 22);
                            let b = getRandomInt(5, B - 5);
                            const side1 = getRandomInt(5, 10); // Lado no paralelo 1
                            const side2 = getRandomInt(5, 10); // Lado no paralelo 2
                            currentDimensions = { B: B, b: b, side1: side1, side2: side2 };
                        } else if (currentShapeType === 'circle') {
                            const radius = getRandomInt(3, 8);
                            currentDimensions = { radius: radius };
                        } else if (currentShapeType === 'composite') {
                            currentDimensions = {}; // Dimensiones fijas para esta figura
                        }

                        correctPerimeter = calculatePerimeter(currentShapeType, currentDimensions);

                        drawShape(currentShapeType, currentDimensions);

                        userPerimeterGuessInput.value = '';
                        feedbackMessage.textContent = '';
                        feedbackMessage.classList.remove('correct-message', 'incorrect-message');
                        hintMessage.classList.add('hidden');
                        checkPerimeterBtn.disabled = false;
                        checkPerimeterBtn.classList.remove('opacity-50', 'cursor-not-allowed');
                        nextChallengeBtn.classList.add('hidden');
                        hintUsedThisChallenge = false;
                        updateScoreDisplay();
                    }

                    // Event listener para verificar la respuesta
                    checkPerimeterBtn.addEventListener('click', () => {
                        const userAnswer = parseFloat(userPerimeterGuessInput.value);

                        if (isNaN(userAnswer)) {
                            feedbackMessage.textContent = 'Por favor, introduce un número válido.';
                            feedbackMessage.classList.add('incorrect-message');
                            return;
                        }

                        if (Math.abs(userAnswer - correctPerimeter) < 0.02) {
                            playCorrectSound(); // Play sound on correct answer
                            feedbackMessage.textContent = `¡Correcto! El perímetro es ${correctPerimeter.toFixed(2)}.`;
                            feedbackMessage.classList.remove('incorrect-message');
                            feedbackMessage.classList.add('correct-message');
                            score += 10;
                            updateScoreDisplay();
                            checkPerimeterBtn.disabled = true;
                            checkPerimeterBtn.classList.add('opacity-50', 'cursor-not-allowed');
                            nextChallengeBtn.classList.remove('hidden');
                            hintBtn.disabled = true;
                            hintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                        } else {
                            playIncorrectSound(); // Play sound on incorrect answer
                            feedbackMessage.textContent = 'Incorrecto. Inténtalo de nuevo.';
                            feedbackMessage.classList.remove('correct-message');
                            feedbackMessage.classList.add('incorrect-message');
                        }
                    });

                    // Event listener para el botón de pista
                    hintBtn.addEventListener('click', () => {
                        if (score >= 20 && !hintUsedThisChallenge) {
                            score -= 20;
                            updateScoreDisplay();
                            displayHint(currentShapeType);
                            hintUsedThisChallenge = true;
                            hintBtn.disabled = true;
                            hintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                        } else if (hintUsedThisChallenge) {
                            feedbackMessage.textContent = 'Ya usaste la pista para este desafío.';
                            feedbackMessage.classList.add('incorrect-message');
                        } else {
                            feedbackMessage.textContent = 'No tienes suficientes puntos para una pista (necesitas 20).';
                            feedbackMessage.classList.add('incorrect-message');
                        }
                    });

                    // Event listener para el siguiente desafío
                    nextChallengeBtn.addEventListener('click', generateNewChallenge);

                    // Permitir presionar Enter para verificar
                    userPerimeterGuessInput.addEventListener('keypress', (event) => {
                        if (event.key === 'Enter' && !checkPerimeterBtn.disabled) {
                            checkPerimeterBtn.click();
                        }
                    });

                    // Lógica de selección de nivel
                    levelButtons.forEach(button => {
                        button.addEventListener('click', (event) => {
                            currentLevel = event.target.dataset.level;
                            levelSelectionDiv.classList.add('hidden');
                            gamePlayDiv.classList.remove('hidden');
                            generateNewChallenge(); // Iniciar el primer desafío del nivel
                            updateScoreDisplay(); // Asegurarse de que la puntuación inicial sea visible
                        });
                    });

                    // Ocultar el juego al inicio, mostrar solo la selección de nivel
                    gamePlayDiv.classList.add('hidden');
                    levelSelectionDiv.classList.remove('hidden');
                }
            },
            'Adivina la Frase Famosa': {
                category: 'Filosofia',
                icon: '💬',
                description: '¿Conoces las frases más célebres? Adivina quién las dijo y gana puntos.',
                gameHtml: `
                    <div id="phraseLevelSelection" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg min-h-[200px]">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Selecciona Nivel de Dificultad</h3>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-lg justify-center">
                            <button class="phrase-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Facil">
                                Fácil (3 opciones)
                            </button>
                            <button class="phrase-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Normal">
                                Normal (4 opciones)
                            </button>
                            <button class="phrase-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Dificil">
                                Difícil (5 opciones)
                            </button>
                        </div>
                    </div>

                    <div id="phraseGamePlay" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg mt-8 hidden">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">¿Quién dijo esta frase?</h3>
                        <div id="phraseScoreDisplay" class="text-2xl font-bold text-purple-700 mb-4">Puntos: 0</div>

                        <p id="famousPhrase" class="text-xl italic text-gray-700 text-center mb-6 px-4 py-3 bg-gray-100 rounded-lg border border-gray-200 w-full max-w-xl">
                            "La imaginación es más importante que el conocimiento."
                        </p>

                        <div id="phraseOptions" class="grid grid-cols-1 md:grid-cols-2 gap-4 w-full max-w-xl">
                            <!-- Opciones se cargarán aquí -->
                        </div>

                        <p id="phraseFeedbackMessage" class="text-xl font-bold mt-4"></p>
                        <p id="phraseHintMessage" class="text-gray-700 text-center mt-2 hidden text-3xl"></p> <!-- Nuevo para la pista de emojis -->
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-sm">
                            <button id="nextPhraseBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md hidden flex-1">
                                Siguiente Frase
                            </button>
                            <button id="phraseHintBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md flex-1">
                                Pista (-20 Puntos)
                            </button>
                        </div>
                    </div>
                `,
                gameScript: function() {
                    const phraseLevelSelectionDiv = document.getElementById('phraseLevelSelection');
                    const phraseGamePlayDiv = document.getElementById('phraseGamePlay');
                    const phraseLevelButtons = document.querySelectorAll('.phrase-level-button');

                    const famousPhraseDisplay = document.getElementById('famousPhrase');
                    const phraseOptionsContainer = document.getElementById('phraseOptions');
                    const phraseFeedbackMessage = document.getElementById('phraseFeedbackMessage');
                    const nextPhraseBtn = document.getElementById('nextPhraseBtn');
                    const phraseScoreDisplay = document.getElementById('phraseScoreDisplay');
                    const phraseHintBtn = document.getElementById('phraseHintBtn'); // Nuevo
                    const phraseHintMessage = document.getElementById('phraseHintMessage'); // Nuevo

                    let currentPhraseLevel = null;
                    let currentCorrectAuthor = '';
                    let phraseScore = 0;
                    let phraseHintUsedThisChallenge = false; // Nuevo
                    let phrasesPlayedInCurrentLevel = []; // Nuevo: Para evitar repeticiones

                    const famousPhrases = {
                        'Facil': [
                            { phrase: "Solo sé que no sé nada.", author: "Sócrates", incorrectAuthors: ["Platón", "Aristóteles"], emojis: ["🏛️", "🤔"] },
                            { phrase: "Pienso, luego existo.", author: "René Descartes", incorrectAuthors: ["Friedrich Nietzsche", "Albert Einstein"], emojis: ["🧠", "💡"] },
                            { phrase: "La vida es aquello que te va sucediendo mientras estás ocupado haciendo otros planes.", author: "John Lennon", incorrectAuthors: ["Paul McCartney", "George Harrison"], emojis: ["🎸", "☮️"] },
                            { phrase: "Sé el cambio que quieres ver en el mundo.", author: "Mahatma Gandhi", incorrectAuthors: ["Nelson Mandela", "Martin Luther King Jr."], emojis: ["🕊️", "✊"] },
                            { phrase: "El único modo de hacer un gran trabajo es amar lo que haces.", author: "Steve Jobs", incorrectAuthors: ["Bill Gates", "Elon Musk"], emojis: ["🍎", "💻"] },
                            { phrase: "La educación es el arma más poderosa que puedes usar para cambiar el mundo.", author: "Nelson Mandela", incorrectAuthors: ["Martin Luther King Jr.", "Malala Yousafzai"], emojis: ["🇿🇦", "📚"] },
                            { phrase: "No preguntes qué puede hacer tu país por ti, pregunta qué puedes hacer tú por tu país.", author: "John F. Kennedy", incorrectAuthors: ["Abraham Lincoln", "George Washington"], emojis: ["🇺🇸", "🗣️"] }
                        ],
                        'Normal': [
                            { phrase: "Dos cosas son infinitas: el universo y la estupidez humana; y del universo no estoy muy seguro.", author: "Albert Einstein", incorrectAuthors: ["Stephen Hawking", "Isaac Newton", "Galileo Galilei"], emojis: ["⚛️", "✨"] },
                            { phrase: "La única forma de hacer un gran trabajo es amar lo que haces.", author: "Steve Jobs", incorrectAuthors: ["Bill Gates", "Mark Zuckerberg", "Elon Musk"], emojis: ["🍎", "💻"] },
                            { phrase: "No llores porque terminó, sonríe porque sucedió.", author: "Gabriel García Márquez", incorrectAuthors: ["Julio Cortázar", "Jorge Luis Borges", "Mario Vargas Llosa"], emojis: ["✍️", "🦋"] },
                            { phrase: "El arte de la guerra se basa en el engaño.", author: "Sun Tzu", incorrectAuthors: ["Alejandro Magno", "Julio César", "Napoleón Bonaparte"], emojis: ["⚔️", "🐉"] },
                            { phrase: "La imaginación es más importante que el conocimiento.", author: "Albert Einstein", incorrectAuthors: ["Stephen Hawking", "Isaac Newton", "Marie Curie"], emojis: ["🧠", "🌌"] },
                            { phrase: "Vive como si fueras a morir mañana. Aprende como si fueras a vivir para siempre.", author: "Mahatma Gandhi", incorrectAuthors: ["Nelson Mandela", "Dalai Lama", "Teresa de Calcuta"], emojis: ["🧘", "📖"] },
                            { phrase: "La felicidad no es algo ya hecho. Viene de tus propias acciones.", author: "Dalai Lama", incorrectAuthors: ["Buda", "Confucio", "Lao Tsé"], emojis: ["🙏", "😊"] },
                            { phrase: "Lo importante no es lo que te ocurre, sino cómo reaccionas a ello.", author: "Epicteto", incorrectAuthors: ["Séneca", "Marco Aurelio", "Platón"], emojis: ["🏛️", "🧘‍♂️"] }
                        ],
                        'Dificil': [
                            { phrase: "¡Oh, Capitán! ¡Mi Capitán! Nuestro terrible viaje ha terminado.", author: "Walt Whitman", incorrectAuthors: ["Edgar Allan Poe", "Emily Dickinson", "Robert Frost", "Henry David Thoreau"], emojis: ["📜", "⚓"] },
                            { phrase: "El hombre es un lobo para el hombre.", author: "Thomas Hobbes", incorrectAuthors: ["Jean-Jacques Rousseau", "John Locke", "Baruch Spinoza", "Immanuel Kant"], emojis: ["🐺", "👑"] },
                            { phrase: "La única cosa de la que debemos tener miedo es del miedo mismo.", author: "Franklin D. Roosevelt", incorrectAuthors: ["Winston Churchill", "Abraham Lincoln", "Theodore Roosevelt", "John F. Kennedy"], emojis: ["🏛️", "🇺🇸"] },
                            { phrase: "No es la especie más fuerte la que sobrevive, ni la más inteligente, sino la que mejor se adapta al cambio.", author: "Charles Darwin", incorrectAuthors: ["Gregor Mendel", "Louis Pasteur", "Marie Curie", "Nikola Tesla"], emojis: ["🐒", "🌳"] },
                            { phrase: "La verdad os hará libres.", author: "Jesucristo", incorrectAuthors: ["Buda", "Mahoma", "Moisés", "Confucio"], emojis: ["🕊️", "✝️"] },
                            { phrase: "El infierno son los otros.", author: "Jean-Paul Sartre", incorrectAuthors: ["Albert Camus", "Simone de Beauvoir", "Friedrich Nietzsche", "Martin Heidegger"], emojis: ["🎭", "🇫🇷"] },
                            { phrase: "Todo lo que vemos o parecemos es solo un sueño dentro de un sueño.", author: "Edgar Allan Poe", incorrectAuthors: ["H.P. Lovecraft", "Mary Shelley", "Bram Stoker", "Lord Byron"], emojis: ["🦇", "✍️"] },
                            { phrase: "La vida debe ser comprendida hacia atrás, pero debe ser vivida hacia adelante.", author: "Søren Kierkegaard", incorrectAuthors: ["Friedrich Nietzsche", "Arthur Schopenhauer", "Georg Wilhelm Friedrich Hegel", "Immanuel Kant"], emojis: ["🇩🇰", "🧐"] },
                            { phrase: "El problema de la humanidad es que los estúpidos están seguros de todo y los inteligentes están llenos de dudas.", author: "Bertrand Russell", incorrectAuthors: ["Albert Einstein", "Stephen Hawking", "Richard Feynman", "Carl Sagan"], emojis: ["🇬🇧", "🤔"] }
                        ]
                    };

                    // Mapeo de autores a sus datos completos para acceder a los emojis
                    let authorDataMap = {};
                    for (const level in famousPhrases) {
                        famousPhrases[level].forEach(item => {
                            authorDataMap[item.author] = item;
                        });
                    }

                    function updatePhraseScoreDisplay() {
                        phraseScoreDisplay.textContent = `Puntos: ${phraseScore}`;
                        if (phraseScore < 20 || phraseHintUsedThisChallenge) {
                            phraseHintBtn.disabled = true;
                            phraseHintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                        } else {
                            phraseHintBtn.disabled = false;
                            phraseHintBtn.classList.remove('opacity-50', 'cursor-not-allowed');
                        }
                    }

                    function generateNewPhrase() {
                        const phrasesForLevel = famousPhrases[currentPhraseLevel];
                        let availablePhrases = phrasesForLevel.filter(phrase => !phrasesPlayedInCurrentLevel.includes(phrase.phrase));

                        // Si todas las frases han sido usadas, resetear la lista de frases jugadas
                        if (availablePhrases.length === 0) {
                            phrasesPlayedInCurrentLevel = [];
                            availablePhrases = phrasesForLevel;
                        }

                        const randomPhraseData = availablePhrases[getRandomInt(0, availablePhrases.length - 1)];

                        famousPhraseDisplay.textContent = `"${randomPhraseData.phrase}"`;
                        currentCorrectAuthor = randomPhraseData.author;
                        phrasesPlayedInCurrentLevel.push(randomPhraseData.phrase); // Añadir a la lista de jugadas

                        let options = [randomPhraseData.author, ...randomPhraseData.incorrectAuthors];
                        // Limitar opciones según el nivel
                        if (currentPhraseLevel === 'Facil') {
                            options = options.slice(0, 3);
                        } else if (currentPhraseLevel === 'Normal') {
                            options = options.slice(0, 4);
                        } else if (currentPhraseLevel === 'Dificil') {
                            options = options.slice(0, 5);
                        }
                        options = shuffleArray(options);

                        phraseOptionsContainer.innerHTML = '';
                        options.forEach(option => {
                            const button = document.createElement('button');
                            button.classList.add('phrase-option-button');
                            button.textContent = option;
                            button.dataset.author = option;
                            button.addEventListener('click', checkPhraseAnswer);
                            phraseOptionsContainer.appendChild(button);
                        });

                        phraseFeedbackMessage.textContent = '';
                        phraseFeedbackMessage.classList.remove('correct-message', 'incorrect-message');
                        phraseHintMessage.classList.add('hidden'); // Ocultar pista al generar nueva frase
                        nextPhraseBtn.classList.add('hidden');
                        phraseOptionsContainer.querySelectorAll('.phrase-option-button').forEach(btn => {
                            btn.disabled = false;
                            btn.classList.remove('correct', 'incorrect');
                        });
                        phraseHintUsedThisChallenge = false; // Resetear el uso de pista
                        updatePhraseScoreDisplay(); // Actualizar el estado del botón de pista
                    }

                    function checkPhraseAnswer(event) {
                        const selectedAuthor = event.target.dataset.author;
                        const allOptionButtons = phraseOptionsContainer.querySelectorAll('.phrase-option-button');

                        allOptionButtons.forEach(btn => {
                            btn.disabled = true; // Deshabilitar todos los botones después de una selección
                            if (btn.dataset.author === currentCorrectAuthor) {
                                btn.classList.add('correct');
                            } else if (btn.dataset.author === selectedAuthor) {
                                btn.classList.add('incorrect');
                            }
                        });

                        if (selectedAuthor === currentCorrectAuthor) {
                            playCorrectSound(); // Play sound on correct answer
                            phraseFeedbackMessage.textContent = '¡Correcto!';
                            phraseFeedbackMessage.classList.remove('incorrect-message');
                            phraseFeedbackMessage.classList.add('correct-message');
                            phraseScore += 10;
                            updatePhraseScoreDisplay();
                        } else {
                            playIncorrectSound(); // Play sound on incorrect answer
                            phraseFeedbackMessage.textContent = `Incorrecto. El autor correcto era: ${currentCorrectAuthor}`;
                            phraseFeedbackMessage.classList.remove('correct-message');
                            phraseFeedbackMessage.classList.add('incorrect-message');
                        }
                        nextPhraseBtn.classList.remove('hidden');
                        phraseHintBtn.disabled = true; // Deshabilitar pista una vez respondido
                        phraseHintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                    }

                    // Función para mostrar la pista de emojis
                    function displayPhraseHint() {
                        if (phraseScore >= 20 && !phraseHintUsedThisChallenge) {
                            phraseScore -= 20;
                            updatePhraseScoreDisplay();
                            phraseHintUsedThisChallenge = true;
                            phraseHintBtn.disabled = true;
                            phraseHintBtn.classList.add('opacity-50', 'cursor-not-allowed');

                            const authorInfo = authorDataMap[currentCorrectAuthor];
                            if (authorInfo && authorInfo.emojis && authorInfo.emojis.length > 0) {
                                phraseHintMessage.textContent = authorInfo.emojis.join(' ');
                            } else {
                                phraseHintMessage.textContent = 'No hay pista de emojis disponible para este autor.';
                            }
                            phraseHintMessage.classList.remove('hidden');
                        } else if (phraseHintUsedThisChallenge) {
                            phraseFeedbackMessage.textContent = 'Ya usaste la pista para esta frase.';
                            phraseFeedbackMessage.classList.add('incorrect-message');
                        } else {
                            feedbackMessage.textContent = 'No tienes suficientes puntos para una pista (necesitas 20).';
                            feedbackMessage.classList.add('incorrect-message');
                        }
                    }

                    nextPhraseBtn.addEventListener('click', generateNewPhrase);
                    phraseHintBtn.addEventListener('click', displayPhraseHint); // Nuevo listener para la pista

                    phraseLevelButtons.forEach(button => {
                        button.addEventListener('click', (event) => {
                            currentPhraseLevel = event.target.dataset.level;
                            phraseLevelSelectionDiv.classList.add('hidden');
                            phraseGamePlayDiv.classList.remove('hidden');
                            phraseScore = 0; // Reset score for new game
                            phrasesPlayedInCurrentLevel = []; // Resetear frases jugadas al cambiar de nivel
                            updatePhraseScoreDisplay();
                            generateNewPhrase();
                        });
                    });

                    // Ocultar el juego al inicio, mostrar solo la selección de nivel
                    phraseGamePlayDiv.classList.add('hidden');
                    phraseLevelSelectionDiv.classList.remove('hidden');
                }
            },
            'Adivina el Autor de la Obra': {
                category: 'Filosofia',
                icon: '📚',
                description: '¿Conoces los autores de obras famosas? ¡Demuestra tus conocimientos y gana puntos!',
                gameHtml: `
                    <div id="workLevelSelection" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg min-h-[200px]">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Selecciona Nivel de Dificultad</h3>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-lg justify-center">
                            <button class="work-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Facil">
                                Fácil (3 opciones)
                            </button>
                            <button class="work-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Normal">
                                Normal (4 opciones)
                            </button>
                            <button class="work-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Dificil">
                                Difícil (5 opciones)
                            </button>
                        </div>
                    </div>

                    <div id="workGamePlay" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg mt-8 hidden">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">¿Quién escribió esta obra?</h3>
                        <div id="workScoreDisplay" class="text-2xl font-bold text-purple-700 mb-4">Puntos: 0</div>

                        <p id="famousWork" class="text-xl italic text-gray-700 text-center mb-6 px-4 py-3 bg-gray-100 rounded-lg border border-gray-200 w-full max-w-xl">
                            "Cien años de soledad"
                        </p>

                        <div id="workOptions" class="grid grid-cols-1 md:grid-cols-2 gap-4 w-full max-w-xl">
                            <!-- Opciones se cargarán aquí -->
                        </div>

                        <p id="workFeedbackMessage" class="text-xl font-bold mt-4"></p>
                        <p id="workHintMessage" class="text-gray-700 text-center mt-2 hidden text-3xl"></p>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-sm">
                            <button id="nextWorkBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md hidden flex-1">
                                Siguiente Obra
                            </button>
                            <button id="workHintBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md flex-1">
                                Pista (-20 Puntos)
                            </button>
                        </div>
                    </div>
                `,
                gameScript: function() {
                    const workLevelSelectionDiv = document.getElementById('workLevelSelection');
                    const workGamePlayDiv = document.getElementById('workGamePlay');
                    const workLevelButtons = document.querySelectorAll('.work-level-button');

                    const famousWorkDisplay = document.getElementById('famousWork');
                    const workOptionsContainer = document.getElementById('workOptions');
                    const workFeedbackMessage = document.getElementById('workFeedbackMessage');
                    const nextWorkBtn = document.getElementById('nextWorkBtn');
                    const workScoreDisplay = document.getElementById('workScoreDisplay');
                    const workHintBtn = document.getElementById('workHintBtn');
                    const workHintMessage = document.getElementById('workHintMessage');

                    let currentWorkLevel = null;
                    let currentCorrectAuthor = '';
                    let workScore = 0;
                    let workHintUsedThisChallenge = false;
                    let worksPlayedInCurrentLevel = []; // Para evitar repeticiones

                    const famousWorks = {
                        'Facil': [
                            { work: "Don Quijote de la Mancha", author: "Miguel de Cervantes", incorrectAuthors: ["Lope de Vega", "Tirso de Molina"], emojis: ["🇪🇸", "📖"] },
                            { work: "Cien años de soledad", author: "Gabriel García Márquez", incorrectAuthors: ["Julio Cortázar", "Mario Vargas Llosa"], emojis: ["🇨🇴", "🦋"] },
                            { work: "Romeo y Julieta", author: "William Shakespeare", incorrectAuthors: ["Charles Dickens", "Jane Austen"], emojis: ["🎭", "🇬🇧"] },
                            { work: "El Principito", author: "Antoine de Saint-Exupéry", incorrectAuthors: ["Victor Hugo", "Julio Verne"], emojis: ["🇫🇷", "🦊"] },
                            { work: "1984", author: "George Orwell", incorrectAuthors: ["Aldous Huxley", "Ray Bradbury"], emojis: ["👁️", "🇬🇧"] }
                        ],
                        'Normal': [
                            { work: "Orgullo y Prejuicio", author: "Jane Austen", incorrectAuthors: ["Emily Brontë", "Mary Shelley", "Virginia Woolf"], emojis: ["🎩", "🌸"] },
                            { work: "Crimen y Castigo", author: "Fiódor Dostoievski", incorrectAuthors: ["León Tolstói", "Antón Chéjov", "Nikolai Gógol"], emojis: ["🇷🇺", "🔪"] },
                            { work: "Moby Dick", author: "Herman Melville", incorrectAuthors: ["Mark Twain", "Jack London", "Ernest Hemingway"], emojis: ["🐳", "⚓"] },
                            { work: "La Odisea", author: "Homero", incorrectAuthors: ["Virgilio", "Hesíodo", "Esquilo"], emojis: ["🏛️", "⛵"] },
                            { work: "Un mundo feliz", author: "Aldous Huxley", incorrectAuthors: ["George Orwell", "Ray Bradbury", "Isaac Asimov"], emojis: ["🧪", "💊"] },
                            { work: "Anna Karenina", author: "León Tolstói", incorrectAuthors: ["Fiódor Dostoievski", "Iván Turguénev", "Antón Chéjov"], emojis: ["🇷🇺", "💔"] }
                        ],
                        'Dificil': [
                            { work: "En busca del tiempo perdido", author: "Marcel Proust", incorrectAuthors: ["James Joyce", "Franz Kafka", "Virginia Woolf", "Hermann Hesse"], emojis: ["🕰️", "🥐"] },
                            { work: "Ulises", author: "James Joyce", incorrectAuthors: ["Marcel Proust", "Virginia Woolf", "Franz Kafka", "T.S. Eliot"], emojis: ["🇮🇪", "🏙️"] },
                            { work: "El ruido y la furia", author: "William Faulkner", incorrectAuthors: ["Ernest Hemingway", "F. Scott Fitzgerald", "John Steinbeck", "Carson McCullers"], emojis: ["🇺🇸", "🌾"] },
                            { work: "Pedro Páramo", author: "Juan Rulfo", incorrectAuthors: ["Carlos Fuentes", "Elena Garro", "Octavio Paz", "Jorge Ibargüengoitia"], emojis: ["🇲🇽", "👻"] },
                            { work: "La montaña mágica", author: "Thomas Mann", incorrectAuthors: ["Hermann Hesse", "Franz Kafka", "Bertolt Brecht", "Stefan Zweig"], emojis: ["🇩🇪", "🏔️"] },
                            { work: "La Divina Comedia", author: "Dante Alighieri", incorrectAuthors: ["Giovanni Boccaccio", "Francesco Petrarca", "Virgilio", "Homero"], emojis: ["🇮🇹", "🔥"] },
                            { work: "Así habló Zaratustra", author: "Friedrich Nietzsche", incorrectAuthors: ["Arthur Schopenhauer", "Søren Kierkegaard", "Immanuel Kant", "Georg Wilhelm Friedrich Hegel"], emojis: ["👨‍🦳", "🌄"] }
                        ]
                    };

                    // Mapeo de autores a sus datos completos para acceder a los emojis
                    let workAuthorDataMap = {};
                    for (const level in famousWorks) {
                        famousWorks[level].forEach(item => {
                            workAuthorDataMap[item.author] = item;
                        });
                    }

                    function updateWorkScoreDisplay() {
                        workScoreDisplay.textContent = `Puntos: ${workScore}`;
                        if (workScore < 20 || workHintUsedThisChallenge) {
                            workHintBtn.disabled = true;
                            workHintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                        } else {
                            workHintBtn.disabled = false;
                            workHintBtn.classList.remove('opacity-50', 'cursor-not-allowed');
                        }
                    }

                    function generateNewWork() {
                        const worksForLevel = famousWorks[currentWorkLevel];
                        let availableWorks = worksForLevel.filter(work => !worksPlayedInCurrentLevel.includes(work.work));

                        // Si todas las obras han sido usadas, resetear la lista de obras jugadas
                        if (availableWorks.length === 0) {
                            worksPlayedInCurrentLevel = [];
                            availableWorks = worksForLevel;
                        }

                        const randomWorkData = availableWorks[getRandomInt(0, availableWorks.length - 1)];

                        famousWorkDisplay.textContent = `"${randomWorkData.work}"`;
                        currentCorrectAuthor = randomWorkData.author;
                        worksPlayedInCurrentLevel.push(randomWorkData.work); // Añadir a la lista de jugadas

                        let options = [randomWorkData.author, ...randomWorkData.incorrectAuthors];
                        // Limitar opciones según el nivel
                        if (currentWorkLevel === 'Facil') {
                            options = options.slice(0, 3);
                        } else if (currentWorkLevel === 'Normal') {
                            options = options.slice(0, 4);
                        } else if (currentWorkLevel === 'Dificil') {
                            options = options.slice(0, 5);
                        }
                        options = shuffleArray(options);

                        workOptionsContainer.innerHTML = '';
                        options.forEach(option => {
                            const button = document.createElement('button');
                            button.classList.add('work-option-button');
                            button.textContent = option;
                            button.dataset.author = option;
                            button.addEventListener('click', checkWorkAnswer);
                            workOptionsContainer.appendChild(button);
                        });

                        workFeedbackMessage.textContent = '';
                        workFeedbackMessage.classList.remove('correct-message', 'incorrect-message');
                        workHintMessage.classList.add('hidden'); // Ocultar pista al generar nueva obra
                        nextWorkBtn.classList.add('hidden');
                        workOptionsContainer.querySelectorAll('.work-option-button').forEach(btn => {
                            btn.disabled = false;
                            btn.classList.remove('correct', 'incorrect');
                        });
                        workHintUsedThisChallenge = false; // Resetear el uso de pista
                        updateWorkScoreDisplay(); // Actualizar el estado del botón de pista
                    }

                    function checkWorkAnswer(event) {
                        const selectedAuthor = event.target.dataset.author;
                        const allOptionButtons = workOptionsContainer.querySelectorAll('.work-option-button');

                        allOptionButtons.forEach(btn => {
                            btn.disabled = true; // Deshabilitar todos los botones después de una selección
                            if (btn.dataset.author === currentCorrectAuthor) {
                                btn.classList.add('correct');
                            } else if (btn.dataset.author === selectedAuthor) {
                                btn.classList.add('incorrect');
                            }
                        });

                        if (selectedAuthor === currentCorrectAuthor) {
                            playCorrectSound(); // Play sound on correct answer
                            workFeedbackMessage.textContent = '¡Correcto!';
                            workFeedbackMessage.classList.remove('incorrect-message');
                            workFeedbackMessage.classList.add('correct-message');
                            workScore += 10;
                            updateWorkScoreDisplay();
                        } else {
                            playIncorrectSound(); // Play sound on incorrect answer
                            workFeedbackMessage.textContent = `Incorrecto. El autor correcto era: ${currentCorrectAuthor}`;
                            workFeedbackMessage.classList.remove('correct-message');
                            workFeedbackMessage.classList.add('incorrect-message');
                        }
                        nextWorkBtn.classList.remove('hidden');
                        workHintBtn.disabled = true; // Deshabilitar pista una vez respondido
                        workHintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                    }

                    // Función para mostrar la pista de emojis
                    function displayWorkHint() {
                        if (workScore >= 20 && !workHintUsedThisChallenge) {
                            workScore -= 20;
                            updateWorkScoreDisplay();
                            workHintUsedThisChallenge = true;
                            workHintBtn.disabled = true;
                            workHintBtn.classList.add('opacity-50', 'cursor-not-allowed');

                            const authorInfo = workAuthorDataMap[currentCorrectAuthor];
                            if (authorInfo && authorInfo.emojis && authorInfo.emojis.length > 0) {
                                workHintMessage.textContent = authorInfo.emojis.join(' ');
                            } else {
                                workHintMessage.textContent = 'No hay pista de emojis disponible para este autor.';
                            }
                            workHintMessage.classList.remove('hidden');
                        } else if (workHintUsedThisChallenge) {
                            workFeedbackMessage.textContent = 'Ya usaste la pista para esta obra.';
                            workFeedbackMessage.classList.add('incorrect-message');
                        } else {
                            workFeedbackMessage.textContent = 'No tienes suficientes puntos para una pista (necesitas 20).';
                            workFeedbackMessage.classList.add('incorrect-message');
                        }
                    }

                    nextWorkBtn.addEventListener('click', generateNewWork);
                    workHintBtn.addEventListener('click', displayWorkHint);

                    workLevelButtons.forEach(button => {
                        button.addEventListener('click', (event) => {
                            currentWorkLevel = event.target.dataset.level;
                            workLevelSelectionDiv.classList.add('hidden');
                            workGamePlayDiv.classList.remove('hidden');
                            workScore = 0; // Reset score for new game
                            worksPlayedInCurrentLevel = []; // Resetear obras jugadas al cambiar de nivel
                            updateWorkScoreDisplay();
                            generateNewWork();
                        });
                    });

                    // Ocultar el juego al inicio, mostrar solo la selección de nivel
                                        workGamePlayDiv.classList.add('hidden');
                    workLevelSelectionDiv.classList.remove('hidden');
                }
            },
            'Adivina la Respuesta Correcta': {
                category: 'Ingles',
                icon: '🇬🇧',
                description: 'Pon a prueba tus conocimientos de inglés. ¡Adivina la respuesta correcta y gana puntos!',
                gameHtml: `
                    <div id="englishLevelSelection" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg min-h-[200px]">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Selecciona Nivel de Dificultad</h3>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-lg justify-center">
                            <button class="english-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Basico">
                                Básico (3 opciones)
                            </button>
                            <button class="english-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Intermedio">
                                Intermedio (4 opciones)
                            </button>
                            <button class="english-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Avanzado">
                                Avanzado (5 opciones)
                            </button>
                        </div>
                    </div>

                    <div id="englishGamePlay" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg mt-8 hidden">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Adivina la Respuesta Correcta</h3>
                        <div id="englishScoreDisplay" class="text-2xl font-bold text-purple-700 mb-4">Puntos: 0</div>

                        <p id="englishQuestion" class="text-xl italic text-gray-700 text-center mb-6 px-4 py-3 bg-gray-100 rounded-lg border border-gray-200 w-full max-w-xl">
                            "What is the capital of France?"
                        </p>

                        <div id="englishOptions" class="grid grid-cols-1 md:grid-cols-2 gap-4 w-full max-w-xl">
                            <!-- Opciones se cargarán aquí -->
                        </div>

                        <p id="englishFeedbackMessage" class="text-xl font-bold mt-4"></p>
                        <p id="englishHintMessage" class="text-gray-700 text-center mt-2 hidden text-3xl"></p>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-sm">
                            <button id="nextEnglishBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md hidden flex-1">
                                Siguiente Pregunta
                            </button>
                            <button id="englishHintBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md flex-1">
                                Pista (-20 Puntos)
                            </button>
                        </div>
                    </div>
                `,
                gameScript: function() {
                    const englishLevelSelectionDiv = document.getElementById('englishLevelSelection');
                    const englishGamePlayDiv = document.getElementById('englishGamePlay');
                    const englishLevelButtons = document.querySelectorAll('.english-level-button');

                    const englishQuestionDisplay = document.getElementById('englishQuestion');
                    const englishOptionsContainer = document.getElementById('englishOptions');
                    const englishFeedbackMessage = document.getElementById('englishFeedbackMessage');
                    const nextEnglishBtn = document.getElementById('nextEnglishBtn');
                    const englishScoreDisplay = document.getElementById('englishScoreDisplay');
                    const englishHintBtn = document.getElementById('englishHintBtn');
                    const englishHintMessage = document.getElementById('englishHintMessage');

                    let currentEnglishLevel = null;
                    let currentCorrectAnswer = '';
                    let englishScore = 0;
                    let englishHintUsedThisChallenge = false;
                    let englishQuestionsPlayedInCurrentLevel = [];

                    const englishQuestions = {
                        'Basico': [
                            { question: "What is the capital of France?", correctAnswer: "Paris", incorrectAnswers: ["Berlin", "Rome"], emojis: ["🇫🇷", "🗼"] },
                            { question: "How many days are in a week?", correctAnswer: "Seven", incorrectAnswers: ["Five", "Ten"], emojis: ["🗓️", "7️⃣"] },
                            { question: "What color is the sky?", correctAnswer: "Blue", incorrectAnswers: ["Red", "Green"], emojis: ["☁️", "💙"] },
                            { question: "What animal says 'moo'?", correctAnswer: "Cow", incorrectAnswers: ["Dog", "Cat"], emojis: ["🐄", "🥛"] },
                            { question: "What is 2 + 2?", correctAnswer: "Four", incorrectAnswers: ["Three", "Five"], emojis: ["➕", "4️⃣"] }
                        ],
                        'Intermedio': [
                            { question: "What is the past tense of 'eat'?", correctAnswer: "Ate", incorrectAnswers: ["Eaten", "Eating", "Eat"], emojis: ["🍽️", "⏰"] },
                            { question: "Which of these is a synonym for 'happy'?", correctAnswer: "Joyful", incorrectAnswers: ["Sad", "Angry", "Tired"], emojis: ["😊", "☀️"] },
                            { question: "Complete the sentence: 'She ___ to the store yesterday.'", correctAnswer: "went", incorrectAnswers: ["go", "goes", "going"], emojis: ["🚶‍♀️", "🛒"] },
                            { question: "What is a group of fish called?", correctAnswer: "School", incorrectAnswers: ["Herd", "Flock", "Pack"], emojis: ["🐠", "🏫"] },
                            { question: "What is the opposite of 'brave'?", correctAnswer: "Cowardly", incorrectAnswers: ["Strong", "Kind", "Fast"], emojis: ["🦁", "🐰"] }
                        ],
                        'Avanzado': [
                            { question: "Which literary device involves a contrast between what is said and what is actually meant?", correctAnswer: "Irony", incorrectAnswers: ["Metaphor", "Simile", "Alliteration", "Hyperbole"], emojis: ["🎭", "🤫"] },
                            { question: "Who wrote 'To Kill a Mockingbird'?", correctAnswer: "Harper Lee", incorrectAnswers: ["F. Scott Fitzgerald", "J.D. Salinger", "Ernest Hemingway", "John Steinbeck"], emojis: ["⚖️", "🐦"] },
                            { question: "What is the chemical symbol for gold?", correctAnswer: "Au", incorrectAnswers: ["Ag", "Fe", "Hg", "Pb"], emojis: ["✨", "💰"] },
                            { question: "Which philosopher is known for the concept of 'tabula rasa'?", correctAnswer: "John Locke", incorrectAnswers: ["René Descartes", "Immanuel Kant", "David Hume", "Baruch Spinoza"], emojis: ["🧠", "👶"] },
                            { question: "In which year did the Battle of Hastings take place?", correctAnswer: "1066", incorrectAnswers: ["1492", "1776", "1812", "1914"], emojis: ["⚔️", "📅"] }
                        ]
                    };

                    // Mapeo de respuestas correctas a sus datos completos para acceder a los emojis
                    let englishQuestionDataMap = {};
                    for (const level in englishQuestions) {
                        englishQuestions[level].forEach(item => {
                            englishQuestionDataMap[item.correctAnswer] = item;
                        });
                    }

                    function updateEnglishScoreDisplay() {
                        englishScoreDisplay.textContent = `Puntos: ${englishScore}`;
                        if (englishScore < 20 || englishHintUsedThisChallenge) {
                            englishHintBtn.disabled = true;
                            englishHintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                        } else {
                            englishHintBtn.disabled = false;
                            englishHintBtn.classList.remove('opacity-50', 'cursor-not-allowed');
                        }
                    }

                    function generateNewEnglishQuestion() {
                        const questionsForLevel = englishQuestions[currentEnglishLevel];
                        let availableQuestions = questionsForLevel.filter(q => !englishQuestionsPlayedInCurrentLevel.includes(q.question));

                        // Si todas las preguntas han sido usadas, resetear la lista de preguntas jugadas
                        if (availableQuestions.length === 0) {
                            englishQuestionsPlayedInCurrentLevel = [];
                            availableQuestions = questionsForLevel;
                        }

                        const randomQuestionData = availableQuestions[getRandomInt(0, availableQuestions.length - 1)];

                        englishQuestionDisplay.textContent = `"${randomQuestionData.question}"`;
                        currentCorrectAnswer = randomQuestionData.correctAnswer;
                        englishQuestionsPlayedInCurrentLevel.push(randomQuestionData.question); // Añadir a la lista de jugadas

                        let options = [randomQuestionData.correctAnswer, ...randomQuestionData.incorrectAnswers];
                        // Limitar opciones según el nivel
                        if (currentEnglishLevel === 'Basico') {
                            options = options.slice(0, 3);
                        } else if (currentEnglishLevel === 'Intermedio') {
                            options = options.slice(0, 4);
                        } else if (currentEnglishLevel === 'Avanzado') {
                            options = options.slice(0, 5);
                        }
                        options = shuffleArray(options);

                        englishOptionsContainer.innerHTML = '';
                        options.forEach(option => {
                            const button = document.createElement('button');
                            button.classList.add('english-option-button');
                            button.textContent = option;
                            button.dataset.answer = option;
                            button.addEventListener('click', checkEnglishAnswer);
                            englishOptionsContainer.appendChild(button);
                        });

                        englishFeedbackMessage.textContent = '';
                        englishFeedbackMessage.classList.remove('correct-message', 'incorrect-message');
                        englishHintMessage.classList.add('hidden'); // Ocultar pista al generar nueva pregunta
                        nextEnglishBtn.classList.add('hidden');
                        englishOptionsContainer.querySelectorAll('.english-option-button').forEach(btn => {
                            btn.disabled = false;
                            btn.classList.remove('correct', 'incorrect');
                        });
                        englishHintUsedThisChallenge = false; // Resetear el uso de pista
                        updateEnglishScoreDisplay(); // Actualizar el estado del botón de pista
                    }

                    function checkEnglishAnswer(event) {
                        const selectedAnswer = event.target.dataset.answer;
                        const allOptionButtons = englishOptionsContainer.querySelectorAll('.english-option-button');

                        allOptionButtons.forEach(btn => {
                            btn.disabled = true; // Deshabilitar todos los botones después de una selección
                            if (btn.dataset.answer === currentCorrectAnswer) {
                                btn.classList.add('correct');
                            } else if (btn.dataset.answer === selectedAnswer) {
                                btn.classList.add('incorrect');
                            }
                        });

                        if (selectedAnswer === currentCorrectAnswer) {
                            playCorrectSound(); // Play sound on correct answer
                            englishFeedbackMessage.textContent = '¡Correcto!';
                            englishFeedbackMessage.classList.remove('incorrect-message');
                            englishFeedbackMessage.classList.add('correct-message');
                            englishScore += 10;
                            updateEnglishScoreDisplay();
                        } else {
                            playIncorrectSound(); // Play sound on incorrect answer
                            englishFeedbackMessage.textContent = `Incorrecto. La respuesta correcta era: ${currentCorrectAnswer}`;
                            englishFeedbackMessage.classList.remove('correct-message');
                            englishFeedbackMessage.classList.add('incorrect-message');
                        }
                        nextEnglishBtn.classList.remove('hidden');
                        englishHintBtn.disabled = true; // Deshabilitar pista una vez respondido
                        englishHintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                    }

                    // Función para mostrar la pista de emojis
                    function displayEnglishHint() {
                        if (englishScore >= 20 && !englishHintUsedThisChallenge) {
                            englishScore -= 20;
                            updateEnglishScoreDisplay();
                            englishHintUsedThisChallenge = true;
                            englishHintBtn.disabled = true;
                            englishHintBtn.classList.add('opacity-50', 'cursor-not-allowed');

                            const questionInfo = englishQuestionDataMap[currentCorrectAnswer];
                            if (questionInfo && questionInfo.emojis && questionInfo.emojis.length > 0) {
                                englishHintMessage.textContent = questionInfo.emojis.join(' ');
                            } else {
                                englishHintMessage.textContent = 'No hay pista de emojis disponible para esta pregunta.';
                            }
                            englishHintMessage.classList.remove('hidden');
                        } else if (englishHintUsedThisChallenge) {
                            englishFeedbackMessage.textContent = 'Ya usaste la pista para esta pregunta.';
                            englishFeedbackMessage.classList.add('incorrect-message');
                        } else {
                            englishFeedbackMessage.textContent = 'No tienes suficientes puntos para una pista (necesitas 20).';
                            englishFeedbackMessage.classList.add('incorrect-message');
                        }
                    }

                    nextEnglishBtn.addEventListener('click', generateNewEnglishQuestion);
                    englishHintBtn.addEventListener('click', displayEnglishHint);

                    englishLevelButtons.forEach(button => {
                        button.addEventListener('click', (event) => {
                            currentEnglishLevel = event.target.dataset.level;
                            englishLevelSelectionDiv.classList.add('hidden');
                            englishGamePlayDiv.classList.remove('hidden');
                            englishScore = 0; // Reset score for new game
                            englishQuestionsPlayedInCurrentLevel = []; // Resetear preguntas jugadas al cambiar de nivel
                            updateEnglishScoreDisplay();
                            generateNewEnglishQuestion();
                        });
                    });

                    // Ocultar el juego al inicio, mostrar solo la selección de nivel
                    englishGamePlayDiv.classList.add('hidden');
                    englishLevelSelectionDiv.classList.remove('hidden');
                }
            },
            'Adivina el Verbo Correcto': {
                category: 'Ingles',
                icon: '✍️',
                description: 'Elige la forma verbal correcta para completar la oración. ¡Mejora tu gramática!',
                gameHtml: `
                    <div id="verbLevelSelection" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg min-h-[200px]">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Selecciona Nivel de Dificultad</h3>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-lg justify-center">
                            <button class="verb-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Basico">
                                Básico (3 opciones)
                            </button>
                            <button class="verb-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Intermedio">
                                Intermedio (4 opciones)
                            </button>
                            <button class="verb-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Avanzado">
                                Avanzado (5 opciones)
                            </button>
                        </div>
                    </div>

                    <div id="verbGamePlay" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg mt-8 hidden">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Completa la oración:</h3>
                        <div id="verbScoreDisplay" class="text-2xl font-bold text-purple-700 mb-4">Puntos: 0</div>

                        <p id="verbSentence" class="text-xl italic text-gray-700 text-center mb-6 px-4 py-3 bg-gray-100 rounded-lg border border-gray-200 w-full max-w-xl">
                            "She ___ to the store yesterday."
                        </p>

                        <div id="verbOptions" class="grid grid-cols-1 md:grid-cols-2 gap-4 w-full max-w-xl">
                            <!-- Opciones se cargarán aquí -->
                        </div>

                        <p id="verbFeedbackMessage" class="text-xl font-bold mt-4"></p>
                        <p id="verbHintMessage" class="text-gray-700 text-center mt-2 hidden text-3xl"></p>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-sm">
                            <button id="nextVerbBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md hidden flex-1">
                                Siguiente Pregunta
                            </button>
                            <button id="verbHintBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md flex-1">
                                Pista (-20 Puntos)
                            </button>
                        </div>
                    </div>
                `,
                gameScript: function() {
                    const verbLevelSelectionDiv = document.getElementById('verbLevelSelection');
                    const verbGamePlayDiv = document.getElementById('verbGamePlay');
                    const verbLevelButtons = document.querySelectorAll('.verb-level-button');

                    const verbSentenceDisplay = document.getElementById('verbSentence');
                    const verbOptionsContainer = document.getElementById('verbOptions');
                    const verbFeedbackMessage = document.getElementById('verbFeedbackMessage');
                    const nextVerbBtn = document.getElementById('nextVerbBtn');
                    const verbScoreDisplay = document.getElementById('verbScoreDisplay');
                    const verbHintBtn = document.getElementById('verbHintBtn');
                    const verbHintMessage = document.getElementById('verbHintMessage');

                    let currentVerbLevel = null;
                    let currentCorrectVerb = '';
                    let verbScore = 0;
                    let verbHintUsedThisChallenge = false;
                    let verbQuestionsPlayedInCurrentLevel = [];

                    const verbQuestions = {
                        'Basico': [
                            { sentence: "She ___ to the park yesterday.", correctAnswer: "went", incorrectAnswers: ["go", "goes"], emojis: ["🚶‍♀️", "🌳"], hint: "Pasado simple de 'go'." },
                            { sentence: "They ___ football every Sunday.", correctAnswer: "play", incorrectAnswers: ["plays", "played"], emojis: ["⚽", "🗓️"], hint: "Presente simple, tercera persona del plural." },
                            { sentence: "I ___ a delicious dinner last night.", correctAnswer: "cooked", incorrectAnswers: ["cook", "cooking"], emojis: ["👨‍🍳", "🍲"], hint: "Pasado simple de 'cook'." },
                            { sentence: "He ___ a new book now.", correctAnswer: "is reading", incorrectAnswers: ["read", "reads"], emojis: ["📚", "👀"], hint: "Presente continuo." },
                            { sentence: "We ___ happy to see you.", correctAnswer: "are", incorrectAnswers: ["is", "am"], emojis: ["😊", "🤝"], hint: "Verbo 'to be' para 'we'." }
                        ],
                        'Intermedio': [
                            { sentence: "If I ___ a bird, I would fly.", correctAnswer: "were", incorrectAnswers: ["was", "am", "is"], emojis: ["🐦", "☁️"], hint: "Segundo condicional, para situaciones hipotéticas." },
                            { sentence: "She has ___ that movie many times.", correctAnswer: "seen", incorrectAnswers: ["saw", "see", "seeing"], emojis: ["🎬", "👁️"], hint: "Participio pasado de 'see'." },
                            { sentence: "The book ___ by a famous author.", correctAnswer: "was written", incorrectAnswers: ["wrote", "writes", "is writing"], emojis: ["📝", "📜"], hint: "Voz pasiva en pasado." },
                            { sentence: "They ___ for hours before they found it.", correctAnswer: "had been searching", incorrectAnswers: ["searched", "were searching", "have searched"], emojis: ["🔍", "🕰️"], hint: "Pasado perfecto continuo." },
                            { sentence: "By next year, I ___ my degree.", correctAnswer: "will have finished", incorrectAnswers: ["will finish", "am finishing", "finish"], emojis: ["🎓", "✅"], hint: "Futuro perfecto." }
                        ],
                        'Avanzado': [
                            { sentence: "Had I known, I ___ differently.", correctAnswer: "would have acted", incorrectAnswers: ["would act", "will act", "acted", "had acted"], emojis: ["🤔", "🔄"], hint: "Tercer condicional." },
                            { sentence: "The old house, ___ for years, finally collapsed.", correctAnswer: "having been neglected", incorrectAnswers: ["neglected", "was neglected", "had neglected", "being neglected"], emojis: ["🏚️", "⏳"], hint: "Participio perfecto pasivo." },
                            { sentence: "It's high time you ___ your responsibilities.", correctAnswer: "accepted", incorrectAnswers: ["accept", "are accepting", "will accept", "have accepted"], emojis: ["⏰", "✅"], hint: "Expresión 'It's high time' seguida de pasado simple." },
                            { sentence: "Neither John nor Mary ___ the answer.", correctAnswer: "knows", incorrectAnswers: ["know", "are knowing", "have known", "knowing"], emojis: ["🤷", "💡"], hint: "Concordancia con 'neither... nor' (singular)." },
                            { sentence: "The committee ___ to vote on the proposal.", correctAnswer: "is going", incorrectAnswers: ["are going", "go", "goes", "will go"], emojis: ["🗳️", "🔜"], hint: "Sujeto colectivo singular." }
                        ]
                    };

                    // Mapeo de respuestas correctas a sus datos completos para acceder a los emojis y hints textuales
                    let verbQuestionDataMap = {};
                    for (const level in verbQuestions) {
                        verbQuestions[level].forEach(item => {
                            verbQuestionDataMap[item.correctAnswer] = item;
                        });
                    }

                    function updateVerbScoreDisplay() {
                        verbScoreDisplay.textContent = `Puntos: ${verbScore}`;
                        if (verbScore < 20 || verbHintUsedThisChallenge) {
                            verbHintBtn.disabled = true;
                            verbHintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                        } else {
                            verbHintBtn.disabled = false;
                            verbHintBtn.classList.remove('opacity-50', 'cursor-not-allowed');
                        }
                    }

                    function generateNewVerbQuestion() {
                        const questionsForLevel = verbQuestions[currentVerbLevel];
                        let availableQuestions = questionsForLevel.filter(q => !verbQuestionsPlayedInCurrentLevel.includes(q.sentence));

                        // Si todas las preguntas han sido usadas, resetear la lista de preguntas jugadas
                        if (availableQuestions.length === 0) {
                            verbQuestionsPlayedInCurrentLevel = [];
                            availableQuestions = questionsForLevel;
                        }

                        const randomQuestionData = availableQuestions[getRandomInt(0, availableQuestions.length - 1)];

                        verbSentenceDisplay.textContent = `"${randomQuestionData.sentence}"`;
                        currentCorrectVerb = randomQuestionData.correctAnswer;
                        verbQuestionsPlayedInCurrentLevel.push(randomQuestionData.sentence); // Añadir a la lista de jugadas

                        let options = [randomQuestionData.correctAnswer, ...randomQuestionData.incorrectAnswers];
                        // Limitar opciones según el nivel
                        if (currentVerbLevel === 'Basico') {
                            options = options.slice(0, 3);
                        } else if (currentVerbLevel === 'Intermedio') {
                            options = options.slice(0, 4);
                        } else if (currentVerbLevel === 'Avanzado') {
                            options = options.slice(0, 5);
                        }
                        options = shuffleArray(options);

                        verbOptionsContainer.innerHTML = '';
                        options.forEach(option => {
                            const button = document.createElement('button');
                            button.classList.add('verb-option-button');
                            button.textContent = option;
                            button.dataset.answer = option;
                            button.addEventListener('click', checkVerbAnswer);
                            verbOptionsContainer.appendChild(button);
                        });

                        verbFeedbackMessage.textContent = '';
                        verbFeedbackMessage.classList.remove('correct-message', 'incorrect-message');
                        verbHintMessage.classList.add('hidden'); // Ocultar pista al generar nueva pregunta
                        nextVerbBtn.classList.add('hidden');
                        verbOptionsContainer.querySelectorAll('.verb-option-button').forEach(btn => {
                            btn.disabled = false;
                            btn.classList.remove('correct', 'incorrect');
                        });
                        verbHintUsedThisChallenge = false; // Resetear el uso de pista
                        updateVerbScoreDisplay(); // Actualizar el estado del botón de pista
                    }

                    function checkVerbAnswer(event) {
                        const selectedAnswer = event.target.dataset.answer;
                        const allOptionButtons = verbOptionsContainer.querySelectorAll('.verb-option-button');

                        allOptionButtons.forEach(btn => {
                            btn.disabled = true; // Deshabilitar todos los botones después de una selección
                            if (btn.dataset.answer === currentCorrectVerb) {
                                btn.classList.add('correct');
                            } else if (btn.dataset.answer === selectedAnswer) {
                                btn.classList.add('incorrect');
                            }
                        });

                        if (selectedAnswer === currentCorrectVerb) {
                            playCorrectSound(); // Play sound on correct answer
                            verbFeedbackMessage.textContent = '¡Correcto!';
                            verbFeedbackMessage.classList.remove('incorrect-message');
                            verbFeedbackMessage.classList.add('correct-message');
                            verbScore += 10;
                            updateVerbScoreDisplay();
                        } else {
                            playIncorrectSound(); // Play sound on incorrect answer
                            verbFeedbackMessage.textContent = `Incorrecto. La respuesta correcta era: ${currentCorrectVerb}`;
                            verbFeedbackMessage.classList.remove('correct-message');
                            verbFeedbackMessage.classList.add('incorrect-message');
                        }
                        nextVerbBtn.classList.remove('hidden');
                        verbHintBtn.disabled = true; // Deshabilitar pista una vez respondido
                        verbHintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                    }

                    // Función para mostrar la pista de emojis
                    function displayVerbHint() {
                        if (verbScore >= 20 && !verbHintUsedThisChallenge) {
                            verbScore -= 20;
                            updateVerbScoreDisplay();
                            verbHintUsedThisChallenge = true;
                            verbHintBtn.disabled = true;
                            verbHintBtn.classList.add('opacity-50', 'cursor-not-allowed');

                            const questionInfo = verbQuestionDataMap[currentCorrectVerb];
                            let hintText = '';
                            if (questionInfo && questionInfo.emojis && questionInfo.emojis.length > 0) {
                                hintText += questionInfo.emojis.join(' ');
                            }
                            if (questionInfo && questionInfo.hint) {
                                hintText += (hintText ? ' - ' : '') + questionInfo.hint;
                            }

                            if (hintText) {
                                verbHintMessage.textContent = hintText;
                            } else {
                                verbHintMessage.textContent = 'No hay pista disponible para esta pregunta.';
                            }
                            verbHintMessage.classList.remove('hidden');
                        } else if (verbHintUsedThisChallenge) {
                            verbFeedbackMessage.textContent = 'Ya usaste la pista para esta pregunta.';
                            verbFeedbackMessage.classList.add('incorrect-message');
                        } else {
                            verbFeedbackMessage.textContent = 'No tienes suficientes puntos para una pista (necesitas 20).';
                            verbFeedbackMessage.classList.add('incorrect-message');
                        }
                    }

                    nextVerbBtn.addEventListener('click', generateNewVerbQuestion);
                    verbHintBtn.addEventListener('click', displayVerbHint);

                    verbLevelButtons.forEach(button => {
                        button.addEventListener('click', (event) => {
                            currentVerbLevel = event.target.dataset.level;
                            verbLevelSelectionDiv.classList.add('hidden');
                            verbGamePlayDiv.classList.remove('hidden');
                            verbScore = 0; // Reset score for new game
                            verbQuestionsPlayedInCurrentLevel = []; // Resetear preguntas jugadas al cambiar de nivel
                            updateVerbScoreDisplay();
                            generateNewVerbQuestion();
                        });
                    });

                    // Ocultar el juego al inicio, mostrar solo la selección de nivel
                    verbGamePlayDiv.classList.add('hidden');
                    verbLevelSelectionDiv.classList.remove('hidden');
                }
            },
            'Preguntas de Cultura General': {
                category: 'Cultura General',
                icon: '🌍',
                description: 'Pon a prueba tus conocimientos sobre historia, geografía, ciencia y más. ¡Aprende y diviértete!',
                gameHtml: `
                    <div id="generalKnowledgeLevelSelection" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg min-h-[200px]">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Selecciona Nivel de Dificultad</h3>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-lg justify-center">
                            <button class="general-knowledge-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Basico">
                                Básico (3 opciones)
                            </button>
                            <button class="general-knowledge-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Intermedio">
                                Intermedio (4 opciones)
                            </button>
                            <button class="general-knowledge-level-button py-3 px-8 rounded-full shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-300 flex-1" data-level="Avanzado">
                                Avanzado (5 opciones)
                            </button>
                        </div>
                    </div>

                    <div id="generalKnowledgeGamePlay" class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg mt-8 hidden">
                        <h3 class="text-2xl font-bold text-gray-800 mb-4">Pregunta de Cultura General</h3>
                        <div id="generalKnowledgeScoreDisplay" class="text-2xl font-bold text-purple-700 mb-4">Puntos: 0</div>

                        <p id="generalKnowledgeQuestion" class="text-xl italic text-gray-700 text-center mb-6 px-4 py-3 bg-gray-100 rounded-lg border border-gray-200 w-full max-w-xl">
                            "¿Cuál es el río más largo del mundo?"
                        </p>

                        <div id="generalKnowledgeOptions" class="grid grid-cols-1 md:grid-cols-2 gap-4 w-full max-w-xl">
                            <!-- Opciones se cargarán aquí -->
                        </div>

                        <p id="generalKnowledgeFeedbackMessage" class="text-xl font-bold mt-4"></p>
                        <p id="generalKnowledgeHintMessage" class="text-gray-700 text-center mt-2 hidden text-3xl"></p>
                        <div class="flex flex-col md:flex-row space-y-4 md:space-y-0 md:space-x-4 w-full max-w-sm">
                            <button id="nextGeneralKnowledgeBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md hidden flex-1">
                                Siguiente Pregunta
                            </button>
                            <button id="generalKnowledgeHintBtn" class="game-action-button text-white font-bold py-2 px-6 rounded-full shadow-md flex-1">
                                Pista (-20 Puntos)
                            </button>
                        </div>
                    </div>
                `,
                gameScript: function() {
                    const generalKnowledgeLevelSelectionDiv = document.getElementById('generalKnowledgeLevelSelection');
                    const generalKnowledgeGamePlayDiv = document.getElementById('generalKnowledgeGamePlay');
                    const generalKnowledgeLevelButtons = document.querySelectorAll('.general-knowledge-level-button');

                    const generalKnowledgeQuestionDisplay = document.getElementById('generalKnowledgeQuestion');
                    const generalKnowledgeOptionsContainer = document.getElementById('generalKnowledgeOptions');
                    const generalKnowledgeFeedbackMessage = document.getElementById('generalKnowledgeFeedbackMessage');
                    const nextGeneralKnowledgeBtn = document.getElementById('nextGeneralKnowledgeBtn');
                    const generalKnowledgeScoreDisplay = document.getElementById('generalKnowledgeScoreDisplay');
                    const generalKnowledgeHintBtn = document.getElementById('generalKnowledgeHintBtn');
                    const generalKnowledgeHintMessage = document.getElementById('generalKnowledgeHintMessage');

                    let currentGeneralKnowledgeLevel = null;
                    let currentCorrectAnswer = '';
                    let generalKnowledgeScore = 0;
                    let generalKnowledgeHintUsedThisChallenge = false;
                    let generalKnowledgeQuestionsPlayedInCurrentLevel = [];

                    const generalKnowledgeQuestions = {
                        'Basico': [
                            { question: "¿Cuál es el río más largo del mundo?", correctAnswer: "Nilo", incorrectAnswers: ["Amazonas", "Yangtsé"], emojis: ["🏞️", "🇪🇬"], hint: "Está en África y es famoso por una antigua civilización." },
                            { question: "¿Qué animal es conocido como el 'rey de la selva'?", correctAnswer: "León", incorrectAnswers: ["Tigre", "Elefante"], emojis: ["🦁", "👑"], hint: "Tiene una gran melena." },
                            { question: "¿Cuántos planetas hay en nuestro sistema solar?", correctAnswer: "Ocho", incorrectAnswers: ["Nueve", "Siete"], emojis: ["🪐", "8️⃣"], hint: "Plutón ya no cuenta." },
                            { question: "¿Cuál es el país más grande del mundo por superficie?", correctAnswer: "Rusia", incorrectAnswers: ["Canadá", "China"], emojis: ["🇷🇺", "🗺️"], hint: "Es el más grande y está en Eurasia." },
                            { question: "¿Qué gas respiramos los seres humanos?", correctAnswer: "Oxígeno", incorrectAnswers: ["Dióxido de carbono", "Nitrógeno"], emojis: ["🌬️", "💨"], hint: "Es vital para la vida." }
                        ],
                        'Intermedio': [
                            { question: "¿Quién pintó la Mona Lisa?", correctAnswer: "Leonardo da Vinci", incorrectAnswers: ["Vincent van Gogh", "Pablo Picasso", "Claude Monet"], emojis: ["🎨", "🇮🇹"], hint: "Un genio del Renacimiento." },
                            { question: "¿Cuál es la capital de Australia?", correctAnswer: "Canberra", incorrectAnswers: ["Sídney", "Melbourne", "Brisbane"], emojis: ["🇦🇺", "🏙️"], hint: "No es la ciudad más grande, pero es la sede del gobierno." },
                            { question: "¿En qué año llegó el hombre a la Luna?", correctAnswer: "1969", incorrectAnswers: ["1957", "1972", "1981"], emojis: ["🚀", "🌕"], hint: "Un pequeño paso para el hombre..." },
                            { question: "¿Cuál es el océano más grande del mundo?", correctAnswer: "Pacífico", incorrectAnswers: ["Atlántico", "Índico", "Ártico"], emojis: ["🌊", "🌏"], hint: "Cubre casi un tercio de la superficie terrestre." },
                            { question: "¿Quién escribió 'Romeo y Julieta'?", correctAnswer: "William Shakespeare", incorrectAnswers: ["Charles Dickens", "Jane Austen", "Lord Byron"], emojis: ["🎭", "🇬🇧"], hint: "El Bardo de Avon." }
                        ],
                        'Avanzado': [
                            { question: "¿Cuál es el nombre de la galaxia en la que se encuentra nuestro sistema solar?", correctAnswer: "Vía Láctea", incorrectAnswers: ["Andrómeda", "Triángulo", "Sombrero", "Cigarro"], emojis: ["🌌", "🥛"], hint: "Su nombre hace referencia a su apariencia." },
                            { question: "¿Qué elemento químico tiene el símbolo 'Fe'?", correctAnswer: "Hierro", incorrectAnswers: ["Oro", "Plata", "Cobre", "Plomo"], emojis: ["🧪", "🔩"], hint: "Es un metal muy común y magnético." },
                            { question: "¿Quién formuló la teoría de la relatividad?", correctAnswer: "Albert Einstein", incorrectAnswers: ["Isaac Newton", "Stephen Hawking", "Niels Bohr", "Max Planck"], emojis: ["⚛️", "🤯"], hint: "Famoso por su ecuación E=mc²." },
                            { question: "¿Cuál es la obra musical más famosa de Ludwig van Beethoven?", correctAnswer: "Novena Sinfonía", incorrectAnswers: ["Quinta Sinfonía", "Claro de Luna", "Para Elisa", "Fidelio"], emojis: ["🎶", "👂"], hint: "Incluye el 'Himno a la Alegría'." },
                            { question: "¿Qué país fue el primero en otorgar el derecho al voto a las mujeres a nivel nacional?", correctAnswer: "Nueva Zelanda", incorrectAnswers: ["Finlandia", "Australia", "Noruega", "Estados Unidos"], emojis: ["🇳🇿", "🗳️"], hint: "Fue en 1893." }
                        ]
                    };

                    // Mapeo de respuestas correctas a sus datos completos para acceder a los emojis y hints textuales
                    let generalKnowledgeQuestionDataMap = {};
                    for (const level in generalKnowledgeQuestions) {
                        generalKnowledgeQuestions[level].forEach(item => {
                            generalKnowledgeQuestionDataMap[item.correctAnswer] = item;
                        });
                    }

                    function updateGeneralKnowledgeScoreDisplay() {
                        generalKnowledgeScoreDisplay.textContent = `Puntos: ${generalKnowledgeScore}`;
                        if (generalKnowledgeScore < 20 || generalKnowledgeHintUsedThisChallenge) {
                            generalKnowledgeHintBtn.disabled = true;
                            generalKnowledgeHintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                        } else {
                            generalKnowledgeHintBtn.disabled = false;
                            generalKnowledgeHintBtn.classList.remove('opacity-50', 'cursor-not-allowed');
                        }
                    }

                    function generateNewGeneralKnowledgeQuestion() {
                        const questionsForLevel = generalKnowledgeQuestions[currentGeneralKnowledgeLevel];
                        let availableQuestions = questionsForLevel.filter(q => !generalKnowledgeQuestionsPlayedInCurrentLevel.includes(q.question));

                        // Si todas las preguntas han sido usadas, resetear la lista de preguntas jugadas
                        if (availableQuestions.length === 0) {
                            generalKnowledgeQuestionsPlayedInCurrentLevel = [];
                            availableQuestions = questionsForLevel;
                        }

                        const randomQuestionData = availableQuestions[getRandomInt(0, availableQuestions.length - 1)];

                        generalKnowledgeQuestionDisplay.textContent = `"${randomQuestionData.question}"`;
                        currentCorrectAnswer = randomQuestionData.correctAnswer;
                        generalKnowledgeQuestionsPlayedInCurrentLevel.push(randomQuestionData.question); // Añadir a la lista de jugadas

                        let options = [randomQuestionData.correctAnswer, ...randomQuestionData.incorrectAnswers];
                        // Limitar opciones según el nivel
                        if (currentGeneralKnowledgeLevel === 'Basico') {
                            options = options.slice(0, 3);
                        } else if (currentGeneralKnowledgeLevel === 'Intermedio') {
                            options = options.slice(0, 4);
                        } else if (currentGeneralKnowledgeLevel === 'Avanzado') {
                            options = options.slice(0, 5);
                        }
                        options = shuffleArray(options);

                        generalKnowledgeOptionsContainer.innerHTML = '';
                        options.forEach(option => {
                            const button = document.createElement('button');
                            button.classList.add('general-knowledge-option-button');
                            button.textContent = option;
                            button.dataset.answer = option;
                            button.addEventListener('click', checkGeneralKnowledgeAnswer);
                            generalKnowledgeOptionsContainer.appendChild(button);
                        });

                        generalKnowledgeFeedbackMessage.textContent = '';
                        generalKnowledgeFeedbackMessage.classList.remove('correct-message', 'incorrect-message');
                        generalKnowledgeHintMessage.classList.add('hidden'); // Ocultar pista al generar nueva pregunta
                        nextGeneralKnowledgeBtn.classList.add('hidden');
                        generalKnowledgeOptionsContainer.querySelectorAll('.general-knowledge-option-button').forEach(btn => {
                            btn.disabled = false;
                            btn.classList.remove('correct', 'incorrect');
                        });
                        generalKnowledgeHintUsedThisChallenge = false; // Resetear el uso de pista
                        updateGeneralKnowledgeScoreDisplay(); // Actualizar el estado del botón de pista
                    }

                    function checkGeneralKnowledgeAnswer(event) {
                        const selectedAnswer = event.target.dataset.answer;
                        const allOptionButtons = generalKnowledgeOptionsContainer.querySelectorAll('.general-knowledge-option-button');

                        allOptionButtons.forEach(btn => {
                            btn.disabled = true; // Deshabilitar todos los botones después de una selección
                            if (btn.dataset.answer === currentCorrectAnswer) {
                                btn.classList.add('correct');
                            } else if (btn.dataset.answer === selectedAnswer) {
                                btn.classList.add('incorrect');
                            }
                        });

                        if (selectedAnswer === currentCorrectAnswer) {
                            playCorrectSound(); // Play sound on correct answer
                            generalKnowledgeFeedbackMessage.textContent = '¡Correcto!';
                            generalKnowledgeFeedbackMessage.classList.remove('incorrect-message');
                            generalKnowledgeFeedbackMessage.classList.add('correct-message');
                            generalKnowledgeScore += 10;
                            updateGeneralKnowledgeScoreDisplay();
                        } else {
                            playIncorrectSound(); // Play sound on incorrect answer
                            generalKnowledgeFeedbackMessage.textContent = `Incorrecto. La respuesta correcta era: ${currentCorrectAnswer}`;
                            generalKnowledgeFeedbackMessage.classList.remove('correct-message');
                            generalKnowledgeFeedbackMessage.classList.add('incorrect-message');
                        }
                        nextGeneralKnowledgeBtn.classList.remove('hidden');
                        generalKnowledgeHintBtn.disabled = true; // Deshabilitar pista una vez respondido
                        generalKnowledgeHintBtn.classList.add('opacity-50', 'cursor-not-allowed');
                    }

                    // Función para mostrar la pista de emojis
                    function displayGeneralKnowledgeHint() {
                        if (generalKnowledgeScore >= 20 && !generalKnowledgeHintUsedThisChallenge) {
                            generalKnowledgeScore -= 20;
                            updateGeneralKnowledgeScoreDisplay();
                            generalKnowledgeHintUsedThisChallenge = true;
                            generalKnowledgeHintBtn.disabled = true;
                            generalKnowledgeHintBtn.classList.add('opacity-50', 'cursor-not-allowed');

                            const questionInfo = generalKnowledgeQuestionDataMap[currentCorrectAnswer];
                            let hintText = '';
                            if (questionInfo && questionInfo.emojis && questionInfo.emojis.length > 0) {
                                hintText += questionInfo.emojis.join(' ');
                            }
                            if (questionInfo && questionInfo.hint) {
                                hintText += (hintText ? ' - ' : '') + questionInfo.hint;
                            }

                            if (hintText) {
                                generalKnowledgeHintMessage.textContent = hintText;
                            } else {
                                generalKnowledgeHintMessage.textContent = 'No hay pista disponible para esta pregunta.';
                            }
                            generalKnowledgeHintMessage.classList.remove('hidden');
                        } else if (generalKnowledgeHintUsedThisChallenge) {
                            generalKnowledgeFeedbackMessage.textContent = 'Ya usaste la pista para esta pregunta.';
                            generalKnowledgeFeedbackMessage.classList.add('incorrect-message');
                        } else {
                            generalKnowledgeFeedbackMessage.textContent = 'No tienes suficientes puntos para una pista (necesitas 20).';
                            generalKnowledgeFeedbackMessage.classList.add('incorrect-message');
                        }
                    }

                    nextGeneralKnowledgeBtn.addEventListener('click', generateNewGeneralKnowledgeQuestion);
                    generalKnowledgeHintBtn.addEventListener('click', displayGeneralKnowledgeHint);

                    generalKnowledgeLevelButtons.forEach(button => {
                        button.addEventListener('click', (event) => {
                            currentGeneralKnowledgeLevel = event.target.dataset.level;
                            generalKnowledgeLevelSelectionDiv.classList.add('hidden');
                            generalKnowledgeGamePlayDiv.classList.remove('hidden');
                            generalKnowledgeScore = 0; // Reset score for new game
                            generalKnowledgeQuestionsPlayedInCurrentLevel = []; // Resetear preguntas jugadas al cambiar de nivel
                            updateGeneralKnowledgeScoreDisplay();
                            generateNewGeneralKnowledgeQuestion();
                        });
                    });

                    // Ocultar el juego al inicio, mostrar solo la selección de nivel
                    generalKnowledgeGamePlayDiv.classList.add('hidden');
                    generalKnowledgeLevelSelectionDiv.classList.remove('hidden');
                }
            },
            'Proximamente': {
                category: 'Otros',
                icon: '⏳',
                description: '¡Estamos trabajando en más minijuegos para ti! Vuelve pronto para descubrir nuevas aventuras.',
                gameHtml: `
                    <div class="flex flex-col items-center space-y-6 p-6 bg-white rounded-xl shadow-lg min-h-[200px]">
                        <div class="text-6xl mb-4">✨</div>
                        <h3 class="text-2xl font-bold text-gray-800 mb-3">¡Más Minijuegos en Camino!</h3>
                        <p class="text-gray-600 text-center">Estamos desarrollando nuevos y emocionantes desafíos para que sigas aprendiendo y divirtiéndote.</p>
                        <p class="text-gray-600 text-center">¡Vuelve pronto para descubrir las novedades!</p>
                    </div>
                `,
                gameScript: function() {
                    // No hay lógica de juego para esta tarjeta
                }
            }
        };

        // --- Estado de la Aplicación ---
        let selectedGameModule = null;

        const appContainer = document.getElementById('app');
        const homeLink = document.getElementById('homeLink');

        // --- Funciones de Renderizado ---

        function renderAreaGameCards() {
            let mathGamesHtml = '';
            let filosofiaGamesHtml = '';
            let inglesGamesHtml = '';
            let culturaGeneralGamesHtml = '';
            let otrosGamesHtml = '';

            for (const gameName in areaGamesContent) {
                const game = areaGamesContent[gameName];
                const cardHtml = `
                    <div class="game-card bg-white rounded-xl shadow-lg p-6 flex flex-col items-center text-center" data-game="${gameName}">
                        <div class="text-6xl mb-4">${game.icon}</div>
                        <h3 class="text-2xl font-bold text-gray-800 mb-3">${gameName}</h3>
                        <p class="text-gray-600 mb-4">${game.description}</p>
                        <button class="nav-button text-white font-bold py-3 px-6 rounded-full shadow-lg hover:shadow-xl focus:outline-none focus:ring-4 focus:ring-indigo-300 w-full">
                            Jugar Ahora
                        </button>
                    </div>
                `;
                if (game.category === 'Matematicas') {
                    mathGamesHtml += cardHtml;
                } else if (game.category === 'Filosofia') {
                    filosofiaGamesHtml += cardHtml;
                } else if (game.category === 'Ingles') {
                    inglesGamesHtml += cardHtml;
                } else if (game.category === 'Cultura General') {
                    culturaGeneralGamesHtml += cardHtml;
                } else if (game.category === 'Otros') {
                    otrosGamesHtml += cardHtml;
                }
            }

            let html = `
                <section id="games" class="mb-12">
                    <h2 class="text-4xl font-extrabold text-center text-gray-800 mb-10">Explora los Minijuegos de Chisfuerzo</h2>

                    <h3 class="text-3xl font-bold text-gray-700 mb-6 text-center">Zona Matemática</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8 mb-12">
                        ${mathGamesHtml}
                    </div>

                    <h3 class="text-3xl font-bold text-gray-700 mb-6 text-center">Zona Filosofía</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8 mb-12">
                        ${filosofiaGamesHtml}
                    </div>

                    <h3 class="text-3xl font-bold text-gray-700 mb-6 text-center">Zona Inglés</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8 mb-12">
                        ${inglesGamesHtml}
                    </div>

                    <h3 class="text-3xl font-bold text-gray-700 mb-6 text-center">Zona Cultura General</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8 mb-12">
                        ${culturaGeneralGamesHtml}
                    </div>

                    <h3 class="text-3xl font-bold text-gray-700 mb-6 text-center">Otros</h3>
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
                        ${otrosGamesHtml}
                    </div>
                </section>
            `;
            appContainer.innerHTML = html;

            // Añadir event listeners a los botones "Jugar Ahora"
            document.querySelectorAll('.game-card button').forEach(button => {
                button.addEventListener('click', (event) => {
                    const gameName = event.target.closest('.game-card').dataset.game;
                    loadGame(gameName);
                });
            });
        }

        function loadGame(gameName) {
            const game = areaGamesContent[gameName];
            if (game) {
                appContainer.innerHTML = `
                    <section id="currentGame" class="flex flex-col items-center justify-center min-h-[60vh]">
                        <h2 class="text-4xl font-extrabold text-center text-gray-800 mb-6">${gameName}</h2>
                        ${game.gameHtml}
                    </section>
                `;
                // Ejecutar el script del juego
                if (game.gameScript) {
                    game.gameScript();
                }
            } else {
                appContainer.innerHTML = `<p class="text-center text-xl text-red-500">Juego no encontrado.</p>`;
            }
        }

        // --- Inicialización ---
        document.addEventListener('DOMContentLoaded', () => {
            renderAreaGameCards(); // Cargar las tarjetas de juego al inicio

            homeLink.addEventListener('click', (event) => {
                event.preventDefault();
                renderAreaGameCards();
            });
        });
    </script>
</body>
</html>
