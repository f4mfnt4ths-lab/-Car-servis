<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Car Service</title>

<style>
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    background: #f2f2f7;
    color: #111;
}

header {
    background: #111827;
    color: white;
    padding: 25px 18px;
    padding-top: calc(25px + env(safe-area-inset-top));
}

header h1 {
    margin: 0;
    font-size: 28px;
}

header p {
    margin: 5px 0 0;
    color: #cbd5e1;
    font-size: 14px;
}

.container {
    max-width: 700px;
    margin: auto;
    padding: 18px;
}

button {
    border: 0;
    border-radius: 12px;
    padding: 12px 16px;
    font-size: 15px;
    font-weight: 600;
    cursor: pointer;
    transition: opacity 0.2s;
}

button:active {
    opacity: 0.7;
}

.primary {
    background: #2563eb;
    color: white;
}

.secondary {
    background: #e5e7eb;
    color: #111;
}

.danger {
    background: #fee2e2;
    color: #b91c1c;
}

.card {
    background: white;
    border-radius: 18px;
    padding: 18px;
    margin-bottom: 14px;
    box-shadow: 0 3px 12px rgba(0,0,0,.05);
    cursor: pointer;
}

.card h3 {
    margin: 0 0 6px;
    font-size: 20px;
}

.muted {
    color: #6b7280;
    font-size: 14px;
}

.row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 10px;
    margin-bottom: 15px;
}

.actions {
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
    margin: 15px 0 25px;
}

.empty {
    text-align: center;
    padding: 50px 20px;
    background: white;
    border-radius: 18px;
}

.empty .icon {
    font-size: 50px;
    margin-bottom: 10px;
}

.service {
    background: white;
    border-radius: 16px;
    padding: 16px;
    margin: 10px 0;
    box-shadow: 0 2px 8px rgba(0,0,0,.04);
}

.badge {
    display: inline-block;
    background: #dbeafe;
    color: #1d4ed8;
    padding: 4px 10px;
    border-radius: 20px;
    font-size: 13px;
    font-weight: 600;
    margin-bottom: 8px;
}

.modal {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,.55);
    display: flex;
    align-items: flex-end;
    z-index: 100;
}

.modal-content {
    width: 100%;
    max-height: 90vh;
    overflow-y: auto;
    background: white;
    border-radius: 24px 24px 0 0;
    padding: 20px;
    padding-bottom: calc(25px + env(safe-area-inset-bottom));
}

.modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
}

.modal-header h2 {
    margin: 0;
    font-size: 20px;
}

.close {
    background: #eee;
    border-radius: 50%;
    width: 36px;
    height: 36px;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 18px;
}

label {
    display: block;
    margin-top: 12px;
    font-weight: 600;
    font-size: 14px;
}

input, select, textarea {
    width: 100%;
    margin-top: 6px;
    padding: 12px;
    border: 1px solid #d1d5db;
    border-radius: 12px;
    font-size: 16px;
    background: #fff;
}

textarea {
    min-height: 80px;
}

.submit {
    width: 100%;
    margin-top: 20px;
}

.hidden {
    display: none !important;
}
</style>
</head>

<body>

<header>
    <div class="container">
        <h1>🚗 Car Service</h1>
        <p>Облік обслуговування автомобілів</p>
    </div>
</header>

<div class="container">

    <!-- СПИСОК АВТО -->
    <section id="carsPage">
        <div class="row">
            <h2 style="margin:0;">Мої автомобілі</h2>
            <button class="primary" onclick="openCarModal()">＋ Додати авто</button>
        </div>
        <div id="carsList"></div>
    </section>

    <!-- ДЕТАЛІ АВТО -->
    <section id="carPage" class="hidden">
        <button class="secondary" onclick="showCarsList()">← Назад до списку</button>

        <div id="carInfo" style="margin-top:15px;"></div>

        <div class="actions">
            <button class="primary" onclick="openServiceModal()">＋ Додати запис</button>
            <button class="danger" onclick="deleteCurrentCar()">🗑 Видалити авто</button>
        </div>

        <h3 style="margin-bottom: 10px;">Історія обслуговування</h3>
        <div id="serviceList"></div>
    </section>

</div>

<!-- МОДАЛЬНЕ ВІКНО -->
<div id="modal" class="modal hidden">
    <div class="modal-content">
        <div class="modal-header">
            <h2 id="modalTitle"></h2>
            <button class="close" onclick="closeModal()">✕</button>
        </div>
        <form id="modalForm" onsubmit="handleFormSubmit(event)"></form>
    </div>
</div>

<script>
// Стан додатка
let cars = JSON.parse(localStorage.getItem('car_service_data')) || [];
let currentCarId = null;
let currentFormType = null; // 'car' або 'service'

function saveData() {
    localStorage.setItem('car_service_data', JSON.stringify(cars));
}

function renderCars() {
    const list = document.getElementById('carsList');
    if (cars.length === 0) {
        list.innerHTML = `
            <div class="empty">
                <div class="icon">🚘</div>
                <p class="muted">У вас поки немає доданих авто.</p>
            </div>`;
        return;
    }

    list.innerHTML = cars.map(car => `
        <div class="card" onclick="openCarDetails(${car.id})">
            <h3>${car.brand} ${car.model} (${car.year})</h3>
            <div class="muted">Пробіг: <b>${car.mileage.toLocaleString()} км</b> | Держ. номер: <b>${car.number || 'не вказано'}</b></div>
        </div>
    `).join('');
}

