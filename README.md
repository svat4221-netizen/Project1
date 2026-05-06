<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Family Courier — Система с Авторизацией</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        body { font-family: 'Inter', sans-serif; transition: all 0.3s ease; }
        .glovo-yellow { background-color: #FFC244; }
        .glovo-green { background-color: #00A082; }
        .hidden { display: none !important; }
        .modal { position: fixed; inset: 0; background: rgba(0,0,0,0.8); z-index: 50; display: flex; align-items: center; justify-content: center; padding: 20px; }
    </style>
</head>
<body class="bg-gray-100 min-h-screen">

    <div id="authScreen" class="modal">
        <div class="bg-white w-full max-w-sm rounded-[40px] p-10 text-center shadow-2xl">
            <h1 class="text-3xl font-black italic mb-2 tracking-tighter">FAMILY COURIER</h1>
            <p class="text-gray-400 text-sm mb-10 font-bold uppercase tracking-widest">Выберите профиль</p>
            
            <button onclick="login('courier')" class="w-full glovo-yellow py-5 rounded-2xl font-black uppercase mb-4 hover:scale-105 transition-transform shadow-lg">
                Вход: КУРЬЕР
            </button>
            <button onclick="login('dispatcher')" class="w-full bg-slate-800 text-white py-5 rounded-2xl font-black uppercase hover:scale-105 transition-transform shadow-lg">
                Вход: ДИСПЕТЧЕР
            </button>
        </div>
    </div>

    <div id="app" class="hidden">
        <header id="header" class="px-6 pt-10 pb-8 rounded-b-[40px] shadow-lg sticky top-0 z-10 transition-colors">
            <div class="flex justify-between items-start">
                <div>
                    <h1 class="text-2xl font-black italic tracking-tighter leading-none" id="logoText">FAMILY COURIER</h1>
                    <button onclick="logout()" class="mt-2 text-[10px] font-black uppercase underline opacity-60">Выйти из аккаунта</button>
                </div>
                <div class="bg-white/30 backdrop-blur-md rounded-2xl p-3 text-right">
                    <span class="block text-[10px] font-bold opacity-60 uppercase">Рейтинг</span>
                    <span class="text-xl font-black">★ <span id="ratingDisplay">4.95</span></span>
                </div>
            </div>

            <div class="mt-8 bg-white rounded-3xl p-6 shadow-md flex justify-between items-center text-gray-900">
                <div>
                    <p class="text-[10px] font-bold text-gray-400 uppercase">Баланс системы</p>
                    <p class="text-4xl font-black"><span id="tokenCount">1250</span> <span class="text-lg font-medium text-green-600 italic">₮</span></p>
                </div>
                <div id="roleIndicator" class="text-[10px] font-black bg-black text-white px-3 py-1 rounded-full uppercase italic">КУРЬЕР</div>
            </div>
        </header>

        <main class="px-6 py-8">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl font-black uppercase italic">Активные тикеты</h2>
                <button id="addBtn" onclick="toggleModal('createModal', true)" class="hidden glovo-green text-white w-12 h-12 rounded-full shadow-lg text-2xl font-bold flex items-center justify-center">+</button>
            </div>
            <div id="taskList" class="space-y-4 pb-20"></div>
        </main>
    </div>

    <div id="createModal" class="modal hidden">
        <div class="bg-white w-full max-w-md rounded-[32px] p-8">
            <h2 class="text-2xl font-black mb-6 uppercase italic">Новый заказ</h2>
            <input id="taskName" type="text" class="w-full bg-gray-100 rounded-xl p-4 mb-4 font-bold outline-none" placeholder="Что нужно сделать?">
            <input id="taskPrice" type="number" class="w-full bg-gray-100 rounded-xl p-4 mb-6 font-bold outline-none" placeholder="Цена в ₮">
            <div class="flex gap-4">
                <button onclick="toggleModal('createModal', false)" class="flex-1 font-bold text-gray-400">ОТМЕНА</button>
                <button onclick="createTask()" class="flex-[2] glovo-green text-white py-4 rounded-2xl font-black shadow-lg">СОЗДАТЬ</button>
            </div>
        </div>
    </div>

    <div id="chatModal" class="modal hidden">
        <div class="bg-white w-full max-w-md h-[80vh] rounded-[32px] flex flex-col overflow-hidden shadow-2xl">
            <div class="p-6 bg-gray-50 border-b flex justify-between items-center">
                <h3 class="font-black italic uppercase" id="chatTitle">Чат по заказу</h3>
                <button onclick="toggleModal('chatModal', false)" class="text-gray-400 font-bold">ЗАКРЫТЬ</button>
            </div>
            <div id="chatMessages" class="flex-1 p-6 overflow-y-auto space-y-4">
                </div>
            <div class="p-4 border-t flex gap-2 bg-white">
                <input id="chatInput" type="text" class="flex-1 bg-gray-100 rounded-xl px-4 py-3 outline-none font-medium" placeholder="Напишите сообщение...">
                <button onclick="sendMessage()" class="glovo-green text-white px-6 rounded-xl font-bold italic">ОТПР.</button>
            </div>
        </div>
    </div>

    <script>
        let currentUser = null; // 'courier' или 'dispatcher'
        let tokens = 1250;
        let rating = 4.95;
        let currentChatTaskId = null;
        
        let tasks = [
            { id: 1, title: "Уборка кухни", price: 200, type: "Standard", chat: [] },
            { id: 2, title: "Годовая контрольная", price: 5000, type: "Epic", chat: [] }
        ];

        function login(role) {
            currentUser = role;
            document.getElementById('authScreen').classList.add('hidden');
            document.getElementById('app').classList.remove('hidden');
            
            const header = document.getElementById('header');
            const roleIndicator = document.getElementById('roleIndicator');
            const logo = document.getElementById('logoText');

            if (role === 'courier') {
                header.className = "glovo-yellow px-6 pt-10 pb-8 rounded-b-[40px] shadow-lg sticky top-0 z-10 text-gray-900";
                roleIndicator.innerText = "КУРЬЕР";
                document.getElementById('addBtn').classList.add('hidden');
            } else {
                header.className = "bg-slate-900 px-6 pt-10 pb-8 rounded-b-[40px] shadow-lg sticky top-0 z-10 text-white";
                roleIndicator.innerText = "ДИСПЕТЧЕР";
                roleIndicator.className = "text-[10px] font-black bg-yellow-400 text-black px-3 py-1 rounded-full uppercase italic";
                document.getElementById('addBtn').classList.remove('hidden');
            }
            renderTasks();
        }

        function logout() {
            currentUser = null;
            document.getElementById('authScreen').classList.remove('hidden');
            document.getElementById('app').classList.add('hidden');
        }

        function renderTasks() {
            const list = document.getElementById('taskList');
            list.innerHTML = '';

            tasks.forEach(task => {
                const card = document.createElement('div');
                card.className = "bg-white rounded-[28px] p-5 shadow-sm border border-transparent";
                
                let buttons = `<button onclick="openChat(${task.id})" class="flex-1 bg-gray-100 text-gray-900 font-bold py-3 rounded-xl text-xs uppercase tracking-tighter italic">Открыть чат (${task.chat.length})</button>`;
                
                if (currentUser === 'courier') {
                    buttons += `<button onclick="completeTask(${task.id}, ${task.price})" class="flex-[2] glovo-yellow text-gray-900 font-black py-3 rounded-xl text-xs uppercase italic shadow-md">Принять заказ</button>`;
                } else {
                    buttons += `<button onclick="penaltyTask(${task.id}, ${Math.floor(task.price/2)})" class="flex-1 bg-red-100 text-red-600 font-black py-3 rounded-xl text-[10px] uppercase">Штраф</button>`;
                    buttons += `<button onclick="deleteTask(${task.id})" class="bg-gray-100 p-3 rounded-xl text-gray-400">×</button>`;
                }

                card.innerHTML = `
                    <div class="flex justify-between mb-2">
                        <span class="text-[10px] font-black ${task.type === 'Epic' ? 'bg-purple-100 text-purple-700' : 'bg-green-100 text-green-700'} px-2 py-1 rounded-full uppercase">${task.type}</span>
                        <span class="font-black text-gray-900">${task.price} ₮</span>
                    </div>
                    <h3 class="font-bold text-gray-900 mb-4">${task.title}</h3>
                    <div class="flex gap-2 border-t pt-4">${buttons}</div>
                `;
                list.appendChild(card);
            });
        }

        function completeTask(id, price) {
            tokens += price;
            rating = Math.min(5.00, rating + 0.02);
            tasks = tasks.filter(t => t.id !== id);
            updateStats();
            renderTasks();
        }

        function penaltyTask(id, penalty) {
            tokens -= penalty;
            rating = Math.max(1.00, rating - 0.20);
            updateStats();
            alert("Штраф выписан! Рейтинг курьера упал.");
        }

        function deleteTask(id) { tasks = tasks.filter(t => t.id !== id); renderTasks(); }

        function updateStats() {
            document.getElementById('tokenCount').innerText = tokens;
            document.getElementById('ratingDisplay').innerText = rating.toFixed(2);
        }

        function createTask() {
            const name = document.getElementById('taskName').value;
            const price = parseInt(document.getElementById('taskPrice').value);
            if(name && price) {
                tasks.push({ id: Date.now(), title: name, price: price, type: price > 1000 ? "Epic" : "Standard", chat: [] });
                toggleModal('createModal', false);
                renderTasks();
            }
        }

        // ЧАТ ЛОГИКА
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
            task.chat.push({ role: currentUser, text: input.value, time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}) });
            input.value = '';
            renderMessages();
        }

        function renderMessages() {
            const container = document.getElementById('chatMessages');
            const task = tasks.find(t => t.id === currentChatTaskId);
            container.innerHTML = task.chat.map(m => `
                <div class="flex ${m.role === currentUser ? 'justify-end' : 'justify-start'}">
                    <div class="${m.role === currentUser ? 'bg-green-600 text-white' : 'bg-gray-100 text-gray-900'} max-w-[80%] p-4 rounded-2xl shadow-sm">
                        <p class="text-xs font-black uppercase opacity-50 mb-1">${m.role === 'courier' ? 'Курьер' : 'Диспетчер'}</p>
                        <p class="text-sm font-medium">${m.text}</p>
                        <p class="text-[9px] mt-1 text-right opacity-40">${m.time}</p>
                    </div>
                </div>
            `).join('');
            container.scrollTop = container.scrollHeight;
        }

        function toggleModal(id, show) { document.getElementById(id).classList.toggle('hidden', !show); }
    </script>
</body>
</html>


