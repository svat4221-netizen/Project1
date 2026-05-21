<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>JS Мой Майнкрафт</title>
    <style>
        /* Полный сброс отступов, чтобы игра занимала 100% экрана */
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
            position: fixed; /* Предотвращает случайные сдвиги экрана на телефоне */
        }

        canvas {
            width: 100% !important;
            height: 100% !important;
            display: block;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1; /* Игра строго на заднем фоне */
        }
        
        /* Интерфейс (UI) поверх игры */
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
            max-width: 90%;
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
        
        /* Кнопки управления (Джойстик слева) */
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

        /* Действия справа */
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
        // --- ИНИЦИАЛИЗАЦИЯ СЦЕНЫ ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x7ec0ee);
        scene.fog = new THREE.FogExp2(0x7ec0ee, 0.06);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // Оптимизация под Retina/AMOLED экраны
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // Свет
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);
        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
        directionalLight.position.set(20, 40, 20);
        scene.add(directionalLight);

        // Игрок и физика
        let moveForward = false, moveBackward = false, moveLeft = false, moveRight = false, canJump = false;
        let velocity = new THREE.Vector3();
        let direction = new THREE.Vector3();
        const playerSpeed = 0.08;
        const gravity = 0.008;
        
        camera.position.set(5, 3, 5);
        let pitch = 0, yaw = 0;

        // Блоки
        const BLOCK_TYPES = {
            1: { name: 'Земля', color: 0x553311, isBlock: true },
            2: { name: 'Песок', color: 0xddcc99, isBlock: true },
            3: { name: 'Гравий', color: 0x777777, isBlock: true },
            4: { name: 'Камень', color: 0x888888, isBlock: true },
            5: { name: 'Кирка', isBlock: false },
            6: { name: 'Меч', isBlock: false },
            7: { name: 'Печь', color: 0x333333, isBlock: true }
        };
        let activeSlot = 1;
        const worldBlocks = [];
        const geometry = new THREE.BoxGeometry(1, 1, 1);

        // Генерация мира (оптимизированная под телефоны)
        const worldSize = 10;
        function createBlock(x, y, z, typeId) {
            const material = new THREE.MeshLambertMaterial({ color: BLOCK_TYPES[typeId].color });
            const mesh = new THREE.Mesh(geometry, material);
            mesh.position.set(x, y, z);
            mesh.blockTypeId = typeId;
            scene.add(mesh);
            worldBlocks.push(mesh);
        }

        for(let x = 0; x < worldSize; x++) {
            for(let z = 0; z < worldSize; z++) {
                createBlock(x, 0, z, 4); // Нижний слой камня
                let rand = Math.random();
                let topType = 1;
                if (rand < 0.15) topType = 2;
                else if (rand < 0.3) topType = 3;
                createBlock(x, 1, z, topType); // Верхний слой ландшафта
            }
        }
        createBlock(4, 2, 4, 7); // Стартовая печка

        // Взаимодействие (Ломать / Ставить)
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
                            document.getElementById('stats').innerText = "Вы поставили печь! Нажмите на неё Киркой.";
                        }
                    } else if (activeSlot == 5 && hitBlock.blockTypeId == 7) {
                        document.getElementById('stats').innerText = "Переплавка гравия в камень...";
                        setTimeout(() => { document.getElementById('stats').innerText = "Готово! Получен Камень."; }, 2000);
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

        // --- УПРАВЛЕНИЕ И СЕНСОРНЫЙ ВЗГЛЯД ---
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

        // Настройка экранных кнопок
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

        // Поддержка ПК клавиатуры на всякий случай
        window.addEventListener('keydown', (e) => {
            if(e.code === 'KeyW') moveForward = true;
            if(e.code === 'KeyS') moveBackward = true;
            if(e.code === 'KeyA') moveLeft = true;
            if(e.code === 'KeyD') moveRight = true;
            if(e.code === 'Space' && canJump) { velocity.y = 0.12; canJump = false; }
        });
        window.addEventListener('keyup', (e) => {
            if(e.code === 'KeyW') moveForward = false;
            if(e.code === 'KeyS') moveBackward = false;
            if(e.code === 'KeyA') moveLeft = false;
            if(e.code === 'KeyD') moveRight = false;
        });

        // --- ИГРОВОЙ ЦИКЛ ---
        function animate() {
            requestAnimationFrame(animate);

            // Вращение камеры
            const qx = new THREE.Quaternion();
            qx.setFromAxisAngle(new THREE.Vector3(1, 0, 0), pitch);
            const qy = new THREE.Quaternion();
            qy.setFromAxisAngle(new THREE.Vector3(0, 1, 0), yaw);
            camera.quaternion.copy(qy).multiply(qx);

            // Физика гравитации
            velocity.y -= gravity;
            camera.position.y += velocity.y;

            if (camera.position.y < 2.5) {
                velocity.y = 0;
                camera.position.y = 2.5;
                canJump = true;
            }

            // Движение
            direction.z = Number(moveForward) - Number(moveBackward);
            direction.x = Number(moveRight) - Number(moveLeft);
            direction.normalize();

            const camYawMat = new THREE.Matrix4().makeRotationY(yaw);
            const moveVector = new THREE.Vector3(direction.x, 0, -direction.z).applyMatrix4(camYawMat);
            camera.position.addScaledVector(moveVector, playerSpeed);

            renderer.render(scene, camera);
        }

        // Адаптация под поворот экрана смартфона
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>







