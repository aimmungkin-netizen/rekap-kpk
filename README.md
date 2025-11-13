<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Kalkulator Multiple Taruhan 2D (Multi-Pasaran + Pola A–T)</title>
<style>
  body { font-family: Arial, sans-serif; margin: 25px; background: #f9f9f9; color: #333; }
  h2 { color: #0077cc; }
  select, input { padding: 6px; margin-top: 4px; border-radius: 5px; border: 1px solid #aaa; }
  button { padding: 8px 14px; border: none; border-radius: 5px; cursor: pointer; font-size: 14px; margin: 5px 5px 5px 0; }
  button:hover { opacity: 0.95; }
  .primary { background: #0077cc; color: white; }
  .danger { background: #cc0000; color: white; }
  .success { background: #009933; color: white; }
  .container { background: #fff; padding: 15px; border-radius: 10px; box-shadow: 0 2px 6px rgba(0,0,0,0.1); margin-bottom: 20px; }
  table { border-collapse: collapse; width: 100%; margin-top: 10px; }
  th, td { border: 1px solid #ccc; padding: 6px; text-align: center; }
  th { background: #e0f0ff; }
  .summary { margin-top: 10px; background: #eef8ee; padding: 10px; border-radius: 8px; }
  .controls-row { display:flex; gap:8px; align-items:center; flex-wrap:wrap; margin-top:8px; }
  .top-row { display:flex; align-items:center; gap:10px; flex-wrap:wrap; }
</style>
</head>
<body>

<h2>Kalkulator Multiple Taruhan 2D — Multi Pasaran + Pola</h2>

<div class="container">
  <div class="top-row">
    <label>Pasaran:</label>
    <select id="pasaranSelect" onchange="gantiPasaranAtauPola()">
      <option>KYNDERS</option>
      <option>ORLANDO DAY</option>
      <option>HK SIANG</option>
      <option>CAMBODIA</option>
      <option>BULLSEYE</option>
      <option>SIDNEY</option>
      <option>SIDNEY LOTTO</option>
      <option>SINGAPORE</option>
      <option>ORLANDOEVE</option>
      <option>PCSO</option>
      <option>TAIWAN</option>
      <option>HONGKONG</option>
      <option>HONGKONG LOTTO</option>
      <option>TOTO WUHAN</option>
      <option>OREGONLOTTERY</option>
      <option>KINGSTOWN</option>
    </select>

    <label>Pola:</label>
    <select id="polaSelect" onchange="gantiPasaranAtauPola()">
      <option>A</option>
      <option>B</option>
      <option>C</option>
      <option>D</option>
      <option>E</option>
      <option>F</option>
      <option>G</option>
      <option>H</option>
      <option>I</option>
      <option>J</option>
      <option>K</option>
      <option>L</option>
      <option>M</option>
      <option>N</option>
      <option>O</option>
      <option>P</option>
      <option>Q</option>
      <option>R</option>
      <option>S</option>
      <option>T</option>
    </select>
  </div>
</div>

<div class="container">
  <label>Jumlah nomor dipasang (M):</label>
  <input type="number" id="jumlahNomor" value="10" min="1" max="99">
  
  <label>Payout ratio:</label>
  <input type="number" id="payout" value="98">
  
  <label>Target keuntungan per menang:</label>
  <input type="number" id="target" value="50000">
  
  <label>Minimal taruhan per nomor:</label>
  <input type="number" id="minTaruhan" value="100" step="100">
  
  <p>Total kekalahan (otomatis): <input id="kerugian" value="0" readonly></p>
  <p>Total modal kumulatif: <input id="totalModal" value="0" readonly></p>

  <div class="controls-row">
    <button class="primary" onclick="hitung()">Hitung Multiple</button>
    <button class="success" onclick="tambahKalah()">Tambah Kekalahan</button>
    <button class="success" onclick="tambahMenang()">Menang Hari Ini</button>
    <button onclick="simulasi()">Simulasi 10 Hari</button>
    <button class="danger" onclick="resetKombinasi()">Reset Pola</button>
    <button onclick="exportData()">Ekspor</button>
    <button onclick="triggerImport()">Impor</button>
    <input type="file" id="importFile" accept=".json" style="display:none" onchange="importData(event)">
  </div>
</div>

<div class="container" id="hasil"></div>

<div class="container">
  <h3>Histori Harian</h3>
  <table id="tabelHistori">
    <tr>
      <th>Hari</th>
      <th>Status</th>
      <th>Modal</th>
      <th>Total Kekalahan</th>
      <th>Total Modal</th>
      <th>Tanggal</th> <!-- kolom tanggal baru -->
    </tr>
  </table>
  <div class="summary" id="ringkasan"></div>
</div>

<script>
/* util: format dd-mm-yyyy */
function pad2(n){ return String(n).padStart(2,'0'); }
function todayDdMmYyyy(offsetDays=0){
  const now = new Date();
  if (offsetDays) now.setDate(now.getDate() + offsetDays);
  const d = pad2(now.getDate());
  const m = pad2(now.getMonth()+1);
  const y = now.getFullYear();
  return `${d}-${m}-${y}`;
}

/* data */
let dataSemua = JSON.parse(localStorage.getItem("dataMultiPasaranPola") || "{}");
let pasaran = localStorage.getItem("lastPasaran") || document.getElementById("pasaranSelect").value;
let pola = localStorage.getItem("lastPola") || document.getElementById("polaSelect").value;

function keyAktif() {
  return pasaran + "|" + pola;
}

function ensureKombinasiExist() {
  if (!dataSemua[keyAktif()]) {
    dataSemua[keyAktif()] = { totalKalah: 0, totalModal: 0, histori: [], modalTerakhir: 0 };
  }
}

function simpanData() {
  localStorage.setItem("dataMultiPasaranPola", JSON.stringify(dataSemua));
  localStorage.setItem("lastPasaran", pasaran);
  localStorage.setItem("lastPola", pola);
}

function gantiPasaranAtauPola() {
  pasaran = document.getElementById("pasaranSelect").value;
  pola = document.getElementById("polaSelect").value;
  ensureKombinasiExist();
  simpanData();
  tampilkanData();
}

function hitung() {
  const f = parseInt(document.getElementById("payout").value);
  const T = parseInt(document.getElementById("target").value);
  const M = parseInt(document.getElementById("jumlahNomor").value);
  const L = dataSemua[keyAktif()] ? dataSemua[keyAktif()].totalKalah : 0;
  const minBet = parseInt(document.getElementById("minTaruhan").value);

  if (isNaN(f) || isNaN(T) || isNaN(M) || isNaN(minBet)) { alert("Masukkan nilai numerik yang valid."); return; }
  if (M >= f) return alert("Jumlah nomor tidak boleh ≥ payout ratio!");
  let s = (T + L) / (f - M);
  s = Math.ceil(s / minBet) * minBet;
  const totalModal = M * s;
  const totalMenang = f * s;
  const net = totalMenang - totalModal;

  ensureKombinasiExist();
  dataSemua[keyAktif()].modalTerakhir = totalModal;
  simpanData();

  document.getElementById("hasil").innerHTML = `
    <p>Pola Aktif: <strong>${pola}</strong> | Pasaran: <strong>${pasaran}</strong></p>
    <p>Stake/nomor: <strong>Rp${s.toLocaleString()}</strong></p>
    <p>Total Modal: <strong>Rp${totalModal.toLocaleString()}</strong></p>
    <p>Total Menang: <strong>Rp${totalMenang.toLocaleString()}</strong></p>
    <p>Net: <strong>Rp${net.toLocaleString()}</strong></p>
  `;

  tampilkanData();
}

function tambahKalah() {
  const d = dataSemua[keyAktif()];
  if (!d || !d.modalTerakhir) return alert("Hitung dulu!");
  d.totalKalah += d.modalTerakhir;
  d.totalModal += d.modalTerakhir;
  // tambahkan tanggal saat ini ke histori
  d.histori.push({
    hari: d.histori.length+1,
    status: "Kalah",
    modal: d.modalTerakhir,
    totalKalah: d.totalKalah,
    totalModal: d.totalModal,
    tanggal: todayDdMmYyyy()
  });
  simpanData();
  tampilkanData();
}

function tambahMenang() {
  const d = dataSemua[keyAktif()];
  if (!d || !d.modalTerakhir) return alert("Hitung dulu!");
  const f = parseInt(document.getElementById("payout").value), M = parseInt(document.getElementById("jumlahNomor").value);
  const s = d.modalTerakhir / M;
  const totalMenang = f * s;
  const net = totalMenang - d.modalTerakhir;
  d.totalKalah = Math.max(0, d.totalKalah - net);
  d.totalModal += d.modalTerakhir;
  // tambahkan tanggal saat ini ke histori
  d.histori.push({
    hari: d.histori.length+1,
    status: "Menang",
    modal: d.modalTerakhir,
    totalKalah: d.totalKalah,
    totalModal: d.totalModal,
    tanggal: todayDdMmYyyy()
  });
  simpanData();
  tampilkanData();
}

function tampilkanData() {
  ensureKombinasiExist();
  const d = dataSemua[keyAktif()];
  document.getElementById("kerugian").value = d.totalKalah;
  document.getElementById("totalModal").value = d.totalModal;

  const t = document.getElementById("tabelHistori");
  t.innerHTML = `<tr>
    <th>Hari</th><th>Status</th><th>Modal</th><th>Total Kekalahan</th><th>Total Modal</th><th>Tanggal</th>
  </tr>`;
  (d.histori || []).forEach(r=>{
    const tr = document.createElement("tr");
    const tanggalText = r.tanggal ? r.tanggal : "-";
    tr.innerHTML = `<td>${r.hari}</td><td>${r.status}</td><td>Rp${r.modal.toLocaleString()}</td><td>Rp${r.totalKalah.toLocaleString()}</td><td>Rp${r.totalModal.toLocaleString()}</td><td>${tanggalText}</td>`;
    tr.style.background = r.status=="Menang"?"#d0ffd0":"#ffd0d0";
    t.appendChild(tr);
  });

  const menang = (d.histori || []).filter(x=>x.status=="Menang").length;
  const kalah = (d.histori || []).filter(x=>x.status=="Kalah").length;

  document.getElementById("ringkasan").innerHTML = `
    Kombinasi: <strong>${pasaran} (${pola})</strong><br>
    Hari: ${d.histori.length} | Menang: ${menang} | Kalah: ${kalah}<br>
    Total Modal: Rp${d.totalModal.toLocaleString()}<br>
    Total Kekalahan: Rp${d.totalKalah.toLocaleString()}
  `;
}

function resetKombinasi() {
  if (!confirm("Hapus semua data untuk kombinasi ini?")) return;
  dataSemua[keyAktif()] = { totalKalah:0, totalModal:0, histori:[], modalTerakhir:0 };
  simpanData();
  tampilkanData();
}

/* Simulasi: tambahkan tanggal di output simulasi (mulai hari ini, bertambah 1 hari setiap baris) */
function simulasi() {
  const f = parseInt(document.getElementById("payout").value, 10);
  const T = parseInt(document.getElementById("target").value, 10);
  const M = parseInt(document.getElementById("jumlahNomor").value, 10);
  const minBet = parseInt(document.getElementById("minTaruhan").value, 10);
  let L = 0;
  let log = "";

  for (let i = 1; i <= 10; i++) {
    let s = (T + L) / (f - M);
    s = Math.ceil(s / minBet) * minBet;
    const totalModal = M * s;
    const totalMenang = f * s;
    const net = totalMenang - totalModal;
    const menang = (i === 10);
    if (menang) { L = Math.max(0, L - net); } else { L += totalModal; }
    const tanggal = todayDdMmYyyy(i-1); // hari simulasi: hari ini + (i-1)
    log += `<tr><td>${i}</td><td>${menang ? "Menang" : "Kalah"}</td><td>Rp${totalModal.toLocaleString()}</td><td>Rp${L.toLocaleString()}</td><td>${tanggal}</td></tr>`;
  }

  document.getElementById("hasil").innerHTML = `
    <h4>Simulasi 10 Hari (Menang di hari ke-10)</h4>
    <table>
      <tr><th>Hari</th><th>Status</th><th>Modal</th><th>Total Kekalahan</th><th>Tanggal</th></tr>
      ${log}
    </table>
  `;
}

/* Export / Import */
function exportData() {
  const payload = { exportedAt: new Date().toISOString(), data: dataSemua };
  const blob = new Blob([JSON.stringify(payload,null,2)],{type:"application/json"});
  const a=document.createElement("a");
  a.href=URL.createObjectURL(blob);
  a.download="backup_togel_pola.json";
  a.click();
  alert("Berhasil diekspor!");
}
function triggerImport(){ document.getElementById("importFile").click(); }
function importData(e){
  const f=e.target.files[0]; if(!f)return;
  const r=new FileReader();
  r.onload=function(x){
    try{
      const j=JSON.parse(x.target.result);
      dataSemua=j.data||{};
      simpanData(); tampilkanData();
      alert("Data impor berhasil!");
    }catch(err){alert("Gagal impor: "+err);}
  };
  r.readAsText(f);
}

window.onload=()=>{
  document.getElementById("pasaranSelect").value=pasaran;
  document.getElementById("polaSelect").value=pola;
  ensureKombinasiExist();
  tampilkanData();
};
</script>

</body>
</html>
