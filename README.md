# ing
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>AR с анимированной моделью</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <!-- A-Frame -->
    <script src="https://aframe.io/releases/1.4.0/aframe.min.js"></script>
    <!-- AR.js -->
    <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js"></script>
    <!-- Animation компонент -->
    <script src="https://cdn.jsdelivr.net/npm/aframe-extras@6.1.1/dist/aframe-extras.loaders.min.js"></script>
    <style>
        body { 
            margin: 0; 
            overflow: hidden; 
            font-family: Arial, sans-serif;
            background: #000;
        }
        .ar-overlay {
            position: absolute;
            top: 15px;
            left: 15px;
            right: 15px;
            background: rgba(0,0,0,0.7);
            color: white;
            padding: 12px;
            border-radius: 8px;
            z-index: 100;
            max-width: 320px;
            backdrop-filter: blur(5px);
        }
        .ar-status {
            color: #ff5555;
            font-weight: bold;
            margin-top: 8px;
            font-size: 14px;
        }
    </style>
</head>
<body>
    <div class="ar-overlay">
        <h3 style="margin: 0 0 8px 0;">AR Демонстрация</h3>
        <ol style="margin: 0; padding-left: 20px;">
            <li>Разрешите доступ к камере</li>
            <li>Наведите на маркер Hiro</li>
        </ol>
        <p class="ar-status" id="status">Статус: Загрузка...</p>
    </div>

    <!-- AR-сцена -->
    <a-scene
        vr-mode-ui="enabled: false"
        embedded
        arjs="sourceType: webcam; detectionMode: mono_and_matrix; debugUIEnabled: false;"
        renderer="colorManagement: true; antialias: true;"
    >
        <!-- Ресурсы -->
        <a-assets>
            <a-asset-item id="animatedModel" src="ing/engine.glb"></a-asset-item>
        </a-assets>

        <!-- Маркер Hiro -->
        <a-marker preset="hiro" id="marker">
            <a-entity
                gltf-model="#animatedModel"
                position="0 0.5 0"
                scale="0.5 0.5 0.5"
                rotation="0 180 0"
                animation-mixer="clip: *; loop: repeat; crossFadeDuration: 0.5"
                shadow="receive: false">
            </a-entity>
        </a-marker>

        <!-- Освещение -->
        <a-entity light="type: ambient; intensity: 0.8"></a-entity>
        <a-entity light="type: directional; intensity: 1.0" position="-1 2 1"></a-entity>
        <a-entity camera></a-entity>
    </a-scene>

    <script>
        const statusEl = document.getElementById('status');
        const sceneEl = document.querySelector('a-scene');

        // Проверка поддержки
        function checkCompatibility() {
            if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                statusEl.textContent = "Ваш браузер не поддерживает доступ к камере";
                return false;
            }
            return true;
        }

        // Основная инициализация
        sceneEl.addEventListener('loaded', () => {
            if (!checkCompatibility()) return;

            const marker = document.getElementById('marker');
            
            marker.addEventListener('markerFound', () => {
                statusEl.textContent = "Маркер распознан! Анимация запущена";
                statusEl.style.color = "#00ff00";
                
                // Принудительный запуск анимации
                const model = marker.querySelector('[gltf-model]');
                if (model && model.components['animation-mixer']) {
                    model.components['animation-mixer'].playAll();
                }
            });

            marker.addEventListener('markerLost', () => {
                statusEl.textContent = "Наведите на маркер Hiro";
                statusEl.style.color = "#ff5555";
            });

            statusEl.textContent = "Готово! Наведите на маркер";
            statusEl.style.color = "#00ff88";
        });

        // Обработка ошибок
        sceneEl.addEventListener('error', (e) => {
            statusEl.textContent = "Ошибка: " + (e.message || "Неизвестная ошибка");
            statusEl.style.color = "#ff0000";
            console.error(e);
        });
    </script>
</body>
</html>
