<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8" />
<title>Змейка Online</title>
<style>
body{margin:0;height:100vh;display:flex;justify-content:center;align-items:center;background:#020617;color:white;font-family:Arial}
canvas{background:#020617;border-radius:14px;box-shadow:0 0 40px rgba(56,189,248,.35)}
#ui{position:fixed;top:10px;left:10px;font-size:14px;opacity:.9}
</style>
</head>
<body>
<div id="ui">🔌 Онлайн-режим</div>
<canvas id="c" width="400" height="400"></canvas>

<script>
// ⚠️ ПРОСТОЙ ONLINE-ПРИМЕР (WebSocket)
// нужен сервер, пример ниже

const c=document.getElementById('c'),ctx=c.getContext('2d');
const box=20,size=20;

let myId=Math.random().toString(36).slice(2);
let players={};
let food={x:5,y:5};
let dir={x:1,y:0};

const ws=new WebSocket('ws://localhost:3000');

ws.onopen=()=>{
  players[myId]={snake:[{x:10,y:10}],color:'#4ade80'};
  send()
};

ws.onmessage=e=>{
  let data=JSON.parse(e.data);
  players=data.players;
  food=data.food;
};

addEventListener('keydown',e=>{
  if(e.key==='ArrowUp')dir={x:0,y:-1}
  if(e.key==='ArrowDown')dir={x:0,y:1}
  if(e.key==='ArrowLeft')dir={x:-1,y:0}
  if(e.key==='ArrowRight')dir={x:1,y:0}
  send()
});

function send(){
  ws.send(JSON.stringify({id:myId,dir}))
}

function draw(){
  ctx.clearRect(0,0,400,400);

  // food
  ctx.fillStyle='red';
  ctx.fillRect(food.x*box,food.y*box,box,box);

  // players
  for(let id in players){
    let p=players[id];
    ctx.fillStyle=p.color||'#22c55e';
    p.snake.forEach(s=>ctx.fillRect(s.x*box,s.y*box,box,box));
  }
}

setInterval(draw,50);
</script>
</body>
</html>

<!-- ================= SERVER (Node.js) =================

npm i ws

server.js:

const WebSocket=require('ws');
const wss=new WebSocket.Server({port:3000});
let players={};
let food={x:5,y:5};

function step(){
  for(let id in players){
    let p=players[id];
    let h={x:p.snake[0].x+p.dir.x,y:p.snake[0].y+p.dir.y};
    p.snake.unshift(h);p.snake.pop();
    if(h.x===food.x&&h.y===food.y){
      p.snake.push({...h});
      food={x:Math.floor(Math.random()*20),y:Math.floor(Math.random()*20)}
    }
  }
}

wss.on('connection',ws=>{
  ws.on('message',msg=>{
    let d=JSON.parse(msg);
    if(!players[d.id])players[d.id]={snake:[{x:10,y:10}],dir:d.dir,color:'#'+Math.floor(Math.random()*16777215).toString(16)};
    players[d.id].dir=d.dir;
  });
});

setInterval(()=>{
  step();
  let data=JSON.stringify({players,food});
  wss.clients.forEach(c=>c.readyState===1&&c.send(data));
},120);

====================================================== -->
