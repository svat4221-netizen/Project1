<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>3D Voxel Game: Алмазный Меч</title>
    <style>
        body { margin: 0; padding: 0; overflow: hidden; background-color: #000; }
        canvas { display: block; width: 100vw; height: 100vh; }
        #ui-container {
            position: absolute;
            top: 10px;
            left: 10px;
            color: white;
            font-family: sans-serif;
            font-size: 14px;
            text-shadow: 1px 1px 2px black;
            pointer-events: none;
        }
    </style>
    <!-- Подключаем библиотеку Three.js для работы с 3D -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <div id="ui-container">
        <div>Оружие в руке: Объёмный Алмазный Меч</div>
        <div id="online-status">Статус: Готов к подключению мультиплеера</div>
    </div>

    <script>
        // --- ГЛОБАЛЬНЫЕ ПЕРЕМЕННЫЕ ДВИЖКА ---
        let scene, camera, renderer;
        let currentWeaponMesh;
        let terrainBlocks = [];

        // --- 1. ГЛАВНАЯ НАСТРОЙКА ИГРЫ (ИНИЦИАЛИЗАЦИЯ) ---
        function init() {
            // Создаем 3D сцену
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x78a7ff); // Красивое голубое небо
            scene.fog = new THREE.FogExp2(0x78a7ff, 0.04);

            // Настраиваем камеру (глаза игрока)
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 2, 5); // Поднимаем камеру на уровень роста

            // Рендерер для вывода графики на экран телефона
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            document.body.appendChild(renderer.domElement);

            // ВАЖНО: Освещение сцены (без него объемные кубы сольются в кашу)
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.6); // Равномерный свет со всех сторон[span_4](start_span)[span_4](end_span)
            scene.add(ambientLight);

            const sunLight = new THREE.DirectionalLight(0xffffff, 0.8); // Яркое солнце, создающее тени[span_5](start_span)[span_5](end_span)
            sunLight.position.set(12, 24, 12);
            sunLight.castShadow = true;
            scene.add(sunLight);

            // Генерируем землю из воксельных кубов
            createWorld();

            // --- ДОБАВЛЕНИЕ ИНСТРУМЕНТА В РУКУ ИГРОКА ---
            // Генерируем наш Алмазный Меч через функцию[span_6](start_span)[span_6](end_span)
            currentWeaponMesh = createVoxelTool('diamond_sword');
            
            // Прикрепляем его жестко к камере, чтобы он двигался за взглядом[span_7](start_span)[span_7](end_span)
            camera.add(currentWeaponMesh);
            scene.add(camera); // Добавляем камеру с мечом в сцену

            // Идеальное позиционирование в правой нижней части экрана[span_8](start_span)[span_8](end_span)
            currentWeaponMesh.position.set(0.35, -0.45, -0.7); 

            // НАКЛОН И ПОВОРОТ: Меч стоит ВЕРТИКАЛЬНО и слегка наклонен вперед в руке[span_9](start_span)[span_9](end_span)[span_10](start_span)[span_10](end_span)!
            currentWeaponMesh.rotation.set(Math.PI / 5, -Math.PI / 7, Math.PI / 12);

            // Следим за поворотом экрана смартфона
            window.addEventListener('resize', onWindowResize, false);
            
            // Запускаем бесконечный игровой цикл обновления кадров
            animate();
        }

        // --- 2. ФУНКЦИЯ СБОРКИ ОБЪЁМНЫХ 3D-ИНСТРУМЕНТОВ ---
        function createVoxelTool(type) {
            const toolGroup = new THREE.Group();
            
            // Материалы для кубиков оружия[span_11](start_span)[span_11](end_span)
            const woodMat = new THREE.MeshStandardMaterial({ color: 0x5c4033, roughness: 0.9 }); // Рукоять[span_12](start_span)[span_12](end_span)
            const ironMat = new THREE.MeshStandardMaterial({ color: 0xdcdcdc, roughness: 0.4, metalness: 0.3 }); // Железо[span_13](start_span)[span_13](end_span)
            const diamondMat = new THREE.MeshStandardMaterial({ 
                color: 0x00ffff, 
                roughness: 0.1, 
                metalness: 0.6,
                emissive: 0x001a24 // Красивый внутренний отсвет алмаза[span_14](start_span)[span_14](end_span)
            });

            if (type === 'diamond_sword') {
                // =========================================================
                // НАСТОЯЩИЙ ОБЪЁМНЫЙ АЛМАЗНЫЙ МЕЧ
                // =========================================================
                
                // 1. Рукоятка (Деревянный брус с объемом)[span_15](start_span)[span_15](end_span)
                const handle = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.4, 0.06), woodMat);
                handle.position.y = 0.2; // Точка привязки (pivot) в самом низу[span_16](start_span)[span_16](end_span)
                toolGroup.add(handle);

                // 2. Гарда / Крестовина (Толстый защитный блок)[span_17](start_span)[span_17](end_span)
                const guard = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.08, 0.12), diamondMat);
                guard.position.y = 0.4;
                toolGroup.add(guard);

                // 3. Лезвие (Широкая, толстая алмазная плита — не палка!)[span_18](start_span)[span_18](end_span)
                const blade = new THREE.Mesh(new THREE.BoxGeometry(0.14, 1.1, 0.06), diamondMat);
                blade.position.y = 0.9;
                toolGroup.add(blade);
                
                // 4. Острие меча (Ромб на самом верху)[span_19](start_span)[span_19](end_span)
                const tip = new THREE.Mesh(new THREE.BoxGeometry(0.1, 0.1, 0.06), diamondMat);
                tip.position.y = 1.45;
                tip.rotation.z = Math.PI / 4; // Поворачиваем кубик под углом 45°[span_20](start_span)[span_20](end_span)
                toolGroup.add(tip);

            } else if (type === 'pickaxe') {
                // =========================================================
                // ОБЪЁМНАЯ КИРКА (Если захочешь переключить)
                // =========================================================
                const shaft = new THREE.Mesh(new THREE.BoxGeometry(0.06, 1.3, 0.06), woodMat);
                shaft.position.y = 0.65;
                toolGroup.add(shaft);

                const pickHead = new THREE.Mesh(new THREE.BoxGeometry(0.8, 0.1, 0.1), ironMat);
                pickHead.position.y = 1.25;
                toolGroup.add(pickHead);

                const tipL = new THREE.Mesh(new THREE.BoxGeometry(0.08, 0.08, 0.1), ironMat);
                tipL.position.set(-0.4, 1.21, 0);
                toolGroup.add(tipL);

                const tipR = tipL.clone();
                tipR.position.x = 0.4;
                toolGroup.add(tipR);
            }

            // ЖЕСТКИЙ СБРОС ВРАЩЕНИЯ: Выпрямляем объект строго по вертикали[span_21](start_span)[span_21](end_span)!
            toolGroup.rotation.set(0, 0, 0); 
            return toolGroup;
        }

        // --- 3. СОЗДАНИЕ КАРТЫ ИЗ БЛОКОВ (КАК В MINECRAFT) ---
        function createWorld() {
            const blockGeo = new THREE.BoxGeometry(1, 1, 1);
            const grassMat = new THREE.MeshStandardMaterial({ color: 0x559933, roughness: 0.8 });

            // Строим зелёную платформу под ногами размером 12х12 кубиков
            for (let x = -6; x < 6; x++) {
                for (let z = -6; z < 6; z++) {
                    const block = new THREE.Mesh(blockGeo, grassMat);
                    block.position.set(x, 0, z);
                    block.receiveShadow = true;
                    scene.add(block);
                    terrainBlocks.push(block); // Сохраняем блоки в массив
                }
            }
        }

        // --- 4. МЕСТО ДЛЯ ПОДКЛЮЧЕНИЯ СЕТЕВОГО ОНЛАЙНА ---
        function sendMyCoordinatesToNetwork() {
            // Когда мы подключим Firebase или Socket.io, этот код будет брать 
            // координаты твоей камеры и мгновенно отсылать их друзьям:
            /*
            database.ref('players/' + myID).set({
                x: camera.position.x,
                y: camera.position.y,
                z: camera.position.z
            });
            */
        }

        // --- 5. ИГРОВОЙ ЦИКЛ ОБНОВЛЕНИЯ АНИМАЦИИ ---
        function animate() {
            requestAnimationFrame(animate);

            // Эффект "живого" дыхания: меч плавно покачивается вверх-вниз в руке
            const animationTime = Date.now() * 0.0025;
            if (currentWeaponMesh) {
                currentWeaponMesh.position.y = -0.45 + Math.sin(animationTime) * 0.012;
                currentWeaponMesh.position.x = 0.35 + Math.cos(animationTime) * 0.005;
            }

            // Постоянно отправляем наши координаты в сеть (заготовка для онлайна)
            sendMyCoordinatesToNetwork();

            // Отрисовываем сцену через камеру
            renderer.render(scene, camera);
        }

        // Корректное отображение при повороте экрана телефона
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // Запуск игры сразу после загрузки страницы в браузере телефона
        window.onload = init;
    </script>
</body>
</html>

