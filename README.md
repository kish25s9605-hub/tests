<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>정하기 게임</title>
<style>
body {margin:0;font-family:"Pretendard","Noto Sans KR",sans-serif;background:#fff;color:#000;text-align:center;}
header {padding:20px;background:#f0f0f0;border-bottom:3px solid #000;font-size:28px;font-weight:bold;}
.page {display:none;padding:20px;}
.active {display:block;}
h2 {margin:15px 0;}
.menu-btn {width:80%;max-width:280px;padding:15px;margin:15px auto;border:none;border-radius:12px;font-size:20px;background:#000;color:white;cursor:pointer;display:block;transition:0.25s;}
.menu-btn:hover {background:#333;transform:scale(1.05);}
.back-btn {margin-top:20px;padding:10px 25px;font-size:16px;background:#555;border:none;border-radius:8px;color:white;cursor:pointer;}
.name-input {margin:8px;padding:10px;width:70%;max-width:250px;border-radius:8px;border:1px solid #000;font-size:16px;}
.roulette-container {margin:30px auto;width:300px;height:300px;position:relative;}
#rouletteCanvas {width:100%;height:100%;border-radius:50%;transition: transform 4s cubic-bezier(0.33,1,0.68,1);}
.pointer {width:0;height:0;border-left:20px solid transparent;border-right:20px solid transparent;border-top:50px solid red;position:absolute;top:-50px;left:calc(50% - 20px);}
.ladder-container {margin:20px auto;width:95%;max-width:800px;height:450px;background:#ddd;border-radius:10px;overflow:hidden;position:relative;}
canvas {background:#eee;}
</style>
</head>
<body>

<header>정하기 게임</header>

<div id="main" class="page active">
  <h2>게임을 선택하세요</h2>
  <button class="menu-btn" onclick="showPage('roulette')">🎡 룰렛 정하기</button>
  <button class="menu-btn" onclick="showPage('ladder')">🎢 사다리 타기</button>
</div>

<div id="roulette" class="page">
  <h2>🎡 룰렛 정하기</h2>
  <input class="name-input" id="r1" placeholder="이름 1">
  <input class="name-input" id="r2" placeholder="이름 2">
  <input class="name-input" id="r3" placeholder="이름 3 (선택)">
  <input class="name-input" id="r4" placeholder="이름 4 (선택)">
  <input class="name-input" id="r5" placeholder="이름 5 (선택)">
  <br>
  <button class="menu-btn" onclick="spinRoulette()">🎯 돌리기</button>
  <div class="roulette-container">
    <canvas id="rouletteCanvas" width="300" height="300"></canvas>
    <div class="pointer"></div>
  </div>
  <div id="rouletteResult"></div>
  <button class="back-btn" onclick="showPage('main')">⬅ 메인으로</button>
</div>

<div id="ladder" class="page">
  <h2>🎢 사다리 타기</h2>
  <input class="name-input" id="l1" placeholder="이름 1">
  <input class="name-input" id="l2" placeholder="이름 2">
  <input class="name-input" id="l3" placeholder="이름 3 (선택)">
  <input class="name-input" id="l4" placeholder="이름 4 (선택)">
  <input class="name-input" id="l5" placeholder="이름 5 (선택)">
  <br>
  <button id="startLadderBtn" class="menu-btn" onclick="startLadder()">게임 시작</button>
  <div class="ladder-container">
    <canvas id="ladderCanvas" width="800" height="450"></canvas>
  </div>
  <div id="ladderResult"></div>
  <button id="ladderBackBtn" class="back-btn" onclick="showPage('main')">⬅ 메인으로</button>
</div>

<script>
// ---------- 페이지 ----------
function showPage(id){
  document.querySelectorAll(".page").forEach(p=>p.classList.remove("active"));
  document.getElementById(id).classList.add("active");
}

// ---------- 룰렛 ----------
const rColors=["#27ae60","#c0392b","#f1c40f","#2980b9","#e67e22"];
let rNames=[];

function drawRoulette(names){
  rNames = names;
  const c=document.getElementById("rouletteCanvas");
  const ctx=c.getContext("2d");
  const len=names.length;
  const arc=(2*Math.PI)/len;
  ctx.clearRect(0,0,c.width,c.height);
  for(let i=0;i<len;i++){
    ctx.beginPath();
    ctx.fillStyle=rColors[i];
    ctx.moveTo(c.width/2,c.height/2);
    ctx.arc(c.width/2,c.height/2,c.width/2,arc*i,arc*(i+1));
    ctx.fill();
    ctx.save();
    ctx.translate(c.width/2,c.height/2);
    ctx.rotate(arc*(i+0.5));
    ctx.fillStyle="#000";
    ctx.font="16px Pretendard";
    ctx.fillText(names[i],c.width/4,5);
    ctx.restore();
  }
}

function spinRoulette(){
  let names=[r1.value.trim(),r2.value.trim(),r3.value.trim(),r4.value.trim(),r5.value.trim()].filter(n=>n);
  if(names.length<2){alert("최소 2명을 입력하세요.");return;}
  drawRoulette(names);

  const len=names.length;
  const winnerIndex=Math.floor(Math.random()*len);
  const arcDeg=360/len;
  const rotateDeg=360*5 + (360 - winnerIndex*arcDeg - arcDeg/2);
  const canvas=document.getElementById("rouletteCanvas");
  canvas.style.transition="transform 4s cubic-bezier(0.33,1,0.68,1)";
  canvas.style.transform=`rotate(${rotateDeg}deg)`;

  setTimeout(()=>{
    document.getElementById("rouletteResult").innerHTML=`<h2>🎉 결과: ${names[winnerIndex]}</h2>`;
  },4000);
}

// ---------- 사다리 ----------
const ladderCanvas=document.getElementById("ladderCanvas");
const lctx=ladderCanvas.getContext("2d");
let ladder=[], ladderNames=[], ladderResults=[], animationInProgress=false, ladderEnd=[];

function startLadder(){
  if(animationInProgress) return;
  ladderNames=[l1.value.trim(),l2.value.trim(),l3.value.trim(),l4.value.trim(),l5.value.trim()].filter(n=>n);
  if(ladderNames.length<2){alert("최소 2명 이상 입력"); return;}
  if(ladderNames.length>5){alert("최대 5명까지 가능"); return;}
  animationInProgress=true;
  document.getElementById("startLadderBtn").disabled=true;
  document.getElementById("ladderBackBtn").disabled=true;
  generateLadder(ladderNames.length);
  generateEndResults();
  ladderResults=[];
  drawLadder(true);
  runAllAnimations();
}

function generateLadder(count){
  ladder=[];
  const canvasW=ladderCanvas.width, canvasH=ladderCanvas.height;
  for(let i=0;i<count;i++) ladder.push({x:canvasW/(count+1)*(i+1),bars:[]});
  for(let i=0;i<count-1;i++){
    const barCount=1+Math.floor(Math.random()*2);
    for(let j=0;j<barCount;j++){
      const y=Math.random()*(canvasH-60)+30;
      ladder[i].bars.push({y});
    }
  }
}

function generateEndResults(){
  ladderEnd=Array(ladderNames.length).fill("❌ 꽝");
  ladderEnd[0]="🎁 당첨";
  ladderEnd.sort(()=>Math.random()-0.5);
}

function drawLadder(hidden=false, people=[]){
  const ctx=lctx;
  ctx.clearRect(0,0,ladderCanvas.width,ladderCanvas.height);
  ctx.fillStyle=hidden?"#bbb":"#eee";
  ctx.fillRect(0,0,ladderCanvas.width,ladderCanvas.height);
  ctx.strokeStyle="#000"; ctx.lineWidth=3;
  ladder.forEach(l=>{ctx.beginPath(); ctx.moveTo(l.x,20); ctx.lineTo(l.x,ladderCanvas.height-20); ctx.stroke();});
  ladder.forEach((l,i)=>{
    if(i>=ladder.length-1) return;
    l.bars.forEach(b=>{ctx.beginPath(); ctx.moveTo(l.x,b.y); ctx.lineTo(ladder[i+1].x,b.y); ctx.stroke();});
  });
  ladderNames.forEach((n,i)=>{ctx.fillStyle="#000"; ctx.font="16px Pretendard"; ctx.fillText(n,ladder[i].x-20,18);});
  people.forEach(p=>{ctx.fillStyle="#f00"; ctx.beginPath(); ctx.arc(p.x,p.y,10,0,Math.PI*2); ctx.fill(); ctx.fillStyle="#000"; ctx.fillText(p.name,p.x-15,p.y-15);});
}

function runAllAnimations(){
  let idx=0;
  function next(){if(idx>=ladderNames.length){showLadderResult(); return;} animatePerson(idx,()=>{idx++; setTimeout(next,200);});}
  next();
}

function animatePerson(idx,done){
  const person={x:ladder[idx].x,y:20,name:ladderNames[idx]};
  const pathX=[ladder[idx].x];
  for(let j=0;j<ladder.length-1;j++){
    const bars=ladder[j].bars;
    const bar=bars.length>0?bars[Math.floor(Math.random()*bars.length)]:null;
    if(bar) pathX.push(ladder[j+1].x);
    else pathX.push(pathX[pathX.length-1]);
  }
  pathX.push(pathX[pathX.length-1]);
  let stepY=2, stepIndex=0;
  function move(){
    if(person.y<ladderCanvas.height-20){
      person.y+=stepY;
      const segHeight=ladderCanvas.height/(pathX.length+1);
      if(person.y>segHeight*(stepIndex+1)&&stepIndex<pathX.length-1) stepIndex++;
      person.x+=(pathX[stepIndex]-person.x)*0.1;
      drawLadder(false,[person]);
      requestAnimationFrame(move);
    } else {ladderResults.push(person); done();}
  }
  move();
}

function showLadderResult(){
  let html="<h2>🎉 사다리 결과</h2>";
  ladderResults.forEach((p,i)=>{html+= `${ladderEnd[i]} → ${p.name}<br>`;});
  document.getElementById("ladderResult").innerHTML=html;
  animationInProgress=false;
  document.getElementById("startLadderBtn").disabled=false;
  document.getElementById("ladderBackBtn").disabled=false;
}
</script>
</body>
</html>
