<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scanner + Filtro 3D Real</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: sans-serif; background: #000; color: white; }
        
        /* Estilos do Scanner */
        #scanner-container {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: white; z-index: 10;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
        }
        #reader { width: 100%; max-width: 400px; }
        .aviso { color: black; text-align: center; margin: 10px; padding: 10px; background: #eee; border-radius: 8px; }
        
        /* Estilos do Container AR */
        #ar-container { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 5; background: black;}
        
        /* Aviso de Carregando */
        #ar-loading {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            color: white; font-size: 20px; z-index: 6; display: none; text-align: center;
        }
    </style>
    
    <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-face-aframe.prod.js"></script>
    <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
</head>
<body>

    <div id="scanner-container">
        <h2 style="color:black">Modo Teste 3D</h2>
        <p class="aviso">Aponte para um QR Code para ver o filtro real.</p>
        <div id="reader"></div>
        <p id="status" style="color:gray; margin-top:10px">Iniciando câmera...</p>
    </div>

    <div id="ar-container">
        <div id="ar-loading">Baixando modelo 3D...<br>Isso pode levar alguns segundos.</div>
    </div>

    <script>
        const scannerContainer = document.getElementById('scanner-container');
        const arContainer = document.getElementById('ar-container');
        const arLoading = document.getElementById('ar-loading');
        const statusMsg = document.getElementById('status');
        let html5QrCode;
        let scannerAtivo = true;

        // --- URL DO MODELO 3D DE EXEMPLO (Capacete de Papelão) ---
        // No futuro, você trocará esse link pelo link do seu arquivo .glb do boné
        const MODELO_3D_URL = "https://cdn.jsdelivr.net/gh/hiukim/mind-ar-js@1.2.5/examples/face-tracking/assets/cardboard/scene.gltf";

        function iniciarScanner() {
            html5QrCode = new Html5Qrcode("reader");
            // Usando environment (traseira) para simular o uso real no poste
            html5QrCode.start({ facingMode: "environment" }, { fps: 10, qrbox: 250 }, 
                (decodedText) => {
                    if (scannerAtivo && decodedText) {
                        scannerAtivo = false;
                        statusMsg.innerText = "Lido! Preparando AR...";
                        statusMsg.style.color = "green";
                        arLoading.style.display = 'block'; 
                        
                        // Pequeno delay para a UI atualizar
                        setTimeout(trocarParaAR, 500);
                    }
                },
                () => {}
            ).catch(err => { statusMsg.innerText = "Erro Câmera: " + err; });
        }

        function trocarParaAR() {
            html5QrCode.stop().then(() => {
                scannerContainer.style.display = 'none'; 
                
                // Injetando a cena com o modelo 3D real
                arContainer.innerHTML = `
                    <a-scene mindar-face embedded color-space="sRGB" renderer="colorManagement: true, physicallyCorrectLights" vr-mode-ui="enabled: false" device-orientation-permission-ui="enabled: false">
                        
                        <a-assets>
                            <a-asset-item id="helmetModel" src="${MODELO_3D_URL}"></a-asset-item>
                        </a-assets>

                        <a-camera active="false" position="0 0 0"></a-camera>
                        
                        <a-entity mindar-face-target="anchorIndex: 10">
                            <a-gltf-model src="#helmetModel" position="0 -0.5 0" scale="0.012 0.012 0.012" rotation="0 180 0"></a-gltf-model>
                        </a-entity>
                    </a-scene>
                `;

                // Esconde o aviso de carregando quando o modelo estiver pronto
                document.querySelector('a-scene').addEventListener('loaded', () => {
                     arLoading.style.display = 'none';
                });

            }).catch(err => console.error(err));
        }

        iniciarScanner();
    </script>
</body>
</html>
