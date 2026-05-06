<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Family Courier Pro — Система управления</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        body { font-family: 'Inter', sans-serif; transition: all 0.3s ease; }
        .glovo-yellow { background-color: #FFC244; }
        .glovo-green { background-color: #00A082; }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 50; }
    </style>
</head>
<body class="bg-gray-100 min-h-screen pb-20">

    <header id="headerBg" class="glovo-yellow px-6 pt-10 pb-8 rounded-b-[40px] shadow-lg sticky top-0 z-10 transition-colors">
        <div class="flex justify-between items-start">
            <div>
                <h1 class="text-3xl font-black italic tracking-tighter text-gray-900 leading-none">FAMILY COURIER</h1>
                <div id="roleBadge" class="inline-block mt-2 bg-black text-white text-[10px] font-bold px-2 py-1 rounded uppercase">Роль: Курьер</div>
            </div>
            <button onclick="toggleRole()" class="bg-white/40 backdrop-blur-md rounded-2xl p-3 text-right hover:bg-white/60 transition-all">
                <span class="block text-[10px] font-bold opacity-60 uppercase">Сменить аккаунт</span>
                <span id="roleActionText" class="text-xs font-black uppercase">Стать Диспетчером</span>
            </button>
        </div>

        <div class="mt-8 bg-white rounded-3xl p-6 shadow-md flex justify-between items-center">
            <div>
                <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest">Баланс системы</p>
                <p class="text-4xl font-black text-gray-900"><span id="tokenCount">1250</span> <span class="text-lg font-medium text-green-600">₮</span></p>
            </div>
            <div class="text-right">
                <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest">Рейтинг</p>
                <p class="text-2xl font-black text-gray-900">★ <span id="ratingDisplay">4.95</span></p>
            </div>
        </div>
    </header>

    <main class="px-6 py-8">
        <div class="flex justify-between items-center mb-6">
            <h2 class="text-xl font-black uppercase italic" id="listTitle">Доступные заказы</h2>
            <button id="addBtn" onclick="openModal()" class="hidden glovo-green text-white w-12 h-12 rounded-full shadow-lg text-2xl font-bold flex items-center justify-center active:scale-90 transition-all">
                +
            </button>
        </div>

        <div id="taskList" class="space-y-4">
            </div>
    </main>

    <div id="modal" class="modal flex items-center justify-center px-6">
        <div class="bg-white w-full max-w-md rounded-[32px] p-8 shadow-2xl">
            <h2 class="text-2xl font-black mb-6 uppercase italic">Новый заказ</h2>
            <div class="space-y-4">
                <div>
                    <label class="block text-[10px] font-bold uppercase text-gray-400 mb-1">Название задачи</label>
                    <input id="taskName" type="text" class="w-full bg-gray-100 rounded-xl p-4 border-none focus:ring-2 focus:ring-yellow-400 outline-none font-bold" placeholder="Напр: Вымыть окна">
                </div>
                <div>
                    <label class="block text-[10px] font-bold uppercase text-gray-400 mb-1">Вознаграждение (₮)</label>
                    <input id="taskPrice" type="number" class="w-full bg-gray-100 rounded-xl p-4 border-none focus:ring-2 focus:ring-yellow-400 outline-none font-bold" placeholder="500">
                </div>
                <div class="flex gap-3 pt-4">
                    <button onclick="closeModal()" class="flex-1 py-4 font-bold text-gray-400 uppercase text-sm">Отмена</button>
                    <button onclick="createTask()" class="flex-[2] glovo-green text-white py-4 rounded-2xl font-black uppercase text-sm shadow-lg">Опубликовать</button>
                </div>
            </div>
        </div>
    </div>

    <script>
        let tokens = 1250;
        let rating = 4.95;
        let isDispatcher = false;
        let tasks = [
            { id: 1, title: "Уборка в гостиной", price: 300, type: "Standard" },
            { id: 2, title: "Месяц без замечаний", price: 5000, type: "Epic" }
        ];

        function renderTasks() {
            const list = document.getElementById('taskList');
            list.innerHTML = '';

            tasks.forEach(task => {
                const card = document.createElement('div');
                card.className = "bg-white rounded-[28px] p-5 shadow-sm border border-transparent transition-all";
                
                // Кнопки зависят от роли
                const actionButtons = isDispatcher 
                    ? `<button onclick="penaltyTask(${task.id}, ${Math.floor(task.price/2)})" class="bg-red-100 text-red-600 font-black px-4 py-3 rounded-2xl text-[10px] uppercase">Штраф (-${Math.floor(task.price/2)}₮)</button>
                       <button onclick="deleteTask(${task.id})" class="bg-gray-100 text-gray-400 font-black px-4 py-3 rounded-2xl text-[10px] uppercase">Удалить</button>`
                    : `<button onclick="completeTask(${task.id}, ${task.price})" class="glovo-yellow text-gray-900 font-extrabold px-8 py-3 rounded-2xl text-sm shadow-md active:translate-y-1 transition-all">ПРИНЯТЬ</button>`;

                card.innerHTML = `
                    <div class="flex justify-between items-start mb-2">
                        <span class="text-[10px] font-black ${task.type === 'Epic' ? 'bg-purple-100 text-purple-700' : 'bg-green-100 text-green-700'} px-3 py-1 rounded-full uppercase">${task.type}</span>
                        <span class="text-xl font-black text-gray-900">${task.price} ₮</span>
                    </div>
                    <h3 class="text-lg font-bold leading-tight mb-4">${task.title}</h3>
                    <div class="flex items-center justify-between pt-4 border-t border-gray-50 gap-2">
                        ${actionButtons}
                    </div>
                `;
                list.appendChild(card);
            });
        }

        function toggleRole() {
            isDispatcher = !isDispatcher;
            document.getElementById('headerBg').classList.toggle('glovo-yellow', !isDispatcher);
            document.getElementById('headerBg').classList.toggle('bg-slate-800', isDispatcher);
            document.getElementById('headerBg').classList.toggle('text-white', isDispatcher);
            
            document.getElementById('roleBadge').innerText = isDispatcher ? 'Роль: Диспетчер' : 'Роль: Курьер';
            document.getElementById('roleBadge').className = isDispatcher ? 'inline-block mt-2 bg-yellow-400 text-black text-[10px] font-bold px-2 py-1 rounded uppercase' : 'inline-block mt-2 bg-black text-white text-[10px] font-bold px-2 py-1 rounded uppercase';
            
            document.getElementById('roleActionText').innerText = isDispatcher ? 'Стать Курьером' : 'Стать Диспетчером';
            document.getElementById('addBtn').classList.toggle('hidden', !isDispatcher);
            document.getElementById('listTitle').innerText = isDispatcher ? 'Управление заказами' : 'Доступные заказы';
            
            renderTasks();
        }

        function completeTask(id, price) {
            tokens += price;
            rating = Math.min(5.00, rating + 0.02);
            updateUI();
            tasks = tasks.filter(t => t.id !== id);
            renderTasks();
        }

        function penaltyTask(id, penalty) {
            tokens -= penalty;
            rating = Math.max(1.00, rating - 0.15); // Штраф сильно бьет по рейтингу
            updateUI();
            alert(`Штраф выписан! Списано ${penalty} ₮ и понижен рейтинг.`);
        }

        function deleteTask(id) {
            tasks = tasks.filter(t => t.id !== id);
            renderTasks();
        }

        function updateUI() {
            document.getElementById('tokenCount').innerText = tokens;
            document.getElementById('ratingDisplay').innerText = rating.toFixed(2);
        }

        // Логика Модалки
        function openModal() { document.getElementById('modal').style.display = 'flex'; }
        function closeModal() { document.getElementById('modal').style.display = 'none'; }

        function createTask() {
            const name = document.getElementById('taskName').value;
            const price = parseInt(document.getElementById('taskPrice').value);
            
            if (name && price) {
                tasks.push({
                    id: Date.now(),
                    title: name,
                    price: price,
                    type: price > 1000 ? "Epic" : "Standard"
                });
                renderTasks();
                closeModal();
                document.getElementById('taskName').value = '';
                document.getElementById('taskPrice').value = '';
            }
        }

        renderTasks();
    </script>
</body>
</html>

