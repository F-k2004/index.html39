<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<title>🛰️ Ion Engine Spacecraft HUD</title>
<tyle>
  html,body{
    margin:0;
    overflow:hidden;
    background:#00060f;
    font-family:system-ui,sans-serif;
  }
  canvas{display:block}

  .hud{
    position:absolute;
    right:16px;
    top:16px;
    width:240px;
    padding:14px;
    border-radius:14px;
    background:rgba(255,255,255,0.06);
    backdrop-filter: blur(10px);
    color:#d9f3ff;
    box-shadow:0 0 30px rgba(120,200,255,0.2);
  }
  .hud h3{
    margin:0 0 10px;
    font-size:15px;
    color:#9fdcff;
  }
  .row{
    display:flex;
    justify-content:space-between;
    margin:6px 0;
    font-size:13px;
  }
  .bar{
    height:6px;
    border-radius:4px;
    background:rgba(255,255,255,0.1);
    overflow:hidden;
    margin-top:4px;
  }
  .fill{
    height:100%;
    background:linear-gradient(90deg,#7fdcff,#ffffff);
    width:0%;
  }
  .status{
    margin-top:10px;
    text-align:center;
    font-weight:600;
  }
</style>
</head>
<body>

<canvas id="c"></canvas>

<div class="hud">
  <h3>🛰️ SPACECRAFT HUD</h3>

  <div class="row"><span>Voltage</span><span id="voltage">0 kV</span></div>
  <div class="bar"><div class="fill" id="vBar"></div></div>

  <div class="row"><span>Thrust</span><span id="thrust">0 mN</span></div>
  <div class="bar"><div class="fill" id="tBar"></div></div>

  <div class="row"><span>Velocity</span><span id="vel">0 m/s</span></div>
  <div class="bar"><div class="fill" id="velBar"></div></div>

  <div class="status" id="status">ENGINE OFF</div>
</div>

<script>
const canvas = document.getElementById("c");
const ctx = canvas.getContext("2d");
let w,h;
function resize(){
  w = canvas.width = innerWidth;
  h = canvas.height = innerHeight;
}
resize();
addEventListener("resize", resize);

// HUD refs
const voltageEl = document.getElementById("voltage");
const thrustEl  = document.getElementById("thrust");
const velEl     = document.getElementById("vel");
const vBar = document.getElementById("vBar");
const tBar = document.getElementById("tBar");
const velBar = document.getElementById("velBar");
const statusEl = document.getElementById("status");

// spacecraft
const ship = {
  x: w/2,
  y: h/2,
  vx: 0,
  vy: 0,
  angle: 0
};

let ions = [];
let thrusting = false;

addEventListener("mousedown", ()=> thrusting = true);
addEventListener("mouseup", ()=> thrusting = false);
addEventListener("mousemove", e=>{
  ship.angle = Math.atan2(e.clientY - ship.y, e.clientX - ship.x);
});

function emitIons(){
  if(!thrusting) return;

  for(let i=0;i<5;i++){
    ions.push({
      x: ship.x - Math.cos(ship.angle)*20,
      y: ship.y - Math.sin(ship.angle)*20,
      vx: -Math.cos(ship.angle)*(2+Math.random()),
      vy: -Math.sin(ship.angle)*(2+Math.random()),
      life:160,
      size:Math.random()*1.5+0.8
    });
  }
}

function drawShip(){
  ctx.save();
  ctx.translate(ship.x, ship.y);
  ctx.rotate(ship.angle);

  ctx.strokeStyle="#bfe9ff";
  ctx.lineWidth=2;
  ctx.beginPath();
  ctx.moveTo(14,0);
  ctx.lineTo(-10,-8);
  ctx.lineTo(-6,0);
  ctx.lineTo(-10,8);
  ctx.closePath();
  ctx.stroke();

  ctx.restore();
}

function updatePhysics(){
  if(thrusting){
    ship.vx += Math.cos(ship.angle)*0.04;
    ship.vy += Math.sin(ship.angle)*0.04;
  }
  ship.x += ship.vx;
  ship.y += ship.vy;

  ship.vx *= 0.995;
  ship.vy *= 0.995;
}

function draw(){
  ctx.fillStyle="rgba(0,6,15,0.3)";
  ctx.fillRect(0,0,w,h);

  emitIons();

  ions.forEach(p=>{
    p.x+=p.vx;
    p.y+=p.vy;
    p.life--;

    ctx.shadowBlur=25;
    ctx.shadowColor="rgba(150,220,255,0.9)";
    ctx.fillStyle=`rgba(190,235,255,${p.life/160})`;
    ctx.beginPath();
    ctx.arc(p.x,p.y,p.size,0,Math.PI*2);
    ctx.fill();
  });
  ions = ions.filter(p=>p.life>0);

  updatePhysics();
  drawShip();
  updateHUD();

  requestAnimationFrame(draw);
}

function updateHUD(){
  const speed = Math.hypot(ship.vx, ship.vy)*60;
  const thrust = thrusting ? 80 : 0;
  const voltage = thrusting ? 28 : 0;

  voltageEl.textContent = voltage.toFixed(1)+" kV";
  thrustEl.textContent = thrust.toFixed(1)+" mN";
  velEl.textContent = speed.toFixed(1)+" m/s";

  vBar.style.width = (voltage/30*100)+"%";
  tBar.style.width = (thrust/100*100)+"%";
  velBar.style.width = Math.min(speed/120*100,100)+"%";

  statusEl.textContent = thrusting ? "ENGINE ONLINE" : "ENGINE OFF";
  statusEl.style.color = thrusting ? "#9ff0ff" : "#6a8fa3";
}

draw();
</script>
</body>
</html>
