<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<title>Dashboard IoT Final</title>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body {
    font-family: Arial;
    text-align: center;
    background: #0059ff44;
}

/* JAM */
#datetime {
    position: fixed;
    top: 10px;
    right: 20px;
    font-size: 14px;
    font-weight: bold;
    background: #001eff;
    color: white;
    padding: 8px 12px;
    border-radius: 10px;
}

/* CARD */
.card {
    display: inline-block;
    padding: 20px;
    border-radius: 15px;
    margin: 10px;
    width: 150px;
    background: white;
}

/* NILAI */
.nilai {
    font-size: 25px;
    font-weight: bold;
}

/* BUTTON */
.btn {
    padding: 12px 20px;
    margin: 10px;
    border: none;
    border-radius: 10px;
    cursor: pointer;
    color: white;
}

.off { background: gray; }
.on { background: green; }

/* NOTIF BAHAYA */
#notifBahaya {
    position: fixed;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    background: red;
    color: white;
    padding: 15px 25px;
    border-radius: 10px;
    font-weight: bold;
    display: none;
    z-index: 999;
}

/* GRAFIK */
.grafik-container {
    display: flex;
    justify-content: center;
    gap: 20px;
    flex-wrap: wrap;
}

canvas { max-width: 300px; }

/* TABEL */
table {
    margin: 30px auto;
    border-collapse: collapse;
    width: 85%;
    background: white;
}

th, td {
    border: 1px solid #ccc;
    padding: 8px;
}

th {
    background: #007bff;
    color: white;
}
</style>
</head>

<body>

<div id="datetime"></div>

<h2>Dashboard Monitoring IoT</h2>

<!-- NOTIF -->
<div id="notifBahaya">⚠️ BAHAYA! Pompa Pestisida Aktif! Jangan masuk greenhouse!</div>

<!-- SENSOR -->
<div id="cardSuhu" class="card">
    <div>Suhu</div>
    <div id="suhu" class="nilai">-- °C</div>
</div>

<div class="card">
    <div>Kelembapan</div>
    <div id="kelembapan" class="nilai">-- %</div>
</div>

<div class="card">
    <div>Cahaya</div>
    <div id="cahaya" class="nilai">-- lux</div>
</div>

<!-- BUTTON -->
<h3>Kontrol</h3>

<button id="lampu" class="btn off" onclick="toggle(this)">Lampu OFF</button>
<button id="kipas" class="btn off" onclick="toggle(this)">Kipas OFF</button>
<button id="pestisida" class="btn off" onclick="toggle(this)">Pompa Pestisida OFF</button>
<button id="pendingin" class="btn off" onclick="toggle(this)">Pompa Pendingin OFF</button>

<!-- GRAFIK -->
<div class="grafik-container">
    <canvas id="chartSuhu"></canvas>
    <canvas id="chartKelembapan"></canvas>
    <canvas id="chartCahaya"></canvas>
</div>

<!-- HISTORI SENSOR -->
<h3>Histori Sensor</h3>
<table id="tabelSensor">
<tr>
<th>Waktu</th>
<th>Suhu</th>
<th>Kelembapan</th>
<th>Cahaya</th>
</tr>
</table>

<!-- HISTORI BUTTON -->
<h3>Histori Durasi Perangkat</h3>
<table id="tabelButton">
<tr>
<th>Perangkat</th>
<th>Mulai</th>
<th>Selesai</th>
<th>Durasi</th>
</tr>
</table>

<script>

// ===== JAM =====
function updateJam() {
    let now = new Date();
    document.getElementById("datetime").innerHTML =
        now.toLocaleDateString('id-ID') + "<br>" +
        now.toLocaleTimeString('id-ID');
}
setInterval(updateJam, 1000);

// ===== STATUS =====
let statusPerangkat = {
    lampu: "OFF",
    kipas: "OFF",
    pendingin: "OFF",
    pestisida: "OFF"
};

let waktuMulai = {
    lampu: null,
    kipas: null,
    pendingin: null,
    pestisida: null
};

// ===== NOTIF =====
function showNotifBahaya() {
    let notif = document.getElementById("notifBahaya");
    notif.style.display = "block";

    setTimeout(() => {
        notif.style.display = "none";
    }, 3000);
}

// ===== FORMAT DURASI =====
function formatDurasi(ms) {
    let detik = Math.floor(ms / 1000);
    let jam = Math.floor(detik / 3600);
    let menit = Math.floor((detik % 3600) / 60);
    let sisa = detik % 60;

    return `${String(jam).padStart(2,'0')}:${String(menit).padStart(2,'0')}:${String(sisa).padStart(2,'0')}`;
}

