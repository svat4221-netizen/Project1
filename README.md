<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Minecraft 3D</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; -webkit-user-select: none; user-select: none; }
        html, body { width: 100%; height: 100%; overflow: hidden; background: #7ec0ee; position: fixed; font-family: monospace; }
        canvas { display: block; position: absolute; top: 0; left: 0; width: 100% !important; height: 100% !important; z-index: 1; }
        #ui { position: absolute; top: 10px; left: 10px; color: white; background: rgba(0,0,0,0.6); padding: 8px; border-radius: 4px; z-index: 5; font-size: 12px; }
        #crosshair { position: absolute; top: 50%; left: 50%; width: 10px; height: 10px; background: white; transform: translate(-50%, -50%); z-index: 5; opacity: 0.7; }
        #hotbar { position: absolute; bottom: 10px; left: 50%; transform: translateX(-50%); display: flex; background: rgba(0,0,0,0.7); padding: 4px; border-radius: 6px; z-index: 5; }
        .slot { width: 42px; height: 42px; border: 2px solid #555; margin: 0 2px; display: flex; align-items: center; justify-content: center; color: white; font-size: 9px; text-align: center; background: #222; border-radius: 4px; }
        .slot.active { border-color: #fff; background: #555; }
        #controls { position: absolute; bottom: 15px; left: 15px; display: grid; grid-template-columns: repeat(3, 45px); grid-gap: 5px; z-index: 5; }
        .btn { width: 45px; height: 45px; background: rgba(255,255,255,0.3); border: 2px solid #fff; border-radius: 8px; display: flex; align-items: center; justify-content: center; color: #fff; font-weight: bold; font-size: 16px; }
        #action-btns { position: absolute; bottom: 15px; right: 15px; display: flex; flex-direction: column; gap: 8px; z-index: 5; }
        .action-btn { width: 60px; height: 60px; background: rgba(0,0,0,0.5); border: 2px solid #fff; border-radius: 50%; color: #fff; display: flex; align-items: center; justify-content: center; font-size: 10px; font-weight: bold; }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/build/three.min.js"></script>
</head>
<body>

    <div id="ui">
        <div id="info">Загрузка...</div>
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
        // Проверка: загрузился ли движок Three.js
        window.onload = function() {
            if (typeof THREE === 'undefined') {
                document.getElementById('info').innerText = "Ошибка: не загрузился скрипт Three.js. Проверь интернет!";
                return;
            }
            initGame();
        };

        function initGame() {
            // --- НАСТРОЙКА СЦЕНЫ ---
            const scene = new THREE.Scene();
            scene.background = new THREE.Color(0x7ec0ee);
            scene.fog = new THREE.FogExp2(0x7ec0ee, 0.06);

            const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            const renderer = new THREE.WebGLRenderer({ antialias: false });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            document.body.appendChild(renderer.domElement);

            // Освещение
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.8); scene.add(ambientLight);
            const directionalLight = new THREE.DirectionalLight(0xffffff, 0.4); directionalLight.position.set(10, 20, 10); scene.add(directionalLight);

            // Воксельные сетки инструментов (Цвета)
            const C = { W: 0x5c4033, S: 0xcccccc, D: 0x4dedf0 };
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

            // Генератор пиксельных текстур блоков
            function generateTexture(type) {
                const canvas = document.createElement('canvas'); canvas.width = 16; canvas.height = 16;
                const ctx = canvas.getContext('2d');
                for(let x=0; x<16; x++) {
                    for(let y=0; y<16; y++) {
                        let r, g, b, noise = Math.random()*20 - 10;
                        if(type==='dirt') { r=90+noise; g=60+noise; b=30+noise; if(y<3&&Math.random()>0.3){ r=45; g=120; b=40; } }
                        else if(type==='sand') { r=220+noise; g=200+noise; b=140+noise; }
                        else if(type==='stone') { r=120+noise; g=120+noise; b=120+noise; }
                        else if(type==='skin') { r=220+noise; g=150+noise; b=115+noise; if(y<4){ r=30; g=140; b=140; } }
                        ctx.fillStyle = `rgb(${Math.floor(r)},${Math.floor(g)},${Math.floor(b)})`; ctx.fillRect(x,y,1,1);
                    }
                }
                const tex = new THREE.CanvasTexture(canvas); tex.magFilter = THREE.NearestFilter; tex.minFilter = THREE.NearestFilter;
                return tex;
            }

            const texDirt = generateTexture('dirt'), texSand = generateTexture('sand'), texStone = generateTexture('stone'), texSkin = generateTexture('skin');

            const BLOCK_TYPES = {
                1: { name: 'Земля', texture: texDirt, isBlock: true },
                2: { name: 'Песок', texture: texSand, isBlock: true },
                3: { name: 'Камень', texture: texStone, isBlock: true },
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

            // --- КВАДРАТНАЯ РУКА И ПРЕДМЕТЫ ---
            const handHolder = new THREE.Group();
            const handGeo = new THREE.BoxGeometry(0.18, 0.18, 0.6);
            const handMat = new THREE.MeshLambertMaterial({ map: texSkin });
            const handMesh = new THREE.Mesh(handGeo, handMat);
            handMesh.position.set(0.3, -0.28, -0.45); handMesh.rotation.set(0.1, -0.1, 0);
            handHolder.add(handMesh);

            const itemContainer = new THREE.Group();
            itemContainer.position.set(0.22, -0.15, -0.5);
            handHolder.add(itemContainer);
            scene.add(handHolder);

            const voxelGeo = new THREE.BoxGeometry(0.03, 0.03, 0.03);
            function updateHandItem() {
                while(itemContainer.children.length > 0) { itemContainer.remove(itemContainer.children[0]); }
                const item = BLOCK_TYPES[activeSlot];

                if (item.isBlock) {
                    const bGeo = new THREE.BoxGeometry(0.1, 0.1, 0.1);
                    const bMat = new THREE.MeshLambertMaterial({ map: item.texture });
                    const miniBlock = new THREE.Mesh(bGeo, bMat);
                    itemContainer.add(miniBlock);
                    itemContainer.rotation.set(0.2, Math.PI/4, 0);
                } else {
                    const grid = MODELS[item.modelKey];
                    if(!grid) return;
                    for(let y=0; y<grid.length; y++) {
                        for(let x=0; x<grid[y].length; x++) {
                            if(grid[y][x] !== 0) {
                                const vMat = new THREE.MeshLambertMaterial({ color: grid[y][x] });
                                const voxel = new THREE.Mesh(voxelGeo, vMat);
                                voxel.position.set((x - grid[y].length/2)*0.03, (grid.length/2 - y)*0.03, 0);
                                itemContainer.add(voxel);
                            }
                        }
                    }
                    itemContainer.rotation.set(0, 0, -Math.PI/4);
                }
            }

            // Рендер маленького мира (10х10), чтоб телефон тянул идеально
            function createWorldBlock(x, y, z, typeId) {
                const mat = new THREE.MeshLambertMaterial({ map: BLOCK_TYPES[typeId].texture });
                const mesh = new THREE.Mesh(geometry, mat); mesh.position.set(x, y, z);
                mesh.blockTypeId = typeId; scene.add(mesh); worldBlocks.push(mesh);
            }
            for(let x=0; x<10; x++) {
                for(let z=0; z<10; z++) {
                    createWorldBlock(x, 0, z, 3);
                    createWorldBlock(x, 1, z, Math.random() < 0.2 ? 2 : 1);
                }
            }

            // Действия ломания/установки блоков
            let isAttacking = false, attackTime = 0;
            function doAction(type) {
                if(!isAttacking) { isAttacking = true; attackTime = 0; }
                const raycaster = new THREE.Raycaster();
                raycaster.setFromCamera(new THREE.Vector2(0,0), camera);
                const intersects = raycaster.intersectObjects(worldBlocks);
                if(intersects.length > 0 && intersects[0].distance < 4.5) {
                    const hit = intersects[0].object;
                    if(type === 'break') {
                        scene.remove(hit); worldBlocks.splice(worldBlocks.indexOf(hit), 1);
                    } else if(type === 'place' && BLOCK_TYPES[activeSlot].isBlock) {
                        const normal = intersects[0].face.normal;
                        const pos = hit.position.clone().add(normal);
                        createWorldBlock(pos.x, pos.y, pos.z, activeSlot);
                    }
                }
            }

            window.selectSlot = function(id) {
                activeSlot = id;
                document.querySelectorAll('.slot').forEach((s, i) => s.classList.toggle('active', i === (id-1)));
                document.getElementById('info').innerText = "В руках: " + BLOCK_TYPES[id].name;
                updateHandItem();
            };
            selectSlot(1);

            // Сенсорный обзор пальцем
            let prevTouch;
            window.addEventListener('touchstart', (e) => { if(e.target.tagName === 'CANVAS' || e.target.id === 'crosshair') prevTouch = e.touches[0]; });
            window.addEventListener('touchmove', (e) => {
                if(!prevTouch || e.touches.length > 1) return;
                const touch = e.touches[0];
                yaw -= (touch.pageX - prevTouch.pageX) * 0.007;
                pitch -= (touch.pageY - prevTouch.pageY) * 0.007;
                pitch = Math.max(-Math.PI/2.2, Math.min(Math.PI/2.2, pitch));
                prevTouch = touch;
            });
            window.addEventListener('touchend', () => { prevTouch = null; });

            // Сенсорные кнопки ходьбы/действий
            const bindBtn = (id, press, release) => {
                const b = document.getElementById(id);
                b.addEventListener('touchstart', (e) => { e.preventDefault(); press(); });
                b.addEventListener('touchend', (e) => { e.preventDefault(); release(); });
            };
            bindBtn('btn-w', () => moveForward = true, () => moveForward = false);
            bindBtn('btn-s', () => moveBackward = true, () => moveBackward = false);
            bindBtn('btn-a', () => moveLeft = true, () => moveLeft = false);
            bindBtn('btn-d', () => moveRight = true, () => moveRight = false);
            bindBtn('btn-jump', () => { if(canJump) { velocity.y = 0.12; canJump = false; } }, () => {});
            bindBtn('btn-break', () => doAction('break'), () => {});
            bindBtn('btn-place', () => doAction('place'), () => {});

            // Анимационный игровой цикл
            function loop() {
                requestAnimationFrame(loop);

                // Поворот камеры
                const qx = new THREE.Quaternion(); qx.setFromAxisAngle(new THREE.Vector3(1,0,0), pitch);
                const qy = new THREE.Quaternion(); qy.setFromAxisAngle(new THREE.Vector3(0,1,0), yaw);
                camera.quaternion.copy(qy).multiply(qx);

                handHolder.position.copy(camera.position); handHolder.quaternion.copy(camera.quaternion);

                // Анимация удара рукой
                if(isAttacking) {
                    attackTime += 0.25;
                    handMesh.position.z = -0.45 + Math.sin(attackTime) * 0.1;
                    handMesh.rotation.x = 0.1 + Math.sin(attackTime) * 0.4;
                    itemContainer.position.z = -0.5 + Math.sin(attackTime) * 0.12;
                    if(attackTime >= Math.PI) {
                        isAttacking = false;
                        handMesh.position.set(0.3, -0.28, -0.45); handMesh.rotation.set(0.1, -0.1, 0);
                        itemContainer.position.set(0.22, -0.15, -0.5);
                    }
                }

                // Гравитация и ходьба
                velocity.y -= gravity; camera.position.y += velocity.y;
                if(camera.position.y < 2.5) { velocity.y = 0; camera.position.y = 2.5; canJump = true; }

                direction.z = Number(moveForward) - Number(moveBackward); direction.x = Number(moveRight) - Number(moveLeft);
                direction.normalize();
                const moveVector = new THREE.Vector3(direction.x, 0, -direction.z).applyMatrix4(new THREE.Matrix4().makeRotationY(yaw));
                camera.position.addScaledVector(moveVector, playerSpeed);

                renderer.render(scene, camera);
            }
            loop();

            window.addEventListener('resize', () => {
                camera.aspect = window.innerWidth / window.innerHeight; camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            });
        }
    </script>
</body>
</html>


