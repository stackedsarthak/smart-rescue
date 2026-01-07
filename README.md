<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>SmartRescue Pro | Google HackSprint</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;800&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<link rel="stylesheet" href="https://unpkg.com/leaflet-routing-machine@latest/dist/leaflet-routing-machine.css">

<style>
:root{
--primary:#ff3e3e;
--accent:#00f2ff;
--bg:#05070a;
--card:#0f172a;
--success:#10b981;
--warning:#f59e0b;
}
*{box-sizing:border-box}
body{
margin:0;font-family:'Plus Jakarta Sans',sans-serif;
background:var(--bg);color:white;height:100vh;
display:flex;flex-direction:column
}
header{
padding:12px 25px;
display:flex;justify-content:space-between;align-items:center;
background:rgba(15,23,42,.9);
border-bottom:1px solid rgba(0,242,255,.2)
}
.dashboard{
flex:1;display:grid;
grid-template-columns:360px 1fr 360px;
gap:12px;padding:12px
}
.panel{
background:var(--card);
border-radius:15px;padding:18px;
display:flex;flex-direction:column;
border:1px solid rgba(255,255,255,.05)
}
#map{flex:1;border-radius:12px;border:1px solid var(--accent)}
button{
border:none;padding:12px;
border-radius:10px;
font-weight:800;cursor:pointer;margin-top:10px
}
.red{background:var(--primary);color:white}
.blue{background:var(--accent);color:black}
.green{background:var(--success);color:white}
.alert{
display:none;margin-top:10px;
background:rgba(245,158,11,.2);
border:1px solid var(--warning);
color:var(--warning);padding:10px;border-radius:10px;
animation:pulse 1.5s infinite
}
@keyframes pulse{50%{opacity:.6}}
.ai{
background:rgba(0,242,255,.1);
border:1px solid rgba(0,242,255,.3);
padding:12px;border-radius:10px;margin-bottom:10px
}
.metric{font-size:.8rem;margin-top:6px}
.metric span{float:right;font-weight:700}
input{
background:#020617;border:1px solid #334155;
color:white;padding:10px;border-radius:8px;margin-top:5px
}
.leaflet-routing-container{display:none!important}
</style>
</head>

<body>

<header>
<b><i class="fa-solid fa-bolt" style="color:#ff3e3e"></i> SMARTRESCUE PRO</b>
<small>AI EMERGENCY SYSTEM • GOOGLE HACKSPRINT</small>
</header>

<div class="dashboard">

<!-- LEFT -->
<div class="panel">
<h3>🚨 Emergency Control</h3>

<label>Accident Location</label>
<input id="locationInput" placeholder="Eg: Rajwada, Indore">

<div class="ai">
<b>AI Severity</b>
<div id="severity" style="font-size:1.6rem;color:#ff3e3e">--%</div>
</div>

<button class="blue" onclick="simulateEmergency()">▶ SIMULATE EMERGENCY</button>
<button class="red" onclick="generateRoute()">Generate AI Route</button>

<div id="trafficAlert" class="alert">
🚦 TRAFFIC CONGESTION DETECTED – AI REROUTING
</div>

<div class="ai">
<b>Live Vitals</b>
<div class="metric">Heart Rate <span id="hr">--</span></div>
<div class="metric">Oxygen <span id="spo2">--</span></div>
<div class="metric">BP <span id="bp">--</span></div>
<div class="metric">ETA <span id="eta">--</span></div>
</div>
</div>

<!-- MAP -->
<div class="panel" style="padding:8px">
<div id="map"></div>
</div>

<!-- RIGHT -->
<div class="panel">
<h3>🏥 AI Hospital Selection</h3>

<div class="ai">
<b id="hospitalName">--</b><br>
<span id="hospitalStatus">Waiting for AI</span>
</div>

<div class="ai">
<b>Analytics</b>
<div class="metric">Response Time <span id="resp">--</span></div>
<div class="metric">Lives Impacted <span>+1</span></div>
</div>

<button class="green" onclick="finalize()">Finalize ER Handover</button>
</div>

</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet-routing-machine@latest/dist/leaflet-routing-machine.js"></script>

<script>
// MAP INIT
const map=L.map('map').setView([22.7196,75.8577],13);
L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png').addTo(map);
setTimeout(()=>map.invalidateSize(),300);

// ICONS
const accidentIcon=L.icon({
iconUrl:"https://cdn-icons-png.flaticon.com/512/564/564619.png",
iconSize:[40,40],iconAnchor:[20,40]
});
const hospitalIcon=L.icon({
iconUrl:"https://cdn-icons-png.flaticon.com/512/1484/1484872.png",
iconSize:[40,40],iconAnchor:[20,40]
});
const ambulanceIcon=L.icon({
iconUrl:"https://cdn-icons-png.flaticon.com/512/2967/2967350.png",
iconSize:[35,35],iconAnchor:[17,17]
});

// DATA
let accident=[22.7196,75.8577];
let hospitals=[
{name:"City Hospital",coord:[22.7533,75.8937],er:4},
{name:"CHL Medical Center",coord:[22.7441,75.8874],er:2}
];

let accidentMarker=L.marker(accident,{icon:accidentIcon}).addTo(map)
.bindPopup("🚨 Accident").openPopup();

let control, ambulance;

// SIMULATION
function simulateEmergency(){
play("1354");
let hr=r(95,140),spo2=r(85,98),bp=r(90,150);
let severity=Math.min(100,Math.round((hr+(100-spo2)+bp)/3));

document.getElementById("severity").innerText=severity+"%";
document.getElementById("hr").innerText=hr+" BPM";
document.getElementById("spo2").innerText=spo2+"%";
document.getElementById("bp").innerText=bp;

const best=hospitals.sort((a,b)=>b.er-a.er)[0];
document.getElementById("hospitalName").innerText=best.name;
document.getElementById("hospitalStatus").innerText=
"ER Slots: "+best.er+" • AI Recommended";
}

// ROUTE + AMBULANCE ANIMATION
function generateRoute(){
if(control)map.removeControl(control);
if(ambulance)map.removeLayer(ambulance);

const dest=hospitals[0];
L.marker(dest.coord,{icon:hospitalIcon}).addTo(map)
.bindPopup("🏥 "+dest.name);

control=L.Routing.control({
waypoints:[L.latLng(accident),L.latLng(dest.coord)],
lineOptions:{styles:[{color:'#00f2ff',weight:6}]},
createMarker:()=>null
}).addTo(map);

control.on('routesfound',e=>{
const route=e.routes[0].coordinates;
let i=0;
ambulance=L.marker(route[0],{icon:ambulanceIcon}).addTo(map);

const t=Math.round(e.routes[0].summary.totalTime/60);
document.getElementById("eta").innerText=t+" min";
document.getElementById("resp").innerText=t+" min";

if(t>12)document.getElementById("trafficAlert").style.display="block";

const move=setInterval(()=>{
ambulance.setLatLng(route[i]);
i++;
if(i>=route.length)clearInterval(move);
},80);

map.fitBounds([accident,dest.coord],{padding:[80,80]});
});
}

// FINALIZE
function finalize(){
play("951");
alert("ER LOCKED ✅\nTrauma team ready.\nGolden Hour secured.");
}

// UTILS
function r(a,b){return Math.floor(Math.random()*(b-a+1))+a}
function play(id){
new Audio(`https://assets.mixkit.co/active_storage/sfx/${id}/${id}-preview.mp3`).play();
}
</script>

</body>
</html>