function showCarsList() {
    document.getElementById('carsPage').classList.remove('hidden');
    document.getElementById('carPage').classList.add('hidden');
    currentCarId = null;
    renderCars();
}

function openCarDetails(id) {
    currentCarId = id;
    const car = cars.find(c => c.id === id);
    if (!car) return;

    document.getElementById('carsPage').classList.add('hidden');
    document.getElementById('carPage').classList.remove('hidden');

    document.getElementById('carInfo').innerHTML = `
        <div class="card" style="cursor:default;">
            <h3>${car.brand} ${car.model} (${car.year})</h3>
            <p class="muted">Номер: <b>${car.number || 'не вказано'}</b> | VIN: <b>${car.vin || 'не вказано'}</b></p>
            <p class="muted">Поточний пробіг: <b>${car.mileage.toLocaleString()} км</b></p>
        </div>
    `;

    renderServices(car);
}

function renderServices(car) {
    const list = document.getElementById('serviceList');
    if (!car.services || car.services.length === 0) {
        list.innerHTML = `
            <div class="empty">
                <div class="icon">🛠</div>
                <p class="muted">Історія обслуговування порожня.</p>
            </div>`;
        return;
    }

    list.innerHTML = car.services.map(s => `
        <div class="service">
            <span class="badge">${s.date}</span>
            <span class="badge" style="background:#fef3c7; color:#b45309;">${s.mileage.toLocaleString()} км</span>
            <h4 style="margin: 5px 0;">${s.title}</h4>
            <p class="muted" style="margin: 5px 0;">${s.notes || 'Без опису'}</p>
            <p style="margin: 5px 0 0; font-weight: bold; color: #16a34a;">Ціна: ${s.cost ? s.cost + ' грн' : 'не вказано'}</p>
        </div>
    `).join('');
}

function closeModal() {
    document.getElementById('modal').classList.add('hidden');
}

function openCarModal() {
    currentFormType = 'car';
    document.getElementById('modalTitle').innerText = 'Додати автомобіль';
    document.getElementById('modalForm').innerHTML = `
        <label>Марка *</label>
        <input type="text" id="brand" placeholder="напр. BMW, Toyota" required>
        
        <label>Модель *</label>
        <input type="text" id="model" placeholder="напр. X5, Camry" required>
        
        <label>Рік випуску *</label>
        <input type="number" id="year" value="2018" required>
        
        <label>Поточний пробіг (км) *</label>
        <input type="number" id="mileage" placeholder="150000" required>
        
        <label>Держ. номер</label>
        <input type="text" id="number" placeholder="KA1234BK">

        <label>VIN-код</label>
        <input type="text" id="vin" placeholder="17-значний код">

        <button type="submit" class="primary submit">Зберегти авто</button>
    `;
    document.getElementById('modal').classList.remove('hidden');
}

function openServiceModal() {
    currentFormType = 'service';
    document.getElementById('modalTitle').innerText = 'Додати обслуговування';
    const today = new Date().toISOString().split('T')[0];
    
    document.getElementById('modalForm').innerHTML = `
        <label>Назва роботи / запчастини *</label>
        <input type="text" id="stitle" placeholder="Заміна мастила та фільтрів" required>
        
        <label>Дата *</label>
        <input type="date" id="sdate" value="${today}" required>
        
        <label>Пробіг на момент ТО (км) *</label>
        <input type="number" id="smileage" placeholder="150000" required>
        
        <label>Вартість (грн)</label>
        <input type="number" id="scost" placeholder="2500">

        <label>Деталі / Примітки</label>
        <textarea id="snotes" placeholder="Масло Motul 5W-30, фільтри MANN..."></textarea>

        <button type="submit" class="primary submit">Зберегти запис</button>
    `;
    document.getElementById('modal').classList.remove('hidden');
}

function handleFormSubmit(e) {
    e.preventDefault();

    if (currentFormType === 'car') {
        const newCar = {
            id: Date.now(),
            brand: document.getElementById('brand').value,
            model: document.getElementById('model').value,
            year: Number(document.getElementById('year').value),
            mileage: Number(document.getElementById('mileage').value),
            number: document.getElementById('number').value.toUpperCase(),
            vin: document.getElementById('vin').value.toUpperCase(),
            services: []
        };
        cars.push(newCar);
        saveData();
        closeModal();
        renderCars();
    } 
    else if (currentFormType === 'service') {
        const car = cars.find(c => c.id === currentCarId);
        if (car) {
            const newService = {
                id: Date.now(),
                title: document.getElementById('stitle').value,
                date: document.getElementById('sdate').value,
                mileage: Number(document.getElementById('smileage').value),
                cost: document.getElementById('scost').value,
                notes: document.getElementById('snotes').value
            };
            
            // Якщо пробіг сервісу більший за поточний — оновлюємо пробіг авто
            if (newService.mileage > car.mileage) {
                car.mileage = newService.mileage;
            }

            car.services.unshift(newService);
            saveData();
            closeModal();
            openCarDetails(currentCarId);
        }
    }
}

function deleteCurrentCar() {
    if (confirm('Ви дійсно хочете видалити це авто та всю історію його обслуговування?')) {
        cars = cars.filter(c => c.id !== currentCarId);
        saveData();
        showCarsList();
    }
}

// Ініціалізація
renderCars();
</script>

</body>
</html>
