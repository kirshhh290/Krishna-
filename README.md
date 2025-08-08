<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Doraemon Quiz — Birthday Wishes</title>
<style>
  :root{
    --bg:#87CEFA; /* Doraemon-like sky blue */
    --card:#fff;
    --accent:#0b5ed7;
    --soft:#f6f9ff;
    --font-sans: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  }
  html,body{height:100%; margin:0; font-family:var(--font-sans); background:linear-gradient(180deg,var(--bg) 0%, #cfefff 100%); -webkit-font-smoothing:antialiased; -moz-osx-font-smoothing:grayscale;}
  .center {max-width:900px; margin:20px auto;padding:20px; text-align:center;}
  header h1{margin:6px 0; font-size:28px; letter-spacing:1px;}
  .btn {background:var(--accent); color:#fff; border:none; padding:12px 18px; border-radius:10px; font-weight:600; cursor:pointer; box-shadow:0 6px 18px rgba(11,94,215,0.25);}
  .btn:active{transform:translateY(1px)}
  .screen {background:rgba(255,255,255,0.9); border-radius:14px; padding:18px; box-shadow:0 8px 30px rgba(0,0,0,0.08);}
  /* Quiz area */
  #quiz {margin-top:18px;}
  .question {font-size:20px; margin:14px 0;}
  .options {display:flex; flex-wrap:wrap; justify-content:center; gap:10px;}
  .opt {background:var(--soft); border-radius:10px; padding:12px 18px; min-width:200px; cursor:pointer; box-shadow:0 4px 14px rgba(0,0,0,0.06);}
  .opt.correct{outline:3px solid #4caf50;}
  .opt.wrong{outline:3px solid #e53935;}
  .meta {margin-top:12px; color:#333;}
  /* Result + animated wishes */
  #celebration {position:fixed; inset:0; display:none; align-items:center; justify-content:center; z-index:9999; pointer-events:none;}
  #celebration.active {display:flex; pointer-events:auto;}
  .overlay {position:absolute; inset:0; backdrop-filter:blur(4px); background:linear-gradient(180deg, rgba(0,0,0,0.15), rgba(0,0,0,0.2));}
  .congratsBox {position:relative; z-index:10; text-align:center; color:#fff; padding:28px; border-radius:16px;}
  .congratsBig {font-size:44px; font-weight:800; text-shadow:0 8px 30px rgba(0,0,0,0.4); margin-bottom:6px;}
  .sub {font-size:20px; margin-bottom:10px}
  /* Falling "Congratulations" text style */
  .falling-word {position:fixed; left:0; top:-40px; font-weight:700; pointer-events:none; z-index:5; text-shadow:0 4px 12px rgba(0,0,0,.25); color:#fff;}
  /* Heart rain container */
  .hearts {position:absolute; inset:0; z-index:4; pointer-events:none; overflow:hidden;}
  .heart {
    position:absolute; width:28px; height:28px; transform:translateY(-10px) rotate(15deg);
    background:transparent; pointer-events:none;
    animation:fall linear forwards;
  }
  .heart:before, .heart:after {
    content:""; position:absolute; left:14px; top:0; width:14px; height:22px; background:linear-gradient(180deg,#ff6b9a,#ff4d85); border-radius:12px 12px 0 0;
    transform-origin:0 100%;
  }
  .heart:after{left:0; transform-origin:100% 100%;}
  @keyframes fall {
    to { transform: translateY(120vh) rotate(-10deg); opacity:0.05; }
  }
  /* Wishes container and animations */
  .wishes {position:relative; z-index:11; display:flex; gap:12px; flex-direction:column; align-items:center; margin-top:8px;}
  .wish {
    min-width:280px; max-width:86%; padding:16px 20px; border-radius:14px; font-weight:700;
    display:inline-block; color:#111; background:rgba(255,255,255,0.95);
    transform-origin:center; opacity:0; box-shadow:0 8px 30px rgba(0,0,0,0.12);
  }
  /* Individual animation classes */
  .anim-confetti { animation:popIn 700ms ease both; }
  .anim-hearts { animation:floatIn 700ms cubic-bezier(.2,.9,.3,1) both; }
  .anim-sparkle { animation:glowIn 700ms ease both; }
  .anim-zoom { animation:zoomIn 700ms cubic-bezier(.22,.9,.36,1) both; }
  .anim-bounce { animation:bounceIn 1000ms cubic-bezier(.34,1.56,.64,1) both; }
  @keyframes popIn { from { transform:translateY(18px) scale(.9); opacity:0 } to { transform:none; opacity:1 } }
  @keyframes floatIn { from { transform:translateY(36px) rotate(-6deg); opacity:0 } to { transform:none; opacity:1 } }
  @keyframes glowIn { from { transform:scale(.95); opacity:0; box-shadow:0 0 0 rgba(255,255,255,0) } 50% { box-shadow:0 8px 50px rgba(255,200,200,0.12)} to { opacity:1 } }
  @keyframes zoomIn { from { transform:scale(.6); opacity:0 } to { transform:scale(1); opacity:1 } }
  @keyframes bounceIn { 0%{ transform:translateY(-200%); opacity:0 } 60%{ transform:translateY(12px); opacity:1 } 100%{ transform:translateY(0) } }
  /* small sparkle */
  .sparkle { width:12px; height:12px; border-radius:50%; background:linear-gradient(180deg,#ffd4d4,#ff9ab3); box-shadow:0 6px 18px rgba(255,122,150,0.28); display:inline-block; margin-left:8px; transform:translateY(-2px); }
  /* footer controls */
  .controls{margin-top:12px; display:flex; justify-content:center; gap:10px;}
  @media (max-width:520px){
    .opt{min-width:140px;}
    .congratsBig{font-size:34px}
  }
  /* canvas for confetti */
  canvas.confetti-canvas {position:fixed; inset:0; z-index:6; pointer-events:none;}
</style>
</head>
<body>
  <div class="center">
    <header>
      <h1>🎮 Doraemon Quiz Adventure</h1>
      <div style="font-size:14px;color:#073763">Finish the short quiz — get a surprise birthday message sequence!</div>
    </header>

    <div class="screen" id="screen">
      <div id="intro">
        <p style="margin:10px 0">Ready? Press start to play.</p>
        <button id="startBtn" class="btn">Shuru Karo 🚀</button>
      </div>

      <div id="quiz" style="display:none">
        <div class="question" id="qtext"></div>
        <div class="options" id="opts"></div>
        <div class="meta">
          <span id="progress">Question 1 / 3</span>
        </div>
      </div>

      <div id="end" style="display:none; padding:12px;">
        <h3 id="finalScore"></h3>
        <div class="controls">
          <button id="replay" class="btn">Replay</button>
        </div>
      </div>
    </div>

    <small style="display:block; margin-top:10px; color:#024a7b">Tip: This is a demo. You can replace questions easily in the JS section.</small>
  </div>

  <!-- Celebration overlay -->
  <div id="celebration">
    <div class="overlay"></div>
      <canvas class="confetti-canvas" id="confettiCanvas"></canvas>
      <div class="hearts" id="heartsContainer"></div>
      <div id="congratsArea" class="congratsBox">
        <div class="congratsBig" id="congratsMain">Congratulations!</div>
        <div class="sub">You did it 🎉</div>
        <div class="wishes" id="wishesList" aria-live="polite"></div>
        <div style="margin-top:12px"><button id="closeCelebration" class="btn">Close</button></div>
      </div>
  </div>

<script>
/* ---------- Simple quiz data (edit questions here) ---------- */
const quiz = [
  { q: "Doraemon ka rang kya hai?", o: ["Neela", "Laal", "Hara", "Peela"], a: 0 },
  { q: "Nobita ka sabse acha dost kaun hai?", o: ["Suneo", "Gian", "Doraemon", "Dekisugi"], a: 2 },
  { q: "Doraemon ka pocket kya rakhta hai?", o: ["Gadgets", "Khazana", "Books", "Snacks"], a: 0 }
];

/* ---------- UI references ---------- */
const startBtn = document.getElementById('startBtn');
const screen = document.getElementById('screen');
const intro = document.getElementById('intro');
const quizEl = document.getElementById('quiz');
const qtext = document.getElementById('qtext');
const opts = document.getElementById('opts');
const progress = document.getElementById('progress');
const endEl = document.getElementById('end');
const finalScore = document.getElementById('finalScore');
const replay = document.getElementById('replay');

let current = 0, score = 0;

/* ---------- Start quiz ---------- */
startBtn.addEventListener('click', ()=> {
  intro.style.display='none';
  quizEl.style.display='block';
  current = 0; score = 0;
  loadQuestion();
});

function loadQuestion(){
  const item = quiz[current];
  qtext.textContent = item.q;
  opts.innerHTML = '';
  item.o.forEach((optText, i) => {
    const b = document.createElement('button');
    b.className = 'opt';
    b.innerText = optText;
    b.onclick = ()=>{ choose(i,b); };
    opts.appendChild(b);
  });
  progress.textContent = `Question ${current+1} / ${quiz.length}`;
}

/* ---------- Choose answer ---------- */
function choose(i, btn){
  const item = quiz[current];
  // Disable further clicks
  Array.from(opts.children).forEach(x=>x.onclick=null);
  if(i === item.a){
    btn.classList.add('correct');
    score += 10;
  } else {
    btn.classList.add('wrong');
    // highlight correct
    opts.children[item.a].classList.add('correct');
  }
  // short delay to next
  setTimeout(()=>{
    current++;
    if(current < quiz.length) loadQuestion();
    else finishQuiz();
  }, 800);
}

/* ---------- Finish ---------- */
function finishQuiz(){
  quizEl.style.display='none';
  endEl.style.display='block';
  finalScore.innerHTML = `Score: ${score} / ${quiz.length*10}`;
  // trigger celebration if good or just always
  setTimeout(()=> showCelebration(), 600);
}

/* replay */
replay.addEventListener('click', ()=>{
  endEl.style.display='none';
  intro.style.display='block';
});

/* ---------- Celebration & message sequence logic ---------- */
const celebration = document.getElementById('celebration');
const confettiCanvas = document.getElementById('confettiCanvas');
const heartsContainer = document.getElementById('heartsContainer');
const wishesList = document.getElementById('wishesList');
const congratsMain = document.getElementById('congratsMain');
const closeCelebration = document.getElementById('closeCelebration');

closeCelebration.addEventListener('click', hideCelebration);

const wishes = [
  "Happy Birthday merii patni g 🥳🥳",
  "Cuti pie 💖",
  "Dharmi patni g 💋",
  "Wiifffffyyyyy 💖👸🏻",
  "Jaaanu 💗💗"
];

/* show overlay */
function showCelebration(){
  celebration.classList.add('active');
  congratsMain.textContent = "Congratulations!";
  wishesList.innerHTML = '';
  startConfetti();
  startFallingWords();
  spawnHearts(8); // gentle heart rain
  // sequence: show wishes one by one with different animations
  setTimeout(()=> showWish(0,'anim-confetti'), 1200);
  setTimeout(()=> showWish(1,'anim-hearts'), 2400);
  setTimeout(()=> showWish(2,'anim-sparkle'), 3600);
  setTimeout(()=> showWish(3,'anim-zoom'), 4800);
  setTimeout(()=> showWish(4,'anim-bounce'), 6200);
  // stop some effects after a while
  setTimeout(()=> stopFallingWords(), 9000);
  setTimeout(()=> stopConfetti(9000), 11000);
}

/* hide */
function hideCelebration(){
  celebration.classList.remove('active');
  wishesList.innerHTML = '';
  // clear canvas and hearts
  stopConfetti(0);
  heartsContainer.innerHTML = '';
  stopFallingWords();
}

/* ---------- Wish rendering ---------- */
function showWish(idx, animClass){
  if(idx < 0 || idx >= wishes.length) return;
  const w = document.createElement('div');
  w.className = 'wish ' + animClass;
  w.innerHTML = escapeHtml(wishes[idx]);
  // add a small sparkle for the sparkle item
  if(animClass === 'anim-sparkle'){
    const sp = document.createElement('span');
    sp.className = 'sparkle';
    w.appendChild(sp);
  }
  wishesList.appendChild(w);
  // small additional effect for confetti wish
  if(animClass === 'anim-confetti') {
    // extra burst
    burstConfetti(18);
  }
}

/* utility for safety */
function escapeHtml(text){ const d = document.createElement('div'); d.innerText = text; return d.innerHTML; }

/* ---------- Confetti (simple canvas) ---------- */
let confettiCtx, confettiParticles = [], confettiAnim;
function startConfetti(){
  confettiCanvas.width = innerWidth; confettiCanvas.height = innerHeight;
  confettiCtx = confettiCanvas.getContext('2d');
  confettiParticles = [];
  for(let i=0;i<80;i++) confettiParticles.push(createConfetti());
  animateConfetti();
}
function stopConfetti(delay=0){
  setTimeout(()=>{ cancelAnimationFrame(confettiAnim); if(confettiCtx) confettiCtx.clearRect(0,0,confettiCanvas.width,confettiCanvas.height); confettiParticles = []; }, delay);
}
function burstConfetti(n=30){
  for(let i=0;i<n;i++) confettiParticles.push(createConfetti(true));
}
function createConfetti(burst=false){
  return {
    x: Math.random()*innerWidth,
    y: burst ? innerHeight*0.4 : Math.random()*-innerHeight,
    w: 6 + Math.random()*10,
    h: 8 + Math.random()*10,
    vx: (Math.random()-0.5)*6,
    vy: 2 + Math.random()*6,
    rot: Math.random()*360,
    vr: (Math.random()-0.5)*8,
    color: `hsl(${Math.random()*60 + 300},70%,60%)`
  }
}
function animateConfetti(){
  confettiCtx.clearRect(0,0,confettiCanvas.width,confettiCanvas.height);
  confettiParticles.forEach((p, i)=>{
    p.x += p.vx; p.y += p.vy; p.rot += p.vr; p.vy += 0.06;
    confettiCtx.save();
    confettiCtx.translate(p.x, p.y);
    confettiCtx.rotate(p.rot * Math.PI/180);
    confettiCtx.fillStyle = p.color;
    confettiCtx.fillRect(-p.w/2, -p.h/2, p.w, p.h);
    confettiCtx.restore();
    // recycle
    if(p.y > innerHeight + 80) {
      confettiParticles[i] = createConfetti();
    }
  });
  confettiAnim = requestAnimationFrame(animateConfetti);
}

/* ---------- Heart rain (DOM hearts) ---------- */
function spawnHearts(n=10){
  for(let i=0;i<n;i++){
    createHeart(Math.random()*innerWidth, -20 - Math.random()*200, 3000 + Math.random()*4000);
  }
  // continuous spawn a bit
  const handle = setInterval(()=> {
    if(!document.getElementById('celebration').classList.contains('active')) { clearInterval(handle); return; }
    createHeart(Math.random()*innerWidth, -20, 3200 + Math.random()*4000);
  }, 700);
}
function createHeart(left, top, dur){
  const h = document.createElement('div');
  h.className = 'heart';
  h.style.left = (left)+'px';
  h.style.top = (top)+'px';
  h.style.animationDuration = (dur/1000)+'s';
  heartsContainer.appendChild(h);
  // remove after animation
  setTimeout(()=> { try{ heartsContainer.removeChild(h);}catch(e){} }, dur+200);
}

/* ---------- Falling "Congratulations" words ---------- */
let fallingInterval;
function startFallingWords(){
  fallingInterval = setInterval(()=> spawnWord(), 220);
}
function stopFallingWords(){
  clearInterval(fallingInterval);
}
function spawnWord(){
  const w = document.createElement('div');
  w.className = 'falling-word';
  w.style.left = (Math.random()*90)+'vw';
  w.style.top = '-30px';
  w.style.fontSize = (12 + Math.random()*20)+'px';
  w.style.opacity = (0.7 + Math.random()*0.3);
  w.innerText = 'Congratulations';
  document.body.appendChild(w);
  const dur = 4200 + Math.random()*3600;
  w.style.transition = `transform ${dur}ms linear, opacity ${dur}ms linear`;
  requestAnimationFrame(()=> {
    w.style.transform = `translateY(${innerHeight + 60}px) rotate(${ (Math.random()*40)-20 }deg)`;
    w.style.opacity = 0.05;
  });
  setTimeout(()=> { try{ document.body.removeChild(w);}catch(e){} }, dur+100);
}

/* handle resize */
window.addEventListener('resize', ()=> {
  if(confettiCanvas) { confettiCanvas.width = innerWidth; confettiCanvas.height = innerHeight; }
});
</script>
</body>
</html>
