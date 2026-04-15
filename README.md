<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PlumoSense</title>

<style>
  body {
    margin: 0;
    font-family: 'Trebuchet MS', Arial;
    background: radial-gradient(circle at top, #1a1a1a, #0b0b0b);
    color: white;
    text-align: center;
  }

  #screen1, #screen2, #screen3, #screen4 {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  #screen2, #screen3, #screen4 {
    display: none;
  }

  .title {
  font-size: 48px;
  font-weight: bold;
  margin-bottom: 10px;
  background: linear-gradient(90deg, #00d4ff, #4cff9a);

  /* Standard property */
  background-clip: text;

  /* WebKit support */
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;

  /* Fallback for browsers that don’t support it */
  color: #00d4ff;
  }

  button {
    width: 240px;
    padding: 14px;
    margin: 10px;
    font-size: 16px;
    border-radius: 14px;
    border: none;
    background: linear-gradient(135deg, #00d4ff, #4cff9a);
    color: black;
    font-weight: bold;
    cursor: pointer;
  }

  video {
    width: 100%;
    max-width: 320px;
    border-radius: 16px;
  }

  .camWrapper { position: relative; }

  .crosshair {
    width: 40px;
    height: 40px;
    border: 2px solid #00d4ff;
    border-radius: 50%;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  #colorBox {
    width: 120px;
    height: 120px;
    margin: 20px;
    border-radius: 16px;
    border: 2px solid white;
  }

  .severe { animation: flash 1s infinite; color: red; }

  @keyframes flash {
    0% { opacity: 1; }
    50% { opacity: 0.3; }
    100% { opacity: 1; }
  }

  .icon {
    font-size: 90px;
    margin: 20px;
  }

  .dots {
    display: flex;
    gap: 8px;
    margin-bottom: 15px;
  }

  .dot {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background: #333;
  }

  .dot.active {
    background: #00d4ff;
  }
</style>
</head>

<body>

<!-- SCREEN 1 -->
<div id="screen1">
  <div class="title">🫁 PlumoSense</div>

  <button onclick="goToScan()">📷 Scan Sample</button>
  <button onclick="goToInstructions()">📖 How to Use</button>
</div>

<!-- SCREEN 2 -->
<div id="screen2">
  <div class="title" style="font-size:28px;">📷 Detect Sample</div>

  <div class="camWrapper">
    <video id="video" autoplay playsinline muted></video>
    <div class="crosshair"></div>
  </div>

  <button onclick="captureColor()">🔍 Scan Now</button>
  <button onclick="goHome()">⬅ Back</button>
</div>

<!-- SCREEN 3 -->
<div id="screen3">
  <div class="title" style="font-size:28px;">🧪 Result</div>

  <div id="colorBox"></div>
  <p id="rgbText"></p>

  <button onclick="goBack()">⬅ Back</button>
</div>

<!-- SCREEN 4 -->
<div id="screen4">
  <div class="title" style="font-size:28px;">📖 Instructions</div>

  <div class="dots" id="dots"></div>

  <div id="instructionPage"></div>

  <div style="margin-top:20px;">
    <!-- NOW ALWAYS VISIBLE -->
    <button id="backBtn" onclick="prevPage()">⬅ Back</button>
    <button id="nextBtn" onclick="nextPage()">Next ➡</button>
  </div>
</div>

<canvas id="canvas" style="display:none;"></canvas>

<script>
let page = 1;
const maxPage = 7;

/* NAV */
function goToScan(){
  screen1.style.display="none";
  screen2.style.display="flex";
}

function goToInstructions(){
  screen1.style.display="none";
  screen4.style.display="flex";
  page=1;
  buildDots();
  showPage();
}

function goToTitle(){
  screen4.style.display="none";
  screen1.style.display="flex";
}

/* CAMERA */
navigator.mediaDevices.getUserMedia({video:{facingMode:"environment"}})
.then(s=>video.srcObject=s);

/* SCAN */
function captureColor(){
  const ctx=canvas.getContext("2d");
  ctx.drawImage(video,0,0,300,300);

  let d=ctx.getImageData(140,140,20,20).data;
  let r=0,g=0,b=0,c=0;

  for(let i=0;i<d.length;i+=4){
    r+=d[i];g+=d[i+1];b+=d[i+2];c++;
  }

  r=Math.floor(r/c);
  g=Math.floor(g/c);
  b=Math.floor(b/c);

  let bright=(r+g+b)/3;
  let t=document.getElementById("rgbText");

  if(bright<70||(b>r&&b>g)){
    t.innerHTML="Severe inflammation possible<br>CONSULT A DOCTOR";
    t.style.color="red";
  } else if(r>g&&r>b){
    t.innerHTML="Healthy";
    t.style.color="green";
  } else {
    t.innerHTML="At risk";
    t.style.color="orange";
  }

  screen2.style.display="none";
  screen3.style.display="flex";
}

/* RESULT BACK */
function goBack(){
  screen3.style.display="none";
  screen2.style.display="flex";
}

/* INSTRUCTIONS */
function buildDots(){
  const d=document.getElementById("dots");
  d.innerHTML="";
  for(let i=1;i<=maxPage;i++){
    let x=document.createElement("div");
    x.className="dot";
    x.id="dot"+i;
    d.appendChild(x);
  }
}

function updateDots(){
  for(let i=1;i<=maxPage;i++){
    let d=document.getElementById("dot"+i);
    if(d) d.classList.remove("active");
  }
  let a=document.getElementById("dot"+page);
  if(a) a.classList.add("active");
}

function showPage(){
  const box=document.getElementById("instructionPage");

  if(page===1) box.innerHTML=`<div class="icon">📦</div><p>Open the box. There will be a basic pipette, a sputum collection tube, a chart to determine results, and a cartridge containing the testing fluid</p>`;
  else if(page===2) box.innerHTML=`<div class="icon">🧪</div><p>Take a few deep breaths and cough hard, afterwards collect sputum in the tube</p>`;
  else if(page===3) box.innerHTML=`<div class="icon">💦</div><p>Use the pipette to draw sputum</p>`;
  else if(page===4) box.innerHTML=`<div class="icon">🧫</div><p>Pour the test fluid in the container, and then add the sputum sample in</p>`;
  else if(page===5) box.innerHTML=`<div class="icon">🕑</div><p>Wait 10-15 minutes for visible clumping</p>`;
  else if(page===6) box.innerHTML=`<div class="icon">🔍</div><p>Scan the sample </p>`;
  else if(page===7) box.innerHTML=`<div class="icon">🔬</div><p>The app will provide a result based on the sample</p>`;

  updateDots();
}

function nextPage(){
  if(page<maxPage){
    page++;
    showPage();
  } else {
    goToTitle();
  }
}

/* FIXED BACK BUTTON LOGIC */
function prevPage(){
  if(page===1){
    goToTitle();   // <-- THIS is the fix you wanted
  } else {
    page--;
    showPage();
  }
}
</script>

</body>
</html># tanishqvithun.github.io
