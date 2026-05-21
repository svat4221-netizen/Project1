<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>JS Мой Майнкрафт</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: sans-serif; user-select: none; }
        canvas { display: block; }
        
        /* Интерфейс (UI) */
        #ui {
            position: absolute; top: 10px; left: 10px; color: white;
            background: rgba(0,0,0,0.5); padding: 10px; border-radius: 5px; pointer-events: none;
        }
        #crosshair {
            position: absolute; top: 50%; left: 50%; width: 10px; height: 10px;
            background: white; transform: translate(-50%, -50%); pointer-events: none; opacity: 0.7;
        }
        #hotbar {
            position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%);
            display: flex; background: rgba(0,0,0,0.6); padding: 5px; border-radius: 5px;
        }
        .slot {
            width: 50px; height: 50px; border: 2px solid #555; margin: 0 3px;
            display: flex; align-items: center; justify-content: center; color: white;
            font-size: 11px; text-align: center; background: #222; cursor: pointer;
        }
        .slot.active { border-color: #fff; background: #444; }
        
        /* Кнопки управления для телефона */
        #controls {
            position: absolute; bottom: 20px; left: 20px; display: grid;
            grid-template-columns: repeat(3, 40px); grid-gap: 5px;
        }
        .btn {
            width: 40px; height: 40px; background: rgba(255,255,255,0.3);
            border: 1px solid white; border-radius: 5px; display: flex;
            align-items: center; justify-content: center; color: white; font-weight: bold;
        }
        #action-btns {
            position: absolute; bottom: 20px; right: 20px; display: flex; flex-direction: column; gap: 10px;
        }
        .action-btn {
            width: 60px; height: 60px; background: rgba(0,0,0,0.5); border: 2px solid white;
            border-radius: 50%; color: white; display: flex; align-items: center; justify-content: center; font-size: 12px;
        }
    </style>
    <!-- Подключаем 3D движок Three.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <div id="ui">
        <div id="info">Инструмент: Земля</div>
        <div id="stats">Печь: готова к работе</div>
    </div>
    <div id="crosshair"></div>
    
    <div id="hotbar">
        <div class="slot active" onclick="selectSlot(1)">Земля</div>
        <div class="slot" onclick="selectSlot(2)">Песок</div>
        <div class="slot" onclick="selectSlot(3)">Гравий</div>
        <div class="slot" onclick="selectSlot(4)">Камень</div>
        <div class="slot" onclick="selectSlot(5)">Кирка</div>
        <div class="slot" onclick="selectSlot(6)">Меч</div>
        <div class="slot" onclick="selectSlot(7)">Печь</div>
    </div>

    <!-- Сенсорное управление для мобилок -->
    <div id="controls">
        <div></div><div class="btn" id="btn-w">W</div><div></div>
        <div class="btn" id="btn-a">A</div><div class="btn" id="btn-s">S</div><div class="btn" id="btn-d">D</div>
    </div>

    <div id="action-btns">
        <div class="action-btn" id="btn-break">ЛОМАТЬ</div>
        <div class="action-btn" id="btn-place">СТАВИТЬ</div>
        <div class="action-btn" id="btn-jump">ПРЫЖОК</div>
    </div>

    <script>
        // --- НАСТРОЙКИ И ПЕРЕМЕННЫЕ ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x7ec0ee); // Цвет неба
        scene.fog = new THREE.FogExp2(0x7ec0ee, 0.05);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // Освещение
        const ambientLight = new THREE.AmbientLight(0xcccccc, 0.6);
        scene.add(ambientLight);
        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.6);
        directionalLight.position.set(10, 20, 15);
        scene.add(directionalLight);

        // Физика и игрок
        let moveForward = false, moveBackward = false, moveLeft = false, moveRight = false, canJump = false;
        let velocity = new THREE.Vector3();
        let direction = new THREE.Vector3();
        const playerSpeed = 0.1;
        const gravity = 0.01;
        
        camera.position.set(5, 5, 5);
        let pitch = 0, yaw = 0; // Для вращения камеры пальцем/мышкой

        // Игровой мир и типы блоков (цвета вместо тяжелых текстур, чтобы не лагало)
        const BLOCK_TYPES = {
            1: { name: 'Земля', color: 0x553311, isBlock: true },
            2: { name: 'Песок', color: 0xddcc99, isBlock: true },
            3: { name: 'Гравий', color: 0x777777, isBlock: true },
            4: { name: 'Камень', color: 0x888888, isBlock: true },
            5: { name: 'Кирка', isBlock: false },
            6: { name: 'Меч', isBlock: false },
            7: { name: 'Печь', color: 0x333333, isBlock: true, isFurnace: true }
        };
        let activeSlot = 1;
        const worldBlocks = []; // Массив для проверки столкновений и кликов
        const geometry = new THREE.BoxGeometry(1, 1, 1);

        // --- ГЕНЕРАЦИЯ ОПТИМИЗИРОВАННОГО МИРА ---
        const worldSize = 12; // Небольшой размер, идеальный для телефонов
        
        function createBlock(x, y, z, typeId) {
            const material = new THREE.MeshLambertMaterial({ color: BLOCK_TYPES[typeId].color });
            const mesh = new THREE.Mesh(geometry, material);
            mesh.position.set(x, y, z);
            mesh.blockTypeId = typeId;
            scene.add(mesh);
            worldBlocks.push(mesh);
        }

        // Строим только верхнюю «корку», чтобы не тратить память смартфона
        for(let x = 0; x < worldSize; x++) {
            for(let z = 0; z < worldSize; z++) {
                createBlock(x, 0, z, 4); // Каменное основание
                
                // Рандомим ландшафт: где-то песок, где-то земля или гравий
                let rand = Math.random();
                let topType = 1; // Земля по умолчанию
                if (rand < 0.2) topType = 2; // Песок
                else if (rand < 0.4) topType = 3; // Гравий

                createBlock(x, 1, z, topType); 
            }
        }

        // Поставим одну готовую печку в мире для примера
        createBlock(5, 2, 5, 7);

        // --- ЛОГИКА ИНСТРУМЕНТОВ И ВЗАИМОДЕЙСТВИЯ ---
        const raycaster = new THREE.Raycaster();
        const mousePointer = new THREE.Vector2(0, 0); // Центр экрана

        function handleAction(type) {
            // Направляем луч из центра экрана вперед
            raycaster.setFromCamera(mousePointer, camera);
            const intersects = raycaster.intersectObjects(worldBlocks);

            if (intersects.length > 0 && intersects[0].distance < 4) {
                const hitBlock = intersects[0].object;

                if (type === 'break') {
                    // Механика печки: если ломаем печь
                    if(hitBlock.blockTypeId == 7) {
                        document.getElementById('stats').innerText = "Печь сломана!";
                    }
                    // Убираем блок из сцены и массива
                    scene.remove(hitBlock);
                    worldBlocks.splice(worldBlocks.indexOf(hitBlock), 1);
                } 
                else if (type === 'place') {
                    const current = BLOCK_TYPES[activeSlot];
                    if (current.isBlock) {
                        // Вычисляем позицию для нового блока на основе нормали стороны
                        const normal = intersects[0].face.normal;
                        const newPos = hitBlock.position.clone().add(normal);
                        createBlock(newPos.x, newPos.y, newPos.z, activeSlot);
                        
                        if(activeSlot == 7) {
                            document.getElementById('stats').innerText = "Вы поставили печь! Нажмите на неё Киркой.";
                        }
                    } else if (activeSlot == 5 && hitBlock.blockTypeId == 7) {
                        // Если в руках кирка и кликаем по печке — «активируем» её
                        document.getElementById('stats').innerText = "Печь работает: Переплавка гравия в камень...";
                        setTimeout(() => { document.getElementById('stats').innerText = "Готово! Получен чистый Камень."; }, 2000);
                    }
                }
            }
        }

        // Выбор слота инвентаря
        window.selectSlot = function(slotId) {
            activeSlot = slotId;
            const slots = document.querySelectorAll('.slot');
            slots.forEach((s, idx) => {
                s.classList.toggle('active', idx === (slotId - 1));
            });
            document.getElementById('info').innerText = "В руках: " + BLOCK_TYPES[slotId].name;
        }

        // --- УПРАВЛЕНИЕ И СЕНСОР (ПК + ТЕЛЕФОН) ---
        
        // Поворот камеры мышкой (ПК) или пальцем (Смартфон)
        let isMovingCamera = false;
        let previousTouch;

        window.addEventListener('mousedown', () => isMovingCamera = true);
        window.addEventListener('mouseup', () => isMovingCamera = false);
        window.addEventListener('mousemove', (e) => {
            if (!isMovingCamera) return;
            yaw -= e.movementX * 0.003;
            pitch -= e.movementY * 0.003;
            pitch = Math.max(-Math.PI/2.1, Math.min(Math.PI/2.1, pitch));
        });

        // Сенсорный поворот для телефонов
        window.addEventListener('touchstart', (e) => {
            if(e.target.tagName !== 'DIV') return; // Чтобы не ломать клики по кнопкам UI
            previousTouch = e.touches[0];
        });
        window.addEventListener('touchmove', (e) => {
            if(!previousTouch || e.touches.length > 1) return;
            const touch = e.touches[0];
            const movementX = touch.pageX - previousTouch.pageX;
            const movementY = touch.pageY - previousTouch.pageY;
            
            yaw -= movementX * 0.005;
            pitch -= movementY * 0.005;
            pitch = Math.max(-Math.PI/2.1, Math.min(Math.PI/2.1, pitch));
            
            previousTouch = touch;
        });

        // Кнопки клавиатуры для ПК
        window.addEventListener('keydown', (e) => {
            if(e.code === 'KeyW') moveForward = true;
            if(e.code === 'KeyS') moveBackward = true;
            if(e.code === 'KeyA') moveLeft = true;
            if(e.code === 'KeyD') moveRight = true;
            if(e.code === 'Space' && canJump) { velocity.y = 0.15; canJump = false; }
            if(e.code === 'Digit1') selectSlot(1);
            if(e.code === 'Digit2') selectSlot(2);
            if(e.code === 'Digit3') selectSlot(3);
            if(e.code === 'Digit4') selectSlot(4);
            if(e.code === 'Digit5') selectSlot(5);
            if(e.code === 'Digit6') selectSlot(6);
            if(e.code === 'Digit7') selectSlot(7);
        });
        window.addEventListener('keyup', (e) => {
            if(e.code === 'KeyW') moveForward = false;
            if(e.code === 'KeyS') moveBackward = false;
            if(e.code === 'KeyA') moveLeft = false;
            if(e.code === 'KeyD') moveRight = false;
        });
        window.addEventListener('click', (e) => {
            if(e.target.tagName === 'CANVAS') handleAction('break');
        });

        // Привязка экранных кнопок для смартфона
        const setupMobileBtn = (id, pressAction, releaseAction) => {
            const btn = document.getElementById(id);
            btn.addEventListener('touchstart', (e) => { e.preventDefault(); pressAction(); });
            btn.addEventListener('touchend', (e) => { e.preventDefault(); releaseAction(); });
        };

        setupMobileBtn('btn-w', () => moveForward = true, () => moveForward = false);
        setupMobileBtn('btn-s', () => moveBackward = true, () => moveBackward = false);
        setupMobileBtn('btn-a', () => moveLeft = true, () => moveLeft = false);
        setupMobileBtn('btn-d', () => moveRight = true, () => moveRight = false);
        setupMobileBtn('btn-jump', () => { if(canJump) { velocity.y = 0.15; canJump = false; } }, () => {});
        setupMobileBtn('btn-break', () => handleAction('break'), () => {});
        setupMobileBtn('btn-place', () => handleAction('place'), () => {});


        // --- ИГРОВОЙ ЦИКЛ (АНИМАЦИЯ И ФИЗИКА) ---
        function animate() {
            requestAnimationFrame(animate);

            // Направление взгляда камеры
            const qx = new THREE.Quaternion();
            qx.setFromAxisAngle(new THREE.Vector3(1, 0, 0), pitch);
            const qy = new THREE.Quaternion();
            qy.setFromAxisAngle(new THREE.Vector3(0, 1, 0), yaw);
            camera.quaternion.copy(qy).multiply(qx);

            // Физика гравитации и падения
            velocity.y -= gravity;
            camera.position.y += velocity.y;

            // Очень простая проверка коллизии с «землей» (высота y=2 — уровень ходьбы по блокам)
            if (camera.position.y < 2.5) {
                velocity.y = 0;
                camera.position.y = 2.5;
                canJump = true;
            }

            // Движение игрока
            direction.z = Number(moveForward) - Number(moveBackward);
            direction.x = Number(moveRight) - Number(moveLeft);
            direction.normalize();

            // Переводим локальное движение относительно взгляда камеры в мировые координаты
            const camYawMat = new THREE.Matrix4().makeRotationY(yaw);
            const moveVector = new THREE.Vector3(direction.x, 0, -direction.z).applyMatrix4(camYawMat);
            
            camera.position.addScaledVector(moveVector, playerSpeed);

            renderer.render(scene, camera);
        }

        // Авто подгон размера при повороте экрана телефона
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        // Запуск
        animate();
    </script>
</body>
</html>




