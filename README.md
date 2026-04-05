# yak23yak23.github.io
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QR Studio Pro - Canlı Kamera Destekli</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- QR Code Generation Library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <!-- QR Code Scanning Library -->
    <script src="https://unpkg.com/jsqr/dist/jsQR.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;600;700&display=swap');
        
        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            background: #f1f5f9;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }

        .glass-card {
            background: white;
            border-radius: 32px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.05);
        }

        .tab-btn.active {
            background: #2563eb;
            color: white;
        }

        #video-preview {
            width: 100%;
            max-width: 400px;
            border-radius: 20px;
            background: #000;
        }

        .scanner-overlay {
            position: relative;
            overflow: hidden;
            width: 100%;
            max-width: 400px;
        }

        .scanner-line {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 2px;
            background: #ef4444;
            box-shadow: 0 0 10px #ef4444;
            animation: scan-move 2s linear infinite;
            z-index: 10;
        }

        @keyframes scan-move {
            0% { top: 0; }
            100% { top: 100%; }
        }

        main {
            flex: 1;
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <main class="max-w-5xl mx-auto space-y-6 w-full">
        <div class="text-center mb-8">
            <h1 class="text-4xl font-extrabold text-gray-900 tracking-tight">QR Studio Pro</h1>
            <p class="text-gray-500 mt-2">Oluştur, Canlı Tara ve Yönet</p>
        </div>

        <div class="flex justify-center mb-8">
            <div class="inline-flex p-1 bg-white rounded-2xl shadow-sm border border-gray-100">
                <button onclick="switchTab('create')" id="tab-create" class="tab-btn active px-6 py-2.5 rounded-xl text-sm font-bold transition-all">Oluştur</button>
                <button onclick="switchTab('scan')" id="tab-scan" class="tab-btn px-6 py-2.5 rounded-xl text-sm font-bold text-gray-500 transition-all">Canlı Tara</button>
                <button onclick="switchTab('history')" id="tab-history" class="tab-btn px-6 py-2.5 rounded-xl text-sm font-bold text-gray-500 transition-all">Geçmiş</button>
            </div>
        </div>

        <!-- CREATE TAB -->
        <div id="view-create" class="grid grid-cols-1 lg:grid-cols-2 gap-8 animate-in fade-in duration-500">
            <div class="glass-card p-8 space-y-6 border border-gray-50">
                <h2 class="text-xl font-bold text-gray-800">Tasarım Ayarları</h2>
                
                <div>
                    <label class="block text-sm font-bold text-gray-700 mb-2">İçerik (URL/Metin)</label>
                    <div class="flex gap-2">
                        <input type="text" id="text-input" placeholder="https://..." class="w-full px-4 py-3 rounded-xl border border-gray-200 outline-none focus:ring-2 focus:ring-blue-500 transition-all">
                        <button onclick="visitLink()" class="bg-gray-100 hover:bg-gray-200 p-3 rounded-xl transition-colors">
                            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg>
                        </button>
                    </div>
                </div>

                <div>
                    <div class="flex justify-between items-center mb-2">
                        <label class="block text-sm font-bold text-gray-700">Merkez Logo</label>
                        <button id="remove-logo" onclick="removeLogo()" class="text-xs text-red-500 hidden">Kaldır</button>
                    </div>
                    <label class="flex flex-col items-center justify-center w-full h-24 rounded-xl border-2 border-dashed border-gray-200 cursor-pointer hover:bg-gray-50 transition-colors">
                        <input type="file" id="logo-input" accept="image/*" class="hidden" onchange="handleLogoUpload(this)">
                        <span id="logo-label" class="text-xs text-gray-500">Logo Seç</span>
                        <img id="logo-preview-img" class="hidden h-full p-2 object-contain">
                    </label>
                </div>

                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <label class="block text-xs font-bold text-gray-400 uppercase mb-1">Kod Rengi</label>
                        <input type="color" id="color-dark" value="#000000" onchange="generateQR()" class="w-full h-10 rounded-lg cursor-pointer">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-gray-400 uppercase mb-1">Arka Plan</label>
                        <input type="color" id="color-light" value="#ffffff" onchange="generateQR()" class="w-full h-10 rounded-lg cursor-pointer">
                    </div>
                </div>

                <button onclick="generateQR()" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-4 rounded-2xl shadow-lg shadow-blue-200 transition-all">QR Kodu Oluştur</button>
            </div>

            <div class="glass-card p-8 flex flex-col items-center justify-center border border-gray-50">
                <div id="qrcode" class="p-4 bg-white border rounded-2xl"></div>
                <div id="action-buttons" class="hidden mt-6 w-full max-w-xs">
                    <button onclick="downloadQR()" class="w-full bg-green-600 hover:bg-green-700 text-white font-semibold py-3 rounded-xl transition-all flex items-center justify-center gap-2">
                         İndir (PNG)
                    </button>
                </div>
            </div>
        </div>

        <!-- SCAN TAB -->
        <div id="view-scan" class="hidden glass-card p-8 md:p-12 text-center space-y-6 animate-in slide-in-from-bottom-4">
            <h2 class="text-2xl font-bold text-gray-800">Canlı QR Tarayıcı</h2>
            
            <div class="flex flex-col items-center justify-center space-y-4">
                <div id="scanner-wrapper" class="hidden scanner-overlay border-4 border-white shadow-xl rounded-3xl">
                    <video id="video-preview" autoplay playsinline muted></video>
                    <div class="scanner-line"></div>
                </div>
                
                <p id="scanner-status" class="text-sm text-gray-500 italic"></p>

                <button id="start-scan-btn" onclick="startCamera()" class="bg-blue-600 hover:bg-blue-700 text-white px-8 py-4 rounded-2xl font-bold shadow-lg transition-all">
                    Kamerayı Başlat
                </button>
                <button id="stop-scan-btn" onclick="stopCamera()" class="hidden bg-red-500 hover:bg-red-600 text-white px-8 py-4 rounded-2xl font-bold transition-all">
                    Kamerayı Kapat
                </button>
            </div>

            <div id="scan-result-box" class="hidden p-6 bg-blue-50 rounded-2xl text-left max-w-xl mx-auto border border-blue-100">
                <span class="text-xs font-bold text-blue-600 uppercase tracking-widest">Tarama Sonucu:</span>
                <p id="scan-result-text" class="text-gray-800 mt-2 font-medium break-all"></p>
                <div class="mt-4 flex gap-2">
                    <button onclick="copyScanResult()" class="bg-white border hover:bg-gray-50 px-4 py-2 rounded-lg text-sm transition-all">Kopyala</button>
                    <button id="scan-visit-btn" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg text-sm transition-all">Siteye Git</button>
                </div>
            </div>
        </div>

        <!-- HISTORY TAB -->
        <div id="view-history" class="hidden glass-card p-8 animate-in slide-in-from-bottom-4">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-bold text-gray-800">Geçmiş</h2>
                <button onclick="clearHistory()" class="text-sm text-red-500 hover:underline font-semibold">Tümünü Temizle</button>
            </div>
            <div id="history-list" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
        </div>
    </main>

    <footer class="max-w-5xl mx-auto w-full py-8 mt-8 border-t border-gray-200">
        <div class="flex flex-col md:flex-row justify-between items-center gap-4 text-sm text-gray-500">
            <p>© <span id="current-year">2026</span> QR Studio Pro. Tüm hakları saklıdır.</p>
            <div class="flex items-center gap-6">
                <a href="https://www.termsfeed.com/live/7bb5c5cf-a086-4e4d-a829-d6de2e6dffc6" target="_blank" class="hover:text-blue-600 transition-colors font-medium flex items-center gap-1">
                    Gizlilik Politikası
                </a>
            </div>
        </div>
    </footer>

    <!-- Gizli yardımcı elemanlar -->
    <canvas id="qr-scan-canvas" class="hidden"></canvas>

    <script>
        let logoImage = null;
        let qrHistory = JSON.parse(localStorage.getItem('qr_history') || '[]');
        let videoStream = null;
        let isScanning = false;
        let animationFrameId = null;

        document.getElementById('current-year').textContent = new Date().getFullYear() >= 2026 ? new Date().getFullYear() : 2026;

        function switchTab(tab) {
            stopCamera();
            ['create', 'scan', 'history'].forEach(t => {
                document.getElementById(`view-${t}`).classList.add('hidden');
                const btn = document.getElementById(`tab-${t}`);
                btn.classList.remove('active', 'text-white');
                btn.classList.add('text-gray-500');
            });
            document.getElementById(`view-${tab}`).classList.remove('hidden');
            document.getElementById(`tab-${tab}`).classList.add('active', 'text-white');
            document.getElementById(`tab-${tab}`).classList.remove('text-gray-500');
            if(tab === 'history') updateHistoryUI();
        }

        async function startCamera() {
            const status = document.getElementById('scanner-status');
            const video = document.getElementById('video-preview');
            
            try {
                status.innerText = "Kamera başlatılıyor...";
                videoStream = await navigator.mediaDevices.getUserMedia({ 
                    video: { facingMode: "environment" },
                    audio: false 
                });
                
                video.srcObject = videoStream;
                // Bazı mobil tarayıcılarda video.play() manuel gerekebilir
                await video.play();

                document.getElementById('scanner-wrapper').classList.remove('hidden');
                document.getElementById('start-scan-btn').classList.add('hidden');
                document.getElementById('stop-scan-btn').classList.remove('hidden');
                status.innerText = "QR kodu merkeze hizalayın";
                
                isScanning = true;
                tick();
            } catch (err) {
                console.error(err);
                status.innerText = "Hata: Kamera izni verilmedi veya desteklenmiyor.";
                alert("Kamera hatası: " + err.message);
            }
        }

        function stopCamera() {
            isScanning = false;
            if (animationFrameId) cancelAnimationFrame(animationFrameId);
            if (videoStream) {
                videoStream.getTracks().forEach(track => track.stop());
                videoStream = null;
            }
            document.getElementById('scanner-wrapper').classList.add('hidden');
            document.getElementById('start-scan-btn').classList.remove('hidden');
            document.getElementById('stop-scan-btn').classList.add('hidden');
            document.getElementById('scanner-status').innerText = "";
        }

        function tick() {
            if (!isScanning) return;
            
            const video = document.getElementById('video-preview');
            const canvas = document.getElementById('qr-scan-canvas');
            const ctx = canvas.getContext('2d', { willReadFrequently: true });

            if (video.readyState === video.HAVE_ENOUGH_DATA) {
                canvas.width = video.videoWidth;
                canvas.height = video.videoHeight;
                ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
                
                const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                const code = jsQR(imageData.data, imageData.width, imageData.height, {
                    inversionAttempts: "dontInvert",
                });
                
                if (code) {
                    console.log("QR Kod bulundu!", code.data);
                    showScanResult(code.data);
                    stopCamera();
                    return; // Döngüyü kır
                }
            }
            animationFrameId = requestAnimationFrame(tick);
        }

        function showScanResult(data) {
            const box = document.getElementById('scan-result-box');
            const text = document.getElementById('scan-result-text');
            const btn = document.getElementById('scan-visit-btn');
            box.classList.remove('hidden');
            text.innerText = data;
            if(data.startsWith('http')) {
                btn.classList.remove('hidden');
                btn.onclick = () => window.open(data, '_blank');
            } else {
                btn.classList.add('hidden');
            }
            // Sonucu geçmişe de ekle
            addToHistory("(Tarandı) " + data);
        }

        function copyScanResult() {
            const text = document.getElementById('scan-result-text').innerText;
            navigator.clipboard.writeText(text).then(() => {
                const btn = event.target;
                btn.innerText = "Kopyalandı!";
                setTimeout(() => btn.innerText = "Kopyala", 2000);
            });
        }

        // CREATE LOGIC
        function generateQR() {
            const text = document.getElementById('text-input').value;
            if(!text) return;
            const qrDiv = document.getElementById('qrcode');
            qrDiv.innerHTML = "";
            new QRCode(qrDiv, {
                text: text, width: 256, height: 256,
                colorDark: document.getElementById('color-dark').value,
                colorLight: document.getElementById('color-light').value,
                correctLevel: QRCode.CorrectLevel.H
            });
            setTimeout(() => {
                renderLogoOnQR();
                addToHistory(text);
                document.getElementById('action-buttons').classList.remove('hidden');
            }, 100);
        }

        function renderLogoOnQR() {
            const canvas = document.querySelector('#qrcode canvas');
            if(!canvas || !logoImage) return;
            const ctx = canvas.getContext('2d');
            const size = canvas.width;
            const logoSize = size * 0.22;
            const pos = (size - logoSize) / 2;
            ctx.fillStyle = document.getElementById('color-light').value;
            ctx.fillRect(pos - 5, pos - 5, logoSize + 10, logoSize + 10);
            ctx.drawImage(logoImage, pos, pos, logoSize, logoSize);
        }

        function handleLogoUpload(input) {
            if (input.files && input.files[0]) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    const img = new Image();
                    img.onload = () => {
                        logoImage = img;
                        document.getElementById('logo-preview-img').src = e.target.result;
                        document.getElementById('logo-preview-img').classList.remove('hidden');
                        document.getElementById('logo-label').classList.add('hidden');
                        document.getElementById('remove-logo').classList.remove('hidden');
                        generateQR();
                    };
                    img.src = e.target.result;
                };
                reader.readAsDataURL(input.files[0]);
            }
        }

        function addToHistory(text) {
            if(qrHistory.length > 0 && qrHistory[0].text === text) return;
            qrHistory.unshift({ text, date: new Date().toLocaleString('tr-TR') });
            if(qrHistory.length > 10) qrHistory.pop();
            localStorage.setItem('qr_history', JSON.stringify(qrHistory));
        }

        function updateHistoryUI() {
            const list = document.getElementById('history-list');
            list.innerHTML = qrHistory.map(item => `
                <div class="p-4 bg-white rounded-2xl border border-gray-100 flex justify-between items-center shadow-sm">
                    <div class="truncate mr-4">
                        <p class="font-bold text-sm text-gray-800 truncate">${item.text}</p>
                        <p class="text-xs text-gray-400">${item.date}</p>
                    </div>
                    <button onclick="document.getElementById('text-input').value='${item.text.replace('(Tarandı) ', '')}'; switchTab('create'); generateQR();" class="text-blue-600 hover:text-blue-800 text-sm font-bold transition-colors">Tekrarla</button>
                </div>
            `).join('') || '<p class="text-gray-400 italic text-center py-10 col-span-full">Henüz bir kayıt bulunmuyor.</p>';
        }

        function clearHistory() {
            if(confirm("Tüm geçmişi silmek istediğinize emin misiniz?")) {
                qrHistory = [];
                localStorage.removeItem('qr_history');
                updateHistoryUI();
            }
        }

        function downloadQR() {
            const canvas = document.querySelector('#qrcode canvas');
            if(!canvas) return;
            const link = document.createElement('a');
            link.download = 'qrcode.png';
            link.href = canvas.toDataURL();
            link.click();
        }

        function removeLogo() {
            logoImage = null;
            document.getElementById('logo-input').value = "";
            document.getElementById('logo-preview-img').classList.add('hidden');
            document.getElementById('logo-label').classList.remove('hidden');
            document.getElementById('remove-logo').classList.add('hidden');
            generateQR();
        }

        function visitLink() {
            const val = document.getElementById('text-input').value;
            if(val) window.open(val.startsWith('http') ? val : 'https://'+val, '_blank');
        }
    </script>
</body>
</html>
