<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Minecraft FPS & 3D Items</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-user-select: none; user-select: none; }
        html, body { width: 100%; height: 100%; overflow: hidden; background: #7ec0ee; position: fixed; font-family: monospace; }
        canvas { display: block; position: absolute; top: 0; left: 0; width: 100% !important; height: 100% !important; z-index: 1; }
        
        #ui { position: absolute; top: 15px; left: 15px; color: white; background: rgba(0,0,0,0.6); padding: 10px; border-radius: 6px; z-index: 10; font-size: 13px; pointer-events: none; }
        #crosshair { position: absolute; top: 50%; left: 50%; width: 10px; height: 10px; background: white; transform: translate(-50%, -50%); z-index: 10; opacity: 0.8; pointer-events: none; }
        
        #hotbar { position: absolute; bottom: 15px; left: 50%; transform: translateX(-50%); display: flex; background: rgba(0,0,0,0.75); padding: 5px; border-radius: 8px; z-index: 10; max-width: 95%; overflow-x: auto; }
        .slot { width: 45px; height: 45px; border: 2px solid #555; margin: 0 3px; display: flex; align-items: center; justify-content: center; color: white; font-size: 10px; text-align: center; background: #222; border-radius: 5px; flex-shrink: 0; }
        .slot.active { border-color: #fff; background: #555; box-shadow: 0 0 8px #fff; }
        
        #controls { position: absolute; bottom: 20px; left: 20px; display: grid; grid-template-columns: repeat(3, 50px); grid-gap: 6px; z-index: 10; }
        .btn { width: 50px; height: 50px; background: rgba(255,255,255,0.25); border: 2px solid rgba(255,255,255,0.7); border-radius: 10px; display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; font-size: 18px; }
        
        #action-btns { position: absolute; bottom: 20px; right: 20px; display: flex; flex-direction: column; gap: 10px; z-index: 10; }
        .action-btn { width: 65px; height: 65px; background: rgba(0,0,0,0.55); border: 2px solid #fff; border-radius: 50%; color: white; display: flex; align-items: center; justify-content: center; font-size: 11px; font-weight: bold; }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <div id="ui">
        <div id="info">Генерация мира и моделей...</div>
    </div>
    
    <div id="crosshair"></div>
    
    <div id="hotbar">
        <div class="slot active" onclick="selectSlot(1)">Земля</div>
        <div class="slot" onclick="selectSlot(2)">Песок</div>
        <div class="slot" onclick="selectSlot(3)">Камень</div>
        <div class="slot" onclick="selectSlot(4)">Кирка D</div>
        <div class="slot" onclick="selectSlot(5)">Меч D</div>
        <div class="slot" onclick="selectSlot(6)">Мотыга D</div>
        <div class="slot" onclick="selectSlot(7)">Лопата W</div>
        <div class="slot" onclick="selectSlot(8)">Лопата D</div>
        <div class="slot" onclick="selectSlot(9)">Мотыга D</div>
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
        // --- ЦВЕТОВАЯ ПАЛИТРА ДЛЯ ИНСТРУМЕНТОВ (0 - пустота) ---
        const W1 = 0x5c4033; // Тёмное дерево
        const W2 = 0x8b5a2b; // Светлое дерево (рукоять)
        const S1 = 0x333333; // Чёрная обводка алмаза (S)
        const S2 = 0x247a7c; // Тёмно-бирюзовый (контур D)
        const D1 = 0x4dedf0; // Яркий алмаз (D)
        const D2 = 0xb2ffff; // Блик на алмазе

        const TOOL_GRIDS = {
            // ТОЧНАЯ СЕТКА АЛМАЗНОГО МЕЧА ИЗ MINECRAFT 16x16
            sword: [
                [0,0,0,0,0,0,0,0,0,0,0,0,0,D2,S1,0],
                [0,0,0,0,0,0,0,0,0,0,0,0,D2,D1,S1,0],
                [0,0,0,0,0,0,0,0,0,0,0,D2,D1,S2,0,0],
                [0,0,0,0,0,0,0,0,0,0,D2,D1,S2,0,0,0],
                [0,0,0,0,0,0,0,0,0,D2,D1,S2,0,0,0,0],
                [0,0,0,0,0,0,0,0,D2,D1,S2,0,0,0,0,0],
                [0,0,0,0,0,0,0,D2,D1,S2,0,0,0,0,0,0],
                [0,0,0,0,0,0,D2,D1,S2,0,0,0,0,0,0,0],
                [0,0,0,0,0,D2,D1,S2,0,0,0,0,0,0,0,0],
                [0,0,0,0,D2,D1,S2,0,0,0,0,0,0,0,0,0],
                [0,0,0,S1,S2,S2,0,0,0,0,0,0,0,0,0,0],
                [0,0,S1,W2,S1,0,0,0,0,0,0,0,0,0,0,0],
                [0,S1,W2,S1,0,0,0,0,0,0,0,0,0,0,0,0],
                [S1,W2,S1,0,0,0,0,0,0,0,0,0,0,0,0,0],
                [S1,S1,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
                [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
            ],
            // СЕТКА АЛМАЗНОЙ КИРКИ 7x8
            pickaxe: [
                [0,S1,S1,S1,S1,S1,S1,S1,S1,0,0],
                [S1,D2,D1,D1,D1,S2,S2,D1,D1,S1,0],
                [S1,D1,D1,S1,S1,0,0,S1,S2,D1,S1],
                [0,S1,S1,0,0,0,0,0,S1,S2,S1],
                [0,0,0,0,0,W1,0,0,0,S1,0],
                [0,0,0,0,W1,0,0,0,0,0,0],
                [0,0,0,W1,0,0,0,0,0,0,0],
                [0,0,W1,0,0,0,0,0,0,0,0],
                [0,W1,0,0,0,0,0,0,0,0,0,0],
                [W1,0,0,0,0,0,0,0,0,0,0,0],
            ],
            // СЕТКА ЛОПАТЫ (АЛМАЗ ИЛИ ДЕРЕВО) 5x9
            shovel: [
                [0,0,S1,S1,0],
                [0,S1,D1,D1,S1],
                [0,S1,D1,D1,S1],
                [0,S1,D1,D1,S1],
                [0,0,S1,W2,S1],
                [0,0,0,W1,0],
                [0,0,W1,0,0],
                [0,W1,0,0,0],
                [W1,0,0,0,0]
            ],
            // СЕТКА МОТЫГИ 7x8
            hoe: [
                [0,S1,S1,S1,S1,0,0],
                [S1,D1,D1,D1,D1,S1,0],
                [S1,D1,S1,S1,S2,D1,S1],
                [0,S1,0,0,S1,S2,S1],
                [0,0,0,0,W1,S1,0],
                [0,0,0,W1,0,0,0],
                [0,0,W1,0,0,0,0],
                [0,W1,0,0,0,0,0],
                [W1,0,0,0,0,0,0]
            ]
        };

        // --- ГЕНЕРАТОР ПРОЦЕДУРНЫХ ТЕКСТУР ---
        function createMinecraftTexture(type) {
            const canvas = document.createElement('canvas'); canvas.width = 16; canvas.height = 16;
            const ctx = canvas.getContext('2d');
            for (let x = 0; x < 16; x++) {
                for (let y = 0; y < 16; y++) {
                    let r, g, b, noise = Math.random() * 16 - 8;
                    if (type === 'dirt') {
                        r = 85 + noise; g = 55 + noise; b = 25 + noise;
                        if (y < 3 && Math.random() > 0.2) { r = 50 + noise; g = 120 + noise; b = 35 + noise; }
                    } else if (type === 'sand') { r = 215 + noise; g = 195 + noise; b = 135 + noise; }
                    else if (type === 'stone') { r = 120 + noise; g = 120 + noise; b = 120 + noise; }
                    else if (type === 'skin') {
                        r = 210 + noise; g = 140 + noise; b = 105 + noise;
                        if (y < 4) { r = 35; g = 145; b = 145; }
                    }
                    ctx.fillStyle = "rgb(" + Math.floor(r) + "," + Math.floor(g) + "," + Math.floor(b) + ")"; ctx.fillRect(x, y, 1, 1);
                }
            }
            const texture = new THREE.CanvasTexture(canvas);
            texture.magFilter = THREE.NearestFilter; texture.minFilter = THREE.NearestFilter;
            return texture;
        }

        window.onload = function() { startWorld(); };

        function startWorld() {
            // --- НАСТРОЙКИ СЦЕНЫ И РЕНДЕРА ---
            const scene = new THREE.Scene(); scene.background = new THREE.Color(0x7ec0ee); scene.fog = new THREE.FogExp2(0x7ec0ee, 0.05);
            const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.05, 500);
            const renderer = new THREE.WebGLRenderer({ antialias: false });
            renderer.setSize(window.innerWidth, window.innerHeight); renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            document.body.appendChild(renderer.domElement);

            // Освещение
            const lightA = new THREE.AmbientLight(0xffffff, 0.85); scene.add(lightA);
            const lightD = new THREE.DirectionalLight(0xffffff, 0.35); lightD.position.set(10, 25, 10); scene.add(lightD);

            // Текстуры блоков и кожи
            const texDirt = createMinecraftTexture('dirt'), texSand = createMinecraftTexture('sand'), texStone = createMinecraftTexture('stone'), texSkin = createMinecraftTexture('skin');

            // --- СИСТЕМА ДАТА-ИНВЕНТАРЯ И ОНЛАЙН-ФУНДАМЕНТ ---
            // Уникальные ID блоков (задел для онлайн-сервера)
            const BLOCK_IDS = { dirt: 1, sand: 2, stone: 3 };

            const DATA_ITEMS = {
                1: { name: 'Земля', texture: texDirt, isBlock: true, blockId: BLOCK_IDS.dirt },
                2: { name: 'Песок', texture: texSand, isBlock: true, blockId: BLOCK_IDS.sand },
                3: { name: 'Камень', texture: texStone, isBlock: true, blockId: BLOCK_IDS.stone },
                4: { name: 'Алмазная Кирка', modelKey: 'pickaxe', material: 'D', isBlock: false },
                5: { name: 'Алмазный Меч', modelKey: 'sword', material: 'D', isBlock: false },
                6: { name: 'Алмазная Мотыга', modelKey: 'hoe', material: 'D', isBlock: false },
                7: { name: 'Деревянная Лопата', modelKey: 'shovel', material: 'W', isBlock: false }, // Специально деревянная
                8: { name: 'Алмазная Лопата', modelKey: 'shovel', material: 'D', isBlock: false },
                9: { name: 'Алмазная Мотыга', modelKey: 'hoe', material: 'D', isBlock: false }
            };

            let activeSlot = 1; const blocksArray = []; const blockGeometry = new THREE.BoxGeometry(1, 1, 1);
            let moveW = false, moveS = false, moveA = false, moveD = false, jumpPossible = false;
            let playerVelocity = new THREE.Vector3(); const pSpeed = 0.075, pGravity = 0.0075;
            camera.position.set(4, 3, 4); let cameraPitch = 0, cameraYaw = 0;

            // --- КВАДРАТНАЯ РУКА И КОНТЕЙНЕР ДЛЯ 3D ИНСТРУМЕНТОВ (FPS) ---
            const cameraRig = new THREE.Group();
            const armGeo = new THREE.BoxGeometry(0.14, 0.14, 0.55);
            const armMat = new THREE.MeshLambertMaterial({ map: texSkin });
            const armMesh = new THREE.Mesh(armGeo, armMat);
            armMesh.position.set(0.28, -0.28, -0.42); armMesh.rotation.set(0.1, -0.1, 0);
            cameraRig.add(armMesh);

            const toolHolder = new THREE.Group();
            // Дефолтная позиция контейнера
            toolHolder.position.set(0.25, -0.16, -0.44); 
            cameraRig.add(toolHolder); scene.add(cameraRig);

            const singleVoxelGeo = new THREE.BoxGeometry(0.022, 0.022, 0.022); // Размер кубика модели

            // --- СБОРКА 3D ИНСТРУМЕНТА ИЗ ВОКСЕЛЕЙ ---
            function redrawItemInHand() {
                while(toolHolder.children.length > 0) { toolHolder.remove(toolHolder.children[0]); }
                const current = DATA_ITEMS[activeSlot];

                if (current.isBlock) {
                    const previewGeo = new THREE.BoxGeometry(0.09, 0.09, 0.09);
                    const previewMat = new THREE.MeshLambertMaterial({ map: current.texture });
                    const previewMesh = new THREE.Mesh(previewGeo, previewMat);
                    toolHolder.add(previewMesh);
                    toolHolder.position.set(0.22, -0.16, -0.46); // Блок держится ровно
                    toolHolder.rotation.set(0.2, Math.PI / 4, 0);
                } else {
                    let matrix = TOOL_GRIDS[current.modelKey];
                    if (!matrix) return;
                    
                    const rows = matrix.length;
                    for (let y = 0; y < rows; y++) {
                        for (let x = 0; x < matrix[y].length; x++) {
                            let color = matrix[y][x];
                            // Если инструмент деревянный (material: 'W'), заменяем цвета алмаза на дерево
                            if (current.material === 'W' && (color === D1 || color === D2 || color === S1 || color === S2)) {
                                color = W2; // Светлое дерево
                            }
                            
                            if (color !== 0) {
                                const vMat = new THREE.MeshLambertMaterial({ color: color });
                                const vMesh = new THREE.Mesh(singleVoxelGeo, vMat);
                                vMesh.position.set((x - matrix[y].length / 2) * 0.022, (rows / 2 - y) * 0.022, 0);
                                toolHolder.add(vMesh);
                            }
                        }
                    }
                    
                    // --- НАСТРОЙКА ПОЛОЖЕНИЯ В РУКЕ (ТО, ЧТО ТЫ ПРОСИЛ) ---
                    if (current.modelKey === 'sword') {
                        // Меч: ВЕРТИКАЛЬНО ВВЕРХ, как в боевой стойке
                        toolHolder.position.set(0.18, -0.05, -0.45);
                        toolHolder.rotation.set(0, 0, -Math.PI / 6); // Красивый боевой наклон
                    } else if (current.modelKey === 'shovel' || current.modelKey === 'hoe' || current.modelKey === 'pickaxe') {
                        // Кирка/Лопата/Мотыга: Готовы копать, ВНИЗ И ВПЕРЕД
                        toolHolder.position.set(0.28, -0.14, -0.48);
                        toolHolder.rotation.set(0.3, Math.PI / 3, 0); 
                    }
                }
            }

            // --- ОНЛАЙН-СИСТЕМА: ПРОЦЕДУРНОЕ ПОСТРОЕНИЕ МИРА ---
            function buildWorldBlock(x, y, z, blockId) {
                // Ищем текстуру по blockId в DATA_ITEMS
                const mat = new THREE.MeshLambertMaterial({ map: DATA_ITEMS[blockId].texture });
                const mesh = new THREE.Mesh(blockGeometry, mat); mesh.position.set(x, y, z);
                mesh.blockId = blockId; // Сохраняем blockId в меше
                scene.add(mesh); blocksArray.push(mesh);
            }

            // Генерируем мир (8х8) через уникальные blockId
            for (let x = 0; x < 8; x++) {
                for (let z = 0; z < 8; z++) {
                    buildWorldBlock(x, 0, z, BLOCK_IDS.stone); // Каменный слой
                    buildWorldBlock(x, 1, z, Math.random() < 0.25 ? BLOCK_IDS.sand : BLOCK_IDS.dirt); // Верхний слой
                }
            }

            // Действия игрока
            let swingActive = false, swingProgress = 0;
            function triggerAction(actionType) {
                if (!swingActive) { swingActive = true; swingProgress = 0; }
                const raycaster = new THREE.Raycaster();
                raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
                const hits = raycaster.intersectObjects(blocksArray);
                if (hits.length > 0 && hits[0].distance < 4.2) {
                    const targeted = hits[0].object;
                    if (actionType === 'break') {
                        // СЛОМАТЬ: Удаляем и из массива, и из сцены
                        scene.remove(targeted);
                        blocksArray.splice(blocksArray.indexOf(targeted), 1);
                        // Задел для онлайна: отправить серверу coordinate( targeted.position ), action: 'break'
                    } 
                    else if (actionType === 'place' && DATA_ITEMS[activeSlot].isBlock) {
                        // ПОСТАВИТЬ: Используем coordinate sideNormal и blockId из инвентаря
                        const sideNormal = hits[0].face.normal; const spawnPos = targeted.position.clone().add(sideNormal);
                        buildWorldBlock(spawnPos.x, spawnPos.y, spawnPos.z, DATA_ITEMS[activeSlot].blockId);
                        // Задел для онлайна: отправить серверу spawnPos, action: 'place', blockId
                    }
                }
            }

            window.selectSlot = function(id) {
                activeSlot = id;
                document.querySelectorAll('.slot').forEach((s, idx) => s.classList.toggle('active', idx === (id - 1)));
                document.getElementById('info').innerText = "В руках: " + DATA_ITEMS[id].name;
                redrawItemInHand();
            };
            selectSlot(1);

            // Тач-управление обзором камеры
            let lastTouchState;
            window.addEventListener('touchstart', function(e) { if (e.target.tagName === 'CANVAS' || e.target.id === 'crosshair') { lastTouchState = e.touches[0]; } }, { passive: true });
            window.addEventListener('touchmove', function(e) {
                if (!lastTouchState || e.touches.length > 1) return;
                const touch = e.touches[0];
                cameraYaw -= (touch.pageX - lastTouchState.pageX) * 0.0065; cameraPitch -= (touch.pageY - lastTouchState.pageY) * 0.0065;
                cameraPitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, cameraPitch)); lastTouchState = touch;
            }, { passive: true });
            window.addEventListener('touchend', function() { lastTouchState = null; });

            // Привязка мобильных кнопок управления
            function registerMobileControl(elementId, startCallback, endCallback) {
                const el = document.getElementById(elementId); if (!el) return;
                el.addEventListener('touchstart', function(e) { e.preventDefault(); startCallback(); });
                el.addEventListener('touchend', function(e) { e.preventDefault(); endCallback(); });
            }
            registerMobileControl('btn-w', () => moveW = true, () => moveW = false); registerMobileControl('btn-s', () => moveS = true, () => moveS = false);
            registerMobileControl('btn-a', () => moveA = true, () => moveA = false); registerMobileControl('btn-d', () => moveD = true, () => moveD = false);
            registerMobileControl('btn-jump', () => { if (jumpPossible) { playerVelocity.y = 0.115; jumpPossible = false; } }, () => {});
            registerMobileControl('btn-break', () => triggerAction('break'), () => {}); registerMobileControl('btn-place', () => triggerAction('place'), () => {});

            // --- ИГРОВОЙ ЦИКЛ (АНИМАЦИЯ И ФИЗИКА) ---
            function updateGameLoop() {
                requestAnimationFrame(updateGameLoop);
                const rotX = new THREE.Quaternion().setFromAxisAngle(new THREE.Vector3(1, 0, 0), cameraPitch);
                const rotY = new THREE.Quaternion().setFromAxisAngle(new THREE.Vector3(0, 1, 0), cameraYaw);
                camera.quaternion.copy(rotY).multiply(rotX);
                cameraRig.position.copy(camera.position); cameraRig.quaternion.copy(camera.quaternion);

                // --- Динамическая Анимация удара для FPS ---
                if (swingActive) {
                    swingProgress += 0.24;
                    armMesh.position.z = -0.42 + Math.sin(swingProgress) * 0.08;
                    armMesh.rotation.x = 0.1 + Math.sin(swingProgress) * 0.45;
                    toolHolder.position.z = (DATA_ITEMS[activeSlot].modelKey === 'sword' ? -0.48 : -0.46) + Math.sin(swingProgress) * 0.1;
                    if (swingProgress >= Math.PI) {
                        swingActive = false;
                        armMesh.position.set(0.28, -0.28, -0.42); armMesh.rotation.set(0.1, -0.1, 0);
                        redrawItemInHand(); // Сброс позиции предмета
                    }
                }

                // Симуляция физики
                playerVelocity.y -= pGravity; camera.position.y += playerVelocity.y;
                if (camera.position.y < 2.5) { playerVelocity.y = 0; camera.position.y = 2.5; jumpPossible = true; }

                // Расчет движения
                let dirZ = Number(moveW) - Number(moveS); let dirX = Number(moveD) - Number(moveA);
                let combinedMove = new THREE.Vector3(dirX, 0, -dirZ).normalize();
                combinedMove.applyMatrix4(new THREE.Matrix4().makeRotationY(cameraYaw)); camera.position.addScaledVector(combinedMove, pSpeed);

                renderer.render(scene, camera);
            }
            updateGameLoop();

            window.addEventListener('resize', function() {
                camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix(); renderer.setSize(window.innerWidth, window.innerHeight);
            });
        }
    </script>
</body>
</html>

