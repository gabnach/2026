<!DOCTYPE html>
<html lang="es" class="h-full">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>VocalScribe Pro - Dynamic Voice Transcriptor</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome Icons CDN -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">
    
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    }
                }
            }
        }
    </script>
    <style>
        @keyframes pulse-ring {
            0% { transform: scale(0.95); opacity: 0.8; }
            50% { transform: scale(1.18); opacity: 0.3; }
            100% { transform: scale(0.95); opacity: 0.8; }
        }
        .recording-pulse {
            animation: pulse-ring 1.8s infinite ease-in-out;
        }
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: transparent;
        }
        ::-webkit-scrollbar-thumb {
            background: rgba(148, 163, 184, 0.3);
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: rgba(148, 163, 184, 0.5);
        }
        .phrase-span {
            display: inline;
            line-height: 1.4;
            transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
            word-wrap: break-word;
        }
        [contenteditable]:empty:before {
            content: attr(data-placeholder);
            color: #64748b;
            font-style: italic;
        }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-full flex flex-col font-sans transition-colors duration-300">

    <header class="border-b border-slate-800/80 bg-slate-900/80 backdrop-blur-md sticky top-0 z-20">
        <div class="max-w-6xl mx-auto px-4 py-4 flex flex-col sm:flex-row sm:items-center justify-between gap-4">
            <div class="flex items-center space-x-3">
                <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-cyan-500 via-indigo-500 to-purple-600 flex items-center justify-center shadow-lg shadow-cyan-500/20">
                    <i class="fa-solid fa-microphone-lines text-white text-xl"></i>
                </div>
                <div>
                    <h1 class="text-xl font-bold bg-gradient-to-r from-white via-cyan-200 to-indigo-300 bg-clip-text text-transparent">VocalScribe Pro</h1>
                    <p class="text-xs text-slate-400">Transcripción con fuente dinámica por volumen y color por frase</p>
                </div>
            </div>
            
            <div class="flex items-center space-x-3">
                <!-- Language Selector -->
                <div class="relative">
                    <select id="languageSelect" class="bg-slate-800 hover:bg-slate-750 text-slate-200 text-sm font-medium rounded-xl px-3.5 py-2 pr-9 border border-slate-700/80 focus:outline-none focus:ring-2 focus:ring-cyan-500 appearance-none cursor-pointer transition">
                        <option value="es-ES" selected>🇪🇸 Español (España)</option>
                        <option value="es-MX">🇲🇽 Español (México)</option>
                        <option value="es-AR">🇦🇷 Español (Argentina)</option>
                        <option value="es-CO">🇨🇴 Español (Colombia)</option>
                        <option value="es-CL">🇨🇱 Español (Chile)</option>
                        <option value="en-US">🇺🇸 English (US)</option>
                        <option value="en-GB">🇬🇧 English (UK)</option>
                        <option value="pt-BR">🇧🇷 Português (Brasil)</option>
                        <option value="fr-FR">🇫🇷 Français</option>
                        <option value="de-DE">🇩🇪 Deutsch</option>
                        <option value="it-IT">🇮🇹 Italiano</option>
                    </select>
                    <i class="fa-solid fa-chevron-down absolute right-3 top-3 text-xs text-slate-400 pointer-events-none"></i>
                </div>
            </div>
        </div>
    </header>

    <main class="flex-1 max-w-6xl w-full mx-auto px-4 py-6 flex flex-col space-y-6">

        <!-- Browser Compatibility Alert -->
        <div id="compatAlert" class="hidden bg-amber-500/10 border border-amber-500/30 rounded-2xl p-4 text-amber-300 text-sm flex items-center space-x-3">
            <i class="fa-solid fa-triangle-exclamation text-lg"></i>
            <span><strong>Navegador No Soportado:</strong> Tu navegador no permite acceso a SpeechRecognition o AudioContext. Te recomendamos utilizar Google Chrome o Microsoft Edge.</span>
        </div>

        <!-- Notification Toast -->
        <div id="toast" class="hidden fixed bottom-6 right-6 z-50 bg-slate-800 border border-slate-700 text-slate-200 px-4 py-3 rounded-2xl shadow-2xl flex items-center space-x-3 transition-all transform duration-300">
            <i id="toastIcon" class="fa-solid fa-circle-check text-emerald-400"></i>
            <span id="toastMessage" class="text-sm font-medium">Acción realizada</span>
        </div>

        <!-- Recording Controls & VU Meter Gauge -->
        <div class="bg-slate-900/60 border border-slate-800/80 rounded-2xl p-6 flex flex-col md:flex-row items-center justify-between gap-6 shadow-xl backdrop-blur-md">
            <div class="flex items-center space-x-4 w-full md:w-auto">
                <div class="relative flex justify-center items-center flex-shrink-0">
                    <div id="pulseBg" class="hidden absolute w-20 h-20 rounded-full bg-red-500/30 recording-pulse"></div>
                    <button id="recordBtn" class="relative z-10 w-16 h-16 rounded-full bg-cyan-600 hover:bg-cyan-500 text-white flex items-center justify-center text-2xl shadow-lg shadow-cyan-600/30 transition-all duration-300 active:scale-95 focus:outline-none focus:ring-4 focus:ring-cyan-500/40">
                        <i id="recordIcon" class="fa-solid fa-microphone"></i>
                    </button>
                </div>
                <div>
                    <div class="flex items-center space-x-2">
                        <span id="statusBadge" class="inline-flex items-center px-3 py-1 rounded-full text-xs font-semibold bg-slate-800 text-slate-300 border border-slate-700">
                            Listo para grabar
                        </span>
                    </div>
                    <p id="statusHint" class="text-xs text-slate-400 mt-1.5">Haz clic en el micrófono para iniciar el dictado por voz</p>
                </div>
            </div>

            <!-- Volume Analysis Gauge & Color Palette Display -->
            <div class="flex flex-col sm:flex-row items-center gap-6 w-full md:w-auto bg-slate-950/70 p-4 rounded-xl border border-slate-800/80">
                <!-- Audio Volume Meter -->
                <div class="w-full sm:w-52 flex flex-col space-y-1.5">
                    <div class="flex justify-between text-xs font-medium text-slate-400">
                        <span>Intensidad de Voz:</span>
                        <span id="volPercent" class="text-cyan-400 font-mono font-bold">0%</span>
                    </div>
                    <div class="w-full h-3 bg-slate-800 rounded-full overflow-hidden border border-slate-700/60">
                        <div id="volumeBar" class="h-full w-0 bg-gradient-to-r from-emerald-500 via-amber-400 to-rose-500 transition-all duration-75"></div>
                    </div>
                </div>

                <!-- Current Active Color Tag -->
                <div class="flex items-center space-x-3 w-full sm:w-auto border-t sm:border-t-0 sm:border-l border-slate-800 pt-3 sm:pt-0 sm:pl-6">
                    <div class="flex flex-col">
                        <span class="text-xs text-slate-400 font-medium">Color de Frase:</span>
                        <div class="flex items-center space-x-2 mt-1">
                            <span id="activeColorDot" class="w-4 h-4 rounded-full bg-cyan-400 shadow-sm border border-slate-700"></span>
                            <span id="activeColorLabel" class="text-xs font-bold text-slate-200">Cyan</span>
                        </div>
                    </div>
                    <button id="nextColorBtn" title="Cambiar color manualmente" class="px-2.5 py-1.5 bg-slate-800 hover:bg-slate-700 text-xs rounded-lg border border-slate-700 text-slate-300 transition active:scale-95">
                        <i class="fa-solid fa-palette"></i>
                    </button>
                </div>
            </div>
        </div>

        <div class="flex-1 flex flex-col bg-slate-900/40 border border-slate-800/80 rounded-2xl overflow-hidden shadow-2xl">
            <!-- Toolbar -->
            <div class="bg-slate-900/90 px-4 py-3 border-b border-slate-800 flex flex-wrap items-center justify-between gap-3">
                <div class="flex items-center space-x-4 text-xs font-medium text-slate-400">
                    <div>Palabras: <span id="wordCount" class="text-cyan-400 font-semibold">0</span></div>
                    <div>Caracteres: <span id="charCount" class="text-cyan-400 font-semibold">0</span></div>
                    <div>Frases: <span id="phraseCount" class="text-indigo-400 font-semibold">0</span></div>
                </div>

                <div class="flex items-center space-x-2">
                    <button id="copyBtn" title="Copiar texto al portapapeles" class="px-3 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-200 text-xs font-medium transition flex items-center space-x-1.5 border border-slate-700 active:scale-95">
                        <i class="fa-regular fa-copy"></i>
                        <span>Copiar</span>
                    </button>
                    <button id="downloadBtn" title="Descargar como archivo TXT" class="px-3 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-200 text-xs font-medium transition flex items-center space-x-1.5 border border-slate-700 active:scale-95">
                        <i class="fa-solid fa-download"></i>
                        <span>Descargar</span>
                    </button>
                    <button id="clearBtn" title="Borrar todo el contenido" class="px-3 py-1.5 rounded-lg bg-red-500/10 hover:bg-red-500/20 text-red-400 text-xs font-medium transition flex items-center space-x-1.5 border border-red-500/20 active:scale-95">
                        <i class="fa-solid fa-trash-can"></i>
                        <span>Limpiar</span>
                    </button>
                </div>
            </div>

            <!-- Dynamic Transcript Display Canvas -->
            <div class="relative flex-1 min-h-[360px] p-6 flex flex-col bg-slate-950/40">
                <div id="transcriptBox" 
                    contenteditable="true"
                    data-placeholder="El texto dictado aparecerá aquí. Si hablas fuerte la letra crecerá, si hablas suave se encogerá..." 
                    class="w-full h-full min-h-[320px] bg-transparent focus:outline-none text-slate-100 leading-relaxed font-normal p-2 rounded-lg"></div>

                <!-- Live Interim Speech Box -->
                <div id="interimContainer" class="mt-4 p-3.5 bg-slate-900/80 border border-dashed border-slate-700/80 rounded-xl hidden flex items-baseline space-x-2">
                    <span class="text-xs text-slate-400 uppercase tracking-wider font-semibold flex-shrink-0">Escuchando:</span>
                    <span id="interimSpan" class="italic transition-all"></span>
                </div>
            </div>
        </div>

        <!-- Legend & Instructions -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4 text-xs text-slate-400 bg-slate-900/30 p-4 rounded-xl border border-slate-800/60">
            <div class="flex items-start space-x-2.5">
                <i class="fa-solid fa-text-height text-cyan-400 text-sm mt-0.5"></i>
                <p><strong class="text-slate-200">Fuente Dinámica por Volumen:</strong> El volumen detectado en tiempo real calcula el tamaño exacto de cada frase (desde 14px hasta más de 42px).</p>
            </div>
            <div class="flex items-start space-x-2.5">
                <i class="fa-solid fa-palette text-purple-400 text-sm mt-0.5"></i>
                <p><strong class="text-slate-200">Color por Frase/Voz:</strong> Cada intervención o pausa rotará a un nuevo color vibrante para estructurar mejor la transcripción.</p>
            </div>
        </div>
    </main>

    <footer class="border-t border-slate-800/60 py-4 text-center text-xs text-slate-500">
        <p>VocalScribe Pro &copy; Speech Recognition & Web Audio Engine</p>
    </footer>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // UI Element References
            const recordBtn = document.getElementById('recordBtn');
            const recordIcon = document.getElementById('recordIcon');
            const pulseBg = document.getElementById('pulseBg');
            const statusBadge = document.getElementById('statusBadge');
            const statusHint = document.getElementById('statusHint');
            const transcriptBox = document.getElementById('transcriptBox');
            const interimContainer = document.getElementById('interimContainer');
            const interimSpan = document.getElementById('interimSpan');
            const languageSelect = document.getElementById('languageSelect');
            const wordCount = document.getElementById('wordCount');
            const charCount = document.getElementById('charCount');
            const phraseCount = document.getElementById('phraseCount');
            const copyBtn = document.getElementById('copyBtn');
            const downloadBtn = document.getElementById('downloadBtn');
            const clearBtn = document.getElementById('clearBtn');
            const compatAlert = document.getElementById('compatAlert');
            const toast = document.getElementById('toast');
            const toastMessage = document.getElementById('toastMessage');
            const toastIcon = document.getElementById('toastIcon');

            // Audio Analysis UI References
            const volumeBar = document.getElementById('volumeBar');
            const volPercent = document.getElementById('volPercent');
            const activeColorDot = document.getElementById('activeColorDot');
            const activeColorLabel = document.getElementById('activeColorLabel');
            const nextColorBtn = document.getElementById('nextColorBtn');

            // Color Palette Definitions for Speech Phrases
            const colorPalette = [
                { name: 'Cyan Neón', class: 'text-cyan-400', hex: '#22d3ee' },
                { name: 'Verde Esmeralda', class: 'text-emerald-400', hex: '#34d399' },
                { name: 'Amarillo Ámbar', class: 'text-amber-300', hex: '#fcd34d' },
                { name: 'Púrpura Neón', class: 'text-purple-400', hex: '#c084fc' },
                { name: 'Rosa Neón', class: 'text-rose-400', hex: '#fb7185' },
                { name: 'Índigo Brillante', class: 'text-indigo-400', hex: '#818cf8' },
                { name: 'Azul Cielo', class: 'text-sky-300', hex: '#7dd3fc' },
                { name: 'Fucsia Neón', class: 'text-fuchsia-400', hex: '#e879f9' }
            ];
            let currentColorIdx = 0;

            // Web Audio API State
            let audioCtx = null;
            let analyser = null;
            let microphoneStream = null;
            let audioAnimFrame = null;
            let currentVolume = 0; // Normalized 0 - 100
            let maxVolumeIndexInPhrase = 0; // Peak volume captured during phrase

            // Speech Recognition Engine State
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            let recognition = null;
            let isRecording = false;

            if (!SpeechRecognition || !(window.AudioContext || window.webkitAudioContext)) {
                compatAlert.classList.remove('hidden');
                recordBtn.disabled = true;
                recordBtn.classList.add('opacity-50', 'cursor-not-allowed');
                showToast('Navegador no compatible con Web Speech o Audio API', 'error');
                return;
            }

            function updateActiveColorUI() {
                const palette = colorPalette[currentColorIdx];
                activeColorDot.style.backgroundColor = palette.hex;
                activeColorLabel.innerText = palette.name;
            }

            nextColorBtn.addEventListener('click', () => {
                currentColorIdx = (currentColorIdx + 1) % colorPalette.length;
                updateActiveColorUI();
            });

            // Calculate Font Size (px) based on peak/instant volume (14px up to 45px)
            function calculateFontSize(vol) {
                const minPx = 14;
                const maxPx = 45;
                const scale = Math.min(Math.max(vol, 0), 100) / 100;
                return Math.round(minPx + (scale * (maxPx - minPx)));
            }

            // Real-time Audio Amplitude Analysis via Web Audio API
            async function startAudioAnalysis() {
                try {
                    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                    microphoneStream = await navigator.mediaDevices.getUserMedia({ audio: true, video: false });
                    
                    const source = audioCtx.createMediaStreamSource(microphoneStream);
                    analyser = audioCtx.createAnalyser();
                    analyser.fftSize = 256;
                    analyser.smoothingTimeConstant = 0.4;
                    source.connect(analyser);

                    const dataArray = new Uint8Array(analyser.frequencyBinCount);

                    function analyze() {
                        if (!isRecording) return;
                        
                        analyser.getByteFrequencyData(dataArray);
                        let sum = 0;
                        for (let i = 0; i < dataArray.length; i++) {
                            sum += dataArray[i];
                        }
                        const average = sum / dataArray.length;
                        
                        // Normalized Volume 0 to 100%
                        currentVolume = Math.min(100, Math.round((average / 128) * 100));
                        
                        if (currentVolume > maxVolumeIndexInPhrase) {
                            maxVolumeIndexInPhrase = currentVolume;
                        }

                        // Update Visual VU Meter
                        volumeBar.style.width = `${currentVolume}%`;
                        volPercent.innerText = `${currentVolume}%`;

                        // Dynamically update interim font size preview
                        const dynamicSizePx = calculateFontSize(currentVolume);
                        interimSpan.style.fontSize = `${dynamicSizePx}px`;

                        audioAnimFrame = requestAnimationFrame(analyze);
                    }

                    analyze();
                } catch (err) {
                    console.error('Mic Access / Audio Context Error:', err);
                    showToast('No se obtuvo acceso al micrófono para análisis de volumen', 'warning');
                }
            }

            function stopAudioAnalysis() {
                if (audioAnimFrame) cancelAnimationFrame(audioAnimFrame);
                if (microphoneStream) {
                    microphoneStream.getTracks().forEach(track => track.stop());
                }
                if (audioCtx && audioCtx.state !== 'closed') {
                    audioCtx.close();
                }
                volumeBar.style.width = '0%';
                volPercent.innerText = '0%';
                currentVolume = 0;
            }

            function initRecognition() {
                recognition = new SpeechRecognition();
                recognition.continuous = true;
                recognition.interimResults = true;
                recognition.lang = languageSelect.value;

                recognition.onstart = () => {
                    isRecording = true;
                    updateUIState(true);
                };

                recognition.onresult = (event) => {
                    let interimText = '';

                    for (let i = event.resultIndex; i < event.results.length; ++i) {
                        const transcriptSegment = event.results[i][0].transcript;
                        if (event.results[i].isFinal) {
                            appendFinalPhrase(transcriptSegment.trim());
                            
                            // Move to next color palette hue for next recognized phrase
                            currentColorIdx = (currentColorIdx + 1) % colorPalette.length;
                            updateActiveColorUI();
                            maxVolumeIndexInPhrase = 0;
                        } else {
                            interimText += transcriptSegment;
                        }
                    }

                    if (interimText.trim()) {
                        const activePalette = colorPalette[currentColorIdx];
                        interimSpan.innerText = interimText;
                        interimSpan.className = `italic transition-all ${activePalette.class}`;
                        interimContainer.classList.remove('hidden');
                    } else {
                        interimContainer.classList.add('hidden');
                    }

                    updateStats();
                };

                recognition.onerror = (event) => {
                    console.error('Speech Recognition Error:', event.error);
                    if (event.error === 'not-allowed') {
                        showToast('Permiso de micrófono denegado', 'error');
                    } else if (event.error !== 'no-speech') {
                        showToast(`Error de reconocimiento: ${event.error}`, 'error');
                    }
                    stopRecording();
                };

                recognition.onend = () => {
                    if (isRecording) {
                        try {
                            recognition.start();
                        } catch (e) {
                            stopRecording();
                        }
                    } else {
                        updateUIState(false);
                    }
                };
            }

            // Append final recognized sentence styled with dynamic font size & phrase color
            function appendFinalPhrase(text) {
                if (!text) return;

                const activePalette = colorPalette[currentColorIdx];
                const peakVol = Math.max(maxVolumeIndexInPhrase, 12);
                const fontSizePx = calculateFontSize(peakVol);

                const span = document.createElement('span');
                span.className = `phrase-span ${activePalette.class} font-semibold mr-2 my-1 inline-block`;
                span.style.fontSize = `${fontSizePx}px`;
                span.innerText = text + '. ';
                span.title = `Pico de Volumen: ${peakVol}% | Tamaño Fuente: ${fontSizePx}px`;

                transcriptBox.appendChild(span);
                
                // Auto scroll to latest transcribed text
                transcriptBox.scrollTop = transcriptBox.scrollHeight;
                updateStats();
            }

            function startRecording() {
                if (!recognition) initRecognition();
                recognition.lang = languageSelect.value;
                maxVolumeIndexInPhrase = 0;
                
                startAudioAnalysis();
                try {
                    recognition.start();
                } catch (e) {
                    console.error('Start recognition error:', e);
                }
            }

            function stopRecording() {
                isRecording = false;
                if (recognition) {
                    recognition.stop();
                }
                stopAudioAnalysis();
                interimContainer.classList.add('hidden');
                updateUIState(false);
            }

            recordBtn.addEventListener('click', () => {
                if (isRecording) {
                    stopRecording();
                } else {
                    startRecording();
                }
            });

            languageSelect.addEventListener('change', () => {
                if (isRecording) {
                    stopRecording();
                    showToast('Idioma cambiado. Inicia la grabación de nuevo.', 'info');
                }
            });

            // Toggle Visual Recording Controls
            function updateUIState(recording) {
                if (recording) {
                    recordBtn.classList.remove('bg-cyan-600', 'hover:bg-cyan-500', 'shadow-cyan-600/30');
                    recordBtn.classList.add('bg-red-600', 'hover:bg-red-500', 'shadow-red-600/40');
                    recordIcon.className = 'fa-solid fa-square';
                    pulseBg.classList.remove('hidden');

                    statusBadge.className = 'inline-flex items-center px-3 py-1 rounded-full text-xs font-semibold bg-red-500/20 text-red-400 border border-red-500/30';
                    statusBadge.innerHTML = '<span class="w-2 h-2 rounded-full bg-red-500 mr-1.5 animate-ping"></span>Grabando y Analizando...';
                    statusHint.innerText = 'Habla fuerte para aumentar la letra o suave para achicarla';
                } else {
                    recordBtn.classList.remove('bg-red-600', 'hover:bg-red-500', 'shadow-red-600/40');
                    recordBtn.classList.add('bg-cyan-600', 'hover:bg-cyan-500', 'shadow-cyan-600/30');
                    recordIcon.className = 'fa-solid fa-microphone';
                    pulseBg.classList.add('hidden');

                    statusBadge.className = 'inline-flex items-center px-3 py-1 rounded-full text-xs font-semibold bg-slate-800 text-slate-300 border border-slate-700';
                    statusBadge.innerText = 'Listo para grabar';
                    statusHint.innerText = 'Haz clic en el micrófono para iniciar el dictado por voz';
                }
            }

            function updateStats() {
                const plainText = transcriptBox.innerText.trim();
                const chars = plainText.length;
                const words = plainText ? plainText.split(/\s+/).filter(w => w.length > 0).length : 0;
                const phrases = transcriptBox.querySelectorAll('.phrase-span').length;

                wordCount.innerText = words;
                charCount.innerText = chars;
                phraseCount.innerText = phrases;
            }

            transcriptBox.addEventListener('input', updateStats);

            // Clipboard Copy
            copyBtn.addEventListener('click', () => {
                const text = transcriptBox.innerText;
                if (!text.trim()) {
                    showToast('No hay texto para copiar', 'warning');
                    return;
                }

                const tempTextArea = document.createElement('textarea');
                tempTextArea.value = text;
                document.body.appendChild(tempTextArea);
                tempTextArea.select();
                try {
                    document.execCommand('copy');
                    showToast('Texto copiado al portapapeles', 'success');
                } catch (err) {
                    showToast('Error al copiar el texto', 'error');
                }
                document.body.removeChild(tempTextArea);
            });

            // TXT Export
            downloadBtn.addEventListener('click', () => {
                const text = transcriptBox.innerText;
                if (!text.trim()) {
                    showToast('No hay texto para descargar', 'warning');
                    return;
                }

                const blob = new Blob([text], { type: 'text/plain;charset=utf-8' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                const date = new Date().toISOString().slice(0, 10);
                
                a.href = url;
                a.download = `transcripcion_${date}.txt`;
                document.body.appendChild(a);
                a.click();
                document.body.removeChild(a);
                URL.revokeObjectURL(url);

                showToast('Transcripción descargada como TXT', 'success');
            });

            // Clear Workspace
            clearBtn.addEventListener('click', () => {
                if (!transcriptBox.innerText.trim()) return;
                transcriptBox.innerHTML = '';
                interimSpan.innerText = '';
                updateStats();
                showToast('Transcripción limpiada', 'info');
            });

            // Toast Notifications
            let toastTimeout;
            function showToast(message, type = 'info') {
                clearTimeout(toastTimeout);
                toastMessage.innerText = message;

                if (type === 'success') {
                    toastIcon.className = 'fa-solid fa-circle-check text-emerald-400';
                } else if (type === 'error') {
                    toastIcon.className = 'fa-solid fa-circle-xmark text-rose-400';
                } else if (type === 'warning') {
                    toastIcon.className = 'fa-solid fa-triangle-exclamation text-amber-400';
                } else {
                    toastIcon.className = 'fa-solid fa-circle-info text-cyan-400';
                }

                toast.classList.remove('hidden');
                toast.classList.add('opacity-100', 'translate-y-0');

                toastTimeout = setTimeout(() => {
                    toast.classList.add('hidden');
                }, 3000);
            }

            // Init App
            updateActiveColorUI();
            updateStats();
        });
    </script>
</body>
</html>
