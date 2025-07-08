<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chisfuerzo: Aprende y Crece</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .list-disc li::marker {
            color: #2563eb;
        }
        .list-circle li::marker {
            color: #1d4ed8;
        }
        .theory-content p {
            line-height: 1.6;
        }
        .theory-content em {
            font-style: italic;
        }
        .theory-content strong {
            font-weight: 600;
        }
    </style>
</head>
<body class="min-h-screen bg-gradient-to-br from-sky-50 to-blue-100 flex flex-col items-center justify-center p-6">

    <div id="app" class="w-full max-w-7xl mx-auto py-10">
    </div>

    <footer class="mt-16 text-center text-gray-600 text-sm">
        <p>App ID: <span id="app-id-display">Cargando...</span></p>
        <p>User ID: <span id="user-id-display">Cargando...</span></p>
        <p>&copy; 2025 Chisfuerzo. Todos los derechos reservados.</p>
    </footer>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let selectedCourse = null;
        let selectedLevel = 'basico';
        let db = null;
        let auth = null;
        let currentUserId = 'Cargando...';
        let isAuthReady = false;
        let userAnswers = {};
        let score = 0;

        document.getElementById('app-id-display').textContent = appId;

        const initFirebase = async () => {
            if (Object.keys(firebaseConfig).length > 0) {
                try {
                    const app = initializeApp(firebaseConfig);
                    db = getFirestore(app);
                    auth = getAuth(app);

                    if (initialAuthToken) {
                        await signInWithCustomToken(auth, initialAuthToken);
                    } else {
                        await signInAnonymously(auth);
                    }

                    onAuthStateChanged(auth, (user) => {
                        if (user) {
                            currentUserId = user.uid;
                        } else {
                            currentUserId = 'No autenticado';
                        }
                        document.getElementById('user-id-display').textContent = currentUserId;
                        isAuthReady = true;
                    });
                } catch (error) {
                    console.error("Error initializing Firebase:", error);
                    currentUserId = 'Error de Firebase';
                    document.getElementById('user-id-display').textContent = currentUserId;
                    isAuthReady = true;
                }
            } else {
                console.warn("Firebase config not available. Running without Firebase features.");
                currentUserId = 'Sin Firebase';
                document.getElementById('user-id-display').textContent = currentUserId;
                isAuthReady = true;
            }
        };

        const courses = [
            { id: 'matematicas', name: 'Matemáticas', description: 'Explora el mundo de los números y la lógica. 🔢' },
            { id: 'comunicacion', name: 'Comunicación', description: 'Mejora tus habilidades de expresión oral y escrita. 🗣️✍️' },
            { id: 'cyt', name: 'Ciencia y Tecnología', description: 'Descubre los principios de la ciencia y los avances tecnológicos. 🔬💡' },
            { id: 'ingles', name: 'Inglés', description: 'Aprende un nuevo idioma y abre nuevas oportunidades. 🇬🇧🇺🇸' },
            { id: 'arte', name: 'Arte', description: 'Explora la creatividad, técnicas y la historia del arte. 🎨🖌️' },
            { id: 'computo', name: 'Cómputo', description: 'Descubre los fundamentos de la informática y la programación. 💻🤖' },
            { id: 'minijuegos', name: 'Minijuegos', description: '¡Diviértete con nuestros minijuegos aquí! 🎮', isExternal: true, url: 'https://cristopher-campos.github.io/chisfuerzo-minijuegos/' }, // New Minigames Course
        ];

        const courseContent = {
            matematicas: {
                basico: {
                    theory: `
                        <p class="mb-6">En el nivel Básico de Matemáticas ➕, te familiarizarás con los pilares fundamentales: los números, las operaciones básicas y las formas geométricas elementales. Este nivel te proporciona las herramientas esenciales para el razonamiento cuantitativo y la resolución de problemas cotidianos. Es el cimiento sobre el cual se construye todo el conocimiento matemático posterior.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Rama: Aritmética 🔢 - Los Fundamentos Numéricos</h4>
                            <p class="mb-2">La aritmética es la rama más básica y fundamental de las matemáticas, centrada en las operaciones con números. Es la base sobre la que se construyen todas las demás ramas, permitiéndonos cuantificar y comparar elementos en el mundo real.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Números Naturales y Enteros:</strong>
                                <p>Los números naturales ($\mathbb{N} = \{1, 2, 3, \dots\}$) son los que usamos para contar y ordenar elementos. Son infinitos y se utilizan en situaciones donde no hay cantidades negativas ni fraccionarias. Los números enteros ($\mathbb{Z} = \{\dots, -2, -1, 0, 1, 2, \dots\}$) incluyen los naturales, el cero y los números negativos. Son esenciales para representar cantidades completas, como temperaturas bajo cero, deudas, o niveles por debajo de un punto de referencia. 🥶</p>
                                <p class="mt-2"><em>Ejemplo:</em> Si tienes 5 manzanas 🍎 y te comes 2, te quedan 3. Esto se representa con números naturales. Si la temperatura es de $5^\circ C$ y baja $7^\circ C$, la nueva temperatura es $-2^\circ C$, un número entero negativo.</p>
                            </li>
                            <li><strong>Operaciones Básicas:</strong>
                                <ul class="list-circle list-inside ml-8">
                                <li><strong>Suma y Resta:</strong> La suma es la operación de combinar cantidades (ej. $7 + 4 = 11$). Es conmutativa ($a+b=b+a$) y asociativa ($(a+b)+c=a+(b+c)$). La resta es la operación inversa de la suma, utilizada para quitar cantidades o encontrar diferencias (ej. $11 - 4 = 7$).
                                <p class="mt-1"><em>Ejemplo:</em> Si tienes 10 caramelos 🍬 y regalas 3, te quedan $10 - 3 = 7$ caramelos. Si sumas 5 euros y 3 euros, tienes 8 euros.</p></li>
                                <li><strong>Multiplicación y División:</strong> La multiplicación es una suma repetida (ej. $3 \times 4 = 3 + 3 + 3 + 3 = 12$). Se utiliza para calcular el total de grupos iguales. La división es el reparto de una cantidad en partes iguales (ej. $12 \div 3 = 4$). Es la operación inversa de la multiplicación. Es crucial recordar que la división por cero no está definida en matemáticas.
                                <p class="mt-1"><em>Ejemplo:</em> Si cada amigo tiene 2 galletas 🍪 y tienes 5 amigos, en total tienes $2 \times 5 = 10$ galletas. Si 15 estudiantes se dividen en grupos de 3, hay $15 \div 3 = 5$ grupos. 🧑‍🎓</p></li>
                                </ul>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Rama: Conceptos Preliminares de Álgebra y Geometría 📐 - Primeras Ideas</h4>
                            <p class="mb-2">Una introducción suave a las ideas que se desarrollarán en niveles posteriores, sentando las bases para el pensamiento abstracto y espacial. Estos conceptos son la puerta de entrada a áreas más complejas de las matemáticas.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Fracciones y Decimales:</strong>
                                <p>Las fracciones representan partes de un todo (ej. $1/2$, $3/4$). Son útiles para expresar divisiones inexactas o porciones. Los decimales son otra forma de representar números no enteros, especialmente útiles para medidas, dinero y cálculos científicos (ej. $0.5$, $0.75$). Aprenderás a entender su relación y realizar operaciones simples como sumar y restar fracciones con el mismo denominador, así como conversiones básicas entre fracciones y decimales. 🍕💰</p>
                                <p class="mt-2"><em>Ejemplo:</em> Si una pizza se divide en 8 porciones y te comes 2, has comido $2/8$ o $1/4$ de la pizza. $0.25$ es lo mismo que $1/4$. Un billete de 50 céntimos es $0.50$ euros.</p>
                            </li>
                            <li><strong>Figuras Planas Básicas:</strong>
                                <p>Reconocimiento de formas geométricas fundamentales como cuadrados 🟦, círculos ⭕, triángulos 🔺 y rectángulos 🟨. Entender sus propiedades básicas como el número de lados, la igualdad de sus lados, si tienen ángulos rectos o si son simétricas. Esto es fundamental para describir el mundo que nos rodea.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Un letrero de "STOP" es un octágono. Una rueda es un círculo. Una ventana rectangular.</p>
                            </li>
                            <li><strong>Perímetro y Área:</strong>
                                <p>El perímetro es la distancia alrededor de una figura (la "frontera"). Se calcula sumando la longitud de todos sus lados. El área es el espacio que ocupa una figura en una superficie bidimensional. Cálculo simple para figuras básicas como cuadrados, rectángulos y triángulos. Estas medidas son esenciales en la vida diaria, desde construir una valla hasta pintar una pared.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Para un cuadrado con lado de 4 cm, el perímetro es $4 \times 4 = 16$ cm (o $4+4+4+4=16$) y el área es $4 \times 4 = 16$ cm$^2$. Si quieres saber cuánta pintura necesitas para una pared, calculas su área. 📏</p>
                            </li>
                            </ul>
                        </div>
                        <p class="mt-4">Este nivel te dará una base sólida para avanzar en tu viaje matemático. ¡La práctica constante y la aplicación de estos conceptos en situaciones reales son clave para un aprendizaje duradero! 💪</p>
                    `,
                    questions: [
                        { id: 'm_b_q1', text: 'Calcula: $15 + 23 - 8$.', type: 'number', answer: 30 },
                        { id: 'm_b_q2', text: 'Si tienes 20 caramelos y los divides entre 4 amigos, ¿cuántos caramelos recibe cada uno?', type: 'number', answer: 5 },
                        { id: 'm_b_q3', text: 'Convierte la fracción $1/2$ a decimal.', type: 'number', answer: 0.5 },
                        { id: 'm_b_q4', text: 'Calcula el perímetro de un rectángulo con lados de 5 cm y 3 cm.', type: 'number', answer: 16 },
                        { id: 'm_b_q5', text: 'Si un cuadrado tiene un lado de 6 cm, ¿cuál es su área en cm$^2$?', type: 'number', answer: 36 },
                        { id: 'm_b_q6', text: '¿Cuál es el resultado de $10 \times 7 + 5$?', type: 'number', answer: 75 },
                        { id: 'm_b_q7', text: 'Si un pastel se divide en 4 partes iguales y te comes 1, ¿qué fracción del pastel te comiste?', type: 'text', answer: '1/4' },
                        { id: 'm_b_q8', text: '¿Cuál es el número entero que representa una deuda de 15 dólares?', type: 'number', answer: -15 },
                        { id: 'm_b_q9', text: 'Si un triángulo tiene lados de 3 cm, 4 cm y 5 cm, ¿cuál es su perímetro?', type: 'number', answer: 12 },
                        { id: 'm_b_q10', text: 'Convierte $0.75$ a fracción.', type: 'text', answer: '3/4' },
                        { id: 'm_b_q11', text: '¿Cuál es el resultado de $25 \div 5 - 3$?', type: 'number', answer: 2 },
                        { id: 'm_b_q12', text: 'Nombra una figura geométrica con 4 lados iguales y 4 ángulos rectos.', type: 'text', answer: 'Cuadrado' },
                        { id: 'm_b_q13', text: 'Si un libro cuesta $8 y una pluma $2, ¿cuánto cuestan 3 libros y 2 plumas?', type: 'number', answer: 28 },
                        { id: 'm_b_q14', text: '¿Cuál es el número natural más pequeño?', type: 'number', answer: 1 },
                        { id: 'm_b_q15', text: 'Calcula el área de un triángulo con base de 10 cm y altura de 4 cm.', type: 'number', answer: 20 },
                        { id: 'm_b_q16', text: 'Si la temperatura es de $5^\circ C$ y baja $7^\circ C$, ¿cuál es la nueva temperatura?', type: 'number', answer: -2 },
                        { id: 'm_b_q17', text: '¿Cuántos minutos hay en 2 horas?', type: 'number', answer: 120 },
                        { id: 'm_b_q18', text: 'Simplifica la fracción $4/8$.', type: 'text', answer: '1/2' },
                        { id: 'm_b_q19', text: 'Si un círculo tiene un radio de 7 cm, ¿cuál es su diámetro?', type: 'number', answer: 14 },
                        { id: 'm_b_q20', text: '¿Cuál es el resultado de $12 \times (5 - 2)$?', type: 'number', answer: 36 },
                    ]
                },
                intermedio: {
                    theory: `
                        <p class="mb-6">En el nivel Intermedio de Matemáticas 📈, profundizaremos en el álgebra para resolver problemas con incógnitas, exploraremos la geometría en un plano de coordenadas y entenderemos las relaciones en triángulos, además de potenciar tus habilidades con números y sus inversas. Este nivel te equipará con herramientas más sofisticadas para el análisis cuantitativo.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Rama: Álgebra 📊 - El Lenguaje de las Incógnitas</h4>
                            <p class="mb-2">El álgebra es el lenguaje de las matemáticas, donde usamos símbolos (variables) para representar cantidades desconocidas y relaciones, permitiendo generalizar y resolver problemas más complejos. Es la transición de lo numérico a lo simbólico, abriendo un vasto campo de aplicaciones.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Ecuaciones Lineales:</strong>
                                <p>Son ecuaciones donde la variable tiene un exponente de 1 (ej. $2x + 5 = 11$). Aprenderás a despejar la incógnita utilizando operaciones inversas (sumar/restar, multiplicar/dividir) para encontrar el valor de la variable que satisface la igualdad. Son la base para modelar relaciones simples en la vida real.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Resuelve $3x - 7 = 8$. Suma 7 a ambos lados: $3x = 15$. Divide por 3: $x = 5$. Si el doble de tu edad más 5 es 25, ¿cuántos años tienes? ($2x+5=25$).</p>
                            </li>
                            <li><strong>Sistemas de Ecuaciones:</strong>
                                <p>Resolver conjuntos de dos o más ecuaciones con múltiples incógnitas (ej. $x+y=5$ y $2x-y=1$). Métodos comunes incluyen sustitución (despejar una variable en una ecuación y sustituirla en la otra) y eliminación (sumar o restar ecuaciones para cancelar una variable). Estos sistemas son cruciales para problemas con múltiples condiciones o variables interdependientes.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Para el sistema anterior, sumando ambas ecuaciones se elimina $y$: $3x = 15 \Rightarrow x=5$. Sustituyendo $x$ en la primera: $5+y=5 \Rightarrow y=0$. Si compras 2 lápices y 3 gomas por $7, y 1 lápiz y 2 gomas por $4, ¿cuánto cuesta cada uno?</p>
                            </li>
                            <li><strong>Introducción a Funciones:</strong>
                                <p>Comprender qué es una función (una relación donde cada entrada tiene una única salida) y cómo representarlas (gráficas, tablas, ecuaciones). Una función asigna a cada elemento de un conjunto (dominio) exactamente un elemento de otro conjunto (codominio). Las funciones son fundamentales para describir relaciones de causa y efecto y modelar fenómenos naturales y económicos. 📈</p>
                                <p class="mt-2"><em>Ejemplo:</em> La función $f(x) = 2x + 1$. Si $x=3$, $f(3) = 2(3) + 1 = 7$. La distancia recorrida por un coche a velocidad constante es una función del tiempo.</p>
                            </li>
                            <li><strong>Polinomios:</strong>
                                <p>Expresiones algebraicas que involucran sumas, restas y multiplicaciones de variables elevadas a potencias enteras no negativas (ej. $3x^2 - 2x + 5$). Aprenderás a sumar, restar y multiplicar polinomios, que son bloques de construcción esenciales en álgebra y cálculo. Se utilizan para modelar curvas y superficies.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Suma $(2x+3) + (x-1) = 3x+2$. Multiplica $(x+1)(x+2) = x^2 + 3x + 2$.</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Rama: Geometría Analítica y Trigonometría 📏 - Formas en Coordenadas</h4>
                            <p class="mb-2">Combinaremos la geometría con el álgebra para describir formas en un plano de coordenadas y estudiar las relaciones en triángulos, lo cual es esencial para muchas aplicaciones prácticas en física, ingeniería, navegación y astronomía. 🗺️</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Plano Cartesiano:</strong>
                                <p>Sistema de coordenadas que permite ubicar puntos y graficar líneas y figuras geométricas utilizando pares ordenados $(x, y)$. Cada punto en el plano se identifica de forma única con un par de números. Es la base para visualizar funciones y relaciones algebraicas.</p>
                                <p class="mt-2"><em>Ejemplo:</em> El punto $(2, 3)$ está 2 unidades a la derecha del origen y 3 unidades hacia arriba. 📍</p>
                            </li>
                            <li><strong>Distancia y Pendiente:</strong>
                                <p>Calcular la distancia entre dos puntos utilizando el teorema de Pitágoras (fórmula de la distancia, $\sqrt{(x_2-x_1)^2 + (y_2-y_1)^2}$) y la inclinación de una línea recta (pendiente, $m = \frac{y_2 - y_1}{x_2 - x_1}$). La pendiente indica qué tan empinada es una línea y su dirección. Estos conceptos son fundamentales en la construcción y la física.</p>
                                <p class="mt-2"><em>Ejemplo:</em> La distancia entre $(0,0)$ y $(3,4)$ es $\sqrt{(3-0)^2 + (4-0)^2} = \sqrt{9+16} = \sqrt{25} = 5$. La pendiente de la línea que pasa por $(1,2)$ y $(3,6)$ es $m = \frac{6-2}{3-1} = \frac{4}{2} = 2$.</p>
                            </li>
                            <li><strong>Introducción a la Trigonometría:</strong>
                                <p>Estudio de las relaciones entre los ángulos y los lados de los triángulos, especialmente los rectángulos. Funciones trigonométricas básicas (seno, coseno, tangente - SOH CAH TOA) y sus aplicaciones en la resolución de problemas de altura y distancia. Identidades trigonométricas fundamentales (ej. $\sin^2\theta + \cos^2\theta = 1$) que permiten simplificar expresiones y resolver ecuaciones. La trigonometría es esencial en la topografía, la navegación y el diseño de edificios. 📐</p>
                                <p class="mt-2"><em>Ejemplo:</em> Si un árbol proyecta una sombra de 10 metros y el ángulo de elevación del sol es $45^\circ$, la altura del árbol es $10 \times \tan(45^\circ) = 10$ metros. Para calcular la altura de una montaña sin escalarla.</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">3. Rama: Potencias, Raíces y Logaritmos ⚡ - Operaciones Avanzadas</h4>
                            <p class="mb-2">Exploraremos operaciones más avanzadas con números y sus inversas, que son cruciales para entender el crecimiento y decaimiento exponencial, así como en aplicaciones financieras, científicas y tecnológicas. Estas herramientas permiten manejar números muy grandes o muy pequeños.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Propiedades de Potencias:</strong>
                                <p>Reglas para operar con exponentes, que simplifican cálculos con números grandes o muy pequeños (ej. $x^a \cdot x^b = x^{a+b}$, $(x^a)^b = x^{ab}$, $x^0=1$, $x^{-a} = 1/x^a$). Las potencias son fundamentales en el crecimiento exponencial (población, inversiones) y en la notación científica.</p>
                                <p class="mt-2"><em>Ejemplo:</em> $2^3 \times 2^2 = 2^{3+2} = 2^5 = 32$. $(2^3)^2 = 2^{3 \times 2} = 2^6 = 64$. Si una población de bacterias se duplica cada hora, su crecimiento es exponencial.</p>
                            </li>
                            <li><strong>Raíces Cuadradas y Cúbicas:</strong>
                                <p>Las operaciones inversas de las potencias. La raíz cuadrada de un número $x$ es un número $y$ tal que $y^2 = x$ (ej. $\sqrt{25} = 5$). La raíz cúbica es similar, pero para la tercera potencia (ej. $\sqrt[3]{8} = 2$). Se utilizan para encontrar dimensiones (lados de cuadrados a partir de áreas) o para resolver ecuaciones que involucran potencias. Los números irracionales (ej. $\sqrt{2}$) también se introducen aquí.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Si un cuadrado tiene un área de 49 cm$^2$, su lado mide $\sqrt{49} = 7$ cm. 🌳</p>
                            </li>
                            <li><strong>Concepto de Logaritmos:</strong>
                                <p>Entender qué son los logaritmos (el exponente al que se debe elevar una base para obtener un número dado) y sus propiedades básicas (ej. $\log_b(xy) = \log_b(x) + \log_b(y)$, $\log_b(x^n) = n \log_b(x)$, $\log_b(x/y) = \log_b(x) - \log_b(y)$). Son fundamentales para resolver ecuaciones exponenciales, analizar escalas logarítmicas (Richter para terremotos, pH para acidez), y en campos como la acústica y la informática. 🧪</p>
                                <p class="mt-2"><em>Ejemplo:</em> $\log_{10}(100) = 2$ porque $10^2 = 100$. Si el sonido de un concierto aumenta 10 veces, el nivel de decibelios aumenta en 10 unidades (escala logarítmica).</p>
                            </li>
                            </ul>
                        </div>
                        <p class="mt-4">Este nivel te preparará para desafíos matemáticos más avanzados y te dará herramientas para modelar situaciones del mundo real con mayor precisión y profundidad. ¡La práctica constante es tu mejor aliada! 💪</p>
                    `,
                    questions: [
                        { id: 'm_i_q1', text: 'Resuelve la ecuación: $5x - 12 = 3x + 4$.', type: 'number', answer: 8 },
                        { id: 'm_i_q2', text: 'Si $f(x) = x^2 + 3$, ¿cuánto es $f(2)$?', type: 'number', answer: 7 },
                        { id: 'm_i_q3', text: 'Calcula el valor de $\cos(60^\circ)$.', type: 'number', answer: 0.5 },
                        { id: 'm_i_q4', text: 'Simplifica la expresión: $(x^4 \cdot x^2)^3$.', type: 'text', answer: 'x^18' },
                        { id: 'm_i_q5', text: 'Resuelve para $x$: $\log_3(x) = 2$.', type: 'number', answer: 9 },
                        { id: 'm_i_q6', text: 'Encuentra el valor de $y$ en el sistema: $x+y=7$, $x-y=3$.', type: 'number', answer: 2 },
                        { id: 'm_i_q7', text: '¿Cuál es la pendiente de la línea que pasa por los puntos $(1,1)$ y $(3,5)$?', type: 'number', answer: 2 },
                        { id: 'm_i_q8', text: 'Calcula $\sqrt{144}$.', type: 'number', answer: 12 },
                        { id: 'm_i_q9', text: 'Expresa $10^4$ como un número.', type: 'number', answer: 10000 },
                        { id: 'm_i_q10', text: 'Si $g(x) = 3x - 5$, ¿cuánto es $g(4)$?', type: 'number', answer: 7 },
                        { id: 'm_i_q11', text: 'Resuelve la ecuación: $2(x+3) = 10$.', type: 'number', answer: 2 },
                        { id: 'm_i_q12', text: '¿Cuál es la distancia entre los puntos $(0,0)$ y $(5,12)$?', type: 'number', answer: 13 },
                        { id: 'm_i_q13', text: 'Calcula $\tan(45^\circ)$.', type: 'number', answer: 1 },
                        { id: 'm_i_q14', text: 'Simplifica $ (a^5 / a^2) $.', type: 'text', answer: 'a^3' },
                        { id: 'm_i_q15', text: 'Resuelve para $x$: $\log_5(x) = 1$.', type: 'number', answer: 5 },
                        { id: 'm_i_q16', text: 'Suma los polinomios: $(2x^2 + 3x) + (x^2 - x)$.', type: 'text', answer: '3x^2 + 2x' },
                        { id: 'm_i_q17', text: 'Si un ángulo de un triángulo rectángulo es $30^\circ$, ¿cuál es el otro ángulo agudo?', type: 'number', answer: 60 },
                        { id: 'm_i_q18', text: 'Calcula $\sqrt[3]{27}$.', type: 'number', answer: 3 },
                        { id: 'm_i_q19', text: 'Expresa $0.001$ como una potencia de 10.', type: 'text', answer: '10^-3' },
                        { id: 'm_i_q20', text: 'Multiplica los polinomios: $(x+2)(x-3)$.', type: 'text', answer: 'x^2 - x - 6' },
                    ]
                },
                avanzado: {
                    theory: `
                        <p class="mb-6">En el nivel Avanzado de Matemáticas 🧠, te sumergirás en el cálculo para analizar el cambio y la acumulación, y explorarás el álgebra lineal y las ecuaciones diferenciales, que son la base de la ciencia y la ingeniería moderna. Este nivel te proporcionará las herramientas más potentes para la investigación y el desarrollo en diversas disciplinas.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Rama: Cálculo Diferencial 🚀 - El Estudio del Cambio</h4>
                            <p class="mb-2">El cálculo diferencial estudia cómo cambian las funciones y la tasa de cambio instantánea, fundamental para entender el movimiento, la optimización y el análisis de sistemas dinámicos. Es la base para modelar fenómenos continuos y predecir su comportamiento futuro.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Límites:</strong>
                                <p>Comprender el comportamiento de una función a medida que la variable se acerca a un punto específico. Es el concepto fundamental sobre el que se construyen las derivadas e integrales. Un límite describe el valor al que una función "se acerca" a medida que la entrada se acerca a un cierto valor. $\lim_{x \to a} f(x)$.</p>
                                <p class="mt-2"><em>Ejemplo:</em> $\lim_{x \to 2} (x^2 + 1) = 2^2 + 1 = 5$. Esto significa que a medida que $x$ se acerca a 2, la función $x^2+1$ se acerca a 5.</p>
                            </li>
                            <li><strong>Derivadas:</strong>
                                <p>Calcular la tasa de cambio instantánea de una función. Representa la pendiente de la recta tangente a la curva en un punto dado. Se utiliza para encontrar velocidades, aceleraciones, y en problemas de optimización (máximos y mínimos), donde se busca el valor óptimo de una cantidad. La derivada es una herramienta poderosa para analizar la "sensibilidad" de una función a los cambios en su entrada. $\frac{dy}{dx}$ o $f'(x)$.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Si $f(x) = x^2$, su derivada es $f'(x) = 2x$. La derivada de la posición con respecto al tiempo es la velocidad. 🏎️ Si queremos encontrar la velocidad instantánea de un coche en un momento dado, usamos la derivada de su función de posición.</p>
                            </li>
                            <li><strong>Reglas de Derivación y Aplicaciones:</strong>
                                <p>Técnicas para derivar funciones complejas (regla de la cadena, del producto, del cociente). La regla de la cadena se usa para funciones compuestas, la del producto para productos de funciones y la del cociente para divisiones. Aplicaciones en optimización (encontrar el valor máximo o mínimo de una función), análisis de gráficos de funciones (puntos críticos, concavidad, puntos de inflexión) y tasas relacionadas (cómo cambian varias cantidades en relación entre sí). Estas reglas son fundamentales para resolver problemas de ingeniería, física y economía.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Para maximizar el área de un rectángulo con perímetro fijo, se usa la derivada para encontrar las dimensiones óptimas. 📈 Si el costo de producción de un artículo es una función de la cantidad producida, la derivada del costo nos da el costo marginal.</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Rama: Cálculo Integral 📊 - El Estudio de la Acumulación</h4>
                            <p class="mb-2">El cálculo integral se enfoca en la acumulación de cantidades y el área bajo las curvas. Es la operación inversa de la diferenciación y permite resolver problemas de volumen, trabajo, y otras acumulaciones. Es crucial para modelar procesos que involucran sumas continuas o totales.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Integrales Indefinidas (Antiderivadas):</strong>
                                <p>Encontrar la función original a partir de su derivada. Se representa con el símbolo $\int f(x) dx$. El resultado es una familia de funciones que difieren por una constante de integración ($+C$), ya que la derivada de una constante es cero. Las antiderivadas son el primer paso para resolver problemas de acumulación.</p>
                                <p class="mt-2"><em>Ejemplo:</em> La antiderivada de $2x$ es $x^2 + C$, donde $C$ es una constante. Si conocemos la velocidad de un objeto, podemos encontrar su posición integrando la velocidad.</p>
                            </li>
                            <li><strong>Integrales Definidas y Aplicaciones:</strong>
                                <p>Calcular el área exacta bajo una curva entre dos puntos específicos. Se utiliza para calcular volúmenes de sólidos de revolución, longitudes de arco, trabajo realizado por una fuerza variable, y probabilidades en estadística. La integral definida da un valor numérico y representa una acumulación total. $\int_{a}^{b} f(x) dx$.</p>
                                <p class="mt-2"><em>Ejemplo:</em> El área bajo la curva $y=x$ desde $x=0$ hasta $x=2$ es $\int_{0}^{2} x dx = [x^2/2]_0^2 = 2^2/2 - 0^2/2 = 2$. 📏 Si queremos calcular la cantidad total de agua que fluye en un río durante un período de tiempo, usamos una integral definida.</p>
                            </li>
                            <li><strong>Teorema Fundamental del Cálculo y Técnicas:</strong>
                                <p>El Teorema Fundamental del Cálculo es la conexión crucial entre la diferenciación y la integración, que simplifica enormemente el cálculo de integrales definidas. Establece que la integral definida de una función puede calcularse evaluando la antiderivada en los límites de integración. Técnicas de integración como sustitución (cambio de variable), integración por partes (para productos de funciones), fracciones parciales (para funciones racionales) y sustituciones trigonométricas son herramientas avanzadas para resolver integrales complejas.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Para $\int x \cos(x) dx$, se usa integración por partes. Estas técnicas son esenciales para resolver problemas en física, ingeniería y economía.</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">3. Rama: Ecuaciones Diferenciales y Álgebra Lineal 🧩 - Modelando Sistemas</h4>
                            <p class="mb-2">Introducción a campos más especializados y aplicados de las matemáticas, fundamentales en ciencias e ingeniería para modelar sistemas complejos, analizar datos y resolver problemas de alta dimensión. Estas áreas son la columna vertebral de la investigación moderna y la innovación tecnológica.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Ecuaciones Diferenciales:</strong>
                                <p>Ecuaciones que involucran funciones y sus derivadas. Se utilizan para modelar fenómenos de cambio en física (movimiento de péndulos, flujo de calor), biología (crecimiento poblacional, propagación de enfermedades), economía (modelos de mercado, crecimiento económico) y química (reacciones). Pueden ser ordinarias (ODE, con una sola variable independiente) o parciales (PDE, con múltiples variables independientes). Son la herramienta principal para describir cómo cambian las cosas con el tiempo o el espacio. 🧪</p>
                                <p class="mt-2"><em>Ejemplo:</em> La ecuación $y' = ky$ modela el crecimiento o decaimiento exponencial (ej. crecimiento de bacterias, desintegración radiactiva). Las ecuaciones diferenciales son el lenguaje de la dinámica de sistemas.</p>
                            </li>
                            <li><strong>Álgebra Lineal:</strong>
                                <p>Estudio de vectores, espacios vectoriales, transformaciones lineales y matrices. Esencial para gráficos por computadora (transformaciones 3D), ciencia de datos (análisis de grandes conjuntos de datos), aprendizaje automático (algoritmos de IA), optimización y resolución de sistemas de ecuaciones complejos. Permite manipular y analizar grandes conjuntos de datos de manera eficiente, siendo la base de muchos algoritmos modernos. 🤖</p>
                                <p class="mt-2"><em>Ejemplo:</em> Multiplicación de matrices para transformaciones gráficas (rotación, escalado) o para resolver sistemas de ecuaciones lineales con muchas variables. Los algoritmos de búsqueda de Google utilizan principios de álgebra lineal.</p>
                            </li>
                            <li><strong>Series y Sucesiones:</strong>
                                <p>Comportamiento de secuencias de números (listas ordenadas de números) y sumas infinitas (series). Incluye series de Taylor (representar funciones como polinomios infinitos) y Fourier (descomponer funciones periódicas en sumas de senos y cosenos). Estas herramientas permiten aproximar funciones complejas con polinomios o descomponer señales en componentes más simples, lo que es crucial en procesamiento de señales, física y análisis numérico. 🎶</p>
                                <p class="mt-2"><em>Ejemplo:</em> La serie de Taylor para $e^x = 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + \dots$ (permite calcular $e^x$ con precisión). Las series de Fourier se usan para analizar ondas de sonido o señales eléctricas.</p>
                            </li>
                            </ul>
                        </div>
                        <p class="mt-4">Este nivel te equipará con herramientas matemáticas poderosas para el análisis y la resolución de problemas en campos científicos y tecnológicos avanzados, preparándote para la investigación y la innovación. ¡El conocimiento es ilimitado!💡</p>
                    `,
                    questions: [
                        { id: 'm_a_q1', text: 'Calcula la derivada de $f(x) = 3x^2 - 5x + 7$.', type: 'text', answer: '6x - 5' },
                        { id: 'm_a_q2', text: 'Evalúa la integral $\int (4x^3) dx$.', type: 'text', answer: 'x^4 + C' },
                        { id: 'm_a_q3', text: '¿Qué tipo de ecuación es $y\'\' + y = 0$?', type: 'text', answer: 'Ecuación diferencial ordinaria' },
                        { id: 'm_a_q4', text: 'Si $A = \\begin{pmatrix} 1 & 0 \\\\ 0 & 1 \\end{pmatrix}$, ¿cuál es la matriz identidad?', type: 'text', answer: 'Sí' },
                        { id: 'm_a_q5', text: '¿Qué es una serie de Taylor?', type: 'text', answer: 'Una representación de una función como una suma infinita de términos calculados a partir de los valores de las derivadas de la función en un solo punto.' },
                        { id: 'm_a_q6', text: 'Calcula $\\lim_{x \\to 0} \\frac{\\sin(x)}{x}$.', type: 'number', answer: 1 },
                        { id: 'm_a_q7', text: 'Si $f(x) = e^{2x}$, ¿cuál es $f\'(x)$?', type: 'text', answer: '2e^2x' },
                        { id: 'm_a_q8', text: 'Evalúa $\\int_{0}^{1} x^2 dx$.', type: 'number', answer: 0.3333 }, // 1/3
                        { id: 'm_a_q9', text: '¿Qué es un vector en álgebra lineal?', type: 'text', answer: 'Un objeto matemático que tiene magnitud y dirección.' },
                        { id: 'm_a_q10', text: 'Resuelve la ecuación diferencial $y\' = 5y$.', type: 'text', answer: 'y = Ce^(5x)' },
                        { id: 'm_a_q11', text: 'Encuentra los puntos críticos de $f(x) = x^3 - 6x^2 + 9x$.', type: 'text', answer: 'x=1, x=3' },
                        { id: 'm_a_q12', text: 'Calcula el producto cruz de los vectores $\\vec{A} = (1,0,0)$ y $\\vec{B} = (0,1,0)$.', type: 'text', answer: '(0,0,1)' },
                        { id: 'm_a_q13', text: '¿Cuál es el teorema fundamental del cálculo?', type: 'text', answer: 'Relaciona la diferenciación y la integración, mostrando que son operaciones inversas.' },
                        { id: 'm_a_q14', text: 'Si una función es continua en un intervalo cerrado $[a,b]$ y diferenciable en $(a,b)$, ¿qué teorema se aplica para garantizar la existencia de un punto $c$ donde la derivada es igual a la pendiente de la secante?', type: 'text', answer: 'Teorema del Valor Medio' },
                        { id: 'm_a_q15', text: 'Evalúa $\\int \\frac{1}{x} dx$.', type: 'text', answer: '\\ln|x| + C' },
                        { id: 'm_a_q16', text: '¿Qué es una transformación lineal en álgebra lineal?', type: 'text', answer: 'Una función entre dos espacios vectoriales que preserva las operaciones de suma de vectores y multiplicación por escalar.' },
                        { id: 'm_a_q17', text: 'Resuelve la ecuación diferencial $y\'\' - 4y = 0$.', type: 'text', answer: 'y = C1e^(2x) + C2e^(-2x)' },
                        { id: 'm_a_q18', text: 'Calcula la derivada de $f(x) = \\ln(x)$.', type: 'text', answer: '1/x' },
                        { id: 'm_a_q19', text: '¿Qué es una serie de Fourier?', type: 'text', answer: 'Una forma de representar funciones periódicas como una suma de senos y cosenos.' },
                        { id: 'm_a_q20', text: 'Encuentra el valor de $\\int_{0}^{\\pi} \\sin(x) dx$.', type: 'number', answer: 2 },
                    ]
                },
            },
            comunicacion: {
                basico: {
                    theory: `
                        <p class="mb-6">En el nivel Básico de Comunicación 🗣️✍️, sentarás las bases para una interacción efectiva, aprendiendo sobre los componentes esenciales del proceso comunicativo y los modos principales de expresión. Este nivel es crucial para cualquier tipo de interacción social y profesional.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Fundamentos de la Comunicación 🤔 - El Intercambio Básico</h4>
                            <p class="mb-2">La comunicación es el proceso fundamental de intercambiar información, ideas, sentimientos y significados entre individuos o grupos. Es la base de toda interacción humana y vital para la vida social, permitiéndonos construir relaciones, coordinar acciones y resolver conflictos. Sin comunicación efectiva, la sociedad no podría funcionar.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Importancia:</strong> Nos permite coordinar acciones, construir relaciones, expresar emociones, resolver conflictos y aprender. Sin comunicación, la sociedad no podría funcionar. Es la base de la colaboración y el entendimiento mutuo.</li>
                            <li><strong>Elementos Clave del Proceso:</strong> Para que la comunicación sea efectiva, varios elementos deben interactuar de manera fluida:
                                <ul class="list-circle list-inside ml-8">
                                <li><strong>Emisor:</strong> Quien inicia el mensaje y lo codifica (convierte sus ideas en un formato comprensible). 🗣️</li>
                                <li><strong>Mensaje:</strong> La información codificada que el emisor desea transmitir. Debe ser claro y adaptado al receptor. 📝</li>
                                <li><strong>Canal:</strong> El medio físico o digital por el que viaja el mensaje (aire, papel, internet, teléfono, etc.). La elección del canal puede influir en la efectividad del mensaje. 🌐</li>
                                <li><strong>Receptor:</strong> Quien recibe el mensaje y lo decodifica (interpreta el mensaje para entenderlo). 👂</li>
                                <li><strong>Código:</strong> El sistema de signos compartido por el emisor y el receptor para que el mensaje sea comprensible (idioma, gestos, símbolos, etc.). Si el código no es compartido, la comunicación falla. 🔡</li>
                                <li><strong>Contexto:</strong> Las circunstancias, el entorno o la situación en la que se produce la comunicación. Incluye el lugar, el tiempo, las relaciones entre los participantes, etc. El contexto puede alterar la interpretación del mensaje. 🌍</li>
                                <li><strong>Retroalimentación (Feedback):</strong> La respuesta del receptor al mensaje del emisor. Es crucial para que el emisor sepa si su mensaje fue comprendido y si necesita ajustarlo. Puede ser verbal o no verbal. 🔄</li>
                                <li><strong>Ruido:</strong> Cualquier interferencia que distorsiona o dificulta la transmisión o recepción del mensaje. Puede ser físico (sonido), semántico (malentendido de palabras) o psicológico (prejuicios). 🚧</li>
                                </ul>
                                <p class="mt-2"><em>Ejemplo:</em> Cuando envías un mensaje de texto a un amigo, tú eres el emisor, el texto es el mensaje, el teléfono es el canal, tu amigo es el receptor, el español es el código y la situación en la que se encuentran es el contexto. Si hay mala señal, eso sería ruido. 📱</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Modos de Comunicación: Verbal y No Verbal 💬 gestures - Más allá de las Palabras</h4>
                            <p class="mb-2">No solo hablamos con palabras; también comunicamos con nuestro cuerpo y otras señales. Entender ambos modos es crucial para la claridad y la efectividad, ya que a menudo se complementan o incluso se contradicen.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Comunicación Verbal:</strong>
                                <p>Se refiere al uso de palabras, ya sea habladas (oral) o escritas. Es el modo más directo de transmitir información. La elección de vocabulario, la gramática y la claridad son fundamentales para que el mensaje sea preciso y efectivo.</p>
                                <p class="mt-2"><em>Ejemplo Oral:</em> Una conversación cara a cara, una llamada telefónica, un discurso público, una presentación. 📞</p>
                                <p class="mt-2"><em>Ejemplo Escrito:</em> Un correo electrónico, una carta, un libro, un mensaje de texto, un informe. 📧</p>
                            </li>
                            <li><strong>Comunicación No Verbal:</strong>
                                <p>Incluye todos los mensajes que transmitimos sin usar palabras. A menudo, esto dice más que las palabras y puede reforzar o contradecir el mensaje verbal. Es una parte significativa de cómo interpretamos las interacciones.</p>
                                <ul class="list-circle list-inside ml-8">
                                <li><strong>Gestos:</strong> Movimientos de manos y cuerpo que complementan o reemplazan las palabras (ej. un pulgar hacia arriba 👍, señalar, saludar).</li>
                                <li><strong>Expresiones Faciales:</strong> Los movimientos de los músculos faciales que transmiten emociones (ej. alegría 😄, tristeza 😢, sorpresa 😮, enojo 😠).</li>
                                <li><strong>Postura:</strong> Cómo se posiciona el cuerpo, que puede indicar confianza, sumisión, apertura o cerrazón (ej. hombros caídos vs. erguidos, brazos cruzados). 🧍‍♀️</li>
                                <li><strong>Contacto Visual:</strong> Mantener o evitar la mirada, que puede indicar interés, honestidad, respeto o incomodidad. 👀</li>
                                <li><strong>Tono de Voz (Paralenguaje):</strong> Volumen, velocidad, ritmo, pausas y entonación. No es lo que se dice, sino cómo se dice. Puede cambiar completamente el significado de una frase. 🔊</li>
                                <li><strong>Proxémica:</strong> El uso del espacio personal y la distancia entre las personas al comunicarse. Varía culturalmente. 📏</li>
                                <li><strong>Apariencia Personal:</strong> La vestimenta, el peinado, los accesorios. También comunican mensajes sobre la persona. 👗👔</li>
                                </ul>
                                <p class="mt-2"><em>Ejemplo:</em> Decir "Estoy bien" con el ceño fruncido y los brazos cruzados envía un mensaje no verbal que contradice el verbal, indicando que no estás realmente bien. 😠</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">3. Habilidades de Recepción: Escucha Activa 👂 - El Arte de Entender</h4>
                            <p class="mb-2">Comunicarse bien no es solo hablar, es también saber escuchar de manera consciente y empática para comprender verdaderamente al interlocutor, no solo sus palabras, sino también sus sentimientos e intenciones. Es una habilidad fundamental para construir relaciones sólidas y resolver problemas.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Prestar Atención Plena:</strong>
                                <p>Implica concentrarse completamente en el hablante, evitando distracciones internas y externas. Mirar a la persona, evitar interrupciones, y mostrar interés genuino a través de señales no verbales (asentir con la cabeza, contacto visual apropiado, expresión facial atenta). El objetivo es absorber el mensaje sin interrupciones ni juicios prematuros.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Apagar tu teléfono mientras alguien te habla, o dejar de lado otras tareas para dedicarle toda tu atención. 📵</p>
                            </li>
                            <li><strong>Comprender y Clarificar:</strong>
                                <p>Intentar entender lo que la otra persona está diciendo y sintiendo, incluyendo el mensaje explícito e implícito. Hacer preguntas abiertas para clarificar puntos confusos o para obtener más detalles. Parafrasear lo que has escuchado con tus propias palabras para confirmar tu comprensión y permitir que el hablante corrija cualquier malentendido. Esto demuestra que estás procesando activamente la información. 🤔</p>
                                <p class="mt-2"><em>Ejemplo:</em> "Entonces, si entiendo bien, ¿estás frustrado porque el proyecto se retrasó?" o "¿Podrías darme un ejemplo de lo que quieres decir con 'falta de apoyo'?"</p>
                            </li>
                            <li><strong>Empatía y No Juicio:</strong>
                                <p>Intentar ponerse en el lugar del otro para comprender su perspectiva y sus emociones, incluso si no estás de acuerdo con ellas. Suspender el juicio y evitar interrumpir con soluciones o consejos prematuros. El objetivo es crear un espacio seguro donde el hablante se sienta escuchado y comprendido. ❤️</p>
                                <p class="mt-2"><em>Ejemplo:</em> En lugar de decir "No deberías sentirte así", decir "Entiendo que te sientas así dada la situación."</p>
                            </li>
                            </ul>
                        </div>
                        <p class="mt-4">Dominar estos conceptos básicos te ayudará a mejorar tus interacciones diarias y a sentar las bases para una comunicación más efectiva, tanto en el ámbito personal como profesional. ¡La práctica constante es la clave para convertir estas habilidades en hábitos! 🌟</p>
                    `,
                    questions: [
                        { id: 'c_b_q1', text: 'Menciona un elemento clave del proceso de comunicación.', type: 'text', answer: 'Emisor' },
                        { id: 'c_b_q2', text: '¿Es el lenguaje corporal un tipo de comunicación verbal o no verbal?', type: 'text', answer: 'No verbal' },
                        { id: 'c_b_q3', text: '¿Qué significa "parafrasear" en la escucha activa?', type: 'text', answer: 'Repetir con tus propias palabras lo que la otra persona ha dicho para confirmar que la has entendido.' },
                        { id: 'c_b_q4', text: 'Da un ejemplo de comunicación verbal escrita.', type: 'text', answer: 'Correo electrónico' },
                        { id: 'c_b_q5', text: '¿Por qué es importante la retroalimentación en la comunicación?', type: 'text', answer: 'Para saber si el mensaje fue comprendido y ajustar la comunicación si es necesario.' },
                        { id: 'c_b_q6', text: '¿Cuál es el propósito principal de la comunicación?', type: 'text', answer: 'Intercambiar información, ideas, sentimientos.' },
                        { id: 'c_b_q7', text: 'Si alguien te sonríe, ¿qué tipo de comunicación es?', type: 'text', answer: 'No verbal' },
                        { id: 'c_b_q8', text: 'Menciona un ejemplo de canal de comunicación.', type: 'text', answer: 'Teléfono' },
                        { id: 'c_b_q9', text: '¿Qué es el código en el proceso de comunicación?', type: 'text', answer: 'El sistema de signos compartido, como el idioma.' },
                        { id: 'c_b_q10', text: '¿Qué es el contexto en la comunicación?', type: 'text', answer: 'Las circunstancias que rodean la comunicación.' },
                        { id: 'c_b_q11', text: '¿Qué habilidad de recepción es fundamental para entender a los demás?', type: 'text', answer: 'Escucha activa' },
                        { id: 'c_b_q12', text: 'Si un mensaje no se entiende, ¿qué elemento de la comunicación falló?', type: 'text', answer: 'Mensaje o código' },
                        { id: 'c_b_q13', text: '¿Qué significa mostrar interés genuino al escuchar?', type: 'text', answer: 'Asentir, mantener contacto visual.' },
                        { id: 'c_b_q14', text: 'Da un ejemplo de un saludo informal.', type: 'text', answer: 'Hola' },
                        { id: 'c_b_q15', text: '¿Por qué es importante no interrumpir al escuchar?', type: 'text', answer: 'Para permitir que la otra persona se exprese completamente.' },
                        { id: 'c_b_q16', text: '¿Qué tipo de comunicación es un discurso?', type: 'text', answer: 'Verbal oral' },
                        { id: 'c_b_q17', text: '¿Cuál es la diferencia entre emisor y receptor?', type: 'text', answer: 'El emisor envía el mensaje, el receptor lo recibe.' },
                        { id: 'c_b_q18', text: '¿Qué función cumple el tono de voz en la comunicación no verbal?', type: 'text', answer: 'Transmite emociones o intenciones.' },
                        { id: 'c_b_q19', text: 'Si alguien cruza los brazos, ¿qué podría comunicar no verbalmente?', type: 'text', answer: 'Defensa o incomodidad.' },
                        { id: 'c_b_q20', text: '¿Qué es la comunicación bidireccional?', type: 'text', answer: 'Cuando el emisor y el receptor intercambian roles.' },
                    ]
                },
                intermedio: {
                    theory: `
                        <p class="mb-6">En el nivel Intermedio de Comunicación 🤝, explorarás cómo identificar y superar las barreras que dificultan el mensaje, aprenderás a expresar tus ideas con asertividad y adaptarás tu estilo comunicativo a diversos contextos. Este nivel te permitirá comunicarte con mayor confianza y eficacia en situaciones más variadas.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Identificación y Superación de Barreras de la Comunicación 🚧 - Obstáculos en el Mensaje</h4>
                            <p class="mb-2">A menudo, algo interfiere con la claridad de nuestros mensajes. Identificar estas barreras es el primer paso para superarlas y asegurar que el mensaje llegue como se desea, mejorando la comprensión mutua y evitando conflictos. Reconocer estos obstáculos es clave para una comunicación efectiva. 🗣️➡️👂</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Ruido Físico:</strong> Sonidos externos, distracciones visuales o cualquier elemento ambiental que impida escuchar o concentrarse en el mensaje. Es lo que físicamente nos impide percibir el mensaje.
                                <p class="mt-2"><em>Ejemplo:</em> Música alta en una cafetería mientras intentas hablar, el sonido de una obra en construcción, una mala conexión de internet durante una videollamada. 🎶</p>
                            </li>
                            <li><strong>Barreras Semánticas:</strong> Diferencias en el significado de las palabras, jergas técnicas, acentos, regionalismos o interpretaciones culturales que llevan a malentendidos. Ocurren cuando el emisor y el receptor no comparten el mismo significado para un término o frase.
                                <p class="mt-2"><em>Ejemplo:</em> Usar "coche" en España y "carro" en Latinoamérica puede causar confusión si no se conoce la variante. 🚗 Otro ejemplo es el uso de tecnicismos en una conversación con alguien que no es experto en el tema.</p>
                            </li>
                            <li><strong>Barreras Psicológicas:</strong> Prejuicios, emociones (ira, miedo, ansiedad), estados de ánimo, estereotipos, percepciones individuales, falta de atención o desinterés que afectan la interpretación del mensaje por parte del receptor o la forma en que el emisor lo codifica. Estas barreras residen en la mente de los comunicadores.
                                <p class="mt-2"><em>Ejemplo:</em> No escuchar a alguien porque ya tienes una opinión negativa sobre esa persona, o estar tan enojado que no puedes procesar lo que te dicen. 😠</p>
                            </li>
                            <li><strong>Barreras Fisiológicas:</strong> Problemas de audición, visión, dicción, afonía o cualquier condición física que dificulte la recepción o emisión clara del mensaje. Son limitaciones biológicas que impiden la comunicación.
                                <p class="mt-2"><em>Ejemplo:</em> Una persona con gripe y voz ronca intentando dar un discurso, o alguien con problemas de visión intentando leer un texto pequeño. 🤒</p>
                            </li>
                            <li><strong>Barreras Administrativas:</strong> Relacionadas con la estructura de la organización, la falta de canales de comunicación adecuados, la sobrecarga de información o la falta de planificación.
                                <p class="mt-2"><em>Ejemplo:</em> Un exceso de correos electrónicos que hace que los mensajes importantes se pierdan, o una jerarquía muy rígida que impide la comunicación fluida entre departamentos.</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Comunicación Asertiva ✅ - Exprésate con Respecto</h4>
                            <p class="mb-2">Aprender a expresar tus opiniones, necesidades y deseos de forma clara, directa y respetuosa, sin agredir ni ser pasivo, es una habilidad clave para las relaciones interpersonales sanas y el éxito profesional. Es el equilibrio entre la agresividad (imponer tus ideas) y la pasividad (no expresar tus ideas), promoviendo la autoestima y el respeto mutuo. ⚖️</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Decir "No" de Forma Constructiva:</strong> Establecer límites de manera educada pero firme, sin sentir culpa ni dañar la relación. Implica explicar brevemente la razón (sin excusas excesivas) y, si es posible, ofrecer una alternativa o una solución parcial.
                                <p class="mt-2"><em>Ejemplo:</em> "Agradezco la invitación, pero hoy no podré ir. Tengo otro compromiso. Quizás la próxima semana sí pueda."</p>
                            </li>
                            <li><strong>Expresar Sentimientos y Necesidades:</strong> Utilizar el "yo siento..." en lugar de culpabilizar ("tú me haces sentir..."), asumiendo la responsabilidad de las propias emociones y expresando lo que necesitas de manera clara y específica. Esto evita la confrontación y fomenta la comprensión.
                                <p class="mt-2"><em>Ejemplo:</em> "Me siento frustrado cuando no se cumplen los plazos, y necesito que me informes con anticipación si hay un retraso para poder ajustar mi planificación."</p>
                            </li>
                            <li><strong>Negociación Básica:</strong> Buscar soluciones donde ambas partes ganen (win-win), priorizando el respeto mutuo, la colaboración y la búsqueda de intereses comunes, en lugar de solo enfocarse en posiciones. Implica escuchar activamente, identificar puntos en común y proponer soluciones creativas.
                                <p class="mt-2"><em>Ejemplo:</em> En una discusión sobre un proyecto, proponer: "Entiendo tu punto sobre la necesidad de rapidez, y yo valoro la calidad. ¿Qué tal si combinamos mi idea A con tu idea B para lograr un mejor resultado que sea rápido y de buena calidad?" 🤝</p>
                            </li>
                            <li><strong>Recibir y Dar Retroalimentación (Feedback):</strong> Aprender a aceptar críticas constructivas sin ponerse a la defensiva y a dar feedback de manera que sea útil y no hiriente. El feedback asertivo se enfoca en el comportamiento, no en la persona.
                                <p class="mt-2"><em>Ejemplo:</em> Dar: "He notado que los informes llegan tarde a menudo, lo que afecta la planificación. ¿Podríamos buscar una forma de mejorar esto?" Recibir: "Gracias por tu feedback, lo tendré en cuenta para mejorar."</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">3. Adaptación Comunicativa en Diferentes Contextos 🌍 - El Camaleón Comunicativo</h4>
                            <p class="mb-2">La forma en que te comunicas cambia significativamente según la situación, la audiencia y el propósito. Saber adaptar tu estilo es una señal de alta competencia comunicativa y te permite ser más efectivo en cualquier interacción, desde una charla informal hasta una presentación profesional. 🎭</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Comunicación Formal vs. Informal:</strong>
                                <p>Adaptar tu lenguaje, tono, vocabulario y estructura a la audiencia y al propósito. La comunicación formal se usa en entornos profesionales, académicos o con personas de autoridad (ej. un jefe, un profesor); la informal, con amigos, familiares o en situaciones relajadas. La formalidad implica mayor precisión y respeto a las normas. </p>
                                <p class="mt-2"><em>Ejemplo:</em> En una entrevista de trabajo, usar "Estimado Sr. Pérez" y un lenguaje respetuoso y estructurado. Con un amigo, usar "¡Hola, Juan!" y un lenguaje coloquial y abreviado. 👔 casual</p>
                            </li>
                            <li><strong>Comunicación Digital y Netiqueta:</strong>
                                <p>Reglas de etiqueta (netiqueta) específicas para plataformas digitales como correos electrónicos 📧, mensajes de texto, redes sociales 📱 y videollamadas. Considerar la brevedad, claridad, profesionalismo y el tono adecuado, ya que la no verbalidad es limitada y los malentendidos son más comunes. La netiqueta busca mantener el respeto y la eficiencia en el entorno digital.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Evitar escribir todo en mayúsculas en un correo electrónico (se percibe como gritar), usar un lenguaje claro y conciso en mensajes de trabajo, y evitar el uso excesivo de emojis en contextos formales. 🚫CAPS</p>
                            </li>
                            <li><strong>Comunicación en Público (Básico):</strong>
                                <p>Primeros pasos para hablar frente a una audiencia. Incluye la importancia del contacto visual (para conectar con la audiencia), la postura (para transmitir confianza), la modulación de la voz (volumen, tono, ritmo para mantener el interés) y la organización de ideas (estructura clara y concisa) para un mensaje claro y conciso. Superar el miedo escénico es un objetivo inicial.</p>
                                <p class="mt-2"><em>Ejemplo:</em> Practicar un breve discurso frente al espejo antes de una presentación escolar, o preparar puntos clave para una reunión de equipo. 🎤</p>
                            </li>
                            <li><strong>Comunicación Escrita Efectiva:</strong>
                                <p>Más allá de la gramática y la ortografía, se trata de estructurar las ideas de forma lógica, usar un lenguaje preciso y conciso, y adaptar el mensaje al lector. Esto incluye la redacción de informes, correos electrónicos y documentos claros y persuasivos. 📝</p>
                                <p class="mt-2"><em>Ejemplo:</em> Escribir un correo electrónico con un asunto claro, un saludo apropiado, un cuerpo conciso y una llamada a la acción. </p>
                            </li>
                            </ul>
                        </div>
                        <p class="mt-4">Este nivel te proporcionará las herramientas para comunicarte con mayor confianza y eficacia en una variedad de situaciones, superando obstáculos y construyendo relaciones más sólidas, tanto en tu vida personal como profesional. ¡Sigue practicando y observando cómo se comunican los demás! 🗣️</p>
                    `,
                    questions: [
                        { id: 'c_i_q1', text: 'Menciona una barrera psicológica de la comunicación.', type: 'text', answer: 'Prejuicios' },
                        { id: 'c_i_q2', text: 'Reescribe la frase "Siempre llegas tarde" de forma asertiva.', type: 'text', answer: 'Me siento frustrado/a cuando llegas tarde.' },
                        { id: 'c_i_q3', text: '¿Qué es la netiqueta?', type: 'text', answer: 'Reglas de etiqueta para la comunicación digital.' },
                        { id: 'c_i_q4', text: 'Da un ejemplo de cómo adaptarías tu lenguaje de una situación formal a una informal.', type: 'text', answer: 'De "Estimado Sr. Pérez" a "Hola, Juan".' },
                        { id: 'c_i_q5', text: '¿Qué tipo de comunicación es un gesto con la mano?', type: 'text', answer: 'No verbal' },
                        { id: 'c_i_q6', text: '¿Qué tipo de barrera es un acento fuerte que dificulta la comprensión?', type: 'text', answer: 'Semántica' },
                        { id: 'c_i_q7', text: '¿Qué implica decir "no" de forma constructiva?', type: 'text', answer: 'Establecer límites de manera educada pero firme.' },
                        { id: 'c_i_q8', text: '¿Por qué es importante evitar las mayúsculas en correos electrónicos?', type: 'text', answer: 'Se percibe como gritar.' },
                        { id: 'c_i_q9', text: 'Menciona una técnica de negociación básica.', type: 'text', answer: 'Buscar soluciones donde ambas partes ganen.' },
                        { id: 'c_i_q10', text: '¿Qué se debe considerar al comunicarse en público por primera vez?', type: 'text', answer: 'Contacto visual, postura, modulación de voz.' },
                        { id: 'c_i_q11', text: '¿Cuál es la diferencia entre comunicación pasiva y asertiva?', type: 'text', answer: 'Pasiva evita expresar, asertiva expresa con respeto.' },
                        { id: 'c_i_q12', text: 'Da un ejemplo de barrera física en la comunicación.', type: 'text', answer: 'Música alta.' },
                        { id: 'c_i_q13', text: '¿Cómo puedes expresar tus sentimientos de manera asertiva?', type: 'text', answer: 'Usando "yo siento..."' },
                        { id: 'c_i_q14', text: '¿Qué es la comunicación formal?', type: 'text', answer: 'Lenguaje respetuoso en entornos profesionales.' },
                        { id: 'c_i_q15', text: '¿Qué significa "win-win" en negociación?', type: 'text', answer: 'Que ambas partes ganan.' },
                        { id: 'c_i_q16', text: '¿Qué tipo de barrera es un problema de audición?', type: 'text', answer: 'Fisiológica' },
                        { id: 'c_i_q17', text: '¿Por qué es importante la brevedad en la comunicación digital?', type: 'text', answer: 'Para ser claro y eficiente.' },
                        { id: 'c_i_q18', text: 'Menciona una ventaja de la comunicación asertiva.', type: 'text', answer: 'Mejora las relaciones interpersonales.' },
                        { id: 'c_i_q19', text: '¿Qué es la comunicación informal?', type: 'text', answer: 'Lenguaje coloquial con amigos y familiares.' },
                        { id: 'c_i_q20', text: '¿Qué es la modulación de voz en la comunicación oral?', type: 'text', answer: 'Variar el tono, volumen y velocidad al hablar.' },
                    ]
                },
                avanzado: {
                    theory: `
                        <p class="mb-6">En el nivel Avanzado de Comunicación 🚀, te enfocarás en la comunicación estratégica, la persuasión ética y el liderazgo a través de la palabra, preparándote para influir y gestionar en escenarios complejos y de alto nivel. Este nivel te convertirá en un comunicador maestro, capaz de navegar las complejidades de la interacción humana.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Comunicación Persuasiva y Retórica 🎯 - El Arte de Influir</h4>
                            <p class="mb-2">Aprender a influir en los demás de manera ética y efectiva para lograr objetivos, utilizando principios de la retórica clásica y moderna. No se trata de manipular, sino de construir argumentos sólidos, conectar con la audiencia y motivar a la acción. Es una habilidad esencial para el liderazgo, las ventas y cualquier rol que requiera influencia.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Principios de la Retórica (Aristóteles):</strong> La retórica es el arte de persuadir. Aristóteles identificó tres pilares fundamentales que todo comunicador debe dominar:
                                <ul class="list-circle list-inside ml-8">
                                <li><strong>Logos (Lógica):</strong> Uso de argumentos racionales, datos, hechos, estadísticas y razonamiento deductivo/inductivo para convencer a la audiencia. Apela a la razón. 📊</li>
                                <li><strong>Ethos (Credibilidad):</strong> Establecer tu autoridad, experiencia, conocimiento, carácter moral y buena voluntad para generar confianza en la audiencia. Apela al carácter del orador. 🏅</li>
                                <li><strong>Pathos (Emoción):</strong> Apelar a las emociones, valores y creencias de la audiencia para conectar a un nivel más profundo y moverlos a la acción. Apela a la emoción. 😢😊</li>
                                </ul>
                                <p class="mt-2"><em>Ejemplo:</em> En una negociación salarial, presentar datos de mercado (logos), resaltar tu experiencia y logros (ethos) y expresar tu entusiasmo por el puesto y tu deseo de contribuir (pathos).</p>
                            </li>
                            <li><strong>Técnicas de Negociación Avanzadas:</strong> Estrategias para alcanzar acuerdos complejos, resolver conflictos a gran escala y manejar objeciones de manera constructiva, buscando siempre el valor mutuo (ganar-ganar) y la creación de soluciones creativas. Incluye el análisis de intereses (no solo posiciones), el uso de alternativas (BATNA), y la construcción de relaciones.
                                <p class="mt-2"><em>Ejemplo:</em> En una fusión de empresas, identificar los intereses subyacentes de ambas partes para encontrar un acuerdo que maximice el valor para todos, en lugar de solo discutir los términos iniciales.</p>
                            </li>
                            <li><strong>Presentaciones de Alto Impacto:</strong> Diseñar y entregar discursos y presentaciones que no solo informen, sino que también cautiven, inspiren y motiven a la acción. Incluye storytelling (narrar historias para conectar emocionalmente), diseño de diapositivas efectivo (visuales claros y minimalistas), manejo del escenario (lenguaje corporal, movimiento) y vocalización (volumen, ritmo, pausas). 🎤✨</p>
                                <p class="mt-2"><em>Ejemplo:</em> Un discurso TED Talk, que combina datos con anécdotas personales para inspirar y persuadir a la audiencia sobre una idea innovadora.</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Comunicación Intercultural y Global 🌐 - Conectando Mundos</h4>
                            <p class="mb-2">Entender y adaptarse a las profundas diferencias culturales en la comunicación para evitar malentendidos, construir relaciones sólidas y operar eficazmente en un entorno globalizado. Es crucial en un mundo cada vez más conectado, donde las interacciones transculturales son la norma.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Dimensiones Culturales (Hofstede y Hall):</strong>
                                <p>Análisis de modelos que explican cómo las culturas varían en dimensiones clave. Por ejemplo, el modelo de Hofstede incluye individualismo vs. colectivismo, distancia de poder, evitación de la incertidumbre, masculinidad vs. feminidad, y orientación a largo/corto plazo. El modelo de Hall introduce la alta vs. baja contextualidad (qué tanto se depende del contexto no verbal y la historia compartida para entender un mensaje). Comprender estas dimensiones ayuda a predecir y adaptar el comportamiento comunicativo. </p>
                                <p class="mt-2"><em>Ejemplo:</em> En una cultura de alto contexto (ej. Japón), un "sí" puede significar "lo entiendo" o "lo consideraré", no necesariamente "estoy de acuerdo", debido a la importancia de la armonía y la comunicación implícita.</p>
                            </li>
                            <li><strong>Sensibilidad Cultural y Adaptación:</strong> Desarrollar la capacidad de observar, interpretar y respetar normas, valores y comportamientos comunicativos de otras culturas. Implica ser flexible, evitar el etnocentrismo (juzgar otras culturas con los propios valores) y practicar la empatía cultural. Se trata de ajustar tu propio estilo de comunicación para ser más efectivo y respetuoso en un contexto diferente.
                                <p class="mt-2"><em>Ejemplo:</em> Evitar el contacto visual directo en algunas culturas asiáticas como señal de respeto, mientras que en Occidente es señal de atención. Adaptar el humor o las referencias culturales para que sean apropiadas para la audiencia.</p>
                            </li>
                            <li><strong>Comunicación en Equipos Globales:</strong> Estrategias para gestionar la comunicación en equipos distribuidos geográficamente y culturalmente diversos. Esto incluye el uso efectivo de la tecnología, el establecimiento de normas claras, la gestión de diferencias horarias y la promoción de un ambiente inclusivo.</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">3. Liderazgo, Gestión y Comunicación de Crisis 🚨 - La Voz en la Tormenta</h4>
                            <p class="mb-2">La comunicación es una herramienta vital para el liderazgo efectivo y la gestión de situaciones difíciles, donde la claridad, la transparencia y la empatía son cruciales para mantener la confianza y minimizar el daño. Un buen líder es un comunicador excepcional, especialmente bajo presión. 🚢</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Comunicación de Liderazgo:</strong> Inspirar, motivar y guiar equipos a través de una comunicación clara, visionaria y empática, fomentando la cohesión, el compromiso y la resiliencia. Un líder comunica la visión, los valores, las expectativas y el propósito, y también escucha activamente a su equipo. La comunicación de un líder debe ser consistente y auténtica.
                                <p class="mt-2"><em>Ejemplo:</em> Steve Jobs presentando un nuevo iPhone, no solo informando sobre el producto, sino inspirando con su visión del futuro y conectando emocionalmente con la audiencia. 🍎</p>
                            </li>
                            <li><strong>Gestión de Crisis:</strong> Desarrollar planes y estrategias para comunicar de manera efectiva durante situaciones de emergencia (desastres naturales, escándalos corporativos, fallas de productos). Implica ser rápido, preciso, transparente y empático para mantener la confianza de los stakeholders (empleados, clientes, inversores, público) y minimizar el daño reputacional. La comunicación de crisis busca controlar la narrativa y ofrecer soluciones.
                                <p class="mt-2"><em>Ejemplo:</em> Una empresa emitiendo un comunicado claro, honesto y ofreciendo soluciones rápidas y concretas tras un error en un producto que afectó a los consumidores. 📢</p>
                            </li>
                            <li><strong>Comunicación de Cambio:</strong> Liderar a través del cambio organizacional, comunicando la necesidad del cambio, los beneficios, los desafíos y los próximos pasos de manera efectiva para reducir la resistencia y fomentar la aceptación y el compromiso de los empleados.</li>
                            <li><strong>Comunicación de Influencia y Redes:</strong> Desarrollar la capacidad de construir y mantener redes profesionales, influir en stakeholders clave y negociar a nivel estratégico para lograr objetivos a largo plazo.</li>
                            </ul>
                        </div>
                        <p class="mt-4">Este nivel te transformará en un comunicador maestro, capaz de liderar, persuadir y construir puentes en cualquier escenario, dominando el arte de la influencia y la conexión humana en un mundo complejo y dinámico. ¡Tu voz tiene un poder transformador! 🌟</p>
                    `,
                    questions: [
                        { id: 'c_a_q1', text: 'Menciona los tres principios de la retórica de Aristóteles.', type: 'text', answer: 'Logos, Ethos, Pathos' },
                        { id: 'c_a_q2', text: '¿Qué es una cultura de "alto contexto" en comunicación intercultural?', type: 'text', answer: 'Una cultura donde gran parte del significado del mensaje se deriva del contexto no verbal y la situación, no solo de las palabras explícitas.' },
                        { id: 'c_a_q3', text: '¿Por qué es crucial la transparencia en la comunicación de crisis?', type: 'text', answer: 'Para mantener la confianza de los stakeholders y evitar especulaciones o rumores.' },
                        { id: 'c_a_q4', text: 'Da un ejemplo de cómo un líder puede usar la comunicación para inspirar a su equipo.', type: 'text', answer: 'Compartiendo una visión clara y apasionada del futuro, y reconociendo el esfuerzo de cada miembro.' },
                        { id: 'c_a_q5', text: '¿Qué es el storytelling en presentaciones de alto impacto?', type: 'text', answer: 'El uso de narrativas o historias para conectar emocionalmente con la audiencia y hacer el mensaje más memorable.' },
                        { id: 'c_a_q6', text: '¿Cuál es el objetivo principal de la comunicación persuasiva?', type: 'text', answer: 'Influir en los demás de manera ética para lograr objetivos.' },
                        { id: 'c_a_q7', text: '¿Qué significa "Ethos" en retórica?', type: 'text', answer: 'Establecer credibilidad o autoridad.' },
                        { id: 'c_a_q8', text: 'Menciona una dimensión cultural según Hofstede.', type: 'text', answer: 'Individualismo vs. Colectivismo' },
                        { id: 'c_a_q9', text: '¿Qué se busca al manejar objeciones en una negociación avanzada?', type: 'text', answer: 'Soluciones constructivas y valor mutual.' },
                        { id: 'c_a_q10', text: '¿Por qué es importante la empatía en la comunicación de liderazgo?', type: 'text', answer: 'Para conectar con el equipo y fomentar el compromiso.' },
                        { id: 'c_a_q11', text: '¿Qué es una "cultura de bajo contexto"?', type: 'text', answer: 'Una cultura donde la comunicación es explícita y directa, con poco significado derivado del contexto no verbal.' },
                        { id: 'c_a_q12', text: 'Menciona una técnica para hacer presentaciones de alto impacto.', type: 'text', answer: 'Storytelling.' },
                        { id: 'c_a_q13', text: '¿Qué es el "Pathos" en retórica?', type: 'text', answer: 'Apelar a las emociones de la audiencia.' },
                        { id: 'c_a_q14', text: '¿Qué es la comunicación estratégica?', type: 'text', answer: 'Comunicación planificada para lograr objetivos específicos.' },
                        { id: 'c_a_q15', text: '¿Por qué es importante la adaptabilidad cultural en la comunicación global?', type: 'text', answer: 'Para evitar malentendidos y construir relaciones sólidas.' },
                        { id: 'c_a_q16', text: 'Menciona un paso clave en la gestión de crisis.', type: 'text', answer: 'Desarrollar planes y estrategias de comunicación.' },
                        { id: 'c_a_q17', text: '¿Qué significa "Logos" en retórica?', type: 'text', answer: 'Uso de argumentos lógicos, datos y hechos.' },
                        { id: 'c_a_q18', text: '¿Qué es el etnocentrismo en comunicación intercultural?', type: 'text', answer: 'Juzgar otras culturas basándose en los propios estándares culturales.' },
                        { id: 'c_a_q19', text: '¿Cómo ayuda la comunicación a fomentar la resiliencia en un equipo?', type: 'text', answer: 'Al mantener la cohesión y el compromiso.' },
                        { id: 'c_a_q20', text: '¿Qué es el "rapport" en negociación?', type: 'text', answer: 'La relación de confianza y comprensión mutua entre las partes.' },
                    ]
                },
            },
            ingles: {
                basico: {
                    theory: `
                        <p class="mb-6">En el nivel Básico de Inglés 🇬🇧🇺🇸, darás tus primeros pasos para construir una base sólida en el idioma, enfocándote en la comunicación esencial para el día a día. Este nivel te permitirá interactuar en situaciones básicas y comprender mensajes simples.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Fundamentos de Comunicación Oral y Escrita 👋 - Primeras Palabras</h4>
                            <p class="mb-2">Aprenderás las frases básicas para interactuar en inglés, tanto al hablar como al escribir mensajes sencillos. Es el punto de partida para cualquier conversación y para construir tu confianza en el idioma. 🗣️✍️</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Saludos y Despedidas:</strong>
                                <p>Frases comunes para iniciar y finalizar interacciones en diferentes momentos del día. "Hello" (hola), "Hi" (hola informal), "Good morning" (buenos días), "Good afternoon" (buenas tardes), "Good evening" (buenas noches al llegar), "Good night" (buenas noches al despedirse), "Goodbye" (adiós), "See you later" (hasta luego), "See you soon" (hasta pronto). Dominar estas frases te permitirá iniciar y cerrar conversaciones de forma natural.</p>
                                <p class="mt-2"><em>Ejemplo:</em> "Hello, how are you?" (Hola, ¿cómo estás?) 👋</p>
                            </li>
                            <li><strong>Presentación Personal:</strong>
                                <p>Cómo presentarte a ti mismo y a otros, incluyendo tu nombre, origen y ocupación. "My name is..." (Mi nombre es...), "I am from..." (Soy de...), "Nice to meet you" (Encantado/a de conocerte), "This is..." (Este/a es...). Estas estructuras son fundamentales para las primeras interacciones sociales.</p>
                                <p class="mt-2"><em>Ejemplo:</em> "My name is Ana. Nice to meet you!" (Mi nombre es Ana. ¡Encantado/a de conocerte!) 😊</p>
                            </li>
                            <li><strong>Preguntas Básicas:</strong>
                                <p>Formular y responder preguntas sencillas sobre información personal y preferencias. "¿What is your name?" (¿Cuál es tu nombre?), "¿Where are you from?" (¿De dónde eres?), "¿How old are you?" (¿Cuántos años tienes?), "¿How are you?" (¿Cómo estás?), "¿What is your favorite...?" (¿Cuál es tu... favorito/a?). Estas preguntas te permitirán obtener y dar información básica.</p>
                                <p class="mt-2"><em>Ejemplo:</em> "What is your favorite color?" (¿Cuál es tu color favorito?) 🌈</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Vocabulario Esencial y Comprensión Simple 📖 - Tus Primeras Palabras</h4>
                            <p class="mb-2">Conocerás palabras y frases comunes para describir tu entorno, personas, objetos y entender instrucciones básicas. Es la base para construir oraciones y comprender el mundo que te rodea en inglés.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Números:</strong>
                                <p>Aprender a contar del 1 al 100 y más allá. Esencial para precios, edades, cantidades, direcciones y números de teléfono. "One, two, three...", "Twenty-five", "One hundred", "One thousand". 🔢</p>
                                <p class="mt-2"><em>Ejemplo:</em> "I am twenty-five years old." (Tengo veinticinco años.)</p>
                            </li>
                            <li><strong>Colores:</strong>
                                <p>Nombres de colores básicos como "red" (rojo), "blue" (azul), "green" (verde), "yellow" (amarillo), "black" (negro), "white" (blanco), "orange" (naranja), "purple" (morado), "pink" (rosa), "brown" (marrón). 🎨</p>
                                <p class="mt-2"><em>Ejemplo:</em> "The car is red." (El coche es rojo.) 🚗🔴</p>
                            </li>
                            <li><strong>Días de la Semana y Meses:</strong>
                                <p>Aprender los días ("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday") y los meses ("January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December") para hablar de fechas, horarios y eventos. 🗓️</p>
                                <p class="mt-2"><em>Ejemplo:</em> "Today is Friday." (Hoy es viernes.) "My birthday is in August." (Mi cumpleaños es en agosto.)</p>
                            </li>
                            <li><strong>Objetos Comunes y Verbos de Acción:</strong>
                                <p>Vocabulario de objetos cotidianos en casa, escuela o trabajo ("table", "chair", "book", "phone", "computer", "pen", "door", "window") y verbos de acción simples para describir actividades diarias ("eat", "drink", "sleep", "run", "walk", "read", "write", "listen", "speak"). 🛋️📱🏃‍♀️</p>
                                <p class="mt-2"><em>Ejemplo:</em> "I eat an apple." (Yo como una manzana.) 🍎 "She reads a book." (Ella lee un libro.)</p>
                            </li>
                            <li><strong>Comprensión Auditiva Básica:</strong>
                                <p>Entender frases simples, preguntas directas y conversaciones muy lentas y claras. La práctica con audios sencillos, como diálogos de principiantes o canciones infantiles, es clave para desarrollar el oído. 👂</p>
                                <p class="mt-2"><em>Ejemplo:</em> Escuchar y entender "¿How are you?" y responder "I'm fine, thank you." (Estoy bien, gracias.)</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">3. Estructuras Gramaticales Básicas 🏗️ - Construyendo Oraciones</h4>
                            <p class="mb-2">Introducción a las estructuras más simples del inglés, que te permitirán formar oraciones coherentes y expresar ideas básicas. Dominar estas estructuras es esencial para la comunicación efectiva.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Verbo "To Be":</strong>
                                <p>El verbo más fundamental en inglés, utilizado para describir estados, características, nacionalidades, profesiones, ubicaciones y emociones. Aprender a usar "am" (para I), "is" (para he, she, it) y "are" (para you, we, they) correctamente es crucial. (ej. "I am happy", "She is a student", "They are friends", "He is tall", "We are in the park").</p>
                                <p class="mt-2"><em>Ejemplo:</em> "He is tall." (Él es alto.) 🧍‍♂️ "I am a teacher." (Soy un/a maestro/a.)</p>
                            </li>
                            <li><strong>Pronombres Personales:</strong>
                                <p>Aprender "I" (yo), "you" (tú/ustedes), "he" (él), "she" (ella), "it" (ello/a para objetos o animales), "we" (nosotros/as), "they" (ellos/as) y su uso correcto como sujetos de la oración. Son esenciales para referirse a personas y cosas sin repetir nombres, haciendo la comunicación más fluida. 🧑‍🤝‍🧑</p>
                                <p class="mt-2"><em>Ejemplo:</em> "They play soccer." (Ellos juegan fútbol.) ⚽ "It is a dog." (Es un perro.)</p>
                            </li>
                            <li><strong>Artículos:</strong>
                                <p>Aprender a usar "a" (un/una, antes de palabras que empiezan con sonido de consonante), "an" (un/una, antes de palabras que empiezan con sonido de vocal) para sustantivos singulares indeterminados, y "the" (el/la/los/las, para sustantivos específicos o ya mencionados), y cuándo utilizarlos. Son pequeños pero muy importantes para la precisión. 🅰️🅱️</p>
                                <p class="mt-2"><em>Ejemplo:</em> "A cat" (un gato), "an apple" (una manzana), "the sun" (el sol). 🐱🍎☀️</p>
                            </li>
                            <li><strong>Formación de Oraciones Simples:</strong>
                                <p>La estructura básica Sujeto + Verbo + Complemento (SVO - Subject-Verb-Object) es la más común en inglés. (ej. "I eat apples", "She reads a book", "He drinks water"). Practicar la construcción de estas oraciones te dará la base para expresarte. </p>
                                <p class="mt-2"><em>Ejemplo:</em> "He drinks water." (Él bebe agua.) 💧 "They like pizza." (A ellos les gusta la pizza.)</p>
                            </li>
                            <li><strong>Adjetivos y Adverbios Básicos:</strong> Introducción a los adjetivos (describen sustantivos, ej. "big", "happy") y adverbios (describen verbos, adjetivos u otros adverbios, ej. "quickly", "very").</li>
                            </ul>
                        </div>
                        <p class="mt-4">La clave en este nivel es la práctica constante y la inmersión en el idioma, incluso con pequeños pasos. ¡No tengas miedo de cometer errores! Son parte del aprendizaje y te ayudarán a mejorar. ¡Disfruta el proceso! 😊</p>
                    `,
                    questions: [
                        { id: 'i_b_q1', text: 'Completa: "Hello, my name ___ John."', type: 'text', answer: 'is' },
                        { id: 'i_b_q2', text: 'Traduce "Buenos días" al inglés.', type: 'text', answer: 'Good morning' },
                        { id: 'i_b_q3', text: 'Escribe el número 15 en inglés.', type: 'text', answer: 'fifteen' },
                        { id: 'i_b_q4', text: 'Completa con "a" o "an": "___ orange".', type: 'text', answer: 'an' },
                        { id: 'i_b_q5', text: '¿Cuál es el pronombre personal para referirse a una mujer?', type: 'text', answer: 'she' },
                        { id: 'i_b_q6', text: 'Traduce "Gracias" al inglés.', type: 'text', answer: 'Thank you' },
                        { id: 'i_b_q7', text: '¿Cómo se dice "rojo" en inglés?', type: 'text', answer: 'red' },
                        { id: 'i_b_q8', text: 'Escribe el día de la semana que sigue a "Tuesday".', type: 'text', answer: 'Wednesday' },
                        { id: 'i_b_q9', text: '¿Cómo se dice "libro" en inglés?', type: 'text', answer: 'book' },
                        { id: 'i_b_q10', text: 'Completa: "I ___ from Spain."', type: 'text', answer: 'am' },
                        { id: 'i_b_q11', text: 'Traduce "Adiós" al inglés.', type: 'text', answer: 'Goodbye' },
                        { id: 'i_b_q12', text: 'Escribe el número 50 en inglés.', type: 'text', answer: 'fifty' },
                        { id: 'i_b_q13', text: '¿Cómo se dice "azul" en inglés?', type: 'text', answer: 'blue' },
                        { id: 'i_b_q14', text: 'Escribe el mes que sigue a "March".', type: 'text', answer: 'April' },
                        { id: 'i_b_q15', text: '¿Cómo se dice "comer" en inglés?', type: 'text', answer: 'eat' },
                        { id: 'i_b_q16', text: 'Completa: "They ___ students."', type: 'text', answer: 'are' },
                        { id: 'i_b_q17', text: 'Traduce "¿Cómo estás?" al inglés.', type: 'text', answer: 'How are you?' },
                        { id: 'i_b_q18', text: 'Escribe el número 100 en inglés.', type: 'text', answer: 'one hundred' },
                        { id: 'i_b_q19', text: '¿Cómo se dice "verde" en inglés?', type: 'text', answer: 'green' },
                        { id: 'i_b_q20', text: 'Escribe el día de la semana que precede a "Sunday".', type: 'text', answer: 'Saturday' },
                    ]
                },
                intermedio: {
                    theory: `
                        <p class="mb-6">En el nivel Intermedio de Inglés 🗣️📚, expandirás tu vocabulario, dominarás tiempos verbales más complejos y mejorarás tu fluidez para mantener conversaciones más significativas y expresarte con mayor precisión. Este nivel te permitirá interactuar con mayor autonomía en diversas situaciones cotidianas y laborales.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Dominio de Tiempos Verbales Clave ⏰ - Viajando en el Tiempo</h4>
                            <p class="mb-2">Aprenderás a expresar acciones en diferentes momentos y con distintos matices, lo cual es fundamental para una comunicación precisa y contextualizada. Esto te permitirá hablar sobre el pasado, el presente y el futuro con mayor detalle. ⏳</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Presente Continuo (Present Continuous):</strong>
                                <p>Para acciones que están ocurriendo ahora mismo (ej. "I am studying English right now" - Estoy estudiando inglés ahora mismo), para planes futuros definidos (ej. "We are meeting tomorrow at 3 PM" - Nos reuniremos mañana a las 3 PM) o para describir tendencias temporales. Se forma con el verbo "to be" (am/is/are) + verbo principal con -ing.</p>
                                <p class="mt-2"><em>Ejemplo:</em> "She is reading a book." (Ella está leyendo un libro.) 📖 "The climate is changing rapidly." (El clima está cambiando rápidamente.)</p>
                            </li>
                            <li><strong>Pasado Simple (Simple Past):</strong>
                                <p>Para acciones terminadas en un momento específico del pasado (ej. "She visited Paris last year" - Ella visitó París el año pasado, "I ate pizza yesterday" - Comí pizza ayer). Se usa con verbos regulares (añadir -ed) e irregulares (que tienen formas especiales, ej. go-went, see-saw, eat-ate). Es fundamental para narrar eventos pasados.</p>
                                <p class="mt-2"><em>Ejemplo:</em> "They played soccer." (Ellos jugaron fútbol.) ⚽ "I went to the cinema last night." (Fui al cine anoche.)</p>
                            </li>
                            <li><strong>Futuro Simple (Simple Future - Will / Be Going To):</strong>
                                <p>Para planes, predicciones y decisiones espontáneas. "Will" se usa para predicciones generales, decisiones tomadas en el momento de hablar, promesas y ofertas (ej. "It will rain tomorrow" - Lloverá mañana, "I'll help you" - Te ayudaré). "Be going to" se usa para planes ya decididos o intenciones, y para predicciones basadas en evidencia presente (ej. "She is going to study medicine" - Ella va a estudiar medicina, "Look at those clouds, it's going to rain" - Mira esas nubes, va a llover). </p>
                                <p class="mt-2"><em>Ejemplo:</em> "It will rain tomorrow." (Lloverá mañana.) ☔ "I am going to buy a new car next month." (Voy a comprar un coche nuevo el próximo mes.)</p>
                            </li>
                            <li><strong>Presente Perfecto (Present Perfect):</strong>
                                <p>Para acciones que comenzaron en el pasado y continúan en el presente (ej. "I have lived here for five years" - He vivido aquí durante cinco años, y sigo viviendo aquí), o para acciones terminadas en el pasado pero con un resultado o relevancia en el presente (ej. "She has finished her homework" - Ella ha terminado su tarea, la tarea está hecha ahora). Se forma con "have/has" + participio pasado del verbo principal. ✅</p>
                                <p class="mt-2"><em>Ejemplo:</em> "I have never seen that movie." (Nunca he visto esa película.) 🎬 "He has lost his keys." (Él ha perdido sus llaves - y no las tiene ahora.)</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Vocabulario Funcional y Expresiones Comunes 💬 - Enriquece tu Lenguaje</h4>
                            <p class="mb-2">Ampliarás tu léxico para hablar sobre temas cotidianos, situaciones específicas y empezar a comprender y usar expresiones idiomáticas, lo que te hará sonar más natural y te permitirá entender mejor a los hablantes nativos. 🗣️</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Vocabulario de Viajes:</strong>
                                <p>Frases y términos esenciales para situaciones en aeropuertos ✈️ (check-in, boarding pass, luggage), hoteles 🏨 (reservation, single/double room), pedir direcciones 🗺️ (turn left/right, go straight, landmark), transporte público (bus, train, subway, ticket) y emergencias (emergency, doctor, police). </p>
                                <p class="mt-2"><em>Ejemplo:</em> "Where is the nearest subway station?" (¿Dónde está la estación de metro más cercana?), "I'd like to check in." (Me gustaría registrarme en el hotel.)</p>
                            </li>
                            <li><strong>Compras y Servicios:</strong>
                                <p>Cómo preguntar precios ("How much is this?"), tallas ("What size is this?", "Do you have this in a larger size?"), describir productos ("It's too big/small", "I'm looking for..."), negociar (Can I get a discount?) y solicitar servicios en tiendas 🛍️🏷️, restaurantes 🍽️ (menu, order, bill), bancos y otros establecimientos.</p>
                                <p class="mt-2"><em>Ejemplo:</em> "Do you have this in a larger size?" (¿Tienes esto en una talla más grande?) "I'll have the chicken, please." (Tomaré el pollo, por favor.)</p>
                            </li>
                            <li><strong>Salud y Bienestar:</strong>
                                <p>Vocabulario para describir síntomas ("I have a headache", "I feel sick"), ir al médico 🤒⚕️ (appointment, prescription), hablar sobre hábitos saludables (exercise, healthy food) y emergencias médicas (ambulance, first aid). </p>
                                <p class="mt-2"><em>Ejemplo:</em> "I have a headache." (Tengo dolor de cabeza.) "I need to see a doctor." (Necesito ver a un médico.)</p>
                            </li>
                            <li><strong>Expresiones Idiomáticas Básicas:</strong>
                                <p>Frases cuyo significado no es literal, pero que son muy comunes en el inglés nativo (ej. "break a leg" - ¡buena suerte!; "it's raining cats and dogs" - está lloviendo a cántaros; "piece of cake" - muy fácil; "hit the road" - ponerse en marcha). Conocerlas te ayudará a entender y sonar más natural. 🌧️�🐕</p>
                                <p class="mt-2"><em>Ejemplo:</em> "Don't worry, it's a piece of cake!" (No te preocupes, ¡es pan comido!).</p>
                            </li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">3. Mejora de Habilidades Comunicativas Integradas 👂✍️ - Fluidez y Comprensión</h4>
                            <p class="mb-2">Enfocarse en la comprensión y producción del idioma de manera más fluida y natural, integrando todas las habilidades (escucha, habla, lectura, escritura) para una comunicación más efectiva. La práctica combinada de estas habilidades acelera el aprendizaje.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Comprensión Auditiva (Listening):</strong>
                                <p>Escuchar podcasts 🎧, noticias sencillas, canciones 🎶, entrevistas y conversaciones cotidianas para mejorar la comprensión de diferentes acentos, velocidades del habla y contextos. La clave es la exposición constante y activa. </p>
                                <p class="mt-2"><em>Ejemplo:</em> Ver series o películas en inglés con subtítulos en inglés, escuchar la radio en inglés mientras haces otras actividades.</p>
                            </li>
                            <li><strong>Expresión Oral (Speaking):</strong>
                                <p>Participar en debates simples, describir experiencias personales, dar opiniones, hacer preguntas de seguimiento y mantener conversaciones más largas. Practicar la fluidez (hablar sin pausas excesivas) y la pronunciación (articulación clara de sonidos y palabras). Unirse a grupos de conversación o practicar con hablantes nativos es muy útil. 🗣️</p>
                                <p class="mt-2"><em>Ejemplo:</em> Unirse a un club de conversación en inglés, grabar tu propia voz para identificar errores, describir tu día a un amigo en inglés.</p>
                            </li>
                            <li><strong>Expresión Escrita (Writing):</strong>
                                <p>Redactar correos electrónicos, descripciones, historias cortas, diarios, blogs y mensajes coherentes sobre temas familiares y personales. Enfocarse en la claridad, la corrección gramatical y la cohesión de las ideas. La escritura regular ayuda a consolidar el vocabulario y la gramática. 📝</p>
                                <p class="mt-2"><em>Ejemplo:</em> Escribir un correo electrónico a un amigo describiendo tus vacaciones, mantener un diario personal en inglés.</p>
                            </li>
                            <li><strong>Comprensión Lectora (Reading):</strong>
                                <p>Leer artículos de noticias, blogs, historias cortas, textos adaptados y libros de nivel intermedio para entender la idea principal, los detalles específicos, el vocabulario en contexto y la intención del autor. La lectura amplía tu vocabulario y te expone a diferentes estructuras gramaticales. 👓📚</p>
                                <p class="mt-2"><em>Ejemplo:</em> Leer noticias en inglés sobre temas de tu interés, leer un libro de ficción adaptado a tu nivel.</p>
                            </li>
                            </ul>
                        </div>
                        <p class="mt-4">Este nivel te permitirá interactuar con mayor confianza y autonomía en situaciones reales, expandiendo tus horizontes de comunicación y abriendo nuevas oportunidades. ¡Sigue practicando y sumérgete en el idioma con curiosidad y perseverancia! 🌊</p>
                    `,
                    questions: [
                        { id: 'i_i_q1', text: 'Conjuga el verbo "to eat" en pasado simple.', type: 'text', answer: 'ate' },
                        { id: 'i_i_q2', text: 'Escribe una oración en presente perfecto usando "have".', type: 'text', answer: 'I have finished my homework.' }, // Allow variations
                        { id: 'i_i_q3', text: 'Imagina que estás de viaje. ¿Cómo preguntarías por el precio de algo?', type: 'text', answer: 'How much is this?' },
                        { id: 'i_i_q4', text: '¿Qué significa la expresión "it\'s raining cats and dogs"?', type: 'text', answer: 'Está lloviendo a cántaros.' },
                        { id: 'i_i_q5', text: 'Menciona una forma de mejorar tu comprensión auditiva en inglés.', type: 'text', answer: 'Escuchar podcasts' },
                        { id: 'i_i_q6', text: 'Completa la oración en presente continuo: "She ___ (study) for the exam."', type: 'text', answer: 'is studying' },
                        { id: 'i_i_q7', text: 'Conjuga el verbo "to go" en pasado simple.', type: 'text', answer: 'went' },
                        { id: 'i_i_q8', text: '¿Cuál es la diferencia entre "will" y "be going to" para el futuro?', type: 'text', answer: '"Will" para predicciones/decisiones espontáneas, "be going to" para planes decididos.' },
                        { id: 'i_i_q9', text: '¿Cómo se dice "aeropuerto" en inglés?', type: 'text', answer: 'airport' },
                        { id: 'i_i_q10', text: '¿Qué significa la expresión "break a leg"?', type: 'text', answer: '¡Buena suerte!' },
                        { id: 'i_i_q11', text: 'Escribe una oración en pasado simple con un verbo irregular.', type: 'text', answer: 'I saw a movie yesterday.' },
                        { id: 'i_i_q12', text: 'Completa: "They have ___ (live) here for ten years."', type: 'text', answer: 'lived' },
                        { id: 'i_i_q13', text: '¿Cómo pedirías la cuenta en un restaurante en inglés?', type: 'text', answer: 'Can I have the bill, please?' },
                        { id: 'i_i_q14', text: '¿Qué significa "a piece of cake"?', type: 'text', answer: 'Muy fácil.' },
                        { id: 'i_i_q15', text: 'Menciona una habilidad clave para la expresión oral en inglés.', type: 'text', answer: 'Fluidez' },
                        { id: 'i_i_q16', text: 'Completa la oración en futuro simple: "I think it ___ (rain) tomorrow."', type: 'text', answer: 'will rain' },
                        { id: 'i_i_q17', text: '¿Cómo se dice "tengo dolor de cabeza" en inglés?', type: 'text', answer: 'I have a headache.' },
                        { id: 'i_i_q18', text: 'Escribe una oración en presente perfecto con "never".', type: 'text', answer: 'I have never been to Paris.' },
                        { id: 'i_i_q19', text: '¿Qué tipo de textos son buenos para practicar la comprensión lectora intermedia?', type: 'text', answer: 'Artículos de noticias sencillos.' },
                        { id: 'i_i_q20', text: '¿Cómo se dice "hotel" en inglés?', type: 'text', answer: 'hotel' },
                    ]
                },
                avanzado: {
                    theory: `
                        <p class="mb-6">En el nivel Avanzado de Inglés 🚀✨, refinarás tu fluidez, precisión y comprensión de matices culturales, preparándote para contextos académicos y profesionales exigentes y para una comunicación sofisticada y matizada. Este nivel te permitirá operar con el inglés a un nivel casi nativo.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Gramática Avanzada y Estructuras Complejas 🧩 - Precisión y Sofisticación</h4>
                            <p class="mb-2">Dominarás las estructuras que dan sofisticación y precisión a tu inglés, permitiéndote expresar ideas complejas con claridad y elegancia, como un hablante nativo. Esto es esencial para la escritura académica, presentaciones profesionales y debates complejos. 🧠</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Voz Pasiva (Passive Voice):</strong>
                                <p>Uso y cuándo aplicarla para enfatizar la acción o el objeto de la acción, en lugar del agente (quien realiza la acción). Es común en contextos académicos, científicos y formales para mantener la objetividad (ej. "The book was written by..." - El libro fue escrito por..., "Mistakes were made" - Se cometieron errores). Se forma con el verbo "to be" + participio pasado.</p>
                                <p class="mt-2"><em>Ejemplo:</em> "The experiment was conducted by the students." (El experimento fue conducido por los estudiantes). "The report was submitted yesterday." (El informe fue entregado ayer).</p>
                            </li>
                            <li><strong>Discurso Indirecto (Reported Speech):</strong>
                                <p>Reportar lo que alguien dijo o preguntó sin citar textualmente. Implica ajustar tiempos verbales (hacia el pasado), pronombres y adverbios de tiempo/lugar. Es crucial para narrar conversaciones y reportar información de terceros (ej. "He said he would come" - Él dijo que vendría, "She asked if I was ready" - Ella preguntó si yo estaba listo/a). </p>
                                <p class="mt-2"><em>Ejemplo:</em> Directo: "I am tired." (Estoy cansado/a.) Indirecto: "He said he was tired." (Él dijo que estaba cansado/a.)</p>
                            </li>
                            <li><strong>Condicionales Mixtos e Inversiones:</strong>
                                <p>Estructuras avanzadas para expresar hipótesis y situaciones complejas que combinan diferentes tiempos condicionales (ej. "If I had studied, I would be happy now" - Si hubiera estudiado, estaría feliz ahora - mezcla de pasado perfecto y condicional simple). Las inversiones (cambiar el orden sujeto-verbo) se usan para énfasis o formalidad, especialmente con adverbios negativos (ej. "Never have I seen such a thing" - Nunca he visto tal cosa, "Had I known..." - Si lo hubiera sabido...).</p>
                                <p class="mt-2"><em>Ejemplo:</em> "Had I known, I would have told you." (Si lo hubiera sabido, te lo habría dicho). "Scarcely had he arrived when the phone rang." (Apenas había llegado cuando sonó el teléfono).</p>
                            </li>
                            <li><strong>Frases Nominales y Adjetivales:</strong>
                                <p>Construir oraciones más densas, precisas y concisas, utilizando cláusulas y frases para añadir detalles y complejidad, lo que mejora la cohesión y la fluidez de tu escritura y habla. Permiten expresar ideas de forma más compacta y sofisticada. ✍️</p>
                                <p class="mt-2"><em>Ejemplo:</em> "The student, who had studied diligently, passed the exam." (El estudiante, que había estudiado diligentemente, aprobó el examen). En lugar de dos oraciones, una más compleja.</p>
                            </li>
                            <li><strong>Conjunciones y Conectores Avanzados:</strong> Uso de palabras y frases para unir ideas de forma lógica y fluida (ej. "furthermore", "consequently", "nevertheless", "on the one hand... on the other hand").</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Vocabulario Académico y Profesional 💼🎓 - Tu Léxico Especializado</h4>
                            <p class="mb-2">Adquirirás el léxico necesario para entornos especializados, permitiéndote operar eficazmente en contextos universitarios y laborales, y comprender publicaciones y discusiones complejas. Este vocabulario es clave para la comunicación de alto nivel.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Jerga de Negocios (Business Jargon):</strong>
                                <p>Términos específicos de finanzas 💰 (ROI, balance sheet), marketing 📈 (branding, target audience), gestión (strategy, efficiency), recursos humanos (recruitment, onboarding) y otras áreas corporativas. Esencial para reuniones, presentaciones, informes y documentos de trabajo. </p>
                                <p class="mt-2"><em>Ejemplo:</em> "ROI (Return on Investment)", "synergy" (sinergia), "bottom line" (resultado final), "core competency" (competencia central).</p>
                            </li>
                            <li><strong>Vocabulario Académico (Academic Vocabulary):</strong>
                                <p>Palabras y frases formales utilizadas en ensayos, investigaciones, presentaciones universitarias, tesis y publicaciones científicas. Es fundamental para la lectura crítica, la escritura académica y la participación en discusiones intelectuales. Incluye términos para argumentar, analizar, comparar y contrastar. 🧑‍🎓</p>
                                <p class="mt-2"><em>Ejemplo:</em> "Furthermore" (además), "consequently" (en consecuencia), "hypothesis" (hipótesis), "empirical evidence" (evidencia empírica), "paradigm" (paradigma), "discourse" (discurso).</p>
                            </li>
                            <li><strong>Sinónimos y Antónimos Avanzados:</strong>
                                <p>Para expresar ideas con mayor precisión, variedad y evitar repeticiones, enriqueciendo tu estilo tanto oral como escrito. Permite elegir la palabra exacta para el matiz deseado. ✍️</p>
                                <p class="mt-2"><em>Ejemplo:</em> En lugar de siempre decir "good", usar "excellent", "superb", "satisfactory", "commendable". En lugar de "bad", usar "detrimental", "subpar", "abysmal".</p>
                            </li>
                            <li><strong>Phrasal Verbs y Collocations:</strong>
                                <p>Dominar las combinaciones de palabras (verbos frasales como "put off" - posponer; "look up" - buscar información; "get over" - superar; y colocaciones como "make a decision" - tomar una decisión; "take a break" - tomar un descanso; "heavy rain" - lluvia intensa) que son esenciales para la fluidez y la comprensión nativa. Son una parte crucial del inglés coloquial y formal. 🧩</p>
                                <p class="mt-2"><em>Ejemplo:</em> "Look up" (buscar información), "take a break" (tomar un descanso). "Commit a crime" (cometer un crimen), no "do a crime".</p>
                            </li>
                            <li><strong>Modismos y Expresiones Idiomáticas:</strong> Comprender y usar expresiones culturales que no se pueden traducir literalmente (ej. "to bite the bullet", "to hit the nail on the head").</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">3. Fluidez, Precisión y Comprensión Cultural 🌐🗣️ - La Maestría del Idioma</h4>
                            <p class="mb-2">Perfeccionarás tu capacidad para comunicarte con naturalidad, entender las sutilezas culturales y participar en discusiones complejas con confianza, como un hablante casi nativo. Este es el nivel de maestría donde el idioma se convierte en una extensión de tu pensamiento.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                            <li><strong>Pronunciación Avanzada:</strong>
                                <p>Refinar la entonación (el patrón de subida y bajada del tono de voz), el ritmo (la velocidad y el énfasis), las reducciones (ej. "gonna" por "going to", "wanna" por "want to") y los "linking sounds" (unión de sonidos entre palabras, ej. "an apple" suena como "a napple") para sonar más natural y comprensible. Se busca la inteligibilidad y la naturalidad. 🔊</p>
                                <p class="mt-2"><em>Ejemplo:</em> Practicar la diferencia entre el sonido de "th" en "think" y "this". Escuchar y repetir frases completas para imitar el ritmo y la entonación nativa.</p>
                            </li>
                            <li><strong>Comprensión de Acentos:</strong>
                                <p>Exposición a y práctica con diferentes acentos nativos (británico 🇬🇧, americano 🇺🇸, australiano 🇦🇺, canadiense, irlandés, etc.) para una comprensión global y adaptabilidad. Esto te permitirá entender a una gama más amplia de hablantes. 🎧</p>
                                <p class="mt-2"><em>Ejemplo:</em> Escuchar podcasts y ver películas de diferentes países de habla inglesa, prestando atención a las variaciones fonéticas y de vocabulario.</p>
                            </li>
                            <li><strong>Debate y Argumentación:</strong>
                                <p>Participar en discusiones complejas, defender puntos de vista con argumentos sólidos y evidencia, refutar argumentos de manera respetuosa y negociar eficazmente. Implica el uso de lenguaje persuasivo, conectores lógicos y la capacidad de pensar críticamente en el idioma. 🗣️💬</p>
                                <p class="mt-2"><em>Ejemplo:</em> Debatir sobre el cambio climático, la economía global o temas sociales complejos, presentando y defendiendo tu postura con argumentos bien estructurados.</p>
                            </li>
                            <li><strong>Análisis Crítico de Textos:</strong>
                                <p>Leer y comprender textos complejos, incluyendo literatura (novelas, poesía), artículos científicos, informes de investigación, análisis de noticias y ensayos filosóficos, identificando el tono, el propósito, las ideas principales, los argumentos subyacentes, las sutilezas y las implicaciones culturales. Requiere habilidades de lectura profunda y pensamiento crítico. 👓📖</p>
                                <p class="mt-2"><em>Ejemplo:</em> Leer un artículo de The Economist o The New York Times y resumir sus puntos clave, analizar el estilo del autor y su postura.</p>
                            </li>
                            <li><strong>Escritura Avanzada:</strong> Redactar ensayos persuasivos, informes técnicos, propuestas de investigación, y correspondencia profesional, demostrando un control total de la gramática, el vocabulario y el estilo, y adaptando el registro al propósito y la audiencia.</li>
                            </ul>
                        </div>
                        <p class="mt-4">Este nivel te permitirá usar el inglés con maestría, abriendo un mundo de oportunidades académicas y profesionales, y permitiéndote una inmersión completa en la cultura angloparlante. ¡El mundo es tu escenario y el inglés tu pasaporte! 🌟</p>
                    `,
                    questions: [
                        { id: 'i_a_q1', text: 'Transforma la siguiente oración a voz pasiva: "The company launched a new product."', type: 'text', answer: 'A new product was launched by the company.' },
                        { id: 'i_a_q2', text: 'Reporta la siguiente pregunta: "Are you coming to the meeting tomorrow?"', type: 'text', answer: 'He asked if I was coming to the meeting the next day.' },
                        { id: 'i_a_q3', text: '¿Qué significa el phrasal verb "put off"?', type: 'text', answer: 'Posponer' },
                        { id: 'i_a_q4', text: 'Menciona un sinónimo avanzado para "good".', type: 'text', answer: 'Excellent' },
                        { id: 'i_a_q5', text: '¿Qué es una "collocation" en inglés?', type: 'text', answer: 'Una combinación de palabras que ocurre frecuentemente de forma natural.' },
                        { id: 'i_a_q6', text: 'Transforma a voz pasiva: "They built a new bridge."', type: 'text', answer: 'A new bridge was built by them.' },
                        { id: 'i_a_q7', text: 'Reporta la siguiente afirmación: "I have finished my project."', type: 'text', answer: 'She said she had finished her project.' },
                        { id: 'i_a_q8', text: '¿Cuál es la diferencia entre un condicional tipo 2 y un condicional mixto?', type: 'text', answer: 'El tipo 2 se refiere a situaciones hipotéticas en el presente/futuro, el mixto combina pasado y presente.' },
                        { id: 'i_a_q9', text: 'Menciona un término de jerga de negocios.', type: 'text', answer: 'ROI' },
                        { id: 'i_a_q10', text: '¿Qué significa el phrasal verb "take off"?', type: 'text', answer: 'Despegar' },
                        { id: 'i_a_q11', text: 'Menciona un antónimo avanzado para "happy".', type: 'text', answer: 'Miserable' },
                        { id: 'i_a_q12', text: '¿Qué es la inversión en gramática inglesa?', type: 'text', answer: 'Cambiar el orden sujeto-verbo para énfasis o formalidad.' },
                        { id: 'i_a_q13', text: '¿Cómo se dice "recursos humanos" en inglés?', type: 'text', answer: 'Human Resources' },
                        { id: 'i_a_q14', text: '¿Qué es el "linking sounds" en pronunciación avanzada?', type: 'text', answer: 'La unión de sonidos entre palabras.' },
                        { id: 'i_a_q15', text: 'Menciona un beneficio de leer artículos científicos en inglés.', type: 'text', answer: 'Mejora el vocabulario académico.' },
                        { id: 'i_a_q16', text: 'Transforma a voz pasiva: "The students are writing essays."', type: 'text', answer: 'Essays are being written by the students.' },
                        { id: 'i_a_q17', text: 'Reporta la siguiente pregunta: "What did you do yesterday?"', type: 'text', answer: 'He asked what I had done the day before.' },
                        { id: 'i_a_q18', text: '¿Qué significa el phrasal verb "get along with"?', type: 'text', answer: 'Llevarse bien con alguien.' },
                        { id: 'i_a_q19', text: 'Menciona un sinónimo avanzado para "important".', type: 'text', answer: 'Crucial' },
                        { id: 'i_a_q20', text: 'Da un ejemplo de "collocation" con el verbo "make".', type: 'text', answer: 'Make a decision.' },
                    ]
                },
            },
            arte: {
                basico: {
                    theory: `
                        <p class="mb-6">En el nivel Básico de Arte 🎨, descubrirás los fundamentos visuales y las herramientas esenciales para dar vida a tus ideas. Es el punto de partida para cualquier artista, sentando las bases para la expresión creativa.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Elementos Básicos del Arte 🖌️</h4>
                            <p class="mb-2">Los elementos básicos son los bloques de construcción de cualquier obra de arte.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Línea:</strong> Es el camino de un punto en movimiento. Puede ser recta, curva, gruesa, fina, etc. Define formas y contornos.</li>
                                <li><strong>Forma:</strong> Un área bidimensional definida por líneas, colores o texturas. Puede ser geométrica (cuadrado, círculo) u orgánica (formas naturales).</li>
                                <li><strong>Color:</strong> La percepción visual generada por la luz. Los colores primarios son rojo, azul y amarillo.</li>
                                <li><strong>Valor:</strong> La claridad u oscuridad de un color o tono. Va del blanco al negro.</li>
                                <li><strong>Textura:</strong> Cómo se siente o se ve una superficie (suave, áspera, brillante).</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Materiales y Técnicas Fundamentales ✏️</h4>
                            <p class="mb-2">Conoce las herramientas básicas para empezar a crear.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Lápiz:</strong> Ideal para bocetos, sombreado y dibujo de líneas. Diferentes durezas (H para duro, B para blando).</li>
                                <li><strong>Carboncillo:</strong> Produce tonos oscuros y suaves, ideal para grandes áreas y efectos dramáticos.</li>
                                <li><strong>Acuarela:</strong> Pintura transparente a base de agua, permite crear capas y efectos sutiles.</li>
                                <li><strong>Pincel:</strong> Herramienta esencial para aplicar pintura. Vienen en diversas formas y tamaños.</li>
                                <li><strong>Papel:</strong> La superficie más común para dibujar y pintar.</li>
                            </ul>
                        </div>
                        <p class="mt-4">¡Empieza a experimentar con estos conceptos y materiales para liberar tu creatividad! ✨</p>
                    `,
                    questions: [
                        { id: 'arte_b_q1', text: '¿Cuál es el camino de un punto en movimiento en el arte?', type: 'text', answer: 'Línea' },
                        { id: 'arte_b_q2', text: 'Menciona uno de los colores primarios.', type: 'text', answer: 'Rojo' },
                        { id: 'arte_b_q3', text: '¿Qué elemento del arte se refiere a la claridad u oscuridad?', type: 'text', answer: 'Valor' },
                        { id: 'arte_b_q4', text: '¿Qué tipo de forma es un cuadrado?', type: 'text', answer: 'Geométrica' },
                        { id: 'arte_b_q5', text: '¿Qué material de dibujo produce tonos oscuros y suaves?', type: 'text', answer: 'Carboncillo' },
                        { id: 'arte_b_q6', text: '¿Cómo se llama la pintura transparente a base de agua?', type: 'text', answer: 'Acuarela' },
                        { id: 'arte_b_q7', text: '¿Qué herramienta es esencial para aplicar pintura?', type: 'text', answer: 'Pincel' },
                        { id: 'arte_b_q8', text: '¿Qué tipo de forma es una hoja de árbol?', type: 'text', answer: 'Orgánica' },
                        { id: 'arte_b_q9', text: 'Menciona otro color primario además del rojo.', type: 'text', answer: 'Azul' },
                        { id: 'arte_b_q10', text: '¿Qué letra indica un lápiz blando?', type: 'text', answer: 'B' },
                        { id: 'arte_b_q11', text: '¿Qué es la textura en el arte?', type: 'text', answer: 'Cómo se siente o se ve una superficie.' },
                        { id: 'arte_b_q12', text: '¿Cuál es la superficie más común para dibujar?', type: 'text', answer: 'Papel' },
                        { id: 'arte_b_q13', text: '¿Qué se utiliza para definir formas y contornos?', type: 'text', answer: 'Línea' },
                        { id: 'arte_b_q14', text: '¿Qué es la forma en el arte?', type: 'text', answer: 'Un área bidimensional definida.' },
                        { id: 'arte_b_q15', text: '¿Qué material de dibujo es ideal para bocetos?', type: 'text', answer: 'Lápiz' },
                        { id: 'arte_b_q16', text: '¿Qué tipo de arte utiliza la luz para generar percepción visual?', type: 'text', answer: 'Color' },
                        { id: 'arte_b_q17', text: 'Si un objeto es muy claro, ¿qué valor tiene?', type: 'text', answer: 'Alto' },
                        { id: 'arte_b_q18', text: '¿Qué tipo de pincel usarías para detalles finos?', type: 'text', answer: 'Fino' },
                        { id: 'arte_b_q19', text: '¿Qué es un tono en el arte?', type: 'text', answer: 'La claridad u oscuridad de un color.' },
                        { id: 'arte_b_q20', text: '¿Qué color se obtiene mezclando rojo y azul?', type: 'text', answer: 'Morado' },
                    ]
                },
                intermedio: {
                    theory: `
                        <p class="mb-6">En el nivel Intermedio de Arte 🖼️, explorarás cómo crear la ilusión de profundidad y espacio, y profundizarás en la teoría del color para dar más vida a tus obras.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Composición y Perspectiva 📐</h4>
                            <p class="mb-2">Organiza los elementos para guiar la mirada del espectador y crear profundidad.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Composición:</strong> La organización de los elementos visuales en una obra de arte. Busca equilibrio, ritmo y énfasis.</li>
                                <li><strong>Regla de los Tercios:</strong> Divide la imagen en nueve secciones iguales con dos líneas horizontales y dos verticales. Colocar elementos importantes en las intersecciones o a lo largo de las líneas crea interés.</li>
                                <li><strong>Perspectiva Lineal:</strong> Crea la ilusión de profundidad utilizando líneas convergentes que se encuentran en un punto de fuga en el horizonte.</li>
                                <li><strong>Punto de Fuga:</strong> El punto en el horizonte donde las líneas paralelas parecen encontrarse.</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Teoría del Color Avanzada 🌈</h4>
                            <p class="mb-2">Más allá de los primarios, comprende cómo los colores interactúan.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Colores Complementarios:</strong> Opuestos en la rueda de color (ej. rojo-verde, azul-naranja). Crean alto contraste y vibración.</li>
                                <li><strong>Colores Análogos:</strong> Adyacentes en la rueda de color (ej. azul, azul-verde, verde). Crean armonía y transiciones suaves.</li>
                                <li><strong>Temperatura del Color:</strong> Colores cálidos (rojos, naranjas, amarillos) y fríos (azules, verdes, morados). Afectan la sensación y la percepción de distancia.</li>
                            </ul>
                        </div>
                        <p class="mt-4">¡Aplica estos principios para transformar tus dibujos en obras con mayor impacto visual! 🌟</p>
                    `,
                    questions: [
                        { id: 'arte_i_q1', text: '¿Cómo se llama la organización de los elementos visuales en una obra de arte?', type: 'text', answer: 'Composición' },
                        { id: 'arte_i_q2', text: '¿Qué regla divide la imagen en nueve secciones para mejorar la composición?', type: 'text', answer: 'Regla de los Tercios' },
                        { id: 'arte_i_q3', text: '¿Qué tipo de perspectiva utiliza un punto de fuga?', type: 'text', answer: 'Perspectiva Lineal' },
                        { id: 'arte_i_q4', text: '¿Cómo se llama el punto en el horizonte donde las líneas paralelas parecen encontrarse?', type: 'text', answer: 'Punto de Fuga' },
                        { id: 'arte_i_q5', text: 'Menciona un par de colores complementarios.', type: 'text', answer: 'Rojo-Verde' },
                        { id: 'arte_i_q6', text: '¿Qué tipo de colores son adyacentes en la rueda de color?', type: 'text', answer: 'Análogos' },
                        { id: 'arte_i_q7', text: 'Menciona un color cálido.', type: 'text', answer: 'Rojo' },
                        { id: 'arte_i_q8', text: '¿Qué sensación crean los colores complementarios?', type: 'text', answer: 'Contraste' },
                        { id: 'arte_i_q9', text: '¿Qué tipo de colores son el azul, el azul-verde y el verde?', type: 'text', answer: 'Análogos' },
                        { id: 'arte_i_q10', text: '¿Qué se busca con la composición en el arte?', type: 'text', answer: 'Equilibrio' },
                        { id: 'arte_i_q11', text: '¿Qué ayuda a crear la ilusión de profundidad en un dibujo?', type: 'text', answer: 'Perspectiva' },
                        { id: 'arte_i_q12', text: 'Si un artista quiere crear armonía, ¿qué tipo de colores usaría?', type: 'text', answer: 'Análogos' },
                        { id: 'arte_i_q13', text: '¿Qué efecto tiene la temperatura del color en la percepción de distancia?', type: 'text', answer: 'Afecta la percepción de distancia.' },
                        { id: 'arte_i_q14', text: '¿Qué color es complementario al azul?', type: 'text', answer: 'Naranja' },
                        { id: 'arte_i_q15', text: '¿Qué es el horizonte en la perspectiva lineal?', type: 'text', answer: 'La línea donde el cielo y la tierra se encuentran.' },
                        { id: 'arte_i_q16', text: '¿Qué se logra con la regla de los tercios?', type: 'text', answer: 'Crear interés visual.' },
                        { id: 'arte_i_q17', text: 'Menciona un color frío.', type: 'text', answer: 'Azul' },
                        { id: 'arte_i_q18', text: '¿Qué tipo de líneas convergen en un punto de fuga?', type: 'text', answer: 'Paralelas' },
                        { id: 'arte_i_q19', text: '¿Qué es la temperatura del color?', type: 'text', answer: 'La clasificación de colores en cálidos y fríos.' },
                        { id: 'arte_i_q20', text: '¿Qué efecto se busca al usar colores complementarios juntos?', type: 'text', answer: 'Vibración' },
                    ]
                },
                avanzado: {
                    theory: `
                        <p class="mb-6">En el nivel Avanzado de Arte 🏛️, te sumergirás en el estudio de la figura humana, explorarás el arte digital y aprenderás a analizar y criticar obras de arte con profundidad.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Figura Humana y Retrato 👤</h4>
                            <p class="mb-2">Dominar la representación del cuerpo y el rostro humano.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Anatomía Artística:</strong> Estudio de la estructura ósea y muscular para dibujar el cuerpo humano de forma realista y dinámica.</li>
                                <li><strong>Proporciones:</strong> Las relaciones de tamaño entre las diferentes partes del cuerpo o rostro. Por ejemplo, la cabeza es una unidad de medida común.</li>
                                <li><strong>Retrato:</strong> Representación artística de una persona, enfocándose en las características faciales y la expresión.</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Arte Digital y Nuevos Medios 💻</h4>
                            <p class="mb-2">Explora las herramientas y técnicas del arte en el entorno digital.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Software de Ilustración:</strong> Programas como Adobe Photoshop o Procreate para crear imágenes digitales.</li>
                                <li><strong>Tableta Gráfica:</strong> Dispositivo de entrada que permite dibujar directamente en la computadora con un lápiz óptico.</li>
                                <li><strong>Concept Art:</strong> Creación de diseños visuales para personajes, entornos y objetos en videojuegos o películas.</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">3. Crítica y Análisis de Arte 🧐</h4>
                            <p class="mb-2">Desarrolla tu capacidad para interpretar y evaluar obras de arte.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Análisis Formal:</strong> Examinar los elementos (línea, color, forma) y principios (composición, ritmo) del arte en una obra.</li>
                                <li><strong>Contexto Histórico y Cultural:</strong> Comprender la obra en relación con la época y sociedad en que fue creada.</li>
                                <li><strong>Interpretación:</strong> Explorar el significado, el mensaje o las emociones que la obra transmite.</li>
                            </ul>
                        </div>
                        <p class="mt-4">¡Con estas habilidades, podrás no solo crear, sino también apreciar y entender el arte en un nivel más profundo! 🚀</p>
                    `,
                    questions: [
                        { id: 'arte_a_q1', text: '¿Qué estudio ayuda a dibujar el cuerpo humano de forma realista?', type: 'text', answer: 'Anatomía Artística' },
                        { id: 'arte_a_q2', text: '¿Cómo se llama la relación de tamaño entre las partes del cuerpo?', type: 'text', answer: 'Proporciones' },
                        { id: 'arte_a_q3', text: '¿Qué tipo de representación artística se enfoca en el rostro?', type: 'text', answer: 'Retrato' },
                        { id: 'arte_a_q4', text: 'Menciona un software popular para ilustración digital.', type: 'text', answer: 'Adobe Photoshop' },
                        { id: 'arte_a_q5', text: '¿Qué dispositivo permite dibujar directamente en la computadora con un lápiz óptico?', type: 'text', answer: 'Tableta Gráfica' },
                        { id: 'arte_a_q6', text: '¿Qué tipo de arte crea diseños visuales para videojuegos o películas?', type: 'text', answer: 'Concept Art' },
                        { id: 'arte_a_q7', text: '¿Qué se examina en un análisis formal de una obra de arte?', type: 'text', answer: 'Elementos y principios del arte.' },
                        { id: 'arte_a_q8', text: '¿Qué se debe comprender para analizar el contexto de una obra?', type: 'text', answer: 'Época y sociedad.' },
                        { id: 'arte_a_q9', text: '¿Qué se explora al interpretar una obra de arte?', type: 'text', answer: 'Significado o emociones.' },
                        { id: 'arte_a_q10', text: '¿Cuál es una unidad de medida común en proporciones artísticas?', type: 'text', answer: 'Cabeza' },
                        { id: 'arte_a_q11', text: '¿Qué tipo de músculos son importantes en la anatomía artística?', type: 'text', answer: 'Musculares' },
                        { id: 'arte_a_q12', text: '¿Qué es un lápiz óptico?', type: 'text', answer: 'Un lápiz para dibujar en tabletas gráficas.' },
                        { id: 'arte_a_q13', text: '¿Qué se busca al dibujar el cuerpo humano de forma dinámica?', type: 'text', answer: 'Movimiento' },
                        { id: 'arte_a_q14', text: 'Menciona otro software de ilustración digital además de Photoshop.', type: 'text', answer: 'Procreate' },
                        { id: 'arte_a_q15', text: '¿Qué aspecto de la obra de arte se evalúa en la crítica?', type: 'text', answer: 'Calidad o impacto.' },
                        { id: 'arte_a_q16', text: '¿Qué tipo de estructura es fundamental en la anatomía artística?', type: 'text', answer: 'Ósea' },
                        { id: 'arte_a_q17', text: '¿Qué es la expresión en un retrato?', type: 'text', answer: 'La emoción o el estado de ánimo del sujeto.' },
                        { id: 'arte_a_q18', text: '¿Qué son los nuevos medios en el arte?', type: 'text', answer: 'Formas de arte que utilizan tecnología digital.' },
                        { id: 'arte_a_q19', text: '¿Qué se considera al analizar el contexto cultural de una obra?', type: 'text', answer: 'Las costumbres o creencias de la sociedad.' },
                        { id: 'arte_a_q20', text: '¿Qué es la crítica de arte?', type: 'text', answer: 'El análisis y evaluación de obras de arte.' },
                    ]
                },
            },
            computo: { // New Computo Course
                basico: {
                    theory: `
                        <p class="mb-6">En el nivel Básico de Cómputo 💻, te familiarizarás con los componentes fundamentales de una computadora y cómo interactúan para realizar tareas básicas. Este nivel es esencial para entender el funcionamiento de los dispositivos que usamos a diario.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Componentes de Hardware 🖥️</h4>
                            <p class="mb-2">El hardware se refiere a todas las partes físicas de una computadora.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>CPU (Unidad Central de Procesamiento):</strong> El "cerebro" de la computadora, encargado de ejecutar instrucciones y procesar datos.</li>
                                <li><strong>RAM (Memoria de Acceso Aleatorio):</strong> Memoria temporal donde se almacenan los datos y programas que se están utilizando activamente. Es volátil.</li>
                                <li><strong>Almacenamiento (Disco Duro/SSD):</strong> Dispositivo donde se guardan permanentemente los archivos y el sistema operativo.</li>
                                <li><strong>Dispositivos de Entrada:</strong> Permiten introducir información a la computadora (teclado, ratón, micrófono).</li>
                                <li><strong>Dispositivos de Salida:</strong> Muestran o entregan información de la computadora (monitor, impresora, altavoces).</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Software Básico 💾</h4>
                            <p class="mb-2">El software son los programas y aplicaciones que hacen funcionar el hardware.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Sistema Operativo (OS):</strong> Programa principal que gestiona el hardware y el software (Windows, macOS, Linux).</li>
                                <li><strong>Aplicaciones:</strong> Programas diseñados para tareas específicas (navegadores web, procesadores de texto, juegos).</li>
                                <li><strong>Archivos y Carpetas:</strong> Cómo se organiza la información digital en la computadora.</li>
                            </ul>
                        </div>
                        <p class="mt-4">Comprender estos conceptos te dará una base sólida para interactuar con cualquier sistema informático. ¡A seguir explorando! ✨</p>
                    `,
                    questions: [
                        { id: 'computo_b_q1', text: '¿Cuál es el "cerebro" de la computadora?', type: 'text', answer: 'CPU' },
                        { id: 'computo_b_q2', text: '¿Dónde se almacenan los datos temporalmente mientras se usan?', type: 'text', answer: 'RAM' },
                        { id: 'computo_b_q3', text: 'Menciona un dispositivo de entrada.', type: 'text', answer: 'Teclado' },
                        { id: 'computo_b_q4', text: 'Menciona un dispositivo de salida.', type: 'text', answer: 'Monitor' },
                        { id: 'computo_b_q5', text: '¿Qué tipo de memoria es volátil?', type: 'text', answer: 'RAM' },
                        { id: 'computo_b_q6', text: '¿Qué programa principal gestiona el hardware y el software?', type: 'text', answer: 'Sistema Operativo' },
                        { id: 'computo_b_q7', text: 'Menciona un ejemplo de sistema operativo.', type: 'text', answer: 'Windows' },
                        { id: 'computo_b_q8', text: '¿Qué son los programas diseñados para tareas específicas?', type: 'text', answer: 'Aplicaciones' },
                        { id: 'computo_b_q9', text: '¿Dónde se guardan permanentemente los archivos?', type: 'text', answer: 'Disco Duro' },
                        { id: 'computo_b_q10', text: '¿Qué es el hardware?', type: 'text', answer: 'Partes físicas de la computadora' },
                        { id: 'computo_b_q11', text: '¿Qué es el software?', type: 'text', answer: 'Programas y aplicaciones' },
                        { id: 'computo_b_q12', text: '¿Qué significa CPU?', type: 'text', answer: 'Unidad Central de Procesamiento' },
                        { id: 'computo_b_q13', text: '¿Qué significa RAM?', type: 'text', answer: 'Memoria de Acceso Aleatorio' },
                        { id: 'computo_b_q14', text: '¿Cómo se organiza la información digital?', type: 'text', answer: 'Archivos y Carpetas' },
                        { id: 'computo_b_q15', text: 'Menciona un ejemplo de aplicación.', type: 'text', answer: 'Navegador web' },
                        { id: 'computo_b_q16', text: '¿Es un ratón un dispositivo de entrada o salida?', type: 'text', answer: 'Entrada' },
                        { id: 'computo_b_q17', text: '¿Es una impresora un dispositivo de entrada o salida?', type: 'text', answer: 'Salida' },
                        { id: 'computo_b_q18', text: '¿Qué tipo de almacenamiento es más rápido, HDD o SSD?', type: 'text', answer: 'SSD' },
                        { id: 'computo_b_q19', text: '¿Qué es un archivo?', type: 'text', answer: 'Un conjunto de datos guardados.' },
                        { id: 'computo_b_q20', text: '¿Qué es una carpeta?', type: 'text', answer: 'Un contenedor para organizar archivos.' },
                    ]
                },
                intermedio: {
                    theory: `
                        <p class="mb-6">En el nivel Intermedio de Cómputo 🌐, explorarás cómo las computadoras se conectan y comunican, entenderás los conceptos básicos de la programación y aprenderás a proteger tu información en el mundo digital.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Redes y Conectividad 📡</h4>
                            <p class="mb-2">Cómo las computadoras se comunican entre sí.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Redes Locales (LAN):</strong> Conexión de computadoras en un área pequeña (casa, oficina).</li>
                                <li><strong>Internet:</strong> Red global de computadoras que permite el intercambio de información a gran escala.</li>
                                <li><strong>World Wide Web (WWW):</strong> Sistema de documentos interconectados accesibles a través de Internet (páginas web).</li>
                                <li><strong>Navegadores Web:</strong> Software para acceder y visualizar páginas web (Chrome, Firefox).</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Fundamentos de Programación ✍️</h4>
                            <p class="mb-2">Las bases para dar instrucciones a una computadora.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Algoritmo:</strong> Una secuencia de pasos lógicos para resolver un problema.</li>
                                <li><strong>Lenguaje de Programación:</strong> Un lenguaje formal para escribir instrucciones que una computadora puede entender (Python, JavaScript).</li>
                                <li><strong>Variables:</strong> Espacios de memoria para almacenar datos que pueden cambiar.</li>
                                <li><strong>Condicionales (If/Else):</strong> Estructuras que permiten al programa tomar decisiones basadas en condiciones.</li>
                                <li><strong>Bucles (Loops):</strong> Estructuras que repiten un bloque de código varias veces.</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">3. Ciberseguridad Básica 🔒</h4>
                            <p class="mb-2">Proteger tu información y dispositivos en línea.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Contraseñas Seguras:</strong> Largas, complejas y únicas para cada cuenta.</li>
                                <li><strong>Phishing:</strong> Intentos de engaño para obtener información personal (correos falsos).</li>
                                <li><strong>Malware (Virus, Troyanos):</strong> Software malicioso que daña o roba información.</li>
                                <li><strong>Antivirus:</strong> Software para detectar y eliminar malware.</li>
                            </ul>
                        </div>
                        <p class="mt-4">Estos conocimientos te permitirán navegar el mundo digital con mayor seguridad y empezar a entender cómo se construyen las aplicaciones. ¡Sigue aprendiendo! 🚀</p>
                    `,
                    questions: [
                        { id: 'computo_i_q1', text: '¿Qué tipo de red conecta computadoras en un área pequeña?', type: 'text', answer: 'LAN' },
                        { id: 'computo_i_q2', text: '¿Cómo se llama la red global de computadoras?', type: 'text', answer: 'Internet' },
                        { id: 'computo_i_q3', text: 'Menciona un navegador web.', type: 'text', answer: 'Chrome' },
                        { id: 'computo_i_q4', text: '¿Qué es una secuencia de pasos lógicos para resolver un problema?', type: 'text', answer: 'Algoritmo' },
                        { id: 'computo_i_q5', text: 'Menciona un lenguaje de programación.', type: 'text', answer: 'Python' },
                        { id: 'computo_i_q6', text: '¿Qué son los espacios de memoria para almacenar datos que pueden cambiar?', type: 'text', answer: 'Variables' },
                        { id: 'computo_i_q7', text: '¿Qué estructura permite al programa tomar decisiones?', type: 'text', answer: 'Condicionales' },
                        { id: 'computo_i_q8', text: '¿Qué estructura repite un bloque de código varias veces?', type: 'text', answer: 'Bucles' },
                        { id: 'computo_i_q9', text: '¿Cómo deben ser las contraseñas para ser seguras?', type: 'text', answer: 'Largas, complejas y únicas' },
                        { id: 'computo_i_q10', text: '¿Qué es el phishing?', type: 'text', answer: 'Intentos de engaño para obtener información personal.' },
                        { id: 'computo_i_q11', text: '¿Qué es el malware?', type: 'text', answer: 'Software malicioso.' },
                        { id: 'computo_i_q12', text: '¿Qué software detecta y elimina malware?', type: 'text', answer: 'Antivirus' },
                        { id: 'computo_i_q13', text: '¿Qué significa WWW?', type: 'text', answer: 'World Wide Web' },
                        { id: 'computo_i_q14', text: '¿Qué es un lenguaje de programación?', type: 'text', answer: 'Un lenguaje para escribir instrucciones para computadoras.' },
                        { id: 'computo_i_q15', text: '¿Qué tipo de red es la que usas en casa?', type: 'text', answer: 'LAN' },
                        { id: 'computo_i_q16', text: 'Si un programa necesita elegir entre dos caminos, ¿qué estructura usaría?', type: 'text', answer: 'Condicional' },
                        { id: 'computo_i_q17', text: 'Si quieres que una acción se repita 100 veces, ¿qué usarías?', type: 'text', answer: 'Bucle' },
                        { id: 'computo_i_q18', text: '¿Qué tipo de ataque de ciberseguridad llega por correo electrónico?', type: 'text', answer: 'Phishing' },
                        { id: 'computo_i_q19', text: '¿Qué es un troyano en ciberseguridad?', type: 'text', answer: 'Un tipo de malware que se disfraza de software legítimo.' },
                        { id: 'computo_i_q20', text: '¿Cuál es el objetivo principal de la ciberseguridad?', type: 'text', answer: 'Proteger datos y privacidad.' },
                    ]
                },
                avanzado: {
                    theory: `
                        <p class="mb-6">En el nivel Avanzado de Cómputo 🧠, te adentrarás en conceptos más complejos que impulsan la tecnología moderna, desde cómo se almacena y gestiona la información a gran escala hasta los principios de la inteligencia artificial y el desarrollo web.</p>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">1. Gestión de Datos y Cloud Computing ☁️</h4>
                            <p class="mb-2">Cómo se manejan grandes volúmenes de información y se accede a servicios a través de Internet.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Bases de Datos:</strong> Sistemas organizados para almacenar, gestionar y recuperar grandes cantidades de datos de forma eficiente.</li>
                                <li><strong>SQL (Structured Query Language):</strong> Lenguaje estándar para interactuar con bases de datos relacionales (consultar, insertar, actualizar, eliminar datos).</li>
                                <li><strong>Cloud Computing:</strong> Entrega de servicios informáticos (servidores, almacenamiento, bases de datos, redes, software, análisis) a través de Internet ("la nube"), en lugar de tenerlos físicamente en tu ubicación.</li>
                                <li><strong>Big Data:</strong> Conjuntos de datos tan grandes y complejos que las aplicaciones de procesamiento de datos tradicionales no pueden manejarlos. Requiere herramientas y técnicas especiales para su análisis.</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">2. Desarrollo Web y Diseño 🌐</h4>
                            <p class="mb-2">Los fundamentos para crear sitios web y aplicaciones interactivas.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>HTML (HyperText Markup Language):</strong> Lenguaje para estructurar el contenido de las páginas web (títulos, párrafos, imágenes, enlaces).</li>
                                <li><strong>CSS (Cascading Style Sheets):</strong> Lenguaje para dar estilo a las páginas web (colores, fuentes, diseño, animaciones).</li>
                                <li><strong>JavaScript (JS):</strong> Lenguaje de programación que añade interactividad y dinamismo a las páginas web (formularios, animaciones, lógica del lado del cliente).</li>
                                <li><strong>Frameworks Web (Concepto):</strong> Colecciones de herramientas y bibliotecas preescritas que facilitan el desarrollo web (ej. React, Angular, Vue.js para frontend; Node.js, Django para backend).</li>
                            </ul>
                        </div>
                        <div class="bg-blue-50 p-6 rounded-lg mb-6 shadow-sm">
                            <h4 class="text-blue-800 text-xl font-semibold mb-3">3. Inteligencia Artificial y Ciencia de Datos 🤖📊</h4>
                            <p class="mb-2">Cómo las máquinas aprenden y se utilizan los datos para obtener información valiosa.</p>
                            <ul class="list-disc list-inside ml-4 mb-4">
                                <li><strong>Inteligencia Artificial (IA):</strong> Campo de la ciencia de la computación que busca crear máquinas que puedan realizar tareas que requieren inteligencia humana.</li>
                                <li><strong>Aprendizaje Automático (Machine Learning - ML):</strong> Un subcampo de la IA que permite a los sistemas aprender de los datos sin ser programados explícitamente.</li>
                                <li><strong>Análisis de Datos:</strong> Proceso de examinar, limpiar, transformar y modelar datos para descubrir información útil, informar conclusiones y apoyar la toma de decisiones.</li>
                                <li><strong>Ética de la IA:</strong> Consideraciones morales y sociales sobre el desarrollo y uso de la inteligencia artificial (sesgos, privacidad, impacto en el empleo).</li>
                            </ul>
                        </div>
                        <p class="mt-4">Estos temas te abrirán las puertas a los campos más innovadores de la tecnología. ¡El futuro está en tus manos! 🚀</p>
                    `,
                    questions: [
                        { id: 'computo_a_q1', text: '¿Qué es SQL?', type: 'text', answer: 'Lenguaje estándar para interactuar con bases de datos relacionales.' },
                        { id: 'computo_a_q2', text: '¿Qué es el Cloud Computing?', type: 'text', answer: 'Entrega de servicios informáticos a través de Internet.' },
                        { id: 'computo_a_q3', text: '¿Para qué se utiliza HTML en el desarrollo web?', type: 'text', answer: 'Estructurar el contenido de las páginas web.' },
                        { id: 'computo_a_q4', text: '¿Para qué se utiliza CSS en el desarrollo web?', type: 'text', answer: 'Dar estilo a las páginas web.' },
                        { id: 'computo_a_q5', text: '¿Qué lenguaje de programación añade interactividad a las páginas web?', type: 'text', answer: 'JavaScript' },
                        { id: 'computo_a_q6', text: '¿Qué es el Aprendizaje Automático (Machine Learning)?', type: 'text', answer: 'Un subcampo de la IA que permite a los sistemas aprender de los datos.' },
                        { id: 'computo_a_q7', text: '¿Qué es una base de datos?', type: 'text', answer: 'Sistema organizado para almacenar y gestionar datos.' },
                        { id: 'computo_a_q8', text: 'Menciona un ejemplo de servicio de Cloud Computing.', type: 'text', answer: 'Almacenamiento en la nube.' },
                        { id: 'computo_a_q9', text: '¿Qué significa la sigla "IA"?', type: 'text', answer: 'Inteligencia Artificial' },
                        { id: 'computo_a_q10', text: '¿Qué son los "Big Data"?', type: 'text', answer: 'Conjuntos de datos tan grandes y complejos que requieren herramientas especiales.' },
                        { id: 'computo_a_q11', text: 'Menciona un framework web para frontend.', type: 'text', answer: 'React' },
                        { id: 'computo_a_q12', text: '¿Qué es el Análisis de Datos?', type: 'text', answer: 'Proceso de examinar datos para descubrir información útil.' },
                        { id: 'computo_a_q13', text: '¿Cuál es una preocupación ética de la IA?', type: 'text', answer: 'Sesgos algorítmicos.' },
                        { id: 'computo_a_q14', text: '¿Qué tipo de base de datos se consulta con SQL?', type: 'text', answer: 'Relacionales' },
                        { id: 'computo_a_q15', text: '¿Qué permite hacer JavaScript en una página web?', type: 'text', answer: 'Añadir interactividad.' },
                        { id: 'computo_a_q16', text: '¿Qué es un framework web?', type: 'text', answer: 'Colección de herramientas y bibliotecas preescritas.' },
                        { id: 'computo_a_q17', text: '¿Qué es el backend en desarrollo web?', type: 'text', answer: 'La parte del sitio web que no es visible para el usuario, donde se procesa la lógica y los datos.' },
                        { id: 'computo_a_q18', text: '¿Qué es el frontend en desarrollo web?', type: 'text', answer: 'La parte del sitio web con la que el usuario interactúa directamente.' },
                        { id: 'computo_a_q19', text: '¿Qué es la ciencia de datos?', type: 'text', answer: 'Un campo interdisciplinario que utiliza métodos científicos, procesos, algoritmos y sistemas para extraer conocimiento o ideas de datos estructurados y no estructurados.' },
                        { id: 'computo_a_q20', text: 'Menciona un beneficio del Cloud Computing.', type: 'text', answer: 'Acceso a servicios desde cualquier lugar.' },
                    ]
                },
            },
        };

        const renderQuestionSection = (questions) => {
            userAnswers = {};
            score = 0;

            const questionSectionDiv = document.createElement('div');
            questionSectionDiv.className = "bg-blue-50 p-8 rounded-2xl text-gray-800 border-2 border-blue-200 text-lg leading-relaxed mt-8 shadow-inner";
            questionSectionDiv.innerHTML = `
                <h3 class="text-2xl font-bold text-blue-800 mb-6">Ejercicios - ¡Pon a prueba tus conocimientos! 📝</h3>
                <p class="mb-6 text-gray-700">Responde las siguientes preguntas.</p>
                <ol class="list-decimal list-inside ml-4 space-y-6" id="questions-list"></ol>
                <button id="check-answers-btn" class="mt-8 px-8 py-4 bg-blue-600 text-white rounded-full shadow-lg hover:bg-blue-700 transition-colors duration-300 text-xl font-bold block mx-auto transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-blue-300">
                    Verificar Respuestas ✨
                </button>
                <div id="results-display" class="mt-8 text-center text-2xl font-bold hidden"></div>
            `;

            const questionsList = questionSectionDiv.querySelector('#questions-list');
            questions.forEach((q, index) => {
                const listItem = document.createElement('li');
                listItem.className = "mb-4";
                listItem.innerHTML = `
                    <p class="font-semibold mb-2 text-gray-900">${index + 1}. ${q.text}</p>
                    <input
                        type="${q.type === 'number' ? 'number' : 'text'}"
                        id="q-${q.id}"
                        class="w-full p-3 rounded-lg border-2 focus:outline-none focus:ring-2 border-gray-300 focus:border-blue-500 focus:ring-blue-300 transition-all duration-200"
                        placeholder="Tu respuesta..."
                    />
                    <p id="feedback-${q.id}" class="mt-2 text-sm font-medium hidden"></p>
                `;
                questionsList.appendChild(listItem);

                const inputElement = listItem.querySelector(`#q-${q.id}`);
                inputElement.addEventListener('input', (e) => {
                    userAnswers[q.id] = e.target.value;
                    document.getElementById('results-display').classList.add('hidden');
                    questions.forEach(question => {
                        const feedbackElement = document.getElementById(`feedback-${question.id}`);
                        const inputField = document.getElementById(`q-${question.id}`);
                        if (feedbackElement) feedbackElement.classList.add('hidden');
                        if (inputField) {
                            inputField.classList.remove('border-green-500', 'focus:ring-green-300', 'border-red-500', 'focus:ring-red-300');
                            inputField.classList.add('border-gray-300', 'focus:border-blue-500', 'focus:ring-blue-300');
                        }
                    });
                });
            });

            questionSectionDiv.querySelector('#check-answers-btn').addEventListener('click', () => {
                let correctCount = 0;
                questions.forEach(q => {
                    const userAnswer = userAnswers[q.id];
                    let isCorrect = false;

                    if (q.type === 'number') {
                        isCorrect = Math.abs(parseFloat(userAnswer) - q.answer) < 0.001;
                    } else if (q.type === 'text') {
                        const correctAnswers = Array.isArray(q.answer) ? q.answer : [q.answer];
                        isCorrect = correctAnswers.some(ans => userAnswer && userAnswer.trim().toLowerCase() === ans.toLowerCase());
                    }

                    if (isCorrect) {
                        correctCount++;
                    }

                    const inputElement = document.getElementById(`q-${q.id}`);
                    const feedbackElement = document.getElementById(`feedback-${q.id}`);

                    inputElement.classList.remove('border-green-500', 'focus:ring-green-300', 'border-red-500', 'focus:ring-red-300', 'border-gray-300', 'focus:border-blue-500', 'focus:ring-blue-300');
                    if (isCorrect) {
                        inputElement.classList.add('border-blue-500', 'focus:ring-blue-300');
                        feedbackElement.textContent = '¡Correcto! ✅';
                        feedbackElement.classList.remove('text-red-600');
                        feedbackElement.classList.add('text-blue-600');
                    } else {
                        inputElement.classList.add('border-red-500', 'focus:ring-red-300');
                        feedbackElement.textContent = `Incorrecto. La respuesta correcta es: ${Array.isArray(q.answer) ? q.answer.join(' / ') : q.answer}`;
                        feedbackElement.classList.remove('text-blue-600');
                        feedbackElement.classList.add('text-red-600');
                    }
                    feedbackElement.classList.remove('hidden');
                });

                score = correctCount;

                const resultsDisplay = document.getElementById('results-display');
                resultsDisplay.innerHTML = `
                    <p>Tu puntuación: <span class="text-blue-700">${score}</span> de <span class="text-blue-700">${questions.length}</span></p>
                    ${score === questions.length ? '<p class="text-blue-600 mt-2">¡Excelente trabajo! 🎉</p>' : '<p class="text-orange-600 mt-2">¡Sigue practicando para mejorar! 💪</p>'}
                `;
                resultsDisplay.classList.remove('hidden');
            });

            return questionSectionDiv;
        };

        const renderCourseDetail = (course) => {
            const appDiv = document.getElementById('app');
            appDiv.innerHTML = '';

            // Handle external link for Minijuegos
            if (course.isExternal) {
                window.open(course.url, '_blank');
                selectedCourse = null; // Reset selected course to return to main page
                renderApp();
                return;
            }

            const currentLevelContent = courseContent[course.id][selectedLevel];

            const detailDiv = document.createElement('div');
            detailDiv.className = "bg-white p-10 rounded-3xl shadow-2xl w-full max-w-4xl border border-blue-100";

            const backButton = document.createElement('button');
            backButton.className = "mb-8 px-6 py-3 bg-gradient-to-r from-sky-500 to-blue-600 text-white rounded-full shadow-lg hover:shadow-xl hover:from-sky-600 hover:to-blue-700 transition-all duration-300 flex items-center text-lg font-semibold transform hover:scale-105 focus:outline-none focus:ring-4 focus:ring-sky-300";
            backButton.innerHTML = `
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 mr-3" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M12.707 5.293a1 1 0 010 1.414L9.414 10l3.293 3.293a1 1 0 01-1.414 1.414l-4-4a1 1 0 010-1.414l4-4a1 1 0 011.414 0z" clip-rule="evenodd" />
                </svg>
                Volver a Cursos
            `;
            backButton.addEventListener('click', () => {
                selectedCourse = null;
                renderApp();
            });
            detailDiv.appendChild(backButton);

            const title = document.createElement('h2');
            title.className = "text-5xl font-extrabold text-blue-900 mb-6 leading-tight";
            title.textContent = course.name;
            detailDiv.appendChild(title);

            const welcomeText = document.createElement('p');
            welcomeText.className = "text-gray-700 text-xl mb-8 leading-relaxed";
            welcomeText.textContent = `¡Bienvenido al curso de ${course.name}! Selecciona un nivel para comenzar tu aprendizaje.`;
            detailDiv.appendChild(welcomeText);

            const levelButtonsContainer = document.createElement('div');
            levelButtonsContainer.className = "mb-8 flex justify-center space-x-4";
            ['basico', 'intermedio', 'avanzado'].forEach(level => {
                const button = document.createElement('button');
                button.textContent = level.charAt(0).toUpperCase() + level.slice(1);
                button.className = `px-6 py-3 rounded-full text-lg font-semibold transition-all duration-200 transform hover:scale-105 focus:outline-none focus:ring-4
                    ${selectedLevel === level
                        ? 'bg-blue-600 text-white shadow-md focus:ring-blue-300'
                        : 'bg-gray-200 text-gray-700 hover:bg-gray-300 focus:ring-gray-300'
                    }`;
                button.addEventListener('click', () => {
                    selectedLevel = level;
                    renderCourseDetail(course);
                });
                levelButtonsContainer.appendChild(button);
            });
            detailDiv.appendChild(levelButtonsContainer);

            const theorySection = document.createElement('div');
            theorySection.className = "bg-blue-50 p-8 rounded-2xl text-gray-800 border-2 border-blue-200 text-lg leading-relaxed theory-content shadow-inner";
            theorySection.innerHTML = `
                <h3 class="text-2xl font-bold text-blue-800 mb-4">Teoría del Curso - Nivel ${selectedLevel.charAt(0).toUpperCase() + selectedLevel.slice(1)}</h3>
                ${currentLevelContent.theory}
            `;
            detailDiv.appendChild(theorySection);

            if (currentLevelContent.questions && currentLevelContent.questions.length > 0) {
                detailDiv.appendChild(renderQuestionSection(currentLevelContent.questions));
            }

            appDiv.appendChild(detailDiv);
        };

        const renderCourseCards = () => {
            const appDiv = document.getElementById('app');
            appDiv.innerHTML = `
                <h1 class="text-6xl font-extrabold text-center text-gray-900 mb-16 drop-shadow-lg leading-tight">
                    <span class="text-blue-600">Chisfuerzo:</span> <span class="text-sky-600">Aprende</span> y <span class="text-indigo-600">Crece</span>
                </h1>
                <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-10" id="course-cards-container"></div>
            `;
            const courseCardsContainer = document.getElementById('course-cards-container');

            courses.forEach(course => {
                const cardDiv = document.createElement('div');
                cardDiv.className = "bg-white p-8 rounded-3xl shadow-xl hover:shadow-2xl transition-all duration-300 cursor-pointer transform hover:-translate-y-2 flex flex-col items-center text-center border-2 border-blue-100 hover:border-blue-300";
                cardDiv.innerHTML = `
                    <div class="text-6xl mb-5 text-blue-700">
                        ${course.id === 'matematicas' ? '➕' : ''}
                        ${course.id === 'comunicacion' ? '🗣️' : ''}
                        ${course.id === 'cyt' ? '🔬' : ''}
                        ${course.id === 'ingles' ? '🇬🇧' : ''}
                        ${course.id === 'arte' ? '🎨' : ''}
                        ${course.id === 'computo' ? '💻' : ''}
                        ${course.id === 'minijuegos' ? '🎮' : ''}
                    </div>
                    <h3 class="text-3xl font-bold text-gray-900 mb-3">${course.name}</h3>
                    <p class="text-gray-600 text-base leading-relaxed">${course.description}</p>
                `;
                cardDiv.addEventListener('click', () => {
                    selectedCourse = course;
                    selectedLevel = 'basico'; // Default to basic, but for external links it won't matter
                    renderApp();
                });
                courseCardsContainer.appendChild(cardDiv);
            });
        };

        const renderApp = () => {
            if (selectedCourse) {
                renderCourseDetail(selectedCourse);
            } else {
                renderCourseCards();
            }
        };

        document.addEventListener('DOMContentLoaded', async () => {
            await initFirebase();
            renderApp();
        });
    </script>
</body>
</html>
�
