<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<title>Dashboard Gerbang Otomatis</title>

<style>
*{box-sizing:border-box;margin:0;padding:0;font-family:Arial;}

body{
background:url("https://images.unsplash.com/photo-1518770660439-4636190af475") no-repeat center/cover;
background-size:cover;
height:100vh;
display:flex;
flex-direction:column;
overflow:hidden;
color:white;
position:relative;
}

/* overlay gelap futuristik */
body::before{
content:"";
position:fixed;
width:100%;
height:100%;
background:rgba(0,0,0,0.75);
z-index:0;
}

/* animasi garis teknologi */
body::after{
content:"";
position:fixed;
width:200%;
height:200%;
background:linear-gradient(120deg,transparent,rgba(0,255,255,0.18),transparent);
animation:moveBg 14s linear infinite;
z-index:0;
}

@keyframes moveBg{
0%{transform:translate(-50%,-50%);}
100%{transform:translate(0,0);}
}

/* HEADER */
.header{
display:flex;
align-items:center;
justify-content:center;
gap:15px;
padding:10px;
position:relative;
z-index:1;
animation:fadeDown 1s ease;
}

.header img{height:45px;}

.header h1{
font-size:22px;
text-align:center;
}

@keyframes fadeDown{
from{opacity:0;transform:translateY(-20px);}
to{opacity:1;transform:translateY(0);}
}

/* MAIN */
.main{
flex:1;
display:grid;
grid-template-rows:45% 55%;
gap:10px;
padding:10px;
position:relative;
z-index:1;
}

.top{
display:grid;
grid-template-columns:2fr 1fr 1fr;
gap:10px;
}

/* CARD FUTURISTIK */
.card{
background:rgba(255,255,255,0.1);
backdrop-filter:blur(18px);
border-radius:12px;
padding:10px;
display:flex;
flex-direction:column;
gap:10px;
overflow:hidden;
box-shadow:0 0 20px rgba(0,0,0,0.4);
transition:0.3s;
animation:fadeUp 0.8s ease;
}

.card:hover{
transform:translateY(-4px) scale(1.01);
box-shadow:0 0 25px rgba(0,255,255,0.4);
}

@keyframes fadeUp{
from{opacity:0;transform:translateY(20px);}
to{opacity:1;transform:translateY(0);}
}

/* STATUS */
.status{
font-size:20px;
text-align:center;
padding:10px;
border-radius:8px;
background:rgba(4, 15, 213, 0.904);
color:rgb(0, 0, 0);
transition:0.3s;
}

