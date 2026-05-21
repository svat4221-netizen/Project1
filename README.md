<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>JS Мой Майнкрафт с Текстурами</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            -webkit-user-select: none;
            user-select: none;
        }
        
        html, body {
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #7ec0ee;
            position: fixed;
        }

        canvas {
            width: 100% !important;
            height: 100% !important;
            display: block;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }
        
        #ui {
            position: absolute;
            top: 15px;
            left: 15px;
            color: white;
            background: rgba(0, 0, 0, 0.6);
            padding: 10px 15px;
            border-radius: 6px;
            pointer-events: none;
            z-index: 10;
            font-size: 14px;
            font-family: monospace;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
        }
        
        #crosshair {
            position: absolute;
            top: 50%;
            left: 50%;
            width: 12px;
            height: 12px;
            background: white;
            transform: translate(-50%, -50%);
            pointer-events: none;
            opacity: 0.8;
            z-index: 10;
        }

        #hotbar {
            position: absolute;
            bottom: 15px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            background: rgba(0, 0, 0, 0.7);
            padding: 6px;
            border-radius: 8px;
            z-index: 10;
            max-width: 95%;
            overflow-x: auto;
        }

        .slot {
            width: 45px;
            height: 45px;
            border: 2px solid #555;
            margin: 0 3px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 10px;
            font-family: sans-serif;
            text-align: center;
            background: #222;
            cursor: pointer;
            border-radius: 4px;
            flex-shrink: 0;
        }

        .slot.active {
            border-color: #fff;
            background: #555;
            box-shadow: 0 0 10px #fff;
        }
        
        #controls {
            position: absolute;
            bottom: 20px;
            left: 20px;
            display: grid;
            grid-template-columns: repeat(3, 50px);
            grid-gap: 8px;
            z-index: 10;
        }

        .btn {
            width: 50px;
            height: 50px;
            background: rgba(255, 255, 255, 0.25);
            border: 2px solid rgba(255, 255, 255, 0.6);
            border-radius: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-weight: bold;
            font-size: 18px;
            backdrop-filter: blur(2px);
        }

        #action-btns {
            position: absolute;
            bottom: 20px;
            right: 20px;
            display: flex;
            flex-direction: column;
            gap: 12px;
            z-index: 10;
        }

        .action-btn {
            width: 65px;
            height: 65px;
            background: rgba(0, 0, 0, 0.5);
            border: 2px solid rgba(255, 255, 255, 0.8);
            border-radius: 50%;
            color: white;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 11px;
            font-weight: bold;
            box-shadow: 0 4px 8px rgba(0,0,0,0.4);
            backdrop-filter: blur(2px);
        }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <div id="ui">
        <div id="info">В руках: Земля</div>
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
        // --- ГЕНЕРАТОР ПИКСЕЛЬНЫХ ТЕКСТУР НА НАЧЕХУ ---
        function generatePixelTexture(type) {
            const canvas = document.createElement('canvas');
            canvas.width = 16;  // Стандартное разрешение Майнкрафта 16х16 пикселей
            canvas.height = 16;
            const ctx = canvas.getContext('2d');

            // Генерация шума для пикселей
            for (let x = 0; x < 16; x++) {
                for (let y = 0; y < 16; y++) {
                    let r, g, b;
                    let noise = Math.random() * 20 - 10; // Рандомный оттенок для пикселя

                    if (type === 'dirt') { // Земля
                        r = 85 + noise; g = 51 + noise; b = 17 + noise;
                        // Сверху добавим немного зеленой травы на верхнюю грань куба
                        if (y < 3 && Math.random() > 0.3) { r = 40 + noise; g = 115 + noise; b = 35 + noise; }
                    } 
                    else if (type === 'sand') { // Песок
                        r = 215 + noise; g = 195 + noise; b = 135 + noise;
                        if (Math.random() > 0.8) { r -= 20; g -= 20; } // Эффект песчинок
                    } 
                    else if (type === 'gravel') { // Гравий
                        r = 110 + noise; g = 110 + noise; b = 115 + noise;
                        if (Math.random() > 0.7) { r -= 30; g -= 30; b -= 30; } // Камушки
                    } 
                    else if (type === 'stone') { // Камень
                        r = 125 + noise; g = 125 + noise; b = 125 + noise;
                    } 
                    else if (type === 'furnace') { // Печка
                        r = 60 + noise; g = 60 + noise; b = 65 + noise;
                        // Рисуем рамку печки
                        if (x === 0 || x === 15 || y === 0 || y === 15) { r = 40; g = 40; b = 40; }
                        // Имитация решетки по центру
                        if (y >= 6 && y <= 10 && x >= 4 && x <= 12) { r = 25; g = 25; b = 25; }
                    }

                    ctx.fillStyle = `rgb(${Math.floor(r)},${Math.floor(g)},${Math.floor(b)})`;
                    ctx.fillRect(x, y, 1, 1);
                }
            }

            const texture = new THREE.CanvasTexture(canvas);
            texture.magFilter = THREE.NearestFilter; // Важно! Убирает размытие, делая текстуру пиксельной
            texture.minFilter = THREE.NearestFilter;
            return texture;
        }

        // --- ИНИЦИАЛИЗАЦИЯ СЦЕНЫ ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x7ec0ee);
        scene.fog = new THREE.FogExp2(0x7ec0ee, 0.06);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: false }); // Отключаем сглаживание для олдскульного вида
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // Освещение (направленное, чтобы блоки имели объемные тени)
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
        scene.add(ambientLight);
        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.4);
        directionalLight.position.set(15, 30, 15);
        scene.add(directionalLight);

        // Создаем текстуры
        const textures = {
            dirt: generatePixelTexture('dirt'),
            sand: generatePixelTexture('sand'),
            gravel: generatePixelTexture('gravel'),
            stone: generatePixelTexture('stone'),
            furnace: generatePixelTexture('furnace')
        };

        // Настройки предметов
        const BLOCK_TYPES = {
            1: { name: 'Земля', texture: textures.dirt, isBlock: true },
            2: { name: 'Песок', texture: textures.sand, isBlock: true },
            3: { name: 'Гравий', texture: textures.gravel, isBlock: true },
            4: { name: 'Камень', texture: textures.stone, isBlock: true },
            5: { name: 'Кирка', isBlock: false },
            6: { name: 'Меч', isBlock: false },
            7: { name: 'Печь', texture: textures.furnace, isBlock: true }
        };
        
        let activeSlot = 1;
        const worldBlocks = [];
        const geometry = new THREE.BoxGeometry(1, 1, 1);

        let moveForward = false, moveBackward = false, moveLeft = false, moveRight = false, canJump = false;
        let velocity = new THREE.Vector3();
        let direction = new THREE.Vector3();
        const playerSpeed = 0.08;
        const gravity = 0.008;
        
        camera.position.set(5, 3, 5);
        let pitch = 0, yaw = 0;

        // Создание блока в мире
        function createBlock(x, y, z, typeId) {
            const material = new THREE.MeshLambertMaterial({ map: BLOCK_TYPES[typeId].texture });
            const mesh = new THREE.Mesh(geometry, material);
            mesh.position.set(x, y, z);
            mesh.blockTypeId = typeId;
            scene.add(mesh);
            worldBlocks.push(mesh);
        }

        // Рендерим мир
        const worldSize = 10;
        for(let x = 0; x < worldSize; x++) {
            for(let z = 0; z < worldSize; z++) {
                createBlock(x, 0, z, 4); // Камень вниз
                let rand = Math.random();
                let topType = 1; // Земля по дефолту
                if (rand < 0.15) topType = 2; // Песок
                else if (rand < 0.3) topType = 3; // Гравий
                createBlock(x, 1, z, topType);
            }
        }
        createBlock(4, 2, 4, 7); // Стартовая печка

        // Ломание и установка блоков
        const raycaster = new THREE.Raycaster();
        const mousePointer = new THREE.Vector2(0, 0);

        function handleAction(type) {
            raycaster.setFromCamera(mousePointer, camera);
            const intersects = raycaster.intersectObjects(worldBlocks);

            if (intersects.length > 0 && intersects[0].distance < 5) {
                const hitBlock = intersects[0].object;

                if (type === 'break') {
                    if(hitBlock.blockTypeId == 7) {
                        document.getElementById('stats').innerText = "Печь сломана!";
                    }
                    scene.remove(hitBlock);
                    worldBlocks.splice(worldBlocks.indexOf(hitBlock), 1);
                } 
                else if (type === 'place') {
                    const current = BLOCK_TYPES[activeSlot];
                    if (current.isBlock) {
                        const normal = intersects[0].face.normal;
                        const newPos = hitBlock.position.clone().add(normal);
                        createBlock(newPos.x, newPos.y, newPos.z, activeSlot);
                        
                        if(activeSlot == 7) {
                            document.getElementById('stats').innerText = "Вы поставили печь! Кликните по ней Киркой.";
                        }
                    } else if (activeSlot == 5 && hitBlock.blockTypeId == 7) {
                        document.getElementById('stats').innerText = "Печь топится: Переплавка гравия...";
                        setTimeout(() => { document.getElementById('stats').innerText = "Успех! Получен чистый Камень."; }, 2000);
                    }
                }
            }
        }

        window.selectSlot = function(slotId) {
            activeSlot = slotId;
            const slots = document.querySelectorAll('.slot');
            slots.forEach((s, idx) => {
                s.classList.toggle('active', idx === (slotId - 1));
            });
            document.getElementById('info').innerText = "В руках: " + BLOCK_TYPES[slotId].name;
        }

        // --- СЕНСОР И КАМЕРА ---
        let previousTouch;

        window.addEventListener('touchstart', (e) => {
            if(e.target.tagName === 'CANVAS' || e.target.id === 'crosshair') {
                previousTouch = e.touches[0];
            }
        }, { passive: false });

        window.addEventListener('touchmove', (e) => {
            if(!previousTouch || e.touches.length > 1) return;
            const touch = e.touches[0];
            const movementX = touch.pageX - previousTouch.pageX;
            const movementY = touch.pageY - previousTouch.pageY;
            
            yaw -= movementX * 0.006;
            pitch -= movementY * 0.006;
            pitch = Math.max(-Math.PI/2.2, Math.min(Math.PI/2.2, pitch));
            
            previousTouch = touch;
        }, { passive: false });

        window.addEventListener('touchend', () => { previousTouch = null; });

        const setupMobileBtn = (id, press, release) => {
            const btn = document.getElementById(id);
            if(!btn) return;
            btn.addEventListener('touchstart', (e) => { e.preventDefault(); press(); });
            btn.addEventListener('touchend', (e) => { e.preventDefault(); release(); });
        };

        setupMobileBtn('btn-w', () => moveForward = true, () => moveForward = false);
        setupMobileBtn('btn-s', () => moveBackward = true, () => moveBackward = false);
        setupMobileBtn('btn-a', () => moveLeft = true, () => moveLeft = false);
        setupMobileBtn('btn-d', () => moveRight = true, () => moveRight = false);
        setupMobileBtn('btn-jump', () => { if(canJump) { velocity.y = 0.12; canJump = false; } }, () => {});
        setupMobileBtn('btn-break', () => handleAction('break'), () => {});
        setupMobileBtn('btn-place', () => handleAction('place'), () => {});

        // --- ИГРОВОЙ ЦИКЛ ---
        function animate() {
            requestAnimationFrame(animate);

            const qx = new THREE.Quaternion();
            qx.setFromAxisAngle(new THREE.Vector3(1, 0, 0), pitch);
            const qy = new THREE.Quaternion();
            qy.setFromAxisAngle(new THREE.Vector3(0, 1, 0), yaw);
            camera.quaternion.copy(qy).multiply(qx);

            velocity.y -= gravity;
            camera.position.y += velocity.y;

            if (camera.position.y < 2.5) {
                velocity.y = 0;
                camera.position.y = 2.5;
                canJump = true;
            }

            direction.z = Number(moveForward) - Number(moveBackward);
            direction.x = Number(moveRight) - Number(moveLeft);
            direction.normalize();

            const camYawMat = new THREE.Matrix4().makeRotationY(yaw);
            const moveVector = new THREE.Vector3(direction.x, 0, -direction.z).applyMatrix4(camYawMat);
            camera.position.addScaledVector(moveVector, playerSpeed);

            renderer.render(scene, camera);
        }

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>



