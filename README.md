# teste-ar

<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scanner -> Filtro AR</title>

    <style>
        body { margin: 0; overflow: hidden; font-family: sans-serif; background: #000; }
        
        #scanner-container {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: white; z-index: 10;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
        }
        
        #ar-container {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            z-index: 5;
        }

        .instrucao { margin-bottom: 20px; font-size: 18px; text-align: center; color: #333; }
        
        /* Ajuste do vídeo do scanner */
        #reader { width: 300px; height: 300px; }
    </style>

    <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-face-aframe.prod.js"></script>
    <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
</head>
<body>

    <div id="scanner-container">
        <div class="instrucao">Aponte para o QR Code do poste</div>
        <div id="reader"></div>
        <p id="status-msg" style="color: grey; margin-top: 10px;">Aguardando câmera...</p>
    </div>

    <div id="ar-container"></div>

    <script>
        // --- CONFIGURAÇÃO ---
        // Troque este link pelo link exato que o QR Code do poste contém
        const LINK_ALVO = "https://www.google.com"; 

        const scannerDiv = document.getElementById('reader');
        const scannerContainer = document.getElementById('scanner-container');
        const arContainer = document.getElementById('ar-container');
        const html5QrCode = new Html5Qrcode("reader");

        // --- FUNÇÃO 1: Iniciar Leitor QR ---
        function iniciarScanner() {
            const config = { fps: 10, qrbox: { width: 250, height: 250 } };
            
            html5QrCode.start({ facingMode: "environment" }, config, (decodedText, decodedResult) => {
                // Sucesso na leitura
                console.log(`Leu: ${decodedText}`);

                if (decodedText === LINK_ALVO) {
                    document.getElementById('status-msg').innerText = "QR Code Confirmado! Iniciando Filtro...";
                    trocarParaAR();
                } else {
                    alert("QR Code errado! Tente o do poste.");
                }
            }, (errorMessage) => {
                // Erro de leitura contínua (ignorar para não poluir log)
            }).catch(err => {
                console.error("Erro ao iniciar câmera", err);
            });
        }

        // --- FUNÇÃO 2: A Transição Crítica ---
        function trocarParaAR() {
            // 1. Matar o scanner para liberar a câmera
            html5QrCode.stop().then(() => {
                // 2. Esconder a UI do scanner
                scannerContainer.style.display = 'none';

                // 3. Injetar o HTML do MindAR dinamicamente
                // Fazemos isso aqui para evitar conflito de câmera no carregamento da página
                arContainer.innerHTML = `
                    <a-scene mindar-face embedded color-space="sRGB" renderer="colorManagement: true, physicallyCorrectLights" vr-mode-ui="enabled: false" device-orientation-permission-ui="enabled: false">
                        <a-assets>
                            </a-assets>

                        <a-camera active="false" position="0 0 0"></a-camera>

                        <a-entity mindar-face-target="anchorIndex: 168">
                            
                            <a-sphere color="red" radius="0.1" position="-0.1 0 0.1"></a-sphere>
                            <a-sphere color="red" radius="0.1" position="0.1 0 0.1"></a-sphere>
                            
                            </a-entity>
                    </a-scene>
                `;
            }).catch(err => {
                console.error("Erro ao parar o scanner", err);
            });
        }

        // Inicia tudo
        iniciarScanner();

    </script>
</body>
</html>
