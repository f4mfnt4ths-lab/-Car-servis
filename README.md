# -Car-<!DOCTYPE html>
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
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
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
    font-size: 30px;
}

header p {
    margin: 5px 0 0;
    color: #cbd5e1;
}

.container {
    max-width: 700px;
    margin: auto;
    padding: 18px;
}

button {
    border: 0;
    border-radius: 12px;
    padding: 13px 16px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
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
    box-shadow: 0 3px 12px rgba(0,0,0,.07);
}

.card h2 {
    margin: 0 0 5px;
}

.muted {
    color: #6b7280;
}

.row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 10px;
}

.actions {
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
    margin: 15px 0;
}

.empty {
    text-align: center;
    padding: 70px 20px;
}

.empty .icon {
    font-size: 60px;
}

.service {
    background: white;
    border-radius: 16px;
    padding: 16px;
    margin: 10px 0;
}

.badge {
    display: inline-block;
    background: #dbeafe;
    color: #1d4ed8;
    padding: 5px 9px;
    border-radius: 20px;
    font-size: 13px;
    font-weight: 600;
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
    max-height: 92vh;
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
}

.modal-header h2 {
    margin: 0;
}

.close {
    background: #eee;
    border-radius: 50%;
    width: 40px;
    height: 40px;
    padding: 0;
}

label {
    display: block;
    margin-top: 15px;
    font-weight: 600;
}

input,
select,
textarea {
    width: 100%;
    margin-top: 6px;
    padding: 13px;
    border: 1px solid #d1d5db;
    border-radius: 12px;
    font-size: 16px;
    background: white;
}

textarea {
    min-height: 90px;
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
            <h2>Мої автомобілі</h2>
            <button class="primary" onclick="openCarForm()">＋ Авто</button>
        </div>

        <div id="carsList"></div>

    </section>


    <!-- СТОРІНКА АВТО -->
    <section id="carPage" class="hidden">

        <button class="secondary" onclick="showCars()">← Назад</button>

        <div id="carInfo" style="margin-top:15px;"></div>

        <div class="actions">
            <button class="primary" onclick="openServiceForm()">
                ＋ Обслуговування
            </button>

            <button class="secondary" onclick="editCar()">
                ✏️ Редагувати
            </button>
        </div>

        <h2>Історія</h2>

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

        <form id="form"></form>

    </div>

</div>


<script>

const