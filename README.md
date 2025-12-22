<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scanner + Filtro Ajustado</title>
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
        
        /* Uma mensagem de "Carregando..." para a fase de AR */
        #ar-loading {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            color: white; font-size: 20px; z-index: 6; display: none;
        }
    </style>
    
    <script src="https://aframe.io/releases/1.5.0/aframe.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-face-aframe.prod.js"></script>
    <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
</head>
<body>

    <div id="scanner-container">
        <h2 style="color:black">Modo Teste</h2>
        <p class="aviso">Aponte para QUALQUER QR Code.</p>
        <div id="reader"></div>
        <p id="status" style="color:gray; margin-top:10px">Iniciando câmera...</p>
    </div>

    <div id="ar-container">
        <div id="ar-loading">Carregando Filtro Facial... Aguarde.</div>
    </div>

    <script>
        const scannerContainer = document.getElementById('scanner-container');
        const arContainer = document.getElementById('ar-container');
        const arLoading = document.getElementById('ar-loading');
        const statusMsg = document.getElementById('status');
        let html5QrCode;
        let scannerAtivo = true;

        function iniciarScanner() {
            html5QrCode = new Html5Qrcode("reader");
            // Usando 'user' (frontal) as vezes é mais rapido para testar no pc, 
            // mas 'environment' (traseira) é o certo para ler QR code de poste.
            html5QrCode.start({ facingMode: "environment" }, { fps: 10, qrbox: 250 }, 
                (decodedText) => {
                    if (scannerAtivo && decodedText) {
                        scannerAtivo = false;
                        statusMsg.innerText = "Lido! Iniciando AR...";
                        statusMsg.style.color = "green";
                        // Mostra aviso de carregando AR
                        arLoading.style.display = 'block'; 
                        
                        setTimeout(trocarParaAR, 1000);
                    }
                },
                () => {}
            ).catch(err => { statusMsg.innerText = "Erro: " + err; });
        }

        function trocarParaAR() {
            html5QrCode.stop().then(() => {
                scannerContainer.style.display = 'none'; 
                
                // --- AQUI ESTÁ A CORREÇÃO DE POSIÇÃO ---
                // Aumentei o radius para 0.15 (maiores)
                // Mudei a posição Z para 0.5 (bem para frente do rosto)
                arContainer.innerHTML = `
                    <a-scene mindar-face embedded color-space="sRGB" renderer="colorManagement: true, physicallyCorrectLights" vr-mode-ui="enabled: false" device-orientation-permission-ui="enabled: false">
                        <a-camera active="false" position="0 0 0"></a-camera>
                        
                        <a-entity mindar-face-target="anchorIndex: 168">
                            <a-sphere color="red" radius="0.15" position="-0.12 -0.05 0.5"></a-sphere>
                            <a-sphere color="red" radius="0.15" position="0.12 -0.05 0.5"></a-sphere>
                        </a-entity>
                    </a-scene>
                `;

                // Adiciona um ouvinte para saber quando o MindAR está pronto e esconder o "Carregando"
                document.querySelector('a-scene').addEventListener('mindar-face-ready', () => {
                     arLoading.style.display = 'none';
                     console.log("Filtro pronto e rastreando!");
                });

            }).catch(err => console.error(err));
        }

        iniciarScanner();
    </script>
</body>
</html>
