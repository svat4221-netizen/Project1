<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Minecraft Mobile 3D</title>
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
        <div id="info">Инициализация генератора...</div>
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
        // Проверка загрузки библиотеки, чтобы избежать зависания намертво
        window.onerror = function(msg, url, line) {
            document.getElementById('info').innerText = "Ошибка: " + msg + " (Строка: " + line + ")";
            return false;
        };

        // Цветовая палитра вокселей (0 - пустота)
        const W = 0x5c4033; // Дерево (коричневый)
        const S = 0xaaaaaa; // Сталь (серый)
        const D = 0x4dedf0; // Алмаз (голубой)

        const MODELS = {
            pickaxe: [
                [0,0,D,D,D,D,0],
                [0,D,D,S,S,D,D],
                [0,D,S,0,0,S,D],
                [0,0,0,0,W,0,0],
                [0,0,0,W,0,0,0],
                [0,0,W,0,0,0,0],
                [0,W,0,0,0,0,0],
                [W,0,0,0,0,0,0]
            ],
            sword: [
                [0,0,0,0,D],
                [0,0,0,D,D],
                [0,0,D,D,0],
                [0,D,D,0,0],
                [D,D,0,0,0],
                [0,S,0,0,0],
                [0,0,W,0,0]
            ],
            shovel: [
                [0,0,D,D],
                [0,D,D,D],
                [0,0,W,0],
                [0,W,0,0],
                [W,0,0,0]
            ],
            hoe: [
                [0,D,D,D],
                [0,D,S,S],
                [0,0,W,0],
                [0,W,0,0],
                [W,0,0,0]
            ]
        };

        // Генерация процедурных текстур
        function createMinecraftTexture(type) {
            const canvas = document.createElement('canvas');
            canvas.width = 16;
            canvas.height = 16;
            const ctx = canvas.getContext('2d');

            for (let x = 0; x < 16; x++) {
                for (let y = 0; y < 16; y++) {
                    let r, g, b, noise = Math.random() * 16 - 8;
                    if (type === 'dirt') {
                        r = 85 + noise; g = 55 + noise; b = 25 + noise;
                        if (y < 3 && Math.random() > 0.2) { r = 50 + noise; g = 120 + noise; b = 35 + noise; }
                    } else if (type === 'sand') {
                        r = 215 + noise; g = 195 + noise; b = 135 + noise;
                    } else if (type === 'stone') {
                        r = 120 + noise; g = 120 + noise; b = 120 + noise;
                    } else if (type === 'skin') {
                        r = 210 + noise; g = 140 + noise; b = 105 + noise;
                        if (y < 4) { r = 35; g = 145; b = 145; } // Майка Стива
                    }
                    ctx.fillStyle = "rgb(" + Math.floor(r) + "," + Math.floor(g) + "," + Math.floor(b) + ")";
                    ctx.fillRect(x, y, 1, 1);
                }
            }
            const texture = new THREE.CanvasTexture(canvas);
            texture.magFilter = THREE.NearestFilter;
            texture.minFilter = THREE.NearestFilter;
            return texture;
        }

        // Запуск основного движка после полной загрузки страницы
        window.onload = function() {
            if (typeof THREE === 'undefined') {
                document.getElementById('info').innerText = "Критическая ошибка: движок Three.js не загружен!";
                return;
            }
            startWorld();
        };

        function startWorld() {
            // Сцена и рендер
            const scene = new THREE.Scene();
            scene.background = new THREE.Color(0x7ec0ee);
            scene.fog = new THREE.FogExp2(0x7ec0ee, 0.05);

            const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 500);
            const renderer = new THREE.WebGLRenderer({ antialias: false });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            document.body.appendChild(renderer.domElement);

            // Свет
            const lightA = new THREE.AmbientLight(0xffffff, 0.85); scene.add(lightA);
            const lightD = new THREE.DirectionalLight(0xffffff, 0.35); lightD.position.set(10, 25, 10); scene.add(lightD);

            // Текстуры
            const texDirt = createMinecraftTexture('dirt');
            const texSand = createMinecraftTexture('sand');
            const texStone = createMinecraftTexture('stone');
            const texSkin = createMinecraftTexture('skin');

            const DATA_ITEMS = {
                1: { name: 'Земля', texture: texDirt, isBlock: true },
                2: { name: 'Песок', texture: texSand, isBlock: true },
                3: { name: 'Камень', texture: texStone, isBlock: true },
                4: { name: 'Кирка', modelKey: 'pickaxe', isBlock: false },
                5: { name: 'Меч', modelKey: 'sword', isBlock: false },
                6: { name: 'Лопата', modelKey: 'shovel', isBlock: false },
                7: { name: 'Мотыга', modelKey: 'hoe', isBlock: false }
            };

            let activeSlot = 1;
            const blocksArray = [];
            const blockGeometry = new THREE.BoxGeometry(1, 1, 1);

            let moveW = false, moveS = false, moveA = false, moveD = false, jumpPossible = false;
            let playerVelocity = new THREE.Vector3();
            const pSpeed = 0.075, pGravity = 0.0075;
            camera.position.set(4, 3, 4);
            let cameraPitch = 0, cameraYaw = 0;

            // --- РУКА И КОНТЕЙНЕР ДЛЯ 3D ИНСТРУМЕНТОВ ---
            const cameraRig = new THREE.Group();
            
            const armGeo = new THREE.BoxGeometry(0.16, 0.16, 0.55);
            const armMat = new THREE.MeshLambertMaterial({ map: texSkin });
            const armMesh = new THREE.Mesh(armGeo, armMat);
            armMesh.position.set(0.28, -0.28, -0.42);
            armMesh.rotation.set(0.1, -0.1, 0);
            cameraRig.add(armMesh);

            const toolHolder = new THREE.Group();
            toolHolder.position.set(0.22, -0.16, -0.46);
            cameraRig.add(toolHolder);
            scene.add(cameraRig);

            const singleVoxelGeo = new THREE.BoxGeometry(0.025, 0.025, 0.025);

            function redrawItemInHand() {
                while(toolHolder.children.length > 0) { toolHolder.remove(toolHolder.children[0]); }
                const current = DATA_ITEMS[activeSlot];

                if (current.isBlock) {
                    const previewGeo = new THREE.BoxGeometry(0.09, 0.09, 0.09);
                    const previewMat = new THREE.MeshLambertMaterial({ map: current.texture });
                    const previewMesh = new THREE.Mesh(previewGeo, previewMat);
                    toolHolder.add(previewMesh);
                    toolHolder.rotation.set(0.2, Math.PI / 4, 0);
                } else {
                    const matrix = MODELS[current.modelKey];
                    if (!matrix) return;
                    for (let y = 0; y < matrix.length; y++) {
                        for (let x = 0; x < matrix[y].length; x++) {
                            const color = matrix[y][x];
                            if (color !== 0) {
                                const vMat = new THREE.MeshLambertMaterial({ color: color });
                                const vMesh = new THREE.Mesh(singleVoxelGeo, vMat);
                                vMesh.position.set((x - matrix[y].length / 2) * 0.025, (matrix.length / 2 - y) * 0.025, 0);
                                toolHolder.add(vMesh);
                            }
                        }
                    }
                    toolHolder.rotation.set(0, 0, -Math.PI / 4);
                }
            }

            // Создание блоков в мире
            function buildWorldBlock(x, y, z, id) {
                const mat = new THREE.MeshLambertMaterial({ map: DATA_ITEMS[id].texture });
                const mesh = new THREE.Mesh(blockGeometry, mat);
                mesh.position.set(x, y, z);
                mesh.blockId = id;
                scene.add(mesh);
                blocksArray.push(mesh);
            }

            // Маленький стабильный мир 8х8 блоков
            for (let x = 0; x < 8; x++) {
                for (let z = 0; z < 8; z++) {
                    buildWorldBlock(x, 0, z, 3);
                    buildWorldBlock(x, 1, z, Math.random() < 0.25 ? 2 : 1);
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
                        scene.remove(targeted);
                        blocksArray.splice(blocksArray.indexOf(targeted), 1);
                    } else if (actionType === 'place' && DATA_ITEMS[activeSlot].isBlock) {
                        const sideNormal = hits[0].face.normal;
                        const spawnPos = targeted.position.clone().add(sideNormal);
                        buildWorldBlock(spawnPos.x, spawnPos.y, spawnPos.z, activeSlot);
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
            window.addEventListener('touchstart', function(e) {
                if (e.target.tagName === 'CANVAS' || e.target.id === 'crosshair') { lastTouchState = e.touches[0]; }
            }, { passive: true });

            window.addEventListener('touchmove', function(e) {
                if (!lastTouchState || e.touches.length > 1) return;
                const touch = e.touches[0];
                cameraYaw -= (touch.pageX - lastTouchState.pageX) * 0.0065;
                cameraPitch -= (touch.pageY - lastTouchState.pageY) * 0.0065;
                cameraPitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, cameraPitch));
                lastTouchState = touch;
            }, { passive: true });

            window.addEventListener('touchend', function() { lastTouchState = null; });

            // Привязка мобильных кнопок управления
            function registerMobileControl(elementId, startCallback, endCallback) {
                const el = document.getElementById(elementId);
                if (!el) return;
                el.addEventListener('touchstart', function(e) { e.preventDefault(); startCallback(); });
                el.addEventListener('touchend', function(e) { e.preventDefault(); endCallback(); });
            }

            registerMobileControl('btn-w', () => moveW = true, () => moveW = false);
            registerMobileControl('btn-s', () => moveS = true, () => moveS = false);
            registerMobileControl('btn-a', () => moveA = true, () => moveA = false);
            registerMobileControl('btn-d', () => moveD = true, () => moveD = false);
            registerMobileControl('btn-jump', () => { if (jumpPossible) { playerVelocity.y = 0.115; jumpPossible = false; } }, () => {});
            registerMobileControl('btn-break', () => triggerAction('break'), () => {});
            registerMobileControl('btn-place', () => triggerAction('place'), () => {});

            // Рендер-цикл
            function updateGameLoop() {
                requestAnimationFrame(updateGameLoop);

                // Синхронизация камеры и рук
                const rotX = new THREE.Quaternion().setFromAxisAngle(new THREE.Vector3(1, 0, 0), cameraPitch);
                const rotY = new THREE.Quaternion().setFromAxisAngle(new THREE.Vector3(0, 1, 0), cameraYaw);
                camera.quaternion.copy(rotY).multiply(rotX);

                cameraRig.position.copy(camera.position);
                cameraRig.quaternion.copy(camera.quaternion);

                // Анимация атаки/удара рукой Стива
                if (swingActive) {
                    swingProgress += 0.24;
                    armMesh.position.z = -0.42 + Math.sin(swingProgress) * 0.08;
                    armMesh.rotation.x = 0.1 + Math.sin(swingProgress) * 0.45;
                    toolHolder.position.z = -0.46 + Math.sin(swingProgress) * 0.1;
                    if (swingProgress >= Math.PI) {
                        swingActive = false;
                        armMesh.position.set(0.28, -0.28, -0.42);
                        armMesh.rotation.set(0.1, -0.1, 0);
                        toolHolder.position.set(0.22, -0.16, -0.46);
                    }
                }

                // Симуляция физики и ходьбы
                playerVelocity.y -= pGravity;
                camera.position.y += playerVelocity.y;

                if (camera.position.y < 2.5) {
                    playerVelocity.y = 0;
                    camera.position.y = 2.5;
                    jumpPossible = true;
                }

                let dirZ = Number(moveW) - Number(moveS);
                let dirX = Number(moveD) - Number(moveA);
                let combinedMove = new THREE.Vector3(dirX, 0, -dirZ).normalize();
                combinedMove.applyMatrix4(new THREE.Matrix4().makeRotationY(cameraYaw));
                camera.position.addScaledVector(combinedMove, pSpeed);

                renderer.render(scene, camera);
            }
            updateGameLoop();

            window.addEventListener('resize', function() {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            });
        }
    </script>
</body>
</html>



