<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8"/>
  <title>Kutay Intro (Custom Cursor — Canvas Geometric BG)</title>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>
  <style>
    html, body { margin:0; height:100%; background:#111827; }
    .wrap { position:relative; width:100%; max-width:1200px; height:300px; margin:0 auto; }
    #bgcanvas { position:absolute; inset:0; z-index:1; }
    .title {
      position:absolute; inset:0; display:flex; align-items:center; justify-content:center;
      font-family:"Fira Code",ui-monospace,Menlo,Consolas,monospace;
      font-size:52px; color:#eaeef7; font-weight:700; letter-spacing:1px;
      text-shadow:0 2px 18px rgba(0,255,204,.25); white-space:pre; text-align:center; padding:0 16px;
      z-index:2;
    }
    .cursor { display:inline-block; width:1ch; text-align:center; color:#00ffcc; animation: blink 1s step-start infinite; }
    @keyframes blink { 50% { opacity:0 } }
    .badge { position:absolute; bottom:14px; right:18px; font-size:14px; color:#9aa6b2; opacity:.6; z-index:2; }
    @media (max-width:768px){ .title{font-size:36px} .wrap{height:220px} }
  </style>
</head>
<body>
  <div class="wrap">
    <canvas id="bgcanvas"></canvas>
    <div class="title" id="typebox"></div>
    <div class="badge">akutayural</div>
  </div>

  <!-- GEOMETRIC BACKGROUND (pure Canvas) -->
<script>
(function(){
  const canvas = document.getElementById('bgcanvas');
  const ctx = canvas.getContext('2d');
  const DPR = Math.max(1, window.devicePixelRatio || 1);
  const palette = ['#00ffcc', '#7aa2f7', '#22d3ee'];
  let W=0, H=0, shapes=[];

  function measure(){
    const p = canvas.parentElement; // .wrap
    W = p.clientWidth; H = p.clientHeight;
    canvas.width  = Math.max(1, Math.floor(W * DPR));
    canvas.height = Math.max(1, Math.floor(H * DPR));
    canvas.style.width  = W + 'px';
    canvas.style.height = H + 'px';
    ctx.setTransform(DPR,0,0,DPR,0,0);
  }

  const rand = (a,b) => a + Math.random()*(b-a);
  const pick = arr => arr[(Math.random()*arr.length)|0];

  // === Orta yasak bölge (text bandı) ===
  function band() {
    const gap = 150;                 // ortada boş bırakılacak yükseklik
    const top = (H/2) - gap/2;
    const bot = (H/2) + gap/2;
    return { top, bot, gap };
  }

  function make(count=44){
    shapes.length = 0;
    const { top, bot } = band();

    for(let i=0;i<count;i++){
      const sides = Math.random() < 0.6 ? 6 : 3;
      const r   = rand(14, 34);
      const x   = rand(r, Math.max(r, W - r));

      // y: orta bandın DIŞINDA kalacak
      let y;
      if (Math.random() < 0.5) {
        y = rand(r, Math.max(r, top - r));
      } else {
        y = rand(Math.min(bot + r, H - r), Math.max(bot + r, H - r));
      }

      const rot   = rand(0, Math.PI*2);
      const speed = rand(0.002, 0.008);
      const vx    = rand(-0.18, 0.18);
      const vy    = rand(-0.18, 0.18);
      const col   = pick(palette);
      const alpha = rand(0.08, 0.16);       // çok düşük opaklık = yazı hep net
      shapes.push({sides,r,x,y,rot,speed,vx,vy,col,alpha});
    }
  }

  function poly(cx, cy, r, sides, rot){
    ctx.beginPath();
    for(let i=0;i<sides;i++){
      const a = rot + i*(2*Math.PI/sides);
      const px = cx + Math.cos(a)*r, py = cy + Math.sin(a)*r;
      i===0 ? ctx.moveTo(px,py) : ctx.lineTo(px,py);
    }
    ctx.closePath();
  }

  function loop(){
    const { top, bot } = band();
    ctx.clearRect(0,0,W,H);

    for(const s of shapes){
      s.rot += s.speed;
      s.x += s.vx; s.y += s.vy;

      // Kenarlardan sekme
      if (s.x < s.r)       { s.x = s.r;         s.vx *= -1; }
      if (s.x > W - s.r)   { s.x = W - s.r;     s.vx *= -1; }
      if (s.y < s.r)       { s.y = s.r;         s.vy *= -1; }
      if (s.y > H - s.r)   { s.y = H - s.r;     s.vy *= -1; }

      // === Orta banda girerse nazikçe it ===
      if (s.y + s.r > top && s.y - s.r < bot) {
        // merkeze göre yukarı/aşağı it
        const center = (top + bot)/2;
        const dir = s.y < center ? -1 : 1;      // üstteyse yukarı, alttaysa aşağı
        s.y += dir * 0.9;                       // itme miktarı
        s.vy += dir * 0.02;                     // yavaş hız ayarı
      }

      poly(s.x, s.y, s.r, s.sides, s.rot);
      ctx.globalAlpha = s.alpha;
      ctx.fillStyle = s.col; ctx.fill();
      ctx.globalAlpha = Math.min(1, s.alpha + 0.12);
      ctx.lineWidth = 1; ctx.strokeStyle = 'rgba(24,48,70,0.6)'; ctx.stroke();
      ctx.globalAlpha = 1;
    }
    requestAnimationFrame(loop);
  }

  function init(){ measure(); make(44); loop(); }
  window.addEventListener('load', init);
  window.addEventListener('resize', () => { measure(); make(shapes.length); });
})();
</script>

  <!-- TYPEWRITER (senin hız ayarların) -->
  <script>
    // --- Hızlar (ms) ---
    const SPEED = {
      type: 150, insert: 150, backspace: 150, move: 150,
      pauseShort: 700, pauseMed: 1100, pauseLong: 1600
    };

    const sleep = (ms) => new Promise(r => setTimeout(r, ms));

    function render({el, prefix, content, cursorIndex}) {
      const before = content.slice(0, cursorIndex);
      const after  = content.slice(cursorIndex);
      el.innerHTML = "";
      el.append(
        document.createTextNode(prefix + before),
        Object.assign(document.createElement("span"), {className:"cursor", textContent:"|"}),
        document.createTextNode(after)
      );
    }
    async function typeAtEnd(state, text, spd=SPEED.type){ for(const ch of text){ state.content+=ch; state.cursorIndex++; render(state); await sleep(spd);} }
    async function moveCursor(state,to,spd=SPEED.move){ while(state.cursorIndex!==to){ state.cursorIndex += (state.cursorIndex<to?1:-1); render(state); await sleep(spd);} }
    async function backspaceAt(state,n=1,spd=SPEED.backspace){ for(let i=0;i<n;i++){ if(state.cursorIndex>0){ state.content=state.content.slice(0,state.cursorIndex-1)+state.content.slice(state.cursorIndex); state.cursorIndex--; render(state); await sleep(spd);} } }
    async function insertAt(state,text,spd=SPEED.insert){ for(const ch of text){ state.content=state.content.slice(0,state.cursorIndex)+ch+state.content.slice(state.cursorIndex); state.cursorIndex++; render(state); await sleep(spd);} }

    async function runOnce(state){
      await typeAtEnd(state,"Kutay");                      await sleep(SPEED.pauseMed);
      await moveCursor(state,0); await sleep(200);         await moveCursor(state,1); await backspaceAt(state,1); await sleep(SPEED.pauseShort);
      await moveCursor(state,0); await insertAt(state,"@"); await insertAt(state,"a"); await insertAt(state,"k"); await sleep(SPEED.pauseMed);
      await moveCursor(state,state.content.length);        await insertAt(state,"ural"); await sleep(SPEED.pauseLong);
      await moveCursor(state,0); await moveCursor(state,1); await backspaceAt(state,1); // '@' sil
      await moveCursor(state,1);                           await insertAt(state,"hmet"); await sleep(SPEED.pauseShort);
      await moveCursor(state,state.content.length);        await insertAt(state,".dev"); await sleep(SPEED.pauseLong);
    }

    (async function main(){
      const el = document.getElementById("typebox");
      const prefix = "👋 Hey There! I'm ";
      const state  = { el, prefix, content:"", cursorIndex:0 };

      render(state);
      while(true){
        state.content=""; state.cursorIndex=0; render(state); await sleep(500);
        await runOnce(state);
        await moveCursor(state,state.content.length); await sleep(200);
        await backspaceAt(state,state.content.length); await sleep(600);
      }
    })();
  </script>
</body>
</html>

<p align="center">
  <img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&size=22&duration=3000&pause=1000&color=00F7FF&center=true&vCenter=true&width=800&lines=Python+Software+Engineer+🐍;FinTech+%7C+E-Wallets+%7C+Crypto+Platforms+💳;FastAPI+%7C+Flask+%7C+Django+⚡;AWS+%7C+Docker+%7C+Kubernetes+%7C+ArgoCD+☁️;Always+learning%2C+always+innovating+🚀" alt="Typing SVG" />
</p>

<div align="center">
  <a href="https://www.ahmetkutayural.dev">
    <img src="https://img.shields.io/badge/Portfolio-FF5722?style=for-the-badge&logo=google-chrome&logoColor=white" />
  </a>
  <a href="https://www.linkedin.com/in/akutayural">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white" />
  </a>
  <a href="https://medium.com/@ahmetkutayural">
    <img src="https://img.shields.io/badge/Medium-000000.svg?style=for-the-badge&logo=medium&logoColor=white" />
  </a>
  <a href="https://dev.to/akutayural">
    <img src="https://img.shields.io/badge/dev.to-0A0A0A?style=for-the-badge&logo=devdotto&logoColor=white" />
  </a>
  <a href="https://github.com/akutayural">
  <img src="https://komarev.com/ghpvc/?username=akutayural&label=Profile%20Views&color=0e75b6&style=for-the-badge" />
  </a>
</div>

---
## 📊 GitHub Stats
<div align="center">
  <img src="https://github-readme-stats-sigma-five.vercel.app/api?username=akutayural&show_icons=true&theme=tokyonight&hide_border=true&count_private=true" width="400px" />
  <img src="https://github-readme-streak-stats.herokuapp.com/?user=akutayural&theme=tokyonight&hide_border=true" width="400px" />
</div>

<div align="center">
  <img src="https://github-readme-stats-sigma-five.vercel.app/api/top-langs/?username=akutayural&layout=compact&theme=tokyonight&hide_border=true" width="400px" />
</div>

---

## 🛠 Tech Stack

### 📊 Data & Analytics
<div align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas" />
  <img src="https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white" />
  <img src="https://img.shields.io/badge/SQLAlchemy-CA1F23?style=for-the-badge" />
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" />
  <img src="https://img.shields.io/badge/MySQL-00758F?style=for-the-badge&logo=mysql&logoColor=white" />
  <img src="https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb&logoColor=white" />
  <img src="https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white" />
  <img src="https://img.shields.io/badge/Kafka-231F20?style=for-the-badge&logo=apachekafka&logoColor=white" />
</div>

### 🌐 Web Development
<div align="center">
  <img src="https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white" />
  <img src="https://img.shields.io/badge/Flask-000000?style=for-the-badge&logo=flask&logoColor=white" />
  <img src="https://img.shields.io/badge/Django-092E20?style=for-the-badge&logo=django&logoColor=white" />
  <img src="https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black" />
  <img src="https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white" />
  <img src="https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB" />
  <img src="https://img.shields.io/badge/Spring-6DB33F?style=for-the-badge&logo=spring&logoColor=white" />
</div>

### 💻 Programming Languages
<div align="center">
  <img src="https://img.shields.io/badge/C-A8B9CC?style=for-the-badge&logo=c&logoColor=black" />
  <img src="https://img.shields.io/badge/C++-00599C?style=for-the-badge&logo=c%2B%2B&logoColor=white" />
  <img src="https://img.shields.io/badge/Java-007396?style=for-the-badge&logo=java&logoColor=white" />
</div>

### ☁️ DevOps & Cloud
<div align="center">
  <img src="https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonaws" />
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes" />
  <img src="https://img.shields.io/badge/ArgoCD-E10098?style=for-the-badge" />
  <img src="https://img.shields.io/badge/S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white" />
  <img src="https://img.shields.io/badge/EC2-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" />
  <img src="https://img.shields.io/badge/ECR-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white" />
  <img src="https://img.shields.io/badge/SQS-FF4F00?style=for-the-badge&logo=amazonsqs&logoColor=white" />
  <img src="https://img.shields.io/badge/CloudFront-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" />
</div>

### 🧰 Tools & Others
<div align="center">
  <img src="https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git" />
  <img src="https://img.shields.io/badge/Bitbucket-0052CC?style=for-the-badge&logo=bitbucket" />
  <img src="https://img.shields.io/badge/Jira-0052CC?style=for-the-badge&logo=jira" />
  <img src="https://img.shields.io/badge/RabbitMQ-FF6600?style=for-the-badge&logo=rabbitmq" />
  <img src="https://img.shields.io/badge/Poetry-60A5FA?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Pydantic-00A8E8?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Pytest-0A9EDC?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black" />
  <img src="https://img.shields.io/badge/Shell_Scripting-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white" />
</div>

---

### 📫 Connect with Me

<div align="center">
  <a href="https://www.linkedin.com/in/akutayural">
    <img src="https://img.shields.io/badge/LinkedIn-0077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white" />
  </a>
  <a href="https://medium.com/@ahmetkutayural">
    <img src="https://img.shields.io/badge/Medium-000000.svg?style=for-the-badge&logo=medium&logoColor=white" />
  </a>
  <a href="https://dev.to/akutayural">
    <img src="https://img.shields.io/badge/dev.to-0A0A0A?style=for-the-badge&logo=devdotto&logoColor=white" />
  </a>
  <a href="https://www.ahmetkutayural.dev">
    <img src="https://img.shields.io/badge/Website-FF5722?style=for-the-badge&logo=google-chrome&logoColor=white" />
  </a>
</div>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/akutayural/akutayural/output/snake-dark.svg" />
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/akutayural/akutayural/output/snake-light.svg" />
  <img alt="github contribution grid snake animation" src="https://raw.githubusercontent.com/akutayural/akutayural/output/snake-dark.svg" />
</picture>

---

## 💼 About Me
I'm a **Python Software Engineer** based in **London, UK**, specializing in **scalable, high-performance systems**.  
I have a strong background in **FinTech applications** such as payment processing, cryptocurrency platforms, and e-wallet solutions.  
With an **MSc in Data Science**, I’m also passionate about **machine learning** and **predictive analytics**.

---

### 📢 **Latest Release: APIException**  
`APIException` is a fully customizable exception-handling toolkit for FastAPI. It enhances your Swagger docs, automatically logs errors, and converts even unregistered exceptions into a clean, consistent response format. Perfect for keeping your API responses standardised and developer-friendly.  
[📄 Documentation](https://akutayural.github.io/APIException/) • [📦 PyPI](https://pypi.org/project/APIException)

📦 Install via pip:
```bash
pip install apiexception
```

Happy coding! 🚀

