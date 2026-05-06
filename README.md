<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Family Courier — Ultimate System</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .glovo-yellow { background-color: #FFC244; }
        .glovo-green { background-color: #00A082; }
        .hidden { display: none !important; }
        .modal { position: fixed; inset: 0; background: rgba(0,0,0,0.85); z-index: 50; display: flex; align-items: center; justify-content: center; padding: 20px; }
    </style>
</head>
<body class="bg-gray-50 min-h-screen">

    <div id="authScreen" class="modal">
        <div class="bg-white w-full max-w-sm rounded-[40px] p-10 text-center shadow-2xl">
            <h1 class="text-3xl font-black italic mb-2 tracking-tighter">FAMILY COURIER</h1>
            <p class="text-gray-400 text-[10px] font-bold uppercase tracking-[0.2em] mb-10">Авторизация</p>
            <button onclick="login('courier')" class="w-full glovo-yellow py-5 rounded-2xl font-black uppercase mb-4 active:scale-95 transition-all">Вход: Курьер</button>
            <button onclick="login('dispatcher')" class="w-full bg-slate-900 text-white py-5 rounded-2xl font-black uppercase active:scale-95 transition-all">Вход: Диспетчер</button>
        </div>
    </div>

    <div id="app" class="hidden">
        <header id="header" class="px-6 pt-10 pb-8 rounded-b-[40px] shadow-lg sticky top-0 z-10 transition-colors">
            <div class="flex justify-between items-start">
                <div>
                    <h1 class="text-2xl font-black italic tracking-tighter leading-none" id="logoText">FAMILY COURIER</h1>
                    <button onclick="logout()" class="mt-2 text-[10px] font-black uppercase underline opacity-60">Сменить профиль</button>
                </div>
                <div class="bg-white/20 backdrop-blur-md rounded-2xl p-3 text-right">
                    <span class="block text-[10px] font-bold opacity-60 uppercase">Рейтинг</span>
                    <span class="text-xl font-black text-white" id="ratingDisplay">★ 4.95</span>
                </div>
            </div>
            <div class="mt-8 bg-white rounded-3xl p-6 shadow-md flex justify-between items-center text-gray-900">
                <div>
                    <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest leading-none mb-1">Ваш Баланс</p>
                    <p class="text-4xl font-black"><span id="tokenCount">1250</span> <span class="text-lg font-medium text-green-600">₮</span></p>
                </div>
                <div id="roleIndicator" class="text-[9px] font-black bg-black text-white px-3 py-1 rounded-full uppercase italic"></div>
            </div>
        </header>

        <main class="px-6 py-8 pb-24">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl font-black uppercase italic" id="mainTitle">Активные заказы</h2>
                <button id="addBtn" onclick="toggleModal('createModal', true)" class="hidden glovo-green text-white w-12 h-12 rounded-full shadow-lg text-2xl font-bold flex items-center justify-center">+</button>
            </div>
            <div id="taskList" class="space-y-4"></div>
        </main>
    </div>

    <div id="createModal" class="modal hidden">
        <div class="bg-white w-full max-w-md rounded-[32px] p-8">
            <h2 class="text-2xl font-black mb-6 uppercase italic">Новое задание</h2>
            <input id="taskName" type="text" class="w-full bg-gray-100 rounded-xl p-4 mb-4 font-bold outline-none" placeholder="Название заказа">
            <input id="taskPrice" type="number" class="w-full bg-gray-100 rounded-xl p-4 mb-6 font-bold outline-none" placeholder="Награда ₮">
            <div class="flex gap-4">
                <button onclick="toggleModal('createModal', false)" class="flex-1 font-bold text-gray-400 uppercase text-xs">Отмена</button>
                <button onclick="createTask()" class="flex-[2] glovo-green text-white py-4 rounded-2xl font-black shadow-lg uppercase text-xs">Опубликовать</button>
            </div>
        </div>
    </div>

    <div id="chatModal" class="modal hidden">
        <div class="bg-white w-full max-w-md h-[80vh] rounded-[32px] flex flex-col overflow-hidden shadow-2xl relative">
            <div class="p-6 bg-gray-50 border-b flex justify-between items-center">
                <h3 class="font-black italic uppercase text-sm" id="chatTitle">Чат</h3>
                <button onclick="toggleModal('chatModal', false)" class="text-gray-400 font-bold">×</button>
            </div>
            <div id="chatMessages" class="flex-1 p-6 overflow-y-auto space-y-4 bg-gray-50"></div>
            <div class="p-4 border-t flex gap-2 bg-white">
                <input id="chatInput" type="text" class="flex-1 bg-gray-100 rounded-xl px-4 py-3 outline-none font-medium text-sm" placeholder="Введите сообщение...">
                <button onclick="sendMessage()" class="glovo-green text-white px-6 rounded-xl font-bold text-xs uppercase italic">Отпр.</button>
            </div>
        </div>
    </div>

    <script>
        let currentUser = null;
        let tokens = 1250;
        let rating = 4.95;
        let currentChatTaskId = null;
        
        let tasks = [
            { id: 1, title: "Генеральная уборка", price: 500, type: "Standard", chat: [] },
            { id: 2, title: "Пятерка за экзамен", price: 3000, type: "Epic", chat: [] }
        ];

        function login(role) {
            currentUser = role;
            document.getElementById('authScreen').classList.add('hidden');
            document.getElementById('app').classList.remove('hidden');
            const header = document.getElementById('header');
            const roleIndicator = document.getElementById('roleIndicator');
            
            if (role === 'courier') {
                header.className = "glovo-yellow px-6 pt-10 pb-8 rounded-b-[40px] shadow-lg sticky top-0 z-10";
                roleIndicator.innerText = "КУРЬЕР";
                roleIndicator.className = "text-[9px] font-black bg-black text-white px-3 py-1 rounded-full uppercase italic";
                document.getElementById('addBtn').classList.add('hidden');
            } else {
                header.className = "bg-slate-900 px-6 pt-10 pb-8 rounded-b-[40px] shadow-lg sticky top-0 z-10 text-white";
                roleIndicator.innerText = "ДИСПЕТЧЕР";
                roleIndicator.className = "text-[9px] font-black bg-yellow-400 text-black px-3 py-1 rounded-full uppercase italic";
                document.getElementById('addBtn').classList.remove('hidden');
            }
            renderTasks();
        }

        function logout() { location.reload(); }

        function renderTasks() {
            const list = document.getElementById('taskList');
            list.innerHTML = '';

            tasks.forEach(task => {
                const card = document.createElement('div');
                card.className = "bg-white rounded-[32px] p-6 shadow-sm border border-transparent";
                
                let controls = `<button onclick="openChat(${task.id})" class="flex-1 bg-gray-100 text-gray-900 font-black py-4 rounded-2xl text-[10px] uppercase italic">Чат (${task.chat.length})</button>`;
                
                if (currentUser === 'courier') {
                    controls += `<button onclick="completeTask(${task.id}, ${task.price})" class="flex-[2] glovo-yellow text-gray-900 font-black py-4 rounded-2xl text-[10px] uppercase italic shadow-md">Завершить: +${task.price}₮</button>`;
                } else {
                    controls += `
                        <button onclick="rewardAction(${task.id})" class="flex-1 bg-green-100 text-green-700 font-black py-4 rounded-2xl text-[10px] uppercase">Бонус</button>
                        <button onclick="penaltyAction(${task.id})" class="flex-1 bg-red-100 text-red-600 font-black py-4 rounded-2xl text-[10px] uppercase">Штраф</button>
                        <button onclick="deleteTask(${task.id})" class="bg-gray-50 px-4 rounded-2xl text-gray-300 font-bold">×</button>
                    `;
                }

                card.innerHTML = `
                    <div class="flex justify-between items-center mb-3">
                        <span class="text-[9px] font-black ${task.type === 'Epic' ? 'bg-purple-100 text-purple-700' : 'bg-green-100 text-green-700'} px-3 py-1 rounded-full uppercase">${task.type}</span>
                        <span class="text-xl font-black text-gray-900">${task.price} ₮</span>
                    </div>
                    <h3 class="text-lg font-bold text-gray-800 mb-6 leading-tight">${task.title}</h3>
                    <div class="flex gap-2 border-t border-gray-50 pt-5">${controls}</div>
                `;
                list.appendChild(card);
            });
        }

        // ДЕЙСТВИЯ ДИСПЕТЧЕРА
        function rewardAction(id) {
            const bonus = prompt("Введите сумму поощрения в токенах:", "100");
            if (bonus && !isNaN(bonus)) {
                tokens += parseInt(bonus);
                rating = Math.min(5.00, rating + 0.05);
                updateStats();
                alert(`Курьеру начислено поощрение: ${bonus} ₮!`);
            }
        }

        function penaltyAction(id) {
            if (confirm("Выдать штраф за невыполнение? (-250 ₮ и рейтинг)")) {
                tokens -= 250;
                rating = Math.max(1.00, rating - 0.20);
                updateStats();
                alert("Штраф применен!");
            }
        }

        function completeTask(id, price) {
            tokens += price;
            rating = Math.min(5.00, rating + 0.02);
            tasks = tasks.filter(t => t.id !== id);
            updateStats();
            renderTasks();
            alert("Заказ выполнен! Средства зачислены.");
        }

        function updateStats() {
            document.getElementById('tokenCount').innerText = tokens;
            document.getElementById('ratingDisplay').innerText = rating.toFixed(2);
        }

        function createTask() {
            const name = document.getElementById('taskName').value;
            const price = parseInt(document.getElementById('taskPrice').value);
            if(name && price) {
                tasks.push({ id: Date.now(), title: name, price: price, type: price >= 1000 ? "Epic" : "Standard", chat: [] });
                toggleModal('createModal', false);
                renderTasks();
                document.getElementById('taskName').value = '';
                document.getElementById('taskPrice').value = '';
            }
        }

        function deleteTask(id) { tasks = tasks.filter(t => t.id !== id); renderTasks(); }

        // ЧАТ
        function openChat(id) {
            currentChatTaskId = id;
            const task = tasks.find(t => t.id === id);
            document.getElementById('chatTitle').innerText = task.title;
            renderMessages();
            toggleModal('chatModal', true);
        }

        function sendMessage() {
            const input = document.getElementById('chatInput');
            if(!input.value) return;
            const task = tasks.find(t => t.id === currentChatTaskId);
            task.chat.push({ sender: currentUser, text: input.value });
            input.value = '';
            renderMessages();
            renderTasks(); // Обновить количество сообщений на карточке
        }

        function renderMessages() {
            const container = document.getElementById('chatMessages');
            const task = tasks.find(t => t.id === currentChatTaskId);
            container.innerHTML = task.chat.map(m => `
                <div class="flex ${m.sender === currentUser ? 'justify-end' : 'justify-start'}">
                    <div class="${m.sender === currentUser ? 'bg-[#00A082] text-white' : 'bg-white text-gray-900 shadow-sm'} max-w-[85%] p-4 rounded-2xl">
                        <p class="text-[8px] font-black uppercase opacity-50 mb-1">${m.sender === 'courier' ? 'Курьер' : 'Диспетчер'}</p>
                        <p class="text-sm font-medium leading-tight">${m.text}</p>
                    </div>
                </div>
            `).join('');
            container.scrollTop = container.scrollHeight;
        }

        function toggleModal(id, show) { document.getElementById(id).classList.toggle('hidden', !show); }
    </script>
</body>
</html>



