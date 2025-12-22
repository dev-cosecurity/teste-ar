<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scanner Teste (Qualquer QR)</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: sans-serif; background: #000; color: white; }
        
        #scanner-container {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: white; z-index: 10;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
        }
        
        #ar-container { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 5; }
        
        #reader { width: 100%; max-width: 400px; }
        
        .aviso { 
            color: black; text-align: center; margin: 10px; font-weight: bold;
            padding: 10px; background: #eee; border-radius: 8px;
        }
    </style>
    
    <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-face-aframe.prod.js"></script>
    <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
</head>
<body>

    <div id="scanner-container">
        <h2 style="color:black">Modo Teste</h2>
        <p class="aviso">Aponte para QUALQUER QR Code para ativar o filtro.</p>
        <div id="reader"></div>
        <p id="status" style="color:gray; margin-top:10px">Iniciando câmera...</p>
    </div>

    <div id="ar-container"></div>

    <script>
        const scannerContainer = document.getElementById('scanner-container');
        const arContainer = document.getElementById('ar-container');
        const statusMsg = document.getElementById('status');
        let html5QrCode;
        let scannerAtivo = true; // Trava para não ler o mesmo QR várias vezes seguidas

        function iniciarScanner() {
            html5QrCode = new Html5Qrcode("reader");

            const config = { fps: 10, qrbox: { width: 250, height: 250 } };
            
            html5QrCode.start({ facingMode: "environment" }, config, 
                (decodedText, decodedResult) => {
                    // --- AQUI ESTÁ A MUDANÇA ---
                    // Se scanner estiver ativo e leu ALGO, a gente aceita.
                    if (scannerAtivo && decodedText) {
                        scannerAtivo = false; // Bloqueia novas leituras imediatas
                        console.log(`Lido: ${decodedText}`);
                        
                        statusMsg.innerText = "QR Lido! Carregando filtro...";
                        statusMsg.style.color = "green";
                        
                        // Pequeno delay para usuário ver que leu
                        setTimeout(trocarParaAR, 500);
                    }
                },
                (errorMessage) => {
                    // Ignora erros de "nenhum QR encontrado" enquanto scaneia
                }
            ).catch(err => {
                statusMsg.innerText = "Erro Câmera: " + err;
                statusMsg.style.color = "red";
            });
        }

        function trocarParaAR() {
            // Para a câmera do scanner
            html5QrCode.stop().then(() => {
                scannerContainer.style.display = 'none'; // Some com o scanner
                
                // Injeta o MindAR (Câmera frontal para selfie)
                arContainer.innerHTML = `
                    <a-scene mindar-face embedded color-space="sRGB" renderer="colorManagement: true, physicallyCorrectLights" vr-mode-ui="enabled: false" device-orientation-permission-ui="enabled: false">
                        <a-camera active="false" position="0 0 0"></a-camera>
                        
                        <a-entity mindar-face-target="anchorIndex: 168">
                            <a-sphere color="red" radius="0.1" position="-0.1 0 0.1"></a-sphere>
                            <a-sphere color="red" radius="0.1" position="0.1 0 0.1"></a-sphere>
                        </a-entity>
                    </a-scene>
                `;
            }).catch(err => {
                console.error("Falha ao parar scanner", err);
            });
        }

        // Inicia
        iniciarScanner();

    </script>
</body>
</html>
