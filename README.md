
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

      /* ALARM */
        #notifBahaya {
            display: none;
            position: fixed;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: red;
            color: white;
            padding: 15px;
            border-radius: 10px;
        }

        #btnStopAlarm {
            display: none;
            position: fixed;
            top: 80px;
            left: 50%;
            transform: translateX(-50%);
            background: black;
            color: white;
            padding: 10px;
            border-radius: 10px;
        }

/* 🚨 KEDIP MERAH */
.blink {
    animation: blinkBg 0.5s infinite;
}

@keyframes blinkBg {
    0% { background-color: red; }
    50% { background-color: white; }
    100% { background-color: red; }
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

    <h2>Dashboard IoT</h2>

    <div id="notifBahaya">⚠️ Pompa Pestisida Aktif!</div>
    <button id="btnStopAlarm" onclick="stopAlarm()">🔕 Matikan Alarm</button>
    <audio id="alarmSound" src="alarm_clock.ogg"></audio>

    <!-- SENSOR -->
    <div class="card">Suhu<br><span id="suhu">--</span></div>
    <div class="card">Kelembapan<br><span id="kelembapan">--</span></div>
    <div class="card">Cahaya<br><span id="cahaya">--</span></div>

    <!-- BUTTON -->
    <h3>Kontrol</h3>
    <button id="lampu" class="btn off" onclick="toggle(this)">Lampu OFF</button>
    <button id="kipas" class="btn off" onclick="toggle(this)">Kipas OFF</button>
    <button id="pestisida" class="btn off" onclick="toggle(this)">Pestisida OFF</button>
    <button id="pendingin" class="btn off" onclick="toggle(this)">Pendingin OFF</button>

    <!-- GRAFIK -->
    <div class="grafik-container">
        <canvas id="chartSuhu"></canvas>
        <canvas id="chartKelembapan"></canvas>
        <canvas id="chartCahaya"></canvas>
    </div>

    <!-- HISTORI SENSOR -->
    <h3>Histori Sensor</h3>
    <table id="tabelSensor">
        <tr><th>Waktu</th><th>Suhu</th><th>Kelembapan</th><th>Cahaya</th></tr>
    </table>

    <!-- HISTORI PERANGKAT -->
    <h3>Histori Perangkat</h3>
    <table id="tabelPerangkat">
        <tr><th>Perangkat</th><th>Mulai</th><th>Selesai</th><th>Durasi</th></tr>
    </table>

    <script>

        // ===== JAM =====
        setInterval(() => {
            let n = new Date();
            datetime.innerHTML = n.toLocaleDateString() + "<br>" + n.toLocaleTimeString();
        }, 1000);

        // ===== STATUS & WAKTU =====
        let statusPerangkat = { lampu: "OFF", kipas: "OFF", pendingin: "OFF", pestisida: "OFF" };
        let waktuMulai = { lampu: null, kipas: null, pendingin: null, pestisida: null };

        // ===== DURASI =====
        function formatDurasi(ms) {
            let s = Math.floor(ms / 1000);
            return String(Math.floor(s / 3600)).padStart(2, '0') + ":" +
                String(Math.floor((s % 3600) / 60)).padStart(2, '0') + ":" +
                String(s % 60).padStart(2, '0');
        }

        // ===== ALARM =====
        function showNotifBahaya() {
            notifBahaya.style.display = "block";
            btnStopAlarm.style.display = "block";
            alarmSound.loop = true; alarmSound.play();
            document.body.classList.add("blink");
        }
        function stopAlarm() {
            notifBahaya.style.display = "none";
            btnStopAlarm.style.display = "none";
            alarmSound.pause();
            document.body.classList.remove("blink");
        }

        // ===== ON/OFF =====
        function setON(btn, nama) {
            if (statusPerangkat[btn.id] === "ON") return;
            btn.classList.replace("off", "on");
            btn.innerHTML = nama + " ON";
            statusPerangkat[btn.id] = "ON";
            waktuMulai[btn.id] = new Date();

            if (btn.id === "pestisida") showNotifBahaya();
        }

        function setOFF(btn, nama) {
            if (statusPerangkat[btn.id] === "OFF") return;

            let selesai = new Date();
            let mulai = waktuMulai[btn.id];
            if (!mulai) return;

            let durasi = formatDurasi(selesai - mulai);

            let row = tabelPerangkat.insertRow();
            row.insertCell(0).innerHTML = btn.id;
            row.insertCell(1).innerHTML = mulai.toLocaleTimeString();
            row.insertCell(2).innerHTML = selesai.toLocaleTimeString();
            row.insertCell(3).innerHTML = durasi;

            btn.classList.replace("on", "off");
            btn.innerHTML = nama + " OFF";

            statusPerangkat[btn.id] = "OFF";
            waktuMulai[btn.id] = null;

            if (btn.id === "pestisida") stopAlarm();
        }

        // ===== TOGGLE =====
        function toggle(btn) {
            btn.classList.contains("off") ?
                setON(btn, btn.innerText.replace(" OFF", "")) :
                setOFF(btn, btn.innerText.replace(" ON", ""));
        }

        // ===== HISTORI =====
        let histori = [];

        // ===== MAIN =====
        window.onload = function () {

            const chartSuhu = new Chart(chartSuhuEl = document.getElementById('chartSuhu'), {
                type: 'line', data: { labels: [], datasets: [{ label: 'Suhu', data: [] }] }, options: { animation: false }
            });
            const chartKelembapan = new Chart(document.getElementById('chartKelembapan'), {
                type: 'line', data: { labels: [], datasets: [{ label: 'Kelembapan', data: [] }] }, options: { animation: false }
            });
            const chartCahaya = new Chart(document.getElementById('chartCahaya'), {
                type: 'line', data: { labels: [], datasets: [{ label: 'Cahaya', data: [] }] }, options: { animation: false }
            });

            function updateData() {

                let waktu = new Date().toLocaleTimeString();
                let suhu = Math.floor(Math.random() * 11 + 25);
                let kelembapan = Math.floor(Math.random() * 20 + 60);
                let cahaya = Math.floor(Math.random() * 5000);

                document.getElementById("suhu").innerHTML = suhu;
                document.getElementById("kelembapan").innerHTML = kelembapan;
                document.getElementById("cahaya").innerHTML = cahaya;

                // otomatis
                suhu > 30 ? setON(kipas, "Kipas") : setOFF(kipas, "Kipas");
                suhu > 30 ? setON(pendingin, "Pendingin") : setOFF(pendingin, "Pendingin");
                cahaya < 2500 ? setON(lampu, "Lampu") : setOFF(lampu, "Lampu");

                // grafik
                chartSuhu.data.labels.push(waktu);
                chartSuhu.data.datasets[0].data.push(suhu);

                chartKelembapan.data.labels.push(waktu);
                chartKelembapan.data.datasets[0].data.push(kelembapan);

                chartCahaya.data.labels.push(waktu);
                chartCahaya.data.datasets[0].data.push(cahaya);

                chartSuhu.update();
                chartKelembapan.update();
                chartCahaya.update();

                // simpan histori
                histori.push({ waktu, suhu, kelembapan, cahaya });
            }

            // interval
            setInterval(updateData, 5000);

            // histori tampil
            setInterval(() => {
                let d = histori[histori.length - 1];
                if (!d) return;
                let r = tabelSensor.insertRow();
                r.insertCell(0).innerHTML = d.waktu;
                r.insertCell(1).innerHTML = d.suhu;
                r.insertCell(2).innerHTML = d.kelembapan;
                r.insertCell(3).innerHTML = d.cahaya;
            }, 30000);

        };
    </script>

</body>
</html>