/* BUTTON */
button{
background:linear-gradient(45deg,#00c6ff,#0072ff);
color:white;
padding:6px;
border:none;
border-radius:6px;
cursor:pointer;
transition:0.3s;
}

button:hover{
transform:scale(1.05);
box-shadow:0 0 12px rgba(0,114,255,0.8);
}

/* VIDEO */
video{
width:100%;
height:100%;
object-fit:cover;
border-radius:8px;
}

/* TABLE */
.bottom{height:100%;overflow:hidden;}

table{
width:100%;
height:100%;
border-collapse:collapse;
background:rgba(255,255,255,0.92);
color:black;
font-size:12px;
border-radius:10px;
overflow:hidden;
}

th,td{
border:1px solid #0d47a1;
padding:5px;
text-align:center;
}

th{
background:#0d47a1;
color:white;
}

tbody{
display:block;
height:85%;
overflow:auto;
}

thead, tbody tr{
display:table;
width:100%;
table-layout:fixed;
}

/* POPUP */
.overlay{
position:fixed;
top:0;
left:0;
width:100%;
height:100%;
background:rgba(0,0,0,0.7);
display:none;
justify-content:center;
align-items:center;
z-index:5;
}

.popup{
background:rgba(255,255,255,0.95);
color:black;
padding:20px;
border-radius:12px;
width:300px;
display:flex;
flex-direction:column;
gap:10px;
animation:zoomIn 0.4s ease;
}

@keyframes zoomIn{
from{opacity:0;transform:scale(0.85);}
to{opacity:1;transform:scale(1);}
}
</style>
</head>

<body>

<div class="header">
<img src="logo.png">
<h1>RANCANG BANGUN GERBANG OTOMATIS BERBASIS ESP32 -CAM DENGAN SCAN QR SECARA REAL TIME</h1>
<img src="logo2.png">
</div>

<div class="main">

<div class="top">

<div class="card">
<h3> KAMERA ESP32 -CAM</h3>
<video id="video" autoplay></video>
<canvas id="canvas" style="display:none;"></canvas>
</div>

<div class="card">
<h3>Status Gerbang</h3>
<div id="gateStatus" class="status">MENUNGGU...</div>
<button onclick="openDaftar()">DAFTAR</button>
<button onclick="openData()">DATA PENGGUNA</button>
</div>

<div class="card">
<h3>WAKTU REAL TIME</h3>
<div id="clock"></div>
</div>

</div>

<div class="card bottom">

<h3>Data Keluar Masuk Pengguna (Foto Realtime)</h3>

<table>
<thead>
<tr>
<th>No</th>
<th>ID</th>
<th>Nama</th>
<th>Aktivitas</th>
<th>Waktu</th>
<th>Foto</th>
</tr>
</thead>

<tbody id="logTable"></tbody>

</table>

</div>

</div>

<!-- OVERLAY DAFTAR -->
<div id="overlayDaftar" class="overlay">
<div class="popup">
<h3>Daftar Pengguna</h3>
<input id="idBaru" placeholder="ID Pengguna">
<input id="namaBaru" placeholder="Nama">
<button onclick="simpanUser()">Simpan</button>
<button onclick="tutupOverlay()">Tutup</button>
</div>
</div>

<!-- OVERLAY DATA -->
<div id="overlayData" class="overlay">
<div class="popup">
<h3>Data Pengguna</h3>
<div id="listUser"></div>
<button onclick="tutupOverlay()">Tutup</button>
</div>
</div>

<script>
// ------------------- VARIABEL -------------------
let users=[];
let no=0;
const video=document.getElementById("video");
const canvas=document.getElementById("canvas");
const ctx=canvas.getContext("2d");

// ------------------- KAMERA -------------------
navigator.mediaDevices.getUserMedia({video:true})
.then(stream=>video.srcObject=stream);

// ------------------- JAM -------------------
function updateClock(){clock.textContent=new Date().toLocaleString();}
setInterval(updateClock,1000);
updateClock();

// ------------------- FOTO -------------------
function captureImage(){
canvas.width=video.videoWidth;
canvas.height=video.videoHeight;
ctx.drawImage(video,0,0);
return canvas.toDataURL("image/png");
}

// ------------------- STATUS GERBANG -------------------
function updateStatus(){
const list=["TERBUKA","TERTUTUP","MELAKUKAN SCAN QR"];
updateUI(list[Math.floor(Math.random()*3)]);
}

function updateUI(status){
gateStatus.textContent=status;
if(status==="BUKA") gateStatus.style.background="#d4edda";
else if(status==="TUTUP") gateStatus.style.background="#f8d7da";
else gateStatus.style.background="#fff3cd";
}

setInterval(updateStatus,2000);

// ------------------- LOG -------------------
function addLog(){
if(users.length===0) return;
no++;
let user=users[Math.floor(Math.random()*users.length)];
let row=logTable.insertRow(0);
row.insertCell(0).textContent=no;
row.insertCell(1).textContent=user.id;
row.insertCell(2).textContent=user.nama;
row.insertCell(3).textContent=Math.random()>0.5?"MASUK":"KELUAR";
row.insertCell(4).textContent=new Date().toLocaleString();
let imgCell=row.insertCell(5);
let img=document.createElement("img");
img.src=captureImage();
img.style.width="60px";
imgCell.appendChild(img);
}
setInterval(addLog,7000);

// ------------------- OVERLAY DAFTAR & DATA -------------------
const overlayDaftar = document.getElementById("overlayDaftar");
const overlayData   = document.getElementById("overlayData");
const idBaru        = document.getElementById("idBaru");
const namaBaru      = document.getElementById("namaBaru");
const listUser      = document.getElementById("listUser");

function openDaftar() { overlayDaftar.style.display = "flex"; }
function openData()   { overlayData.style.display = "flex"; renderUser(); }
function tutupOverlay(){ overlayDaftar.style.display="none"; overlayData.style.display="none"; }

function simpanUser(){
    if(!idBaru.value.trim() || !namaBaru.value.trim()) return;

    users.push({id:idBaru.value.trim(), nama:namaBaru.value.trim()});
    idBaru.value=""; namaBaru.value="";
    tutupOverlay();
}

function renderUser(){
    if(users.length===0){ listUser.innerHTML="<p>Belum ada data pengguna</p>"; return; }
    listUser.innerHTML = users.map(u=>`<p>${u.id} - ${u.nama}</p>`).join("");
}
</script>

</body>
</html>
