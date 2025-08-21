# ControlGasolina
Control de Gasolina
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Administrador de Gasolina</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #e6f1fa; /* Un azul claro */
            display: flex;
            justify-content: center;
            align-items: flex-start; /* Alinea al inicio para permitir scroll */
            min-height: 100vh;
            padding: 2rem 1rem;
        }
        .container {
            max-width: 700px;
            width: 100%;
            background-color: #ffffff;
            border-radius: 1.5rem; /* Bordes más redondeados */
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1); /* Sombra más pronunciada */
            padding: 2.5rem; /* Mayor padding */
            animation: fadeIn 0.5s ease-out; /* Animación de entrada */
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .custom-scrollbar::-webkit-scrollbar {
            width: 8px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: #f1f1f1;
            border-radius: 10px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background: #a8dadc; /* Un azul verdoso suave */
            border-radius: 10px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover {
            background: #81b29a; /* Un verde más oscuro */
        }
        .message-box {
            position: fixed;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            background-color: #4CAF50; /* Green */
            color: white;
            padding: 10px 20px;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
            z-index: 1000;
            opacity: 0;
            transition: opacity 0.5s ease-in-out;
        }
        .message-box.show {
            opacity: 1;
        }
        .message-box.error {
            background-color: #f44336; /* Red */
        }
        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            border-left-color: #ffffff;
            border-radius: 50%;
            width: 24px;
            height: 24px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="selection:bg-blue-200">
    <div id="messageBox" class="message-box"></div>

    <div class="container flex flex-col items-center">
        <h1 class="text-4xl font-bold text-gray-800 mb-8 text-center">Administración de Gasolina</h1>

        <!-- Sección de entrada de datos de carga de gasolina -->
        <div class="w-full mb-8">
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Nueva Carga</h2>
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 mb-4">
                <input
                    type="date"
                    id="dateInput"
                    class="p-4 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 text-lg shadow-sm"
                />
                <input
                    type="number"
                    id="litersInput"
                    placeholder="Litros (ej. 45.50)"
                    class="p-4 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 text-lg shadow-sm"
                    step="0.01"
                />
                <input
                    type="number"
                    id="costInput"
                    placeholder="Costo total (ej. 950.00)"
                    class="p-4 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 text-lg shadow-sm"
                    step="0.01"
                />
                <input
                    type="number"
                    id="odometerInput"
                    placeholder="Lectura de Odómetro (opcional)"
                    class="p-4 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-blue-500 text-lg shadow-sm"
                    step="1"
                />
            </div>
            <button
                id="addFuelingBtn"
                class="w-full px-8 py-4 bg-green-600 text-white font-semibold rounded-xl hover:bg-green-700 transition duration-300 ease-in-out shadow-lg transform hover:scale-105 active:scale-95 text-lg"
            >
                Añadir Carga
            </button>
            <div id="errorMessage" class="text-red-600 text-sm mt-2 hidden">Por favor, introduce una fecha, litros y costo válidos.</div>
        </div>

        <!-- Sección de lista de cargas -->
        <div class="w-full mb-8">
            <h2 class="text-2xl font-semibold text-gray-700 mb-4">Historial de Cargas</h2>
            <p id="loadingDataMessage" class="text-gray-600 text-center flex items-center justify-center gap-2 mb-4 hidden">
                <span class="spinner"></span> Cargando datos...
            </p>
            <ul id="fuelingsList" class="bg-gray-50 p-4 rounded-xl shadow-inner max-h-72 overflow-y-auto custom-scrollbar">
                <!-- Las cargas se añadirán aquí dinámicamente -->
                <li id="noFuelingsMessage" class="text-gray-500 text-center py-4">No hay cargas registradas aún.</li>
            </ul>
        </div>

        <!-- Sección de resumen -->
        <div class="w-full grid grid-cols-1 md:grid-cols-2 gap-4 mb-8">
            <div class="p-6 bg-blue-100 rounded-xl shadow-md flex justify-between items-center">
                <span class="text-xl font-bold text-blue-800">Costo Total:</span>
                <span id="totalCost" class="text-2xl font-extrabold text-blue-900">$0.00</span>
            </div>
            <div class="p-6 bg-green-100 rounded-xl shadow-md flex justify-between items-center">
                <span class="text-xl font-bold text-green-800">Costo Promedio/Litro:</span>
                <span id="avgCostPerLiter" class="text-2xl font-extrabold text-green-900">$0.00</span>
            </div>
        </div>

        <!-- Sección de Gemini API -->
        <div class="w-full">
            <button
                id="getTipsBtn"
                class="w-full px-8 py-4 bg-purple-600 text-white font-semibold rounded-xl hover:bg-purple-700 transition duration-300 ease-in-out shadow-lg transform hover:scale-105 active:scale-95 text-lg flex items-center justify-center gap-2"
            >
                <span class="text-2xl">✨</span> Obtener Consejos de Eficiencia <span class="text-2xl">✨</span>
            </button>
            <div id="tipsOutput" class="mt-6 p-6 bg-white rounded-xl shadow-inner hidden">
                <p id="loadingMessage" class="text-gray-600 text-center hidden">Generando consejos...</p>
                <div id="geminiTipsContent" class="prose max-w-none">
                    <!-- Los consejos de Gemini se insertarán aquí -->
                </div>
            </div>
        </div>
        <div class="w-full text-center text-gray-500 text-sm mt-4">
            ID de Usuario: <span id="userIdDisplay">Cargando...</span>
        </div>
    </div>

    <script type="module">
        // Firebase Imports
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, addDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, where, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // DOM Elements
        const litersInput = document.getElementById('litersInput');
        const costInput = document.getElementById('costInput');
        const odometerInput = document.getElementById('odometerInput');
        const dateInput = document.getElementById('dateInput');
        const addFuelingBtn = document.getElementById('addFuelingBtn');
        const fuelingsList = document.getElementById('fuelingsList');
        const totalCostSpan = document.getElementById('totalCost');
        const avgCostPerLiterSpan = document.getElementById('avgCostPerLiter');
        const errorMessage = document.getElementById('errorMessage');
        const noFuelingsMessage = document.getElementById('noFuelingsMessage');
        const getTipsBtn = document.getElementById('getTipsBtn');
        const tipsOutput = document.getElementById('tipsOutput');
        const loadingMessage = document.getElementById('loadingMessage');
        const geminiTipsContent = document.getElementById('geminiTipsContent');
        const loadingDataMessage = document.getElementById('loadingDataMessage');
        const userIdDisplay = document.getElementById('userIdDisplay');
        const messageBox = document.getElementById('messageBox');


        let fuelings = []; // Almacena objetos { id, date, liters, cost, costPerLiter, odometer, kmPerLiterSinceLastFueling }

        // Firebase variables
        let app;
        let db;
        let auth;
        let userId;
        let isAuthReady = false;

        /**
         * Muestra un mensaje temporal al usuario.
         * @param {string} message - El mensaje a mostrar.
         * @param {boolean} isError - True si es un mensaje de error, false para éxito/información.
         */
        function showMessage(message, isError = false) {
            messageBox.textContent = message;
            messageBox.classList.remove('error', 'show');
            if (isError) {
                messageBox.classList.add('error');
            }
            messageBox.classList.add('show');
            setTimeout(() => {
                messageBox.classList.remove('show');
            }, 3000); // El mensaje desaparece después de 3 segundos
        }

        /**
         * Inicializa Firebase y autentica al usuario.
         */
        async function initializeFirebase() {
            try {
                // Global variables from Canvas environment
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
                const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
                const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

                if (!firebaseConfig || Object.keys(firebaseConfig).length === 0) {
                    console.error("Firebase config no está disponible. No se puede inicializar Firebase.");
                    showMessage("Error: Configuración de Firebase no disponible.", true);
                    return;
                }

                app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                loadingDataMessage.classList.remove('hidden'); // Mostrar spinner al cargar datos

                // Autenticar al usuario
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }

                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        userId = user.uid;
                        userIdDisplay.textContent = userId;
                        isAuthReady = true;
                        console.log("Usuario autenticado. ID:", userId);
                        setupFirestoreListener(); // Configurar listener una vez autenticado
                    } else {
                        console.log("No hay usuario autenticado.");
                        userId = crypto.randomUUID(); // Fallback for anonymous
                        userIdDisplay.textContent = userId + " (Anónimo)";
                        isAuthReady = true;
                        setupFirestoreListener(); // Configurar listener incluso si es anónimo
                    }
                });

            } catch (error) {
                console.error("Error al inicializar Firebase o autenticar:", error);
                showMessage("Error al cargar la aplicación. Inténtalo de nuevo.", true);
                loadingDataMessage.classList.add('hidden');
            }
        }

        /**
         * Configura el listener de Firestore para actualizaciones en tiempo real.
         */
        function setupFirestoreListener() {
            if (!isAuthReady || !db || !userId) {
                console.warn("Firestore o usuario no listos para configurar el listener.");
                return;
            }

            // Path para datos públicos
            const fuelingsCollectionRef = collection(db, `artifacts/${typeof __app_id !== 'undefined' ? __app_id : 'default-app-id'}/public/data/fuelings`);

            onSnapshot(fuelingsCollectionRef, (snapshot) => {
                const newFuelings = [];
                snapshot.forEach(doc => {
                    newFuelings.push({ id: doc.id, ...doc.data() });
                });
                fuelings = newFuelings;
                updateDisplay();
                loadingDataMessage.classList.add('hidden'); // Ocultar spinner una vez cargados los datos
                showMessage("Datos de gasolina actualizados.", false);
            }, (error) => {
                console.error("Error al escuchar cambios en Firestore:", error);
                showMessage("Error al cargar datos. Ver consola para más detalles.", true);
                loadingDataMessage.classList.add('hidden');
            });
        }

        /**
         * Actualiza el display del resumen y la lista de cargas de gasolina.
         */
        function updateDisplay() {
            let totalLiters = 0;
            let totalCost = 0;

            // Ordena las cargas por fecha ascendente para calcular km/L correctamente
            const sortedFuelings = [...fuelings].sort((a, b) => new Date(a.date) - new Date(b.date));

            // Calcula km/L para cada carga basada en la anterior
            for (let i = 0; i < sortedFuelings.length; i++) {
                if (i > 0 && sortedFuelings[i].odometer !== null && sortedFuelings[i-1].odometer !== null) {
                    const distance = sortedFuelings[i].odometer - sortedFuelings[i-1].odometer;
                    if (distance > 0 && sortedFuelings[i].liters > 0) {
                        sortedFuelings[i].kmPerLiterSinceLastFueling = distance / sortedFuelings[i].liters;
                    } else {
                        sortedFuelings[i].kmPerLiterSinceLastFueling = null;
                    }
                } else {
                    sortedFuelings[i].kmPerLiterSinceLastFueling = null;
                }
            }
            // Ahora re-asignamos fuelings para mantener el orden ascendente por fecha,
            // ya que el historial se mostrará en orden inverso (más reciente primero)
            fuelings = sortedFuelings; // Esto asegura que kmPerLiterSinceLastFueling esté configurado en el array `fuelings`.


            // Calcular totales para el resumen (no afectados por km/L)
            fuelings.forEach(fueling => {
                totalLiters += fueling.liters;
                totalCost += fueling.cost;
            });

            // Actualiza el resumen
            totalCostSpan.textContent = `$${totalCost.toFixed(2)}`;
            if (totalLiters > 0) {
                avgCostPerLiterSpan.textContent = `$${(totalCost / totalLiters).toFixed(2)}`;
            } else {
                avgCostPerLiterSpan.textContent = `$0.00`;
            }

            // Borra la lista actual
            fuelingsList.innerHTML = '';

            if (fuelings.length === 0) {
                noFuelingsMessage.classList.remove('hidden');
            } else {
                noFuelingsMessage.classList.add('hidden');
                // Añade cada carga a la lista, mostrando las más recientes primero
                for (let i = fuelings.length - 1; i >= 0; i--) {
                    const fueling = fuelings[i];
                    const listItem = document.createElement('li');
                    listItem.className = 'flex flex-col sm:flex-row justify-between items-start sm:items-center py-3 px-4 mb-2 bg-white rounded-lg shadow-sm last:mb-0 transition-all duration-200 ease-in-out hover:bg-gray-100';

                    let kmPerLiterText = '';
                    if (fueling.kmPerLiterSinceLastFueling !== null) {
                        kmPerLiterText = `<span class="text-yellow-700 ml-2">(${fueling.kmPerLiterSinceLastFueling.toFixed(2)} km/L)</span>`;
                    }

                    listItem.innerHTML = `
                        <div class="flex-grow">
                            <span class="font-semibold text-gray-800">${fueling.date}</span>:
                            <span class="text-blue-700">${fueling.liters.toFixed(2)} L</span>
                            <span class="text-green-700 ml-2">$${fueling.cost.toFixed(2)}</span>
                            <span class="text-gray-500 text-sm ml-2">($${fueling.costPerLiter.toFixed(2)}/L)</span>
                            ${fueling.odometer ? `<span class="text-gray-500 text-sm ml-2">(${fueling.odometer} km)</span>` : ''}
                            ${kmPerLiterText}
                        </div>
                        <button data-id="${fueling.id}" class="delete-btn text-red-500 hover:text-red-700 text-sm font-semibold mt-2 sm:mt-0 transition-colors duration-200">
                            Eliminar
                        </button>
                    `;
                    fuelingsList.appendChild(listItem);
                }

                // Añade event listeners a los nuevos botones de eliminar
                document.querySelectorAll('.delete-btn').forEach(button => {
                    button.addEventListener('click', deleteFueling);
                });
            }
        }

        /**
         * Añade una nueva carga de gasolina a la lista y la guarda en Firestore.
         */
        async function addFueling() {
            if (!isAuthReady || !db) {
                showMessage("La aplicación no está lista aún. Inténtalo en un momento.", true);
                return;
            }

            const litersText = litersInput.value.trim();
            const costText = costInput.value.trim();
            const odometerText = odometerInput.value.trim();
            const date = dateInput.value;

            const liters = parseFloat(litersText);
            const cost = parseFloat(costText);
            const odometer = odometerText === '' ? null : parseInt(odometerText);

            // Validación de la entrada
            if (isNaN(liters) || liters <= 0 || isNaN(cost) || cost <= 0 || date === "") {
                errorMessage.classList.remove('hidden');
                litersInput.classList.add('border-red-500');
                costInput.classList.add('border-red-500');
                dateInput.classList.add('border-red-500');
                return;
            }

            errorMessage.classList.add('hidden');
            litersInput.classList.remove('border-red-500');
            costInput.classList.remove('border-red-500');
            dateInput.classList.remove('border-red-500');

            const costPerLiter = cost / liters;
            const newFuelingData = {
                date: date,
                liters: liters,
                cost: cost,
                costPerLiter: parseFloat(costPerLiter.toFixed(2)), // Redondear para guardar
                odometer: odometer,
                kmPerLiterSinceLastFueling: null // Se calculará al recuperar y mostrar
            };

            try {
                // Path para datos públicos
                const fuelingsCollectionRef = collection(db, `artifacts/${typeof __app_id !== 'undefined' ? __app_id : 'default-app-id'}/public/data/fuelings`);
                await addDoc(fuelingsCollectionRef, newFuelingData);
                showMessage("Carga de gasolina añadida y guardada.", false);

                // Limpia los inputs
                litersInput.value = '';
                costInput.value = '';
                odometerInput.value = '';
                dateInput.valueAsDate = new Date(); // Reset to current date

            } catch (error) {
                console.error("Error al añadir carga a Firestore:", error);
                showMessage("Error al guardar la carga. Inténtalo de nuevo.", true);
            }
        }

        /**
         * Elimina una carga de gasolina de la lista y de Firestore por su ID.
         * @param {Event} event - El evento de click del botón.
         */
        async function deleteFueling(event) {
            if (!isAuthReady || !db) {
                showMessage("La aplicación no está lista aún. Inténtalo en un momento.", true);
                return;
            }
            const idToDelete = event.target.dataset.id;
            try {
                // Path para datos públicos
                const docRef = doc(db, `artifacts/${typeof __app_id !== 'undefined' ? __app_id : 'default-app-id'}/public/data/fuelings`, idToDelete);
                await deleteDoc(docRef);
                showMessage("Carga eliminada correctamente.", false);
            } catch (error) {
                console.error("Error al eliminar carga de Firestore:", error);
                showMessage("Error al eliminar la carga. Inténtalo de nuevo.", true);
            }
        }

        /**
         * Llama a la API de Gemini para obtener consejos de eficiencia.
         */
        async function getFuelEfficiencyTips() {
            if (fuelings.length === 0) {
                showMessage("No hay datos de cargas para analizar. Añade al menos una carga.", true);
                return;
            }

            tipsOutput.classList.remove('hidden');
            loadingMessage.classList.remove('hidden');
            geminiTipsContent.innerHTML = '';
            getTipsBtn.disabled = true; // Deshabilita el botón mientras carga

            // Filtra los datos relevantes para el LLM (excluyendo IDs para no sobrecargar el prompt)
            const dataForLLM = fuelings.map(f => ({
                date: f.date,
                liters: f.liters,
                cost: f.cost,
                odometer: f.odometer,
                kmPerLiterSinceLastFueling: f.kmPerLiterSinceLastFueling
            }));

            let prompt = `Eres un asistente experto en automoción y finanzas personales. Analiza los siguientes datos de cargas de gasolina de un vehículo y proporciona consejos personalizados para mejorar la eficiencia del combustible y ahorrar dinero. Considera los datos de litros, costo, lectura de odómetro y especialmente el rendimiento (km/L) entre cargas (si está disponible). Si hay patrones inusuales o fluctuaciones en la eficiencia o el costo, coméntalos. Si no hay suficientes datos para un análisis profundo (se necesitan al menos dos cargas con lecturas de odómetro consecutivas para el km/L), ofrece consejos generales pero útiles para la gestión de gasolina.
            Datos de cargas de gasolina: ${JSON.stringify(dataForLLM)}`;

            let chatHistory = [];
            chatHistory.push({ role: "user", parts: [{ text: prompt }] });
            const payload = { contents: chatHistory };
            const apiKey = ""; // Canvas will provide this at runtime
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

            let responseText = "No se pudieron obtener los consejos. Inténtalo de nuevo más tarde.";

            // Implementación de reintento exponencial (Exponential Backoff)
            const MAX_RETRIES = 5;
            let retryCount = 0;
            let delay = 1000; // 1 second

            while (retryCount < MAX_RETRIES) {
                try {
                    const response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (response.ok) {
                        const result = await response.json();
                        if (result.candidates && result.candidates.length > 0 &&
                            result.candidates[0].content && result.candidates[0].content.parts &&
                            result.candidates[0].content.parts.length > 0) {
                            responseText = result.candidates[0].content.parts[0].text;
                        } else {
                            responseText = "No se recibieron consejos válidos de la IA.";
                        }
                        break; // Salir del bucle si la llamada fue exitosa
                    } else if (response.status === 429) { // Too Many Requests
                        console.warn(`Demasiadas solicitudes (429). Reintentando en ${delay / 1000} segundos...`);
                        await new Promise(res => setTimeout(res, delay));
                        delay *= 2; // Duplicar el retraso
                        retryCount++;
                    } else {
                        const errorData = await response.json();
                        console.error("Error en la llamada a la API de Gemini:", response.status, errorData);
                        responseText = `Error: ${response.status} - ${errorData.error?.message || "Error desconocido"}`;
                        break; // Salir del bucle para otros errores que no sean 429
                    }
                } catch (error) {
                    console.error("Error de red o CORS al llamar a la API de Gemini:", error);
                    responseText = "Error de conexión. Asegúrate de tener conexión a internet.";
                    break; // Salir del bucle en caso de error de red
                }
            }

            loadingMessage.classList.add('hidden');
            geminiTipsContent.innerHTML = `<p>${responseText.replace(/\n/g, '<br>')}</p>`; // Mostrar saltos de línea
            getTipsBtn.disabled = false; // Habilita el botón de nuevo
        }


        // Event Listeners
        addFuelingBtn.addEventListener('click', addFueling);
        litersInput.addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                addFueling();
            }
        });
        costInput.addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                addFueling();
            }
        });
        odometerInput.addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                addFueling();
            }
        });
        getTipsBtn.addEventListener('click', getFuelEfficiencyTips);

        // Establece la fecha actual por defecto en el campo de fecha
        dateInput.valueAsDate = new Date();

        // Inicializa Firebase al cargar la página
        initializeFirebase();
    </script>
</body>
</html>
