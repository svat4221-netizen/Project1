<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>3D Voxel Game: Фикс Экрана</title>
    <style>
        /* Жестко привязываем игру к размерам мобильного экрана, убирая любые отступы */
        html, body { 
            margin: 0; 
            padding: 0; 
            width: 100%; 
            height: 100%; 
            overflow: hidden; 
            background-color: #000000; 
        }
        canvas { 
            display: block; 
            width: 100vw !important; 
            height: 100vh !important; 
        }
        #ui-container {
            position: absolute;
            top: 15px;
            left: 15px;
            color: white;
            font-family: sans-serif;
            font-size: 14px;
            text-shadow: 2px 2px 2px black;
            pointer-events: none;
            z-index: 10;
        }
    </style>
    <!-- Подключаем библиотеку Three.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <div id="ui-container">
        <div>Оружие: 3D Алмазный Меч</div>
        <div id="online-status">Экран исправлен. Одиночный режим.</div>
    </div>

    <script>
        // Глобальные переменные игры
        let scene, camera, renderer;
        let currentWeaponMesh;
        let terrainBlocks = [];

        // --- ИНИЦИАЛИЗАЦИЯ ИГРЫ ---
        function init() {
            // Создаем 3D сцену
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x78a7ff); // Голубое небо
            scene.fog = new THREE.FogExp2(0x78a7ff, 0.04);

            // Камера (глаза игрока)
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 2, 5); 

            // Рендерер с жесткой привязкой к размерам окна
            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: "high-performance" });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // Оптимизация под Retina/FullHD экраны телефонов
            renderer.shadowMap.enabled = true;
            document.body.appendChild(renderer.domElement);

            // Освещение (настройка объемных теней)
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);[span_4](start_span)[span_4](end_span)
            scene.add(ambientLight);

            const sunLight = new THREE.DirectionalLight(0xffffff, 0.8);[span_5](start_span)[span_5](end_span)
            sunLight.position.set(12, 24, 12);
            sunLight.castShadow = true;
            scene.add(sunLight);

            // Строим землю
            createWorld();

            // Создаем и добавляем вертикальный Алмазный Меч в руку[span_6](start_span)[span_6](end_span)
            currentWeaponMesh = createVoxelTool('diamond_sword');
            camera.add(currentWeaponMesh);
            scene.add(camera); 

            // Позиция меча на экране смартфона[span_7](start_span)[span_7](end_span)
            currentWeaponMesh.position.set(0.35, -0.45, -0.7); 
            // Вертикальное положение и легкий наклон вперед[span_8](start_span)[span_8](end_span)[span_9](start_span)[span_9](end_span)
            currentWeaponMesh.rotation.set(Math.PI / 5, -Math.PI / 7, Math.PI / 12);

            // Слушатель изменения ориентации экрана (горизонтально/вертикально)
            window.addEventListener('resize', onWindowResize, false);
            
            // Запуск рендера
            animate();
        }

        // --- СОЗДАНИЕ ОБЪЁМНОГО ОРУЖИЯ ---
        function createVoxelTool(type) {
            const toolGroup = new THREE.Group();
            
            const woodMat = new THREE.MeshStandardMaterial({ color: 0x5c4033, roughness: 0.9 });[span_10](start_span)[span_10](end_span)
            const diamondMat = new THREE.MeshStandardMaterial({ 
                color: 0x00ffff, 
                roughness: 0.1, 
                metalness: 0.6,
                emissive: 0x001a24 // Внутренний блеск алмаза[span_11](start_span)[span_11](end_span)
            });

            if (type === 'diamond_sword') {
                // Рукоятка[span_12](start_span)[span_12](end_span)
                const handle = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.4, 0.06), woodMat);
                handle.position.y = 0.2;[span_13](start_span)[span_13](end_span)
                toolGroup.add(handle);

                // Крестовина (Гарда)[span_14](start_span)[span_14](end_span)
                const guard = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.08, 0.12), diamondMat);
                guard.position.y = 0.4;
                toolGroup.add(guard);

                // Объёмное 3D лезвие[span_15](start_span)[span_15](end_span)
                const blade = new THREE.Mesh(new THREE.BoxGeometry(0.14, 1.1, 0.06), diamondMat);
                blade.position.y = 0.9;
                toolGroup.add(blade);
                
                // Острие[span_16](start_span)[span_16](end_span)
                const tip = new THREE.Mesh(new THREE.BoxGeometry(0.1, 0.1, 0.06), diamondMat);
                tip.position.y = 1.45;
                tip.rotation.z = Math.PI / 4;[span_17](start_span)[span_17](end_span)
                toolGroup.add(tip);
            }

            // Полное обнуление внутренних поворотов группы, чтобы избежать наклона набок[span_18](start_span)[span_18](end_span)!
            toolGroup.rotation.set(0, 0, 0); 
            return toolGroup;
        }

        // --- СОЗДАНИЕ ЗЕМЛИ ---
        function createWorld() {
            const blockGeo = new THREE.BoxGeometry(1, 1, 1);
            const grassMat = new THREE.MeshStandardMaterial({ color: 0x559933, roughness: 0.8 });

            for (let x = -6; x < 6; x++) {
                for (let z = -6; z < 6; z++) {
                    const block = new THREE.Mesh(blockGeo, grassMat);
                    block.position.set(x, 0, z);
                    block.receiveShadow = true;
                    scene.add(block);
                    terrainBlocks.push(block);
                }
            }
        }

        // --- ИГРОВОЙ ЦИКЛ ОБНОВЛЕНИЯ КАДРОВ ---
        function animate() {
            requestAnimationFrame(animate);

            // Эффект плавного дыхания/покачивания меча в руке
            const time = Date.now() * 0.0025;
            if (currentWeaponMesh) {
                currentWeaponMesh.position.y = -0.45 + Math.sin(time) * 0.012;
                currentWeaponMesh.position.x = 0.35 + Math.cos(time) * 0.005;
            }

            renderer.render(scene, camera);
        }

        // --- АВТОМАТИЧЕСКИЙ ПЕРЕРАСЧЕТ РАЗМЕРОВ ЭКРАНА ---
        function onWindowResize() {
            if (!camera || !renderer) return;
            
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // --- БЕЗОПАСНЫЙ СТАРТ ИГРЫ ДЛЯ МОБИЛЬНЫХ БРАУЗЕРОВ ---
        window.onload = () => {
            // Запускаем движок
            init();
            
            // Через 100мс принудительно адаптируем картинку под экран телефона,
            // это полностью убирает баг получёрного экрана при загрузке.
            setTimeout(() => {
                onWindowResize();
            }, 100);
        };
    </script>
</body>
</html>


