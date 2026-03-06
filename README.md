# rastreio-2027<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Suas Encomendas</title>

<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>

<style>
*{margin:0;padding:0;box-sizing:border-box;font-family:'Inter',sans-serif}

body{
background:linear-gradient(135deg,#0f172a,#1e293b);
color:#fff;
transition:0.4s;
}

.light{
background:#f1f5f9;
color:#111;
}

button{cursor:pointer;border:none}

/* SPLASH */
#splash{
position:fixed;
width:100%;
height:100vh;
background:#0f172a;
display:flex;
align-items:center;
justify-content:center;
flex-direction:column;
z-index:9999;
}

#splash h1{
font-size:28px;
animation:pulse 1.5s infinite;
}

@keyframes pulse{
50%{opacity:0.5}
}

/* LOGIN */
.login{
height:100vh;
display:flex;
align-items:center;
justify-content:center;
}

.login-box{
background:rgba(255,255,255,0.05);
backdrop-filter:blur(15px);
padding:40px;
border-radius:20px;
width:320px;
text-align:center;
}

.login-box input{
width:100%;
padding:12px;
margin-bottom:15px;
border-radius:10px;
border:none;
}

.login-box button{
width:100%;
padding:12px;
border-radius:10px;
background:#22c55e;
color:#fff;
font-weight:bold;
}

/* SISTEMA */
.container{display:none;padding:20px;animation:fade 0.5s}
@keyframes fade{from{opacity:0}to{opacity:1}}

.top{
display:flex;
justify-content:space-between;
margin-bottom:15px;
align-items:center;
}

.theme-btn{
background:#3b82f6;
padding:6px 10px;
border-radius:8px;
color:#fff;
}

.card-grid{
display:grid;
grid-template-columns:repeat(2,1fr);
gap:10px;
margin-bottom:15px;
}

.card{
background:rgba(255,255,255,0.05);
padding:15px;
border-radius:15px;
text-align:center;
}

input[type=text]{
width:100%;
padding:12px;
border-radius:10px;
border:none;
margin-bottom:10px;
}

.rastrear{
width:100%;
padding:12px;
border-radius:10px;
background:#3b82f6;
color:#fff;
font-weight:bold;
}

.progress{
width:100%;
height:12px;
background:#1e293b;
border-radius:10px;
overflow:hidden;
margin:15px 0;
}

.bar{
width:0%;
height:100%;
background:#f59e0b;
transition:1s;
}

.status-box{
background:rgba(255,255,255,0.05);
padding:15px;
border-radius:12px;
margin-bottom:15px;
}

#map{
height:250px;
border-radius:15px;
margin-bottom:15px;
}

.history-item{
background:rgba(255,255,255,0.05);
padding:8px;
border-radius:8px;
margin-bottom:5px;
font-size:13px;
}
</style>
</head>
<body>

<div id="splash">
<h1>📦 Suas Encomendas</h1>
<p>Carregando sistema...</p>
</div>

<div class="login" id="login" style="display:none;">
<div class="login-box">
<h2>Login</h2>
<input type="text" id="user" placeholder="Usuário">
<input type="password" placeholder="Senha">
<button onclick="entrar()">Entrar</button>
</div>
</div>

<div class="container" id="sistema">

<div class="top">
<div id="bemvindo"></div>
<div>
<span id="hora"></span>
<button class="theme-btn" onclick="toggleTheme()">🌙</button>
</div>
</div>

<div class="card-grid">
<div class="card">Em rota<br><b id="rota">0</b></div>
<div class="card">Em trânsito<br><b id="transito">0</b></div>
<div class="card">Entregues<br><b id="entregue">0</b></div>
<div class="card">Total<br><b id="total">0</b></div>
</div>

<input type="text" id="codigo" placeholder="Digite o código de rastreio">
<button class="rastrear" onclick="rastrear()">Rastrear</button>

<div class="progress"><div class="bar" id="bar"></div></div>

<div class="status-box" id="status">Digite um código para rastrear.</div>

<div id="map"></div>

<h3>Histórico</h3>
<div id="historico"></div>

</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<script>
let map;

setTimeout(()=>{
document.getElementById("splash").style.display="none";
document.getElementById("login").style.display="flex";
},2000);

function entrar(){
const nome=document.getElementById("user").value||"Usuário";
document.getElementById("bemvindo").innerHTML="Bem-vindo, "+nome;
document.getElementById("login").style.display="none";
document.getElementById("sistema").style.display="block";
atualizarHora();
setInterval(atualizarHora,1000);
} condition ? true : false

function atualizarHora(){
const agora=new Date();
document.getElementById("hora").innerHTML=
agora.toLocaleDateString()+" "+agora.toLocaleTimeString();
}

function toggleTheme(){
document.body.classList.toggle("light");
}

function rastrear(){
const codigo=document.getElementById("codigo").value.trim();
if(!codigo)return;

let dados=JSON.parse(localStorage.getItem("dadosSite"));
if(!dados){
alert("Configure no painel admin primeiro.");
return;
}

document.getElementById("bar").style.width=dados.porcentagem+"%";

const agora=new Date();
document.getElementById("status").innerHTML=
"📦 Código: "+codigo+"<br>"+
"Status: "+dados.status+" ("+dados.porcentagem+"%)<br>"+
"Local: "+dados.cidade+"<br>"+
"Atualizado em: "+agora.toLocaleString();

document.getElementById("rota").innerHTML=dados.rota;
document.getElementById("transito").innerHTML=dados.transito;
document.getElementById("entregue").innerHTML=dados.entregues;
document.getElementById("total").innerHTML=
Number(dados.rota)+Number(dados.transito)+Number(dados.entregues);

iniciarMapa(dados.cidade);
salvarHistorico(codigo);
}

async function iniciarMapa(cidade){
if(map){map.remove();}
try{
const resposta=await fetch(
"https://nominatim.openstreetmap.org/search?format=json&q="
+encodeURIComponent(cidade)
);
const dados=await resposta.json();
if(dados.length===0){alert("Cidade não encontrada");return;}
const lat=dados[0].lat;
const lon=dados[0].lon;

map=L.map('map').setView([lat,lon],13);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
maxZoom:19
}).addTo(map);

L.marker([lat,lon])
.addTo(map)
.bindPopup("🚚 Objeto em "+cidade)
.openPopup();

}catch(e){alert("Erro ao carregar mapa");}
}

function salvarHistorico(codigo){
const historico=document.getElementById("historico");
const item=document.createElement("div");
item.className="history-item";
item.innerHTML="Código rastreado: "+codigo;
historico.prepend(item);
}
</script>

</body>
</html>
