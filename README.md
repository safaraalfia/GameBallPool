# GameBallPool
Alfia Rohmah Safara (23422039)

<?php
// Game Pool PHP - Banyak Bola + UI Lebih Menarik (SATSET FIX)
?>
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<title>BALLPOOL</title>

<style>
body{
    margin:0;
    min-height:100vh;
    background:linear-gradient(135deg,#0f2027,#203a43,#2c5364);
    display:flex;
    justify-content:center;
    align-items:center;
    font-family:'Segoe UI',sans-serif;
    color:white;
}
.game{
    background:linear-gradient(145deg,#1e293b,#0f172a);
    padding:22px;
    border-radius:22px;
    box-shadow:0 25px 60px rgba(0,0,0,.7);
    width:960px;
}
.header{
    display:flex;
    justify-content:space-between;
    align-items:center;
    margin-bottom:12px;
}
.header h2{
    margin:0;
    letter-spacing:1px;
}
.score{
    background:linear-gradient(145deg,#16a34a,#15803d);
    padding:10px 20px;
    border-radius:14px;
    font-size:18px;
    font-weight:bold;
    box-shadow:inset 0 0 10px rgba(0,0,0,.5);
}
canvas{
    display:block;
    margin:auto;
    background:radial-gradient(circle at center,#0b8a3c,#05612a);
    border:14px solid #5b3a1e;
    border-radius:20px;
    box-shadow:
        inset 0 0 30px rgba(0,0,0,.6),
        0 20px 40px rgba(0,0,0,.8);
}
.footer{
    margin-top:14px;
    display:flex;
    justify-content:space-between;
    align-items:center;
}
.hint{
    font-size:13px;
    opacity:.85;
}
button{
    background:linear-gradient(145deg,#f59e0b,#facc15);
    border:none;
    padding:10px 18px;
    border-radius:14px;
    font-size:14px;
    font-weight:bold;
    cursor:pointer;
    box-shadow:0 8px 18px rgba(0,0,0,.5);
}
button:hover{
    transform:translateY(-2px);
}
</style>
</head>

<body>

<div class="game">
    <div class="header">
        <h2>🎱 BALLPOOL PANGA</h2>
        <div class="score">Skor: <span id="score">0</span></div>
    </div>

    <canvas id="canvas" width="820" height="420"></canvas>

    <div class="footer">
        <div class="hint">Klik & tarik dari bola putih untuk memukul</div>
        <button onclick="resetGame()">🔄 Reset Game</button>
    </div>
</div>

<script>
const canvas=document.getElementById("canvas");
const ctx=canvas.getContext("2d");
let score=0;

// LUBANG
const holes=[
 {x:25,y:25},{x:410,y:25},{x:795,y:25},
 {x:25,y:395},{x:410,y:395},{x:795,y:395}
];

// BOLA (LEBIH BANYAK)
let balls=[];
function initBalls(){
 balls=[
  {x:200,y:210,r:10,dx:0,dy:0,color:"#ffffff",active:true}, // putih

  // segitiga bola
  {x:520,y:210,r:10,dx:0,dy:0,color:"#ff4c4c",active:true},
  {x:540,y:200,r:10,dx:0,dy:0,color:"#4ca6ff",active:true},
  {x:540,y:220,r:10,dx:0,dy:0,color:"#ffd84c",active:true},

  {x:560,y:190,r:10,dx:0,dy:0,color:"#ff9f43",active:true},
  {x:560,y:210,r:10,dx:0,dy:0,color:"#22c55e",active:true},
  {x:560,y:230,r:10,dx:0,dy:0,color:"#a855f7",active:true},

  {x:580,y:200,r:10,dx:0,dy:0,color:"#38bdf8",active:true},
  {x:580,y:220,r:10,dx:0,dy:0,color:"#fb7185",active:true},

  {x:600,y:210,r:10,dx:0,dy:0,color:"#f97316",active:true}
 ];
}
initBalls();

let mouse={x:0,y:0,down:false};

// DRAW
function draw(){
 ctx.clearRect(0,0,canvas.width,canvas.height);

 // lubang
 holes.forEach(h=>{
  ctx.beginPath();
  ctx.arc(h.x,h.y,18,0,Math.PI*2);
  ctx.fillStyle="#000";
  ctx.fill();
 });

 // bola
 balls.forEach(b=>{
  if(!b.active) return;

  // shadow
  ctx.beginPath();
  ctx.arc(b.x+2,b.y+3,b.r,0,Math.PI*2);
  ctx.fillStyle="rgba(0,0,0,.4)";
  ctx.fill();

  ctx.beginPath();
  ctx.arc(b.x,b.y,b.r,0,Math.PI*2);
  ctx.fillStyle=b.color;
  ctx.fill();
 });

 // stick
 if(mouse.down){
  ctx.beginPath();
  ctx.moveTo(balls[0].x,balls[0].y);
  ctx.lineTo(mouse.x,mouse.y);
  ctx.strokeStyle="gold";
  ctx.lineWidth=4;
  ctx.stroke();
 }
}

// FISIKA (TIDAK DIUBAH – SATSET)
function physics(){
 balls.forEach(b=>{
  if(!b.active) return;

  b.x+=b.dx;
  b.y+=b.dy;

  b.dx*=0.995;
  b.dy*=0.995;

  if(Math.abs(b.dx)<0.02) b.dx=0;
  if(Math.abs(b.dy)<0.02) b.dy=0;

  if(b.x<b.r||b.x>canvas.width-b.r) b.dx*=-1;
  if(b.y<b.r||b.y>canvas.height-b.r) b.dy*=-1;

  holes.forEach(h=>{
   if(Math.hypot(b.x-h.x,b.y-h.y)<18 && b.color!=="#ffffff"){
    b.active=false;
    score+=10;
    document.getElementById("score").innerText=score;
   }
  });
 });

 // TABRAKAN
 for(let i=0;i<balls.length;i++){
  for(let j=i+1;j<balls.length;j++){
   let b1=balls[i],b2=balls[j];
   if(!b1.active||!b2.active) continue;

   let dx=b2.x-b1.x, dy=b2.y-b1.y;
   let dist=Math.hypot(dx,dy);
   let min=b1.r+b2.r;

   if(dist<min){
    let nx=dx/dist, ny=dy/dist;
    let p=2*(b1.dx*nx+b1.dy*ny-b2.dx*nx-b2.dy*ny)/2;

    b1.dx-=p*nx; b1.dy-=p*ny;
    b2.dx+=p*nx; b2.dy+=p*ny;

    let overlap=(min-dist)/2;
    b1.x-=overlap*nx; b1.y-=overlap*ny;
    b2.x+=overlap*nx; b2.y+=overlap*ny;
   }
  }
 }
}

function loop(){
 draw();
 physics();
 requestAnimationFrame(loop);
}

// MOUSE
canvas.addEventListener("mousedown",e=>{
 mouse.down=true;
 mouse.x=e.offsetX;
 mouse.y=e.offsetY;
});
canvas.addEventListener("mousemove",e=>{
 if(mouse.down){mouse.x=e.offsetX;mouse.y=e.offsetY;}
});
canvas.addEventListener("mouseup",e=>{
 mouse.down=false;
 let dx=balls[0].x-e.offsetX;
 let dy=balls[0].y-e.offsetY;
 let max=140;
 dx=Math.max(-max,Math.min(max,dx));
 dy=Math.max(-max,Math.min(max,dy));
 balls[0].dx=dx*0.22;
 balls[0].dy=dy*0.22;
});

function resetGame(){
 score=0;
 document.getElementById("score").innerText=score;
 initBalls();
}

loop();
</script>

</body>
</html>
