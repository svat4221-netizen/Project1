<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Family Courier — Личный кабинет</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .glovo-yellow { background-color: #FFC244; }
        .glovo-green { background-color: #00A082; }
    </style>
</head>
<body class="bg-gray-100 min-h-screen">

    <header class="glovo-yellow px-6 pt-10 pb-8 rounded-b-[40px] shadow-lg sticky top-0 z-10">
        <div class="flex justify-between items-start">
            <div>
                <h1 class="text-3xl font-black italic tracking-tighter text-gray-900">FAMILY COURIER</h1>
                <p class="text-[10px] font-bold opacity-70 mt-1 uppercase tracking-wider">Исполнитель: Курьер №1</p>
            </div>
            <div class="bg-white/40 backdrop-blur-md rounded-2xl p-3 text-right">
                <span class="block text-[10px] font-bold opacity-60 uppercase">Рейтинг</span>
                <span class="text-xl font-black" id="ratingDisplay">★ 4.95</span>
            </div>
        </div>

        <div class="mt-8 bg-white rounded-3xl p-6 shadow-md flex justify-between items-center">
            <div>
                <p class="text-[10px] font-bold text-gray-400 uppercase tracking-widest">Доступно к выводу</p>
                <p class="text-4xl font-black text-gray-900"><span id="tokenCount">1250</span> <span class="text-lg font-medium text-green-600">₮</span></p>
            </div>
            <button onclick="alert('Вывод средств запрошен у Диспетчера!')" class="glovo-green text-white px-6 py-3 rounded-2xl font-bold text-sm hover:opacity-90 active:scale-95 transition-all">
                ВЫВЕСТИ
            </button>
        </div>
    </header>

    <main class="px-6 py-8">
        <h2 class="text-xl font-black uppercase italic mb-6">Доступные заказы</h2>

        <div id="taskList" class="space-y-4">
            <div class="bg-white rounded-[28px] p-5 shadow-sm border border-transparent hover:border-green-500 transition-all" id="task-1">
                <div class="flex justify-between items-start mb-2">
                    <span class="text-[10px] font-black bg-green-100 text-green-700 px-3 py-1 rounded-full uppercase">Standard</span>
                    <span class="text-xs font-bold text-gray-400 italic">Таймер: 1h 30m</span>
                </div>
                <h3 class="text-lg font-bold leading-tight mb-2">Доставка: Чистая комната</h3>
                <p class="text-sm text-gray-500 mb-6">Сложить одежду, протереть пыль, заправить кровать. Требуется фото-отчет.</p>
                <div class="flex items-center justify-between pt-4 border-t border-gray-50">
                    <div class="flex flex-col">
                        <span class="text-[10px] font-bold text-gray-400 uppercase italic">Диспетчер: Мама</span>
                        <span class="text-xl font-black text-gray-900">+150 ₮</span>
                    </div>
                    <button onclick="completeTask(1, 150)" class="glovo-yellow text-gray-900 font-extrabold px-6 py-3 rounded-2xl text-sm shadow-md active:translate-y-1 transition-all">
                        ПРИНЯТЬ
                    </button>
                </div>
            </div>

            <div class="bg-white rounded-[28px] p-5 shadow-sm border border-transparent hover:border-purple-500 transition-all" id="task-2">
                <div class="flex justify-between items-start mb-2">
                    <span class="text-[10px] font-black bg-purple-100 text-purple-700 px-3 py-1 rounded-full uppercase">Epic Contract</span>
                    <span class="text-xs font-bold text-gray-400 italic">Таймер: 14d</span>
                </div>
                <h3 class="text-lg font-bold leading-tight mb-2">Контракт: Отличные оценки</h3>
                <p class="text-sm text-gray-500 mb-6">Закрыть учебную неделю без троек. Проверка по электронному журналу.</p>
                <div class="flex items-center justify-between pt-4 border-t border-gray-50">
                    <div class="flex flex-col">
                        <span class="text-[10px] font-bold text-gray-400 uppercase italic">Заказчик: Папа</span>
                        <span class="text-xl font-black text-gray-900">+2000 ₮</span>
                    </div>
                    <button onclick="completeTask(2, 2000)" class="glovo-yellow text-gray-900 font-extrabold px-6 py-3 rounded-2xl text-sm shadow-md active:translate-y-1 transition-all">
                        ПРИНЯТЬ
                    </button>
                </div>
            </div>
        </div>

        <div id="emptyState" class="hidden text-center py-20">
            <div class="text-5xl mb-4">📦</div>
            <p class="font-bold text-gray-400 italic text-lg">Все заказы доставлены!<br>Диспетчер скоро обновит список.</p>
        </div>
    </main>

    <script>
        let tokens = 1250;
        let rating = 4.95;
        let activeTasks = 2;

        function completeTask(id, reward) {
            // Обновляем токены
            tokens += reward;
            document.getElementById('tokenCount').innerText = tokens;

            // Обновляем рейтинг
            rating = Math.min(5.00, rating + 0.01);
            document.getElementById('ratingDisplay').innerText = '★ ' + rating.toFixed(2);

            // Удаляем карточку задания
            const card = document.getElementById('task-' + id);
            card.style.opacity = '0';
            card.style.transform = 'scale(0.9)';
            
            setTimeout(() => {
                card.remove();
                activeTasks--;
                if (activeTasks === 0) {
                    document.getElementById('emptyState').classList.remove('hidden');
                }
            }, 300);
        }
    </script>
</body>
</html>