// ===== ON =====
function setON(btn, nama) {
    if (statusPerangkat[btn.id] === "ON") return;

    btn.classList.replace("off", "on");
    btn.innerHTML = nama + " ON";

    statusPerangkat[btn.id] = "ON";
    waktuMulai[btn.id] = new Date();

    // 🔥 NOTIF KHUSUS PESTISIDA
    if (btn.id === "pestisida") {
        showNotifBahaya();
    }
}

// ===== OFF =====
function setOFF(btn, nama) {
    if (statusPerangkat[btn.id] === "OFF") return;

    btn.classList.replace("on", "off");
    btn.innerHTML = nama + " OFF";

    let selesai = new Date();
    let mulai = waktuMulai[btn.id];
    if (!mulai) return;

    let durasi = formatDurasi(selesai - mulai);

    let row = document.getElementById("tabelButton").insertRow();
    row.insertCell(0).innerHTML = btn.id;
    row.insertCell(1).innerHTML = mulai.toLocaleTimeString();
    row.insertCell(2).innerHTML = selesai.toLocaleTimeString();
    row.insertCell(3).innerHTML = durasi;

    statusPerangkat[btn.id] = "OFF";
    waktuMulai[btn.id] = null;
}

// ===== TOGGLE =====
function toggle(btn) {
    if (btn.classList.contains("off")) {
        setON(btn, btn.innerText.replace(" OFF",""));
    } else {
        setOFF(btn, btn.innerText.replace(" ON",""));
    }
}

// ===== DATA =====
let histori = [];

window.onload = function () {

    const chartSuhu = new Chart(document.getElementById('chartSuhu'), {
        type: 'line',
        data: { labels: [], datasets: [{ label: 'Suhu', data: [] }] },
        options: { animation: false }
    });

    const chartKelembapan = new Chart(document.getElementById('chartKelembapan'), {
        type: 'line',
        data: { labels: [], datasets: [{ label: 'Kelembapan', data: [] }] },
        options: { animation: false }
    });

    const chartCahaya = new Chart(document.getElementById('chartCahaya'), {
        type: 'line',
        data: { labels: [], datasets: [{ label: 'Cahaya', data: [] }] },
        options: { animation: false }
    });

    function updateData() {

        let waktu = new Date().toLocaleTimeString();

        let suhu = Math.floor(Math.random()*11 + 25);
        let kelembapan = Math.floor(Math.random()*20 + 60);
        let cahaya = Math.floor(Math.random()*5000);

        document.getElementById("suhu").innerHTML = suhu + " °C";
        document.getElementById("kelembapan").innerHTML = kelembapan + " %";
        document.getElementById("cahaya").innerHTML = cahaya + " lux";

        // warna suhu
        document.getElementById("cardSuhu").style.backgroundColor =
            suhu > 30 ? "red" : "lightblue";

        // otomatis
        let lampu = document.getElementById("lampu");
        let kipas = document.getElementById("kipas");
        let pendingin = document.getElementById("pendingin");

        suhu > 30 ? setON(kipas,"Kipas") : setOFF(kipas,"Kipas");
        suhu > 30 ? setON(pendingin,"Pendingin") : setOFF(pendingin,"Pendingin");
        cahaya < 2500 ? setON(lampu,"Lampu") : setOFF(lampu,"Lampu");

        // grafik
        chartSuhu.data.labels.push(waktu);
        chartSuhu.data.datasets[0].data.push(suhu);
        chartSuhu.update();

        chartKelembapan.data.labels.push(waktu);
        chartKelembapan.data.datasets[0].data.push(kelembapan);
        chartKelembapan.update();

        chartCahaya.data.labels.push(waktu);
        chartCahaya.data.datasets[0].data.push(cahaya);
        chartCahaya.update();

        histori.push({ waktu, suhu, kelembapan, cahaya });
    }

    setInterval(updateData, 5000);

    setInterval(() => {
        let d = histori[histori.length-1];
        if (!d) return;

        let row = document.getElementById("tabelSensor").insertRow();
        row.insertCell(0).innerHTML = d.waktu;
        row.insertCell(1).innerHTML = d.suhu;
        row.insertCell(2).innerHTML = d.kelembapan;
        row.insertCell(3).innerHTML = d.cahaya;

    }, 30000);
};
</script>

</body>
</html>
