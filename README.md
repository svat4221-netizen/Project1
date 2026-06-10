<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Настоящий Майнкрафт 2D/3D</title>
    <style>
        * { box-sizing: border-box; }
        body { margin: 0; padding: 0; background-color: #111; color: #fff; font-family: 'Courier New', monospace; overflow: hidden; user-select: none; }
        
        /* Главный экран игры */
        #gameCanvas { display: block; background: #87CEEB; margin: 0 auto; width: 100vw; height: 100vh; }
        
        /* Интерфейс управления */
        .hud { position: absolute; background: rgba(0,0,0,0.7); border: 2px solid #444; padding: 15px; border-radius: 5px; pointer-events: auto; }
        #left-hud { top: 10px; left: 10px; width: 280px; font-size: 13px; line-height: 1.4; }
        #right-hud { top: 10px; right: 10px; text-align: right; width: 280px; }
        
        /* Энергия */
        .energy-bar { width: 100%; height: 15px; background: #333; border: 1px solid #fff; margin-top: 5px; position: relative; }
        #energy-fill { width: 100%; height: 100%; background: #ffaa00; transition: width 0.05s; }
        
        /* Хотбар (Инвентарь снизу) */
        #hotbar { position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%); display: flex; background: rgba(0,0,0,0.85); border: 3px solid #555; padding: 5px; border-radius: 4px; }
        .slot { width: 60px; height: 60px; border: 2px solid #444; margin: 0 4px; display: flex; flex-direction: column; align-items: center; justify-content: center; font-size: 11px; text-align: center; cursor: pointer; }
        .slot.selected { border-color: #fff; background: rgba(255,255,255,0.2); }
        
        /* Стили ролей */
        .role-player { color: #aaa; }
        .role-mod { color: #55ff55; font-weight: bold; }
        .role-admin { color: #ff5555; font-weight: bold; }
        .role-curator { color: #aa00aa; font-weight: bold; text-shadow: 0 0 5px #ff00ff; }
    </style>
</head>
<body>

    <canvas id="gameCanvas"></canvas>

    <div class="hud" id="left-hud">
        <h3 style="margin: 0 0 10px 0; color: #ffdd44; text-align: center;">MINECRAFT HTML5</h3>
        <b>W, A, S, D</b> — Движение персонажа<br>
        <b>Клавиша V</b> — Сменить вид (1-е / 3-е лицо)<br>
        <b>Цифры 1-5</b> — Выбрать блок в руке<br>
        <b>ЛКМ (Клик)</b> — Сломать блок перед собой<br>
        <b>ПКМ (Прав. клик)</b> — Поставить блок<br>
        <hr style="border: 1px solid #444; margin: 10px 0;">
        <div id="cam-view-text" style="font-weight: bold; color: #55ff55;">Режим: Вид от 3-го лица</div>
    </div>

    <div class="hud" id="right-hud">
        <div>Вы: <span class="role-curator">[Куратор] Разработчик</span></div>
        <div style="margin-top: 5px;">
            Энергия инструмента:
            <div class="energy-bar"><div id="energy-fill"></div></div>
        </div>
        <h4 style="margin: 15px 0 5px 0; text-align: center; border-bottom: 1px solid #444; padding-bottom: 3px;">Игроки на сервере:</h4>
        <div id="tab-list" style="text-align: left; font-size: 12px; max-height: 120px; overflow-y: auto;"></div>
    </div>

    <div id="hotbar">
        <div class="slot selected" id="slot-1" style="color: #77cc44;">📦<br>Трава (1)</div>
        <div class="slot" id="slot-2" style="color: #866043;">🟫<br>Земля (2)</div>
        <div class="slot" id="slot-3" style="color: #888888;">🪨<br>Камень (3)</div>
        <div class="slot" id="slot-4" style="color: #44ddff;">💎<br>Алмаз (4)</div>
        <div class="slot" id="slot-5" style="color: #ffdd44;">🪙<br>Золото (5)</div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        function updateSize() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', updateSize);
        updateSize();

        // Настройки игры
        let isFirstPerson = false;
        let energy = 100;
        let selectedSlot = 1;

        const BLOCK_TYPES = {
            1: { name: 'Трава', color: '#559933', sideColor: '#866043' },
            2: { name: 'Земля', color: '#866043', sideColor: '#5c402d' },
            3: { name: 'Камень', color: '#777777', sideColor: '#555555' },
            4: { name: 'Алмаз', color: '#00ffff', sideColor: '#335566' },
            5: { name: 'Золото', color: '#ffcc00', sideColor: '#555533' }
        };

        // Главный герой
        let player = {
            x: 200,
            y: 200,
            size: 24,
            speed: 4,
            dirX: 0,
            dirY: 1
        };

        // Другие онлайн-игроки (Сетевая симуляция высокой точности)
        let networkPlayers = [
            { name: 'Player_Pro', role: 'Игрок', x: 120, y: 120, color: '#4444ff', targetX: 120, targetY: 120 },
            { name: 'FixEye_Fan', role: 'Модератор', x: 400, y: 150, color: '#22cc22', targetX: 400, targetY: 150 },
            { name: 'Giga_Admin', role: 'Администратор', x: 250, y: 350, color: '#ff2222', targetX: 250, targetY: 350 }
        ];

        // Генерация плоского игрового мира (сетка чанка)
        let grid = [];
        const gridSize = 14;
        const tileSize = 45;
        const offsetX = 100; // Смещение карты для красоты
        const offsetY = 120;

        for (let x = 0; x < gridSize; x++) {
            grid[x] = [];
            for (let y = 0; y < gridSize; y++) {
                // Заполняем карту: по краям камень, в центре трава, случайно — руды
                let rand = Math.random();
                let type = 1; // Трава
                if (x === 0 || y === 0 || x === gridSize-1 || y === gridSize-1) type = 3; // Камень
                else if (rand < 0.04) type = 4; // Алмазная жила
                else if (rand < 0.09) type = 5; // Золотая жила
                else if (rand < 0.2) type = 2;  // Земля
                
                grid[x][y] = type;
            }
        }

        // Обновление списка игроков справа
        function renderTabList() {
            let html = `<div><span class="role-curator">[Куратор]</span> Разработчик (Вы)</div>`;
            networkPlayers.forEach(p => {
                let roleClass = p.role === 'Администратор' ? 'role-admin' : (p.role === 'Модератор' ? 'role-mod' : 'role-player');
                html += `<div><span class="${roleClass}">[${p.role}]</span> ${p.name}</div>`;
            });
            document.getElementById('tab-list').innerHTML = html;
        }
        renderTabList();

        // Управление кнопками
        let keys = {};
        window.addEventListener('keydown', e => {
            keys[e.key.toLowerCase()] = true;
            
            // Кнопка V — Смена режима камеры
            if (e.key.toLowerCase() === 'v') {
                isFirstPerson = !isFirstPerson;
                document.getElementById('cam-view-text').innerText = isFirstPerson ? "Режим: Вид от 1-го лица" : "Режим: Вид от 3-го лица";
            }

            // Выбор слотов 1-5
            if (e.key >= '1' && e.key <= '5') {
                document.getElementById(`slot-${selectedSlot}`).classList.remove('selected');
                selectedSlot = parseInt(e.key);
                document.getElementById(`slot-${selectedSlot}`).classList.add('selected');
            }
        });
        window.addEventListener('keyup', e => { keys[e.key.toLowerCase()] = false; });

        // Взаимодействие мышкой (Клик по блокам)
        canvas.addEventListener('mousedown', e => {
            // Рассчитываем, на какой блок кликнули на сетке
            let rect = canvas.getBoundingClientRect();
            let clickX = e.clientX - rect.left - offsetX;
            let clickY = e.clientY - rect.top - offsetY;

            let blockX = Math.floor(clickX / tileSize);
            let blockY = Math.floor(clickY / tileSize);

            // Проверка границ клика
            if (blockX >= 0 && blockX < gridSize && blockY >= 0 && blockY < gridSize) {
                if (energy < 20) {
                    return; // Если нет энергии, действие заблокировано
                }

                if (e.button === 0) { // ЛКМ — сломать блок (превратить в воздух / удалить)
                    if (grid[blockX][blockY] !== 0) {
                        grid[blockX][blockY] = 0;
                        energy -= 20; // Тратим энергию
                    }
                } else if (e.button === 2) { // ПКМ — поставить выбранный блок
                    if (grid[blockX][blockY] === 0) {
                        grid[blockX][blockY] = selectedSlot;
                        energy -= 15;
                    }
                }
            }
        });
        window.addEventListener('contextmenu', e => e.preventDefault()); // Отключаем контекстное меню

        // Игровой цикл обновления данных
        function updateGame() {
            // Восстановление энергии
            if (energy < 100) {
                energy = Math.min(100, energy + 0.2);
            }
            document.getElementById('energy-fill').style.width = energy + '%';

            // Логика движения нашего игрока
            let moveX = 0;
            let moveY = 0;
            if (keys['w'] || keys['ц']) { moveY = -1; player.dirX = 0; player.dirY = -1; }
            if (keys['s'] || keys['ы']) { moveY = 1;  player.dirX = 0; player.dirY = 1; }
            if (keys['a'] || keys['ф']) { moveX = -1; player.dirX = -1; player.dirY = 0; }
            if (keys['d'] || keys['в']) { moveX = 1;  player.dirX = 1; player.dirY = 0; }

            player.x += moveX * player.speed;
            player.y += moveY * player.speed;

            // Ограничение игрока рамками карты
            player.x = Math.max(offsetX, Math.min(offsetX + gridSize * tileSize - player.size, player.x));
            player.y = Math.max(offsetY, Math.min(offsetY + gridSize * tileSize - player.size, player.y));

            // Симуляция искусственного интеллекта онлайн-игроков (они ходят по миру)
            networkPlayers.forEach(p => {
                if (Math.abs(p.x - p.targetX) < 5 && Math.abs(p.y - p.targetY) < 5) {
                    if (Math.random() < 0.02) { // Шанс сменить направление движения
                        p.targetX = offsetX + Math.random() * (gridSize * tileSize - player.size);
                        p.targetY = offsetY + Math.random() * (gridSize * tileSize - player.size);
                    }
                }
                // Плавное движение к цели
                if (p.x < p.targetX) p.x += 1; else if (p.x > p.targetX) p.x -= 1;
                if (p.y < p.targetY) p.y += 1; else if (p.y > p.targetY) p.y -= 1;
            });
        }

        // Отрисовка графики (Рендер)
        function renderGame() {
            // Очистка экрана (рисуем небо)
            ctx.fillStyle = '#87CEEB';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // 1. Отрисовка блоков Майнкрафта
            for (let x = 0; x < gridSize; x++) {
                for (let y = 0; y < gridSize; y++) {
                    let blockId = grid[x][y];
                    let posX = offsetX + x * tileSize;
                    let posY = offsetY + y * tileSize;

                    if (blockId !== 0) {
                        let block = BLOCK_TYPES[blockId];
                        
                        // Псевдотрехмерный объем блока (Текстура граней)
                        ctx.fillStyle = block.sideColor;
                        ctx.fillRect(posX, posY, tileSize, tileSize);
                        
                        ctx.fillStyle = block.color;
                        ctx.fillRect(posX + 2, posY + 2, tileSize - 4, tileSize - 4);
                        
                        // Сетка блоков
                        ctx.strokeStyle = 'rgba(0,0,0,0.15)';
                        ctx.strokeRect(posX, posY, tileSize, tileSize);
                    } else {
                        // Пустой блок (Воздух / Шахта)
                        ctx.fillStyle = '#332211';
                        ctx.fillRect(posX, posY, tileSize, tileSize);
                    }
                }
            }

            // 2. Отрисовка других онлайн игроков на карте
            networkPlayers.forEach(p => {
                ctx.fillStyle = p.color;
                ctx.beginPath();
                ctx.arc(p.x + player.size/2, p.y + player.size/2, player.size/2, 0, Math.PI * 2);
                ctx.fill();

                // Ники над другими игроками
                ctx.fillStyle = '#fff';
                ctx.font = '11px Courier New';
                ctx.textAlign = 'center';
                let pref = p.role === 'Администратор' ? '[Админ]' : (p.role === 'Модератор' ? '[Мод]' : '[Игрок]');
                ctx.fillText(pref + ' ' + p.name, p.x + player.size/2, p.y - 6);
            });

            // 3. Отрисовка главного героя (Разработчика)
            if (!isFirstPerson) {
                // Режим ОТ ТРЕТЬЕГО ЛИЦА — видим своего персонажа целиком (как Стив в синей футболке)
                ctx.fillStyle = '#00aaff'; // Синее тело
                ctx.fillRect(player.x, player.y, player.size, player.size);
                
                ctx.fillStyle = '#ffaa44'; // Голова/Кожа
                ctx.fillRect(player.x + 4, player.y + 4, player.size - 8, player.size - 8);

                // Направление взгляда (глаза)
                ctx.fillStyle = '#000';
                let eyeX = player.x + player.size/2 + player.dirX * 6 - 2;
                let eyeY = player.y + player.size/2 + player.dirY * 6 - 2;
                ctx.fillRect(eyeX, eyeY, 4, 4);

                // Наш Никнейм над головой
                ctx.fillStyle = '#aa00aa';
                ctx.font = 'bold 12px Courier New';
                ctx.textAlign = 'center';
                ctx.fillText('[Куратор] Вы', player.x + player.size/2, player.y - 8);
            } else {
                // Режим ОТ ПЕРВОГО ЛИЦА — видим только прицел и инструмент перед глазами
                ctx.strokeStyle = '#ffffff';
                ctx.lineWidth = 2;
                ctx.beginPath();
                // Крестик прицела по центру экрана игрока
                ctx.moveTo(player.x + player.size/2 - 8, player.y + player.size/2);
                ctx.lineTo(player.x + player.size/2 + 8, player.y + player.size/2);
                ctx.moveTo(player.x + player.size/2, player.y + player.size/2 - 8);
                ctx.lineTo(player.x + player.size/2, player.y + player.size/2 + 8);
                ctx.stroke();

                // Отрисовка руки/инструмента в углу экрана (имитация взгляда из глаз)
                ctx.fillStyle = BLOCK_TYPES[selectedSlot].color;
                ctx.fillRect(player.x + player.size - 2, player.y + player.size - 2, 12, 12);
            }
        }

        // Главный цикл
        function mainLoop() {
            updateGame();
            renderGame();
            requestAnimationFrame(mainLoop);
        }

        // Старт
        mainLoop();
    </script>
</body>
</html>



