<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>3D Модели Инструментов в Майнкрафт</title>
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
        <div class="slot" onclick="selectSlot(3)">Камень</div>
        <div class="slot" onclick="selectSlot(4)">Кирка</div>
        <div class="slot" onclick="selectSlot(5)">Меч</div>
        <div class="slot" onclick="selectSlot(6)">Лопата</div>
        <div class="slot" onclick="selectSlot(7)">Мотыга</div>
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
        // --- ЦВЕТА И МАТРИЦЫ ДЛЯ 3D МОДЕЛЕЙ ИНСТРУМЕНТОВ ---
        const C = { W: 0x4a3b2c, S: 0xdddddd, D: 0x4dedf0, B: 0x222222, X: 0x000000 }; // W-дерево, S-сталь, D-алмаз, B-рамка/тень

        // Сетки моделей 12x12 (0 - пустота)
        const MODELS = {
            pickaxe: [
                [0,0,D,D,D,D,0,0,0,0,0,0],
                [0,D,D,S,S,D,D,0,0,0,0,0],
                [0,D,S,0,0,S,D,0,0,0,0,0],
                [0,0,0,0,0,W,D,0,0,0,0,0],
                [0,0,0,0,W,0,0,0,0,0,0,0],
                [0,0,0,W,0,0,0,0,0,0,0,0],
                [0,0,W,0,0,0,0,0,0,0,0,0],
                [0,W,0,0,0,0,0,0,0,0,0,0],
                [W,0,0,0,0,0,0,0,0,0,0,0],
            ],
            sword: [
                [0,0,0,0,0,0,0,0,0,D,D,0],
                [0,0,0,0,0,0,0,0,D,D,D,0],
                [0,0,0,0,0,0,0,D,D,D,0,0],
                [0,0,0,0,0,0,D,D,D,0,0,0],
                [0,0,0,0,0,D,D,D,0,0,0,0],
                [0,0,0,0,D,D,D,0,0,0,0,0],
                [0,0,0,D,D,D,0,0,0,0,0,0],
                [0,0,S,D,D,0,0,0,0,0,0,0],
                [0,S,S,S,0,0,0,0,0,0,0,0],
                [0,0,S,0,0,0,0,0,0,0,0,0],
                [0,W,0,0,0,0,0,0,0,0,0,0],
                [W,0,0,0,0,0,0,0,0,0,0,0],
            ],
            shovel: [
                [0,0,0,0,0,0,0,0,D,D,D,0],
                [0,0,0,0,0,0,0,D,D,D,D,0],
                [0,0,0,0,0,0,0,D,D,D,0,0],
                [0,0,0,0,0,0,0,0,W,0,0,0],
                [0,0,0,0,0,0,0,W,0,0,0,0],
                [0,0,0,0,0,0,W,0,0,0,0,0],
                [0,0,0,0,0,W,0,0,0,0,0,0],
                [0,0,0,0,W,0,0,0,0,0,0,0],
                [0,0,0,W,0,0,0,0,0,0,0,0],
                [0,0,W,0,0,0,0,0,0,0,0,0],
                [0,W,0,0,0,0,0,0,0,0,0,0],
                [W,0,0,0,0,0,0,0,0,0,0,0],
            ],
            hoe: [
                [0,0,D,D,D,D,D,0,0,0,0,0],
                [0,0,D,D,S,S,D,0,0,0,0,0],
                [0,0,0,0,0,S,D,0,0,0,0,0],
                [0,0,0,0,0,W,0,0,0,0,0,0],
                [0,0,0,0,W,0,0,0,0,0,0,0],
                [0,0,0,W,0,0,0,0,0,0,0,0],
                [0,0,W,0,0,0,0,0,0,0,0,0],
                [0,W,0,0,0,0,0,0,0,0,0,0],
                [W,0,0,0,0,0,0,0,0,0,0,0],
            ]
        };

        // Генерация пиксельных текстур блоков
        function generatePixelTexture(type) {
            const canvas = document.createElement('canvas'); canvas.width = 16; canvas.height = 16;
            const ctx = canvas.getContext('2d');
            for (let x = 0; x < 16; x++) {
                for (let y = 0; y < 16; y++) {
                    let r, g, b, noise = Math.random() * 20 - 10;
                    if (type === 'dirt') {
                        r = 85 + noise; g = 51 + noise; b = 17 + noise;
                        if (y < 3 && Math.random() > 0.3) { r = 40 + noise; g = 115 + noise; b = 35 + noise; }
                    } 
                    else if (type === 'sand') { r = 215 + noise; g = 195 + noise; b = 135 + noise; } 
                    else if (type === 'stone') { r = 125 + noise; g = 125 + noise; b = 125 + noise; } 
                    else if (type === 'skin') { r = 215 + noise; g = 145 + noise; b = 110 + noise; if(y < 5) { r = 30; g = 150; b = 150; } }
                    ctx.fillStyle = `rgb(${Math.floor(r)},${Math.floor(g)},${Math.floor(b)})`; ctx.fillRect(x, y, 1, 1);
                }
            }
            const texture = new THREE.CanvasTexture(canvas);
            texture.magFilter = THREE.NearestFilter; texture.minFilter = THREE.NearestFilter;
            return texture;
        }

        // --- ИНИЦИАЛИЗАЦИЯ ДВИЖКА ---
        const scene = new THREE.Scene(); scene.background = new THREE.Color(0x7ec0ee);
        scene.fog = new THREE.FogExp2(0x7ec0ee, 0.06);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: false });
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        const ambientLight = new THREE.AmbientLight(0xffffff, 0.8); scene.add(ambientLight);
        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.4); directionalLight.position.set(15, 30, 15); scene.add(directionalLight);

        const textures = { dirt: generatePixelTexture('dirt'), sand: generatePixelTexture('sand'), stone: generatePixelTexture('stone'), skin: generatePixelTexture('skin') };

        // Спецификации предметов инвентаря
        const BLOCK_TYPES = {
            1: { name: 'Земля', texture: textures.dirt, isBlock: true },
            2: { name: 'Песок', texture: textures.sand, isBlock: true },
            3: { name: 'Камень', texture: textures.stone, isBlock: true },
            4: { name: 'Кирка', modelKey: 'pickaxe', isBlock: false },
            5: { name: 'Меч', modelKey: 'sword', isBlock: false },
            6: { name: 'Лопата', modelKey: 'shovel', isBlock: false },
            7: { name: 'Мотыга', modelKey: 'hoe', isBlock: false }
        };
        
        let activeSlot = 1; const worldBlocks = []; const geometry = new THREE.BoxGeometry(1, 1, 1);
        let moveForward = false, moveBackward = false, moveLeft = false, moveRight = false, canJump = false;
        let velocity = new THREE.Vector3(), direction = new THREE.Vector3();
        const playerSpeed = 0.08, gravity = 0.008;
        camera.position.set(5, 3, 5); let pitch = 0, yaw = 0;

        // --- СОЗДАНИЕ КВАДРАТНОЙ РУКИ (FPS) ---
        const handHolder = new THREE.Group();
        const handGeo = new THREE.BoxGeometry(0.2, 0.2, 0.7);
        const handMat = new THREE.MeshLambertMaterial({ map: textures.skin });
        const handMesh = new THREE.Mesh(handGeo, handMat);
        handMesh.position.set(0.3, -0.3, -0.5); handMesh.rotation.set(0.1, -0.1, 0);
        handHolder.add(handMesh);

        // Контейнер для активного 3D предмета
        const itemContainer = new THREE.Group();
        itemContainer.position.set(0.22, -0.15, -0.55); // Позиция в руке
        itemContainer.rotation.set(0, Math.PI / 4, 0); // Разворот к игроку под углом
        handHolder.add(itemContainer);
        scene.add(handHolder);

        // --- ФУНКЦИЯ СБОРКИ КУБИЧЕСКИХ 3D МОДЕЛЕЙ (ВОКСЕЛИ) ---
        const voxelGeo = new THREE.BoxGeometry(0.03, 0.03, 0.03); // Размер одного «пиксельного» кубика модели

        function build3DModel(modelKey) {
            // Очищаем прошлую модель из руки
            while(itemContainer.children.length > 0){ 
                itemContainer.remove(itemContainer.children[0]); 
            }

            const grid = MODELS[modelKey];
            if (!grid) return;

            const rows = grid.length;
            const cols = grid[0].length;

            // Строим предмет из мелких вокселей по матрице
            for (let y = 0; y < rows; y++) {
                for (let x = 0; x < cols; x++) {
                    const colorHex = grid[y][x];
                    if (colorHex !== 0) {
                        const voxelMat = new THREE.MeshLambertMaterial({ color: colorHex });
                        const voxel = new THREE.Mesh(voxelGeo, voxelMat);
                        
                        // Координаты кубиков (центрируем и переворачиваем сетку)
                        voxel.position.set(
                            (x - cols / 2) * 0.03,
                            (rows / 2 - y) * 0.03,
                            0
                        );
                        itemContainer.add(voxel);
                    }
                }
            }
        }

        // --- Функция для создания блока в руке (если выбран блок) ---
        function build3DBlockInHand(texture) {
            while(itemContainer.children.length > 0){ itemContainer.remove(itemContainer.children[0]); }
            const blockGeo = new THREE.BoxGeometry(0.12, 0.12, 0.12);
            const blockMat = new THREE.MeshLambertMaterial({ map: texture });
            const miniBlock = new THREE.Mesh(blockGeo, blockMat);
            itemContainer.add(miniBlock);
        }

        // Генерация мира
        const worldSize = 10;
        function createBlock(x, y, z, typeId) {
            const material = new THREE.MeshLambertMaterial({ map: BLOCK_TYPES[typeId].texture });
            const mesh = new THREE.Mesh(geometry, material);
            mesh.position.set(x, y, z); mesh.blockTypeId = typeId;
            scene.add(mesh); worldBlocks.push(mesh);
        }

        for(let x = 0; x < worldSize; x++) {
            for(let z = 0; z < worldSize; z++) {
                createBlock(x, 0, z, 3);
                createBlock(x, 1, z, Math.random() < 0.2 ? 2 : 1);
            }
        }

        // Логика действий
        let isAttacking = false; let attackTime = 0;

        function handleAction(type) {
            if (!isAttacking) { isAttacking = true; attackTime = 0; }

            const raycaster = new THREE.Raycaster();
            raycaster.setFromCamera(new THREE.Vector2(0,0), camera);
            const intersects = raycaster.intersectObjects(worldBlocks);

            if (intersects.length > 0 && intersects[0].distance < 5) {
                const hitBlock = intersects[0].object;
                if (type === 'break') {
                    scene.remove(hitBlock);
                    worldBlocks.splice(worldBlocks.indexOf(hitBlock), 1);
                } 
                else if (type === 'place') {
                    const current = BLOCK_TYPES[activeSlot];
                    if (current.isBlock) {
                        const normal = intersects[0].face.normal;
                        const newPos = hitBlock.position.clone().add(normal);
                        createBlock(newPos.x, newPos.y, newPos.z, activeSlot);
                    }
                }
            }
        }

        window.selectSlot = function(slotId) {
            activeSlot = slotId;
            const slots = document.querySelectorAll('.slot');
            slots.forEach((s, idx) => s.classList.toggle('active', idx === (slotId - 1)));
            
            const currentItem = BLOCK_TYPES[slotId];
            document.getElementById('info').innerText = "В руках: " + currentItem.name;
            
            // Если выбрали инструмент — собираем его 3D модель из вокселей, если блок — выводим мини-кубик
            if (!currentItem.isBlock) {
                build3DModel(currentItem.modelKey);
                itemContainer.rotation.set(0, 0, -Math.PI / 4); // Инструменты наклонены красиво вперед
            } else {
                build3DBlockInHand(currentItem.texture);
                itemContainer.rotation.set(0.3, Math.PI / 4, 0); // Блок держится ровно
            }
        }

        selectSlot(1); // Старт с первого слота

        // --- СЕНСОРНЫЙ ВЗГЛЯД ---
        let previousTouch;
        window.addEventListener('touchstart', (e) => { if(e.target.tagName === 'CANVAS' || e.target.id === 'crosshair') previousTouch = e.touches[0]; });
        window.addEventListener('touchmove', (e) => {
            if(!previousTouch || e.touches.length > 1) return;
            const touch = e.touches[0];
            yaw -= (touch.pageX - previousTouch.pageX) * 0.006; pitch -= (touch.pageY - previousTouch.pageY) * 0.006;
            pitch = Math.max(-Math.PI/2.2, Math.min(Math.PI/2.2, pitch)); previousTouch = touch;
        });
        window.addEventListener('touchend', () => { previousTouch = null; });

        const setupMobileBtn = (id, press, release) => {
            const btn = document.getElementById(id);
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

        // ПК клавиши
        window.addEventListener('keydown', (e) => {
            if(e.code === 'KeyW') moveForward = true; if(e.code === 'KeyS') moveBackward = true;
            if(e.code === 'KeyA') moveLeft = true; if(e.code === 'KeyD') moveRight = true;
        });
        window.addEventListener('keyup', (e) => {
            if(e.code === 'KeyW') moveForward = false; if(e.code === 'KeyS') moveBackward = false;
            if(e.code === 'KeyA') moveLeft = false; if(e.code === 'KeyD') moveRight = false;
        });

        // --- ИГРОВОЙ ЦИКЛ ---
        function animate() {
            requestAnimationFrame(animate);

            const qx = new THREE.Quaternion(); qx.setFromAxisAngle(new THREE.Vector3(1, 0, 0), pitch);
            const qy = new THREE.Quaternion(); qy.setFromAxisAngle(new THREE.Vector3(0, 1, 0), yaw);
            camera.quaternion.copy(qy).multiply(qx);

            handHolder.position.copy(camera.position); handHolder.quaternion.copy(camera.quaternion);

            // Анимация полноценного взмаха руки Стива вместе с 3D предметом
            if (isAttacking) {
                attackTime += 0.22;
                handMesh.position.z = -0.5 + Math.sin(attackTime) * 0.12;
                handMesh.rotation.x = 0.1 + Math.sin(attackTime) * 0.5;
                itemContainer.position.z = -0.55 + Math.sin(attackTime) * 0.15;
                itemContainer.rotation.x = Math.sin(attackTime) * 0.6;
                
                if (attackTime >= Math.PI) { 
                    isAttacking = false; 
                    handMesh.position.set(0.3, -0.3, -0.5); handMesh.rotation.set(0.1, -0.1, 0);
                    itemContainer.position.set(0.22, -0.15, -0.55);
                    if (BLOCK_TYPES[activeSlot].isBlock) itemContainer.rotation.set(0.3, Math.PI / 4, 0);
                    else itemContainer.rotation.set(0, 0, -Math.PI / 4);
                }
            }

            velocity.y -= gravity; camera.position.y += velocity.y;
            if (camera.position.y < 2.5) { velocity.y = 0; camera.position.y = 2.5; canJump = true; }

            direction.z = Number(moveForward) - Number(moveBackward); direction.x = Number(moveRight) - Number(moveLeft);
            direction.normalize();
            const camYawMat = new THREE.Matrix4().makeRotationY(yaw);
            const moveVector = new THREE.Vector3(direction.x, 0, -direction.z).applyMatrix4(camYawMat);
            camera.position.addScaledVector(moveVector, playerSpeed);

            renderer.render(scene, camera);
        }

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>

