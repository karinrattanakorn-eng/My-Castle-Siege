<!DOCTYPE html><html lang="th">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1" />
<title>Castle Siege 3D – MVP</title>
<style>
  :root{
    --bg:#0f1221; --panel:#151936; --accent:#6ee7ff; --accent2:#a78bfa; --good:#22c55e; --bad:#ef4444; --text:#e5e7eb;
  }
  *{box-sizing:border-box}
  html,body{height:100%;margin:0;background:radial-gradient(1000px 600px at 20% 10%,#1b2150,#0b0e1f 60%);color:var(--text);font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial}
  button{cursor:pointer;border:0;border-radius:14px;padding:12px 16px;font-weight:700;color:#0b0e1f;background:linear-gradient(135deg,var(--accent),var(--accent2));box-shadow:0 8px 20px rgba(0,0,0,.35)}
  button.ghost{background:transparent;color:var(--text);border:1px solid #2a2f59}
  .screen{display:none;position:absolute;inset:0}
  .screen.active{display:block}
  .center{max-width:980px;margin:0 auto;padding:24px}
  header{display:flex;align-items:center;justify-content:space-between;padding:16px 24px}
  header h1{margin:0;font-size:clamp(20px,3vw,32px);letter-spacing:.5px}
  .grid{display:grid;gap:16px}
  .menu{place-items:center;text-align:center}
  .menu h2{font-size:clamp(22px,3vw,36px);margin:24px 0}
  .card{background:rgba(255,255,255,.06);backdrop-filter:blur(8px);border:1px solid rgba(255,255,255,.08);border-radius:20px;padding:18px}
  .stack{display:flex;gap:12px;flex-wrap:wrap}
  .row{display:flex;gap:12px;align-items:center}
  .label{opacity:.8;font-size:14px}
  input[type="number"],input[type="password"],input[type="text"],select{background:#0e1435;color:var(--text);border:1px solid #263067;border-radius:12px;padding:10px 12px}
  .pill{display:inline-flex;gap:6px;align-items:center;background:#0b1130;border:1px solid #27306b;padding:8px 12px;border-radius:999px}
  .small{font-size:12px;opacity:.85}
  .muted{opacity:.7}
  .sep{height:1px;background:linear-gradient(90deg,transparent,#28306e,transparent);margin:14px 0}
  .link{color:#93c5fd;text-decoration:underline;cursor:pointer}/* Game canvas and HUD */ #gameCanvas{position:absolute;inset:0} #hud{position:absolute;inset:0;pointer-events:none} #hud .top{position:absolute;left:0;right:0;top:10px;display:flex;justify-content:center;gap:20px;pointer-events:none} .bar{width:220px;height:16px;border-radius:999px;background:rgba(255,255,255,.08);border:1px solid rgba(255,255,255,.12);overflow:hidden} .bar>span{display:block;height:100%;background:linear-gradient(90deg,#16a34a,#4ade80);width:100%} .bar.bad>span{background:linear-gradient(90deg,#ef4444,#f97316)} .hud-pill{background:rgba(0,0,0,.45);backdrop-filter:blur(6px);border:1px solid rgba(255,255,255,.1);padding:8px 12px;border-radius:12px;font-weight:700} #timer{font-variant-numeric:tabular-nums}

/* On-screen controls */ #controls{position:absolute;inset:0;pointer-events:none} .joystick{position:absolute;left:18px;bottom:20px;width:140px;height:140px;border-radius:50%;background:rgba(255,255,255,.06);border:1px solid rgba(255,255,255,.15);pointer-events:auto} .joy-nub{position:absolute;left:50%;top:50%;width:64px;height:64px;border-radius:50%;transform:translate(-50%,-50%);background:rgba(255,255,255,.9);mix-blend-mode:luminosity} .skillpad{position:absolute;right:18px;bottom:18px;display:flex;gap:10px;flex-direction:column;pointer-events:auto} .btn-round{width:80px;height:80px;border-radius:50%;display:flex;align-items:center;justify-content:center;background:linear-gradient(135deg,#74f,#9df);color:#001;font-weight:900;border:0;box-shadow:0 8px 22px rgba(0,0,0,.45)} .btn-round.small{width:64px;height:64px} .cooldown{position:absolute;inset:0;background:rgba(0,0,0,.55);border-radius:50%;display:none;align-items:center;justify-content:center;color:#fff;font-weight:800} .draggable{touch-action:none} #repositionTip{position:absolute;left:50%;bottom:8px;transform:translateX(-50%);font-size:12px;opacity:.7;pointer-events:none}

/* Modals */ .modal{position:absolute;inset:0;background:rgba(0,0,0,.6);display:none;align-items:center;justify-content:center} .modal .panel{background:var(--panel);border:1px solid #28306e;border-radius:18px;padding:18px;max-width:520px;width:95%} .modal.show{display:flex}

/* Dev tools */ #devPanel{display:none} #palette{display:grid;grid-template-columns:repeat(3,1fr);gap:8px} .palette-item{background:#0e1435;border:1px solid #263067;border-radius:12px;padding:10px;text-align:center;cursor:pointer}

/* End screen */ #endOverlay{position:absolute;inset:0;background:linear-gradient(180deg,rgba(0,0,0,.2),rgba(0,0,0,.8));display:none;align-items:center;justify-content:center;flex-direction:column;gap:14px;text-align:center} #endOverlay h2{font-size:42px;margin:0}

a.inline{color:#93c5fd} </style>

</head>
<body>
  <div class="screen active" id="menuScreen">
    <header>
      <h1>🏰 Castle Siege 3D</h1>
      <div class="pill" id="wallet">💰 <span id="money">0</span> B</div>
    </header>
    <div class="center menu">
      <div class="card">
        <h2>โหมดผู้เล่น</h2>
        <div class="stack" style="justify-content:center">
          <button id="play3">เริ่มเกม 3v3</button>
          <button id="play5">เริ่มเกม 5v5</button>
          <button class="ghost" id="howBtn">วิธีเล่น</button>
        </div>
      </div>
      <div class="grid" style="grid-template-columns:1fr 1fr">
        <div class="card">
          <h3>ร้านค้า (สกิน)</h3>
          <div class="stack">
            <button data-skin="default" class="skinBtn">สกินเริ่มต้น – ฟรี</button>
            <button data-skin="crimson" class="skinBtn">Crimson – 60B</button>
            <button data-skin="jade" class="skinBtn">Jade – 60B</button>
            <button data-skin="royal" class="skinBtn">Royal – 80B</button>
          </div>
        </div>
        <div class="card">
          <h3>ตั้งค่า</h3>
          <div class="row"><span class="label">ตั้งค่าปุ่มควบคุมในเกม</span><button id="openSettings" class="ghost">เปิดหน้าตั้งค่า</button></div>
          <div class="sep"></div>
          <div class="row"><span class="label">โหมดผู้พัฒนา</span><button id="openDev" class="ghost">เข้าสู่ (ต้องมีรหัส)</button></div>
        </div>
      </div>
    </div>
  </div>  <div class="screen" id="settingsScreen">
    <header>
      <h1>⚙️ ตั้งค่า</h1>
      <div class="stack">
        <button class="ghost" id="backToMenu1">⬅ กลับเมนู</button>
      </div>
    </header>
    <div class="center grid" style="grid-template-columns:1fr 1fr">
      <div class="card">
        <h3>ค่าพื้นฐาน (ผู้เล่นทั่วไป)</h3>
        <div class="grid" style="grid-template-columns:1fr 1fr">
          <label>HP ผู้เล่น <input id="cfgPlayerHP" type="number" min="100" max="5000" value="1200"></label>
          <label>เกราะผู้เล่น <input id="cfgPlayerArmor" type="number" min="0" max="1000" value="250"></label>
          <label>พลังโจมตีผู้เล่น <input id="cfgPlayerAtk" type="number" min="1" max="1000" value="35"></label>
          <label>คูลดาวน์สกิล (วินาที) <input id="cfgSkillCD" type="number" min="2" max="30" value="10"></label>
          <label>เวลาเกม (นาที) <input id="cfgMatchMin" type="number" min="3" max="20" value="10"></label>
        </div>
        <div class="small muted">ค่าทั้งหมดจะถูกบันทึกอัตโนมัติ</div>
      </div>
      <div class="card">
        <h3>เข้าสู่โหมดผู้พัฒนา</h3>
        <div class="row">
          <input id="devPass" type="password" placeholder="ใส่รหัส" />
          <button id="devLogin">ยืนยัน</button>
        </div>
        <div class="small muted">รหัสคือ 7735771234 (เก็บเป็นความลับ)</div>
        <div id="devPanel" class="card" style="margin-top:12px">
          <h3>🛠 เครื่องมือผู้พัฒนา (ตัวอย่าง)</h3>
          <div class="grid" style="grid-template-columns:1fr 1fr;gap:10px">
            <div>
              <div class="label">เลือกวัตถุที่จะวาง</div>
              <div id="palette"></div>
            </div>
            <div>
              <div class="label">แผนที่</div>
              <div class="stack">
                <input id="mapName" placeholder="ชื่อแผนที่" />
                <div class="row">
                  <button id="saveMap">บันทึก</button>
                  <button class="ghost" id="clearMap">ล้างแผนที่</button>
                </div>
                <div class="row">
                  <select id="mapList"></select>
                  <button id="loadMap">โหลด</button>
                </div>
              </div>
              <div class="sep"></div>
              <div class="grid" style="grid-template-columns:1fr 1fr">
                <label>HP เสาธง <input id="cfgFlagHP" type="number" min="100" max="20000" value="5000"></label>
                <label>เกราะเสาธง <input id="cfgFlagArmor" type="number" min="0" max="5000" value="500"></label>
                <label>HP บอท <input id="cfgBotHP" type="number" min="50" max="5000" value="400"></label>
                <label>พลังโจมตีบอท <input id="cfgBotAtk" type="number" min="1" max="500" value="12"></label>
              </div>
              <div class="small muted">*ลากและวางวัตถุบนพื้นสีเขียวเพื่อจัดฉาก</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>  <div class="screen" id="howScreen">
    <header>
      <h1>📘 วิธีเล่น</h1>
      <div class="stack"><button class="ghost" id="backToMenu2">⬅ กลับเมนู</button></div>
    </header>
    <div class="center card">
      <p>เป้าหมาย: บุกทำลาย <b>เสาธง</b> ภายในปราสาทของศัตรูภายในเวลาที่กำหนด (ค่าเริ่มต้น 10 นาที) หากหมดเวลาและยังไม่มีฝ่ายใดทำลายได้ จะถือว่าเสมอ</p>
      <ul>
        <li>มุมมองบุคคลที่สาม (กล้องติดตามหลังตัวละคร)</li>
        <li>จอยเสมือนอยู่ซ้ายล่าง ปุ่มโจมตี + สกิล 2 ปุ่มอยู่ขวาล่าง (สามารถย้ายตำแหน่งได้ในโหมดจัดวาง)</li>
        <li>เลือกสายอาชีพก่อนเริ่ม: ธนู, ดาบ, ค้อน, มีดสั้น, โล่ใหญ่, ปาระเบิด</li>
        <li>ชนะแล้วได้รับ 100B เพื่อซื้อสกินในร้านค้า</li>
      </ul>
      <p class="muted small">*นี่คือรุ่นสาธิต (MVP) เพื่อให้เล่นได้ทันทีและมีโครงสำหรับขยายต่อ เช่น AI เพิ่มจำนวนยูนิต ธีมแผนที่ ฯลฯ</p>
    </div>
  </div>  <div class="screen" id="gameScreen">
    <canvas id="gameCanvas"></canvas>
    <div id="hud">
      <div class="top">
        <div class="hud-pill">ทีม 🔵</div>
        <div class="bar"><span id="hpBar"></span></div>
        <div class="bar bad"><span id="flagBar"></span></div>
        <div class="hud-pill" id="timer">10:00</div>
      </div>
      <div id="endOverlay">
        <h2 id="endTitle">ชัยชนะ!</h2>
        <div id="endDesc" class="muted"></div>
        <div class="stack">
          <button id="backToMenu3">กลับเมนู</button>
          <button class="ghost" id="replayBtn">เล่นซ้ำ</button>
        </div>
      </div>
    </div>
    <div id="controls">
      <div id="repositionTip" class="muted small">กดค้าง 1 วิ ที่จอยหรือปุ่ม เพื่อย้ายตำแหน่ง • แตะสองครั้งเพื่อยึดตำแหน่ง</div>
      <div class="joystick draggable" id="joy">
        <div class="joy-nub" id="joynub"></div>
      </div>
      <div class="skillpad">
        <div style="position:relative">
          <button class="btn-round draggable" id="attackBtn">⚔</button>
          <div class="cooldown" id="atkCD"></div>
        </div>
        <div style="display:flex;gap:10px;justify-content:flex-end">
          <div style="position:relative"><button class="btn-round small draggable" id="skill1Btn">S1</button><div class="cooldown" id="s1CD"></div></div>
          <div style="position:relative"><button class="btn-round small draggable" id="skill2Btn">S2</button><div class="cooldown" id="s2CD"></div></div>
        </div>
      </div>
    </div>
  </div>  <!-- Modals -->  <div class="modal" id="classModal">
    <div class="panel">
      <h3>เลือกสายอาชีพ</h3>
      <div class="stack">
        <div class="row" style="flex-wrap:wrap;gap:8px">
          <button class="ghost pickClass" data-class="archer">ธนู</button>
          <button class="ghost pickClass" data-class="sword">ดาบ</button>
          <button class="ghost pickClass" data-class="hammer">ค้อน</button>
          <button class="ghost pickClass" data-class="dagger">มีดสั้น</button>
          <button class="ghost pickClass" data-class="shield">โล่ใหญ่</button>
          <button class="ghost pickClass" data-class="grenade">ปาระเบิด</button>
        </div>
        <div class="small muted">*สกิลจะแตกต่างกันตามสาย</div>
        <div class="row" style="justify-content:flex-end"><button id="startMatch">เริ่มเกม</button></div>
      </div>
    </div>
  </div><script src="https://unpkg.com/three@0.160.0/build/three.min.js"></script><script>
(()=>{
  // ===== Utilities & Storage =====
  const $=s=>document.querySelector(s); const $$=s=>[...document.querySelectorAll(s)];
  const store={
    get(k,def){try{const v=localStorage.getItem(k);return v?JSON.parse(v):def}catch{return def}},
    set(k,v){localStorage.setItem(k,JSON.stringify(v))}
  };
  const rand=(a,b)=>Math.random()*(b-a)+a;
  const clamp=(n,a,b)=>Math.max(a,Math.min(b,n));
  const lerp=(a,b,t)=>a+(b-a)*t;

  // ===== Global State =====
  const state={
    money: store.get('money',0),
    skin: store.get('skin','default'),
    cfg: store.get('cfg',{
      playerHP:1200, playerArmor:250, playerAtk:35, skillCD:10, matchMin:10,
      flagHP:5000, flagArmor:500, botHP:400, botAtk:12
    }),
    dev:false, placing:null, maps: store.get('maps',{}), currentMap:null,
    queuedMode: {size:3},
    pickedClass:'archer'
  };
  const updateMoneyUI=()=>$('#money').textContent=state.money;
  updateMoneyUI();
  function saveCfg(){
    state.cfg.playerHP=+$('#cfgPlayerHP').value;
    state.cfg.playerArmor=+$('#cfgPlayerArmor').value;
    state.cfg.playerAtk=+$('#cfgPlayerAtk').value;
    state.cfg.skillCD=+$('#cfgSkillCD').value;
    state.cfg.matchMin=+$('#cfgMatchMin').value;
    state.cfg.flagHP=+$('#cfgFlagHP').value;
    state.cfg.flagArmor=+$('#cfgFlagArmor').value;
    state.cfg.botHP=+$('#cfgBotHP').value;
    state.cfg.botAtk=+$('#cfgBotAtk').value;
    store.set('cfg',state.cfg);
  }
  function loadCfgUI(){
    $('#cfgPlayerHP').value=state.cfg.playerHP;
    $('#cfgPlayerArmor').value=state.cfg.playerArmor;
    $('#cfgPlayerAtk').value=state.cfg.playerAtk;
    $('#cfgSkillCD').value=state.cfg.skillCD;
    $('#cfgMatchMin').value=state.cfg.matchMin;
    $('#cfgFlagHP').value=state.cfg.flagHP;
    $('#cfgFlagArmor').value=state.cfg.flagArmor;
    $('#cfgBotHP').value=state.cfg.botHP;
    $('#cfgBotAtk').value=state.cfg.botAtk;
  }
  loadCfgUI();
  $$('#settingsScreen input').forEach(i=>i.addEventListener('change',saveCfg));

  // ===== Screen Navigation =====
  function show(id){$$('.screen').forEach(s=>s.classList.remove('active')); $('#'+id).classList.add('active');}
  $('#openSettings').onclick=()=>{show('settingsScreen')};
  $('#backToMenu1').onclick=()=>{show('menuScreen')};
  $('#howBtn').onclick=()=>{show('howScreen')};
  $('#backToMenu2').onclick=()=>{show('menuScreen')};

  // ===== Shop =====
  $$('.skinBtn').forEach(btn=>{
    btn.onclick=()=>{
      const skin=btn.dataset.skin; const prices={default:0,crimson:60,jade:60,royal:80};
      const price=prices[skin]||0; if(skin!==state.skin){
        if(price>state.money){alert('เงินไม่พอ');return}
        state.money-=price; store.set('money',state.money); updateMoneyUI();
        alert('ซื้อสกินสำเร็จ!');
      }
      state.skin=skin; store.set('skin',skin);
    };
  });

  // ===== Dev Login & Panel =====
  const paletteItems=[
    {id:'wall',label:'กำแพง'},
    {id:'tower',label:'หอคอย'},
    {id:'river',label:'แม่น้ำ'},
    {id:'grass',label:'สนามหญ้า'},
    {id:'tree',label:'ต้นไม้'},
    {id:'cannon',label:'ปืนใหญ่'}
  ];
  function rebuildMapList(){ const sel=$('#mapList'); sel.innerHTML=''; Object.keys(state.maps).forEach(name=>{const o=document.createElement('option'); o.value=o.textContent=name; sel.appendChild(o);}); }
  rebuildMapList();

  $('#openDev').onclick=()=>{show('settingsScreen');$('#devPass').focus()};
  $('#devLogin').onclick=()=>{
    const ok=$('#devPass').value==='7735771234';
    if(!ok){alert('รหัสไม่ถูกต้อง');return}
    state.dev=true; $('#devPanel').style.display='block';
    const pal=$('#palette'); if(!pal.children.length){
      paletteItems.forEach(p=>{const b=document.createElement('div'); b.className='palette-item'; b.textContent=p.label; b.onclick=()=>{state.placing=p.id}; pal.appendChild(b);});
    }
  };
  $('#saveMap').onclick=()=>{
    const name=$('#mapName').value.trim(); if(!name){alert('ตั้งชื่อแผนที่ก่อน');return}
    state.maps[name]=game?game.serializeMap():{}; store.set('maps',state.maps); rebuildMapList(); alert('บันทึกแผนที่แล้ว');
  };
  $('#loadMap').onclick=()=>{
    const name=$('#mapList').value; if(!name){alert('ยังไม่มีแผนที่');return}
    if(game){game.loadMap(state.maps[name]); alert('โหลดแผนที่ '+name)}
  };
  $('#clearMap').onclick=()=>{ if(game){game.clearUserMap(); alert('ล้างแผนที่แล้ว')} };

  // ===== Game Core (Three.js) =====
  let game=null;
  class Game{
    constructor(){
      this.canvas=$('#gameCanvas');
      this.renderer=new THREE.WebGLRenderer({canvas:this.canvas,antialias:true});
      this.renderer.setPixelRatio(Math.min(devicePixelRatio,2));
      this.scene=new THREE.Scene();
      this.scene.background=new THREE.Color(0x0b0e1f);
      this.clock=new THREE.Clock();
      this.camera=new THREE.PerspectiveCamera(65,1,0.1,1000);
      this.camera.position.set(0,10,18);
      this.resize();
      window.addEventListener('resize',()=>this.resize());

      // Lights
      const amb=new THREE.AmbientLight(0xffffff,.6); this.scene.add(amb);
      const dir=new THREE.DirectionalLight(0xffffff,.9); dir.position.set(10,20,5); this.scene.add(dir);

      // Ground (build area for dev)
      const g=new THREE.PlaneGeometry(200,200); const m=new THREE.MeshPhongMaterial({color:0x254a2d});
      this.ground=new THREE.Mesh(g,m); this.ground.rotation.x=-Math.PI/2; this.ground.receiveShadow=true; this.scene.add(this.ground);

      // Groups
      this.world=new THREE.Group(); this.scene.add(this.world);
      this.userMap=new THREE.Group(); this.world.add(this.userMap);

      // Teams & bases
      this.blueBase=this.makeCastle(new THREE.Vector3(-40,0,0),0x3b82f6);
      this.redBase=this.makeCastle(new THREE.Vector3(40,0,0),0xef4444);

      // Flag objects
      this.flagBlue=this.makeFlag(this.blueBase.position.clone().add(new THREE.Vector3(0,0,0)),0x3b82f6);
      this.flagRed=this.makeFlag(this.redBase.position.clone().add(new THREE.Vector3(0,0,0)),0xef4444);

      // Player & bots
      this.player=null; this.team='blue';
      this.units=[]; this.projectiles=[]; this.effects=[]; this.respawnQueue=[];

      // Timers
      this.totalTime=state.cfg.matchMin*60; this.timeLeft=this.totalTime;

      // Input
      this.input={move:new THREE.Vector2(),camYaw:0,camPitch:0,att:false,s1:false,s2:false,dragging:false};
      this.initPointerLook();
      setupOnscreenControls(this.input);

      // Default sample map bits (theme variety)
      this.spawnTheme('thai');

      this.running=false; this.ended=false; this.modeSize=state.queuedMode.size;
      this.loop();
    }
    resize(){const w=innerWidth,h=innerHeight; this.renderer.setSize(w,h); this.camera.aspect=w/h; this.camera.updateProjectionMatrix();}
    makeCastle(pos,color){
      const grp=new THREE.Group(); grp.position.copy(pos);
      const wallM=new THREE.MeshPhongMaterial({color,emissive:0x111111});
      const wall=new THREE.Mesh(new THREE.BoxGeometry(16,6,10),wallM); wall.position.y=3; grp.add(wall);
      const gate=new THREE.Mesh(new THREE.BoxGeometry(4,5.5,1.2),new THREE.MeshPhongMaterial({color:0x555555})); gate.position.set(0,2.75,5.6); grp.add(gate); gate.name='gate'; gate.health=1200; gate.maxHealth=1200;
      const towerL=new THREE.Mesh(new THREE.CylinderGeometry(2,2,8,8),wallM); towerL.position.set(-7,4,-4); grp.add(towerL);
      const towerR=towerL.clone(); towerR.position.x=7; grp.add(towerR);
      this.world.add(grp); return grp;
    }
    makeFlag(pos,color){
      const grp=new THREE.Group(); grp.position.copy(pos).add(new THREE.Vector3(0,0,-2));
      const pole=new THREE.Mesh(new THREE.CylinderGeometry(.2,.2,6,8),new THREE.MeshPhongMaterial({color:0xaaaaaa})); pole.position.y=3; grp.add(pole);
      const flag=new THREE.Mesh(new THREE.BoxGeometry(2,1.2,.1),new THREE.MeshPhongMaterial({color})); flag.position.set(1.2,4.3,0); grp.add(flag);
      grp.health=state.cfg.flagHP; grp.maxHealth=state.cfg.flagHP; grp.armor=state.cfg.flagArmor; grp.name='flag';
      this.world.add(grp); return grp;
    }
    spawnTheme(theme){
      // Simple decorative presets
      const deco=new THREE.MeshPhongMaterial({color:0x888888});
      const makeCol=(x,z)=>{const c=new THREE.Mesh(new THREE.CylinderGeometry(.6,.8,6,8),deco); c.position.set(x,3,z); this.world.add(c)}
      if(theme==='thai'){
        for(let i=-4;i<=4;i++){makeCol(i*3, -20);}
      }
      if(theme==='japan'){
        const toriiM=new THREE.MeshPhongMaterial({color:0xff3344});
        const base=new THREE.Mesh(new THREE.BoxGeometry(8,1,2),toriiM); base.position.set(0,2,20); this.world.add(base);
      }
      if(theme==='roman'){
        for(let i=0;i<8;i++) makeCol(-10+i*2.5,10);
      }
      if(theme==='wall'){
        const wallM=new THREE.MeshPhongMaterial({color:0xcccccc}); const w=new THREE.Mesh(new THREE.BoxGeometry(40,3,2),wallM); w.position.set(0,1.5,-30); this.world.add(w);
      }
    }

    // ===== Player & Units =====
    makeUnit({team='blue',type='soldier',isPlayer=false}){
      const color=team==='blue'?0x3b82f6:0xef4444;
      const scale=isPlayer?2:1; // player larger than bots
      const m=new THREE.MeshPhongMaterial({color});
      const body=new THREE.Mesh(new THREE.CapsuleGeometry(.6*scale,1.2*scale,4,8),m);
      body.position.y=1.4*scale; this.world.add(body);
      const u={mesh:body,team,type,isPlayer,alive:true,spd: isPlayer?6:3.5, hp: (isPlayer?state.cfg.playerHP:state.cfg.botHP), maxHp:(isPlayer?state.cfg.playerHP:state.cfg.botHP), armor:(isPlayer?state.cfg.playerArmor:0), atk:(isPlayer?state.cfg.playerAtk:state.cfg.botAtk), cd1:0,cd2:0,atkcd:0, target:null, respawn:0};
      this.units.push(u); if(isPlayer) this.player=u; return u;
    }

    spawnTeams(){
      // Clear old units
      this.units.forEach(u=>this.world.remove(u.mesh)); this.units=[]; this.player=null;
      // Player (blue)
      const p=this.makeUnit({team:'blue',type:'hero',isPlayer:true});
      p.mesh.position.set(-48,0.1,0);
      this.applySkin(p);

      // Teammate bots
      for(let i=0;i<this.modeSize-1;i++){const b=this.makeUnit({team:'blue'}); b.mesh.position.set(-46+rand(-2,2),0.1,rand(-4,4));}
      // Enemy bots
      for(let i=0;i<this.modeSize;i++){const r=this.makeUnit({team:'red'}); r.mesh.position.set(46+rand(-2,2),0.1,rand(-4,4));}

      // Static defenders: archers & cannons (lightweight representation)
      this.spawnDefenders('blue',this.blueBase.position);
      this.spawnDefenders('red',this.redBase.position);
    }

    spawnDefenders(team,basePos){
      // 20 archers on walls (represented by small spheres that shoot)
      for(let i=0;i<10;i++){
        const a=new THREE.Mesh(new THREE.SphereGeometry(.3,8,8),new THREE.MeshPhongMaterial({color: team==='blue'?0x99bbff:0xff9999}));
        a.position.set(basePos.x+rand(-7,7),5.2,basePos.z+rand(-4,4)); this.world.add(a);
        a.userData={team,role:'archer',cool:rand(.2,1.5)}; this.effects.push(a);
      }
      // 10 cannons with 2 crew each (we'll just make 10 firing points)
      for(let i=0;i<10;i++){
        const c=new THREE.Mesh(new THREE.CylinderGeometry(.6,.8,.8,8),new THREE.MeshPhongMaterial({color:0x333333}));
        c.position.set(basePos.x+rand(-7,7),3.5,basePos.z+5.2); c.rotation.x=Math.PI/2; this.world.add(c);
        c.userData={team,role:'cannon',cool:rand(1,3)}; this.effects.push(c);
      }
      // Cavalry & elephant (AI attackers)
      const cavCount=20, infCount=30; // Represent lightly to keep perf
      for(let i=0;i<6;i++){const cav=this.makeUnit({team,type:'cavalry'}); cav.spd=5; cav.mesh.position.set(basePos.x+rand(-10,10),0.1,rand(-8,8));}
      const ele=this.makeUnit({team,type:'elephant'}); ele.spd=3.2; ele.mesh.scale.set(1.8,1.8,1.8); ele.mesh.position.set(basePos.x+rand(-6,6),0.1,rand(-3,3)); ele.hp*=2; ele.atk+=25; ele.cd1=ele.cd2=0;
      for(let i=0;i<10;i++){const inf=this.makeUnit({team,type:'infantry'}); inf.mesh.position.set(basePos.x+rand(-10,10),0.1,rand(-8,8));}
    }

    applySkin(u){
      const skins={default:0x3b82f6, crimson:0xdc2626, jade:0x10b981, royal:0x7c3aed};
      u.mesh.material.color.setHex(skins[state.skin]||skins.default);
    }

    // ===== Input & Camera =====
    initPointerLook(){
      let dragging=false, lastX=0,lastY=0; const el=this.canvas;
      const onDown=e=>{dragging=true; lastX=(e.touches?e.touches[0].clientX:e.clientX); lastY=(e.touches?e.touches[0].clientY:e.clientY)};
      const onMove=e=>{if(!dragging) return; const x=(e.touches?e.touches[0].clientX:e.clientX), y=(e.touches?e.touches[0].clientY:e.clientY); this.input.camYaw-= (x-lastX)*0.003; this.input.camPitch=clamp(this.input.camPitch-(y-lastY)*0.003,-1.0,0.8); lastX=x; lastY=y;};
      const onUp=()=>dragging=false;
      el.addEventListener('mousedown',onDown); el.addEventListener('mousemove',onMove); addEventListener('mouseup',onUp);
      el.addEventListener('touchstart',onDown,{passive:true}); el.addEventListener('touchmove',onMove,{passive:true}); addEventListener('touchend',onUp);
      // Keyboard fallback
      const keys={}; addEventListener('keydown',e=>{keys[e.key.toLowerCase()]=true}); addEventListener('keyup',e=>{keys[e.key.toLowerCase()]=false});
      const kbLoop=()=>{
        const v=new THREE.Vector2(0,0);
        if(keys['w']||keys['arrowup']) v.y+=1; if(keys['s']||keys['arrowdown']) v.y-=1; if(keys['a']||keys['arrowleft']) v.x-=1; if(keys['d']||keys['arrowright']) v.x+=1; this.input.move.copy(v);
        requestAnimationFrame(kbLoop);
      }; kbLoop();
    }

    // ===== Map serialization =====
    serializeMap(){
      return this.userMap.children.map(o=>({type:o.userData.type, pos:o.position.toArray(), rot:[o.rotation.x,o.rotation.y,o.rotation.z], scl:[o.scale.x,o.scale.y,o.scale.z]}));
    }
    loadMap(data){ this.clearUserMap(); (data||[]).forEach(d=>this.placeFromData(d)); }
    clearUserMap(){ this.userMap.children.slice().forEach(c=>this.userMap.remove(c)); }
    placeFromData(d){ const o=this.makeDeco(d.type); if(!o) return; o.position.fromArray(d.pos); o.rotation.set(d.rot[0],d.rot[1],d.rot[2]); o.scale.set(d.scl[0],d.scl[1],d.scl[2]); }

    makeDeco(type){
      const add=o=>{o.userData.type=type; this.userMap.add(o); return o;};
      if(type==='wall'){return add(new THREE.Mesh(new THREE.BoxGeometry(12,3,2),new THREE.MeshPhongMaterial({color:0xcbd5e1})))}
      if(type==='tower'){const g=new THREE.CylinderGeometry(1.2,1.2,6,8); return add(new THREE.Mesh(g,new THREE.MeshPhongMaterial({color:0x94a3b8})))}
      if(type==='river'){const g=new THREE.BoxGeometry(12,.2,4); return add(new THREE.Mesh(g,new THREE.MeshPhongMaterial({color:0x2563eb,transparent:true,opacity:.8})))}
      if(type==='grass'){const g=new THREE.BoxGeometry(10,.2,10); return add(new THREE.Mesh(g,new THREE.MeshPhongMaterial({color:0x22c55e})))}
      if(type==='tree'){const trunk=new THREE.Mesh(new THREE.CylinderGeometry(.3,.4,2,6),new THREE.MeshPhongMaterial({color:0x8b5a2b})); const crown=new THREE.Mesh(new THREE.SphereGeometry(1.1,12,12),new THREE.MeshPhongMaterial({color:0x16a34a})); const grp=new THREE.Group(); trunk.position.y=1; crown.position.y=2.2; grp.add(trunk,crown); return add(grp)}
      if(type==='cannon'){const g=new THREE.CylinderGeometry(.4,.5,1.4,8); const m=new THREE.MeshPhongMaterial({color:0x111111}); const c=new THREE.Mesh(g,m); c.rotation.z=Math.PI/2; return add(c)}
      return null;
    }

    // ===== Match Flow =====
    start(modeSize){
      this.modeSize=modeSize; this.timeLeft=state.cfg.matchMin*60; this.flagBlue.health=this.flagBlue.maxHealth=state.cfg.flagHP; this.flagBlue.armor=state.cfg.flagArmor; this.flagRed.health=this.flagRed.maxHealth=state.cfg.flagHP; this.flagRed.armor=state.cfg.flagArmor; this.spawnTeams(); this.running=true; this.ended=false; $('#endOverlay').style.display='none';
    }

    damage(target,amount){
      const armored=Math.max(0,amount - (target.armor||0)*0.2); target.health-=Math.max(1,armored);
      if(target.health<=0){ target.health=0; if(target.name==='flag'){ this.end(target===this.flagRed?'win':'lose','เสาธงถูกทำลาย'); } }
    }

    end(result,reason){ if(this.ended) return; this.ended=true; this.running=false; const title=result==='win'?'🎉 ชัยชนะ!':'🔴 พ่ายแพ้'; $('#endTitle').textContent=title; $('#endDesc').textContent=reason||''; $('#endOverlay').style.display='flex'; if(result==='win'){ state.money+=100; store.set('money',state.money); updateMoneyUI(); } }

    updateHUD(){
      $('#hpBar').style.width= (this.player? (this.player.hp/this.player.maxHp*100):0)+'%';
      $('#flagBar').style.width= (this.flagRed.health/this.flagRed.maxHealth*100)+'%';
      const m=Math.floor(this.timeLeft/60), s=Math.floor(this.timeLeft%60).toString().padStart(2,'0'); $('#timer').textContent=`${m}:${s}`;
    }

    aiStep(u,dt){
      if(!u.alive) return;
      const enemyFlag= u.team==='blue'?this.flagRed:this.flagBlue; const myBase= u.team==='blue'?this.blueBase:this.redBase; const enemyBase= u.team==='blue'?this.redBase:this.blueBase;
      const pos=u.mesh.position; const gate=enemyBase.children.find(n=>n.name==='gate');
      const targetPos = (gate.health>0)? enemyBase.position.clone().add(new THREE.Vector3(0,0,6)) : enemyFlag.position;
      const dir=new THREE.Vector3().subVectors(targetPos,pos); const dist=dir.length(); dir.normalize();
      // Attack gate or flag if close
      if(dist<3){ if(gate.health>0){ gate.health-=state.cfg.botAtk*dt*2; if(gate.health<=0) gate.health=0; } else { this.damage(enemyFlag,state.cfg.botAtk*dt*10);} }
      else { pos.add(dir.multiplyScalar(u.spd*dt)); }
      // Basic melee around player
      if(this.player && this.player.alive && this.player.team!==u.team){ const d=this.player.mesh.position.distanceTo(pos); if(d<2.2){ this.player.hp=Math.max(0,this.player.hp - u.atk*dt*2); if(this.player.hp<=0){ this.killUnit(this.player); } } }
    }

    killUnit(u){ if(!u.alive) return; u.alive=false; u.mesh.visible=false; u.respawn=u.isPlayer?60:40; this.respawnQueue.push(u); if(u.isPlayer){ /* show hint */ } }

    tryRespawns(dt){ this.respawnQueue.forEach(u=>{u.respawn-=dt}); const ready=this.respawnQueue.filter(u=>u.respawn<=0); this.respawnQueue=this.respawnQueue.filter(u=>u.respawn>0); ready.forEach(u=>{u.alive=true; u.hp=u.maxHp; u.mesh.visible=true; const base=u.team==='blue'?this.blueBase:this.redBase; u.mesh.position.copy(base.position).add(new THREE.Vector3(u.isPlayer?-8:8,0.1,0)); }); }

    heroAttack(kind){
      if(!this.player || !this.player.alive) return;
      const nowCD=(btn)=>{btn.style.display='flex'}; const offCD=(btn)=>{btn.style.display='none'};
      if(kind==='atk' && this.player.atkcd<=0){ this.player.atkcd=0.6; // basic attack
        this.spawnProjectile(this.player,'basic'); $('#atkCD').style.display='flex'; setTimeout(()=>$('#atkCD').style.display='none',600);
      }
      if(kind==='s1' && this.player.cd1<=0){ this.player.cd1=state.cfg.skillCD; this.useSkill(this.player,1); $('#s1CD').style.display='flex'; cooldownCounter('#s1CD',this.player.cd1); }
      if(kind==='s2' && this.player.cd2<=0){ this.player.cd2=state.cfg.skillCD; this.useSkill(this.player,2); $('#s2CD').style.display='flex'; cooldownCounter('#s2CD',this.player.cd2); }
    }

    useSkill(p,idx){ const cls=state.pickedClass; if(cls==='archer'){ if(idx===1){ for(let i=-1;i<=1;i++) this.spawnProjectile(p,'arrow',i*0.12);} else { this.dash(p,6);} }
      else if(cls==='sword'){ if(idx===1){ this.spinSlash(p,4,45);} else { p.armor+=150; setTimeout(()=>p.armor-=150,3000);} }
      else if(cls==='hammer'){ if(idx===1){ this.groundSlam(p,5,60);} else { p.armor+=200; setTimeout(()=>p.armor-=200,4000);} }
      else if(cls==='dagger'){ if(idx===1){ this.dash(p,8,true);} else { p.spd+=2; setTimeout(()=>p.spd-=2,4000);} }
      else if(cls==='shield'){ if(idx===1){ this.shieldBash(p,3,40);} else { p.armor+=300; setTimeout(()=>p.armor-=300,3000);} }
      else if(cls==='grenade'){ if(idx===1){ this.throwGrenade(p,8,80);} else { this.throwGrenade(p,8,40,true);} }
    }

    spawnProjectile(owner,type,angleOffset=0){
      const pos=owner.mesh.position.clone(); const dir=new THREE.Vector3(Math.sin(this.input.camYaw+angleOffset),0,-Math.cos(this.input.camYaw+angleOffset));
      const geo= new THREE.SphereGeometry(type==='basic'?0.25: type==='arrow'?0.2:0.3,8,8);
      const mat= new THREE.MeshPhongMaterial({color: owner.team==='blue'?0x99ccff:0xff9999});
      const mesh= new THREE.Mesh(geo,mat); mesh.position.copy(pos).add(new THREE.Vector3(0,1.2,0)); this.world.add(mesh);
      const proj={mesh,team:owner.team,vel:dir.multiplyScalar( type==='basic'?14: (type==='arrow'?18:12) ),life:3,damage: owner.atk*(type==='basic'?1:1.2)}; this.projectiles.push(proj);
    }

    dash(p,dist,damage=false){ const dir=new THREE.Vector3(Math.sin(this.input.camYaw),0,-Math.cos(this.input.camYaw)); const from=p.mesh.position.clone(); const to=from.clone().add(dir.multiplyScalar(dist)); p.mesh.position.copy(to); if(damage){ this.units.filter(u=>u.team!==p.team && u.alive && u.mesh.position.distanceTo(to)<3).forEach(u=>{u.hp=Math.max(0,u.hp - (p.atk*2)); if(u.hp<=0) this.killUnit(u);}); } }
    spinSlash(p,r,amount){ this.units.filter(u=>u.team!==p.team && u.alive && u.mesh.position.distanceTo(p.mesh.position)<r).forEach(u=>{u.hp=Math.max(0,u.hp - (p.atk+amount)); if(u.hp<=0) this.killUnit(u);}); }
    groundSlam(p,r,amount){ this.effects.push(this.circleFx(p.mesh.position,r)); this.units.filter(u=>u.team!==p.team && u.alive && u.mesh.position.distanceTo(p.mesh.position)<r).forEach(u=>{u.hp=Math.max(0,u.hp - (p.atk+amount)); if(u.hp<=0) this.killUnit(u);}); }
    shieldBash(p,r,amount){ this.units.filter(u=>u.team!==p.team && u.alive && u.mesh.position.distanceTo(p.mesh.position)<r).forEach(u=>{u.hp=Math.max(0,u.hp - (p.atk+amount)); if(u.hp<=0) this.killUnit(u);}); }
    throwGrenade(p,dist,amount,fire=false){ const dir=new THREE.Vector3(Math.sin(this.input.camYaw),0.6,-Math.cos(this.input.camYaw)); const pos=p.mesh.position.clone().add(new THREE.Vector3(0,1.3,0)); const mesh=new THREE.Mesh(new THREE.SphereGeometry(.28,10,10),new THREE.MeshPhongMaterial({color: fire?0xf97316:0xf43f5e})); mesh.position.copy(pos); this.world.add(mesh); const proj={mesh,team:p.team,vel:dir.multiplyScalar(10),life:2,isGrenade:true,damage:amount,fire}; this.projectiles.push(proj); }
    circleFx(pos,r){ const geo=new THREE.RingGeometry(r-0.1,r,32); const mat=new THREE.MeshBasicMaterial({color:0xffffff,transparent:true,opacity:.7,side:THREE.DoubleSide}); const m=new THREE.Mesh(geo,mat); m.rotation.x=-Math.PI/2; m.position.set(pos.x,0.05,pos.z); this.world.add(m); setTimeout(()=>this.world.remove(m),300); return m; }

    updateProjectiles(dt){
      this.projectiles.forEach(p=>{ p.life-=dt; if(p.isGrenade){ p.vel.y-=9.8*dt; } p.mesh.position.addScaledVector(p.vel,dt);
        // Hit test vs enemies & flag
        const targets=[...this.units.filter(u=>u.alive && u.team!==p.team).map(u=>u.mesh), (p.team==='blue'?this.flagRed:this.flagBlue)];
        for(const t of targets){ const tp=t.position||t; const d=p.mesh.position.distanceTo(tp); if(d< (t.geometry? 1.2: 2.2)){
          if(t.name==='flag'){ this.damage(t,p.damage); } else { const unit=this.units.find(u=>u.mesh===t); unit.hp=Math.max(0,unit.hp-p.damage); if(unit.hp<=0) this.killUnit(unit); }
          p.life=0; break;
        }
      });
      const dead=this.projectiles.filter(p=>p.life<=0); dead.forEach(p=>this.world.remove(p.mesh)); this.projectiles=this.projectiles.filter(p=>p.life>0);
    }

    updateDefenses(dt){
      this.effects.forEach(e=>{
        const ud=e.userData; if(!ud) return; ud.cool-=dt; if(ud.cool<=0){
          if(ud.role==='archer'){
            const target=this.units.find(u=>u.team!==(ud.team) && u.alive); if(target){ this.spawnShotFrom(e.position,ud.team==='blue'?'blue':'red'); }
            ud.cool=rand(0.8,1.6);
          } else if(ud.role==='cannon'){
            const target=this.units.find(u=>u.team!==(ud.team) && u.alive); if(target){ this.fireCannon(e.position, target.mesh.position, ud.team); }
            ud.cool=rand(2.0,4.0);
          }
        }
      });
    }
    spawnShotFrom(pos,team){ const dir=new THREE.Vector3().subVectors(this.player.mesh.position,pos).normalize(); const m=new THREE.Mesh(new THREE.SphereGeometry(.18,8,8),new THREE.MeshPhongMaterial({color: team==='blue'?0x99ccff:0xffaaaa})); m.position.copy(pos); this.world.add(m); this.projectiles.push({mesh:m,team,vel:dir.multiplyScalar(16),life:2,damage:10}); }
    fireCannon(from,to,team){ const dir=new THREE.Vector3().subVectors(to,from).normalize(); const m=new THREE.Mesh(new THREE.SphereGeometry(.4,10,10),new THREE.MeshPhongMaterial({color:0x222222})); m.position.copy(from); this.world.add(m); this.projectiles.push({mesh:m,team,vel:dir.multiplyScalar(12),life:3,damage:25}); this.cameraShake(0.2); }
    cameraShake(intensity){ this.shakeT=0.3; this.shakeI=intensity; }

    updateCamera(dt){ if(!this.player) return; const p=this.player.mesh.position; const offset=new THREE.Vector3(Math.sin(this.input.camYaw)*10, 6+ (this.shakeT?rand(-this.shakeI,this.shakeI):0), -Math.cos(this.input.camYaw)*10); const target=p.clone().add(offset); this.camera.position.lerp(target,0.1); this.camera.lookAt(p.clone().add(new THREE.Vector3(0,2,0))); if(this.shakeT){ this.shakeT-=dt; if(this.shakeT<0) this.shakeT=0; }}

    step(dt){ if(!this.running) return;
      // Countdown
      this.timeLeft-=dt; if(this.timeLeft<=0){ this.end('draw','หมดเวลา');}

      // Update CDs
      if(this.player){ this.player.atkcd=Math.max(0,this.player.atkcd-dt); this.player.cd1=Math.max(0,this.player.cd1-dt); this.player.cd2=Math.max(0,this.player.cd2-dt); if(this.player.alive){ // movement
          const m=this.input.move.clone(); const len=Math.min(1,m.length()); if(len>0){ m.normalize(); // rotate by cam yaw
            const ang=this.input.camYaw; const dx=m.x*Math.cos(ang)+m.y*Math.sin(ang); const dz=-m.x*Math.sin(ang)+m.y*Math.cos(ang); this.player.mesh.position.x+=dx*this.player.spd*dt; this.player.mesh.position.z+=dz*this.player.spd*dt; }
        } else { // dead
        }
      }

      // Simple AI for bots
      this.units.forEach(u=>{ if(!u.isPlayer) this.aiStep(u,dt); });

      // Defenses shooting
      this.updateDefenses(dt);

      // Projectiles
      this.updateProjectiles(dt);

      // Respawns
      this.tryRespawns(dt);

      // Camera & HUD
      this.updateCamera(dt); this.updateHUD();
    }

    loop(){ const dt=this.clock.getDelta(); this.step(dt); this.renderer.render(this.scene,this.camera); requestAnimationFrame(()=>this.loop()); }
  }

  // ===== On-screen controls logic (joystick & draggable buttons) =====
  function setupOnscreenControls(input){
    const joy=$('#joy'); const nub=$('#joynub'); const rect=()=>joy.getBoundingClientRect(); let dragging=false, longPress=false, lpTimer=null, dragMode=false; let base={x:0,y:0};
    const setNub=(x,y)=>{ nub.style.left=(x*50+50)+'%'; nub.style.top=(y*50+50)+'%'; };
    const start=(x,y)=>{ dragging=true; base={x,y}; update(x,y); };
    const move=(x,y)=>{ if(!dragging) return; update(x,y); };
    const end=()=>{ dragging=false; input.move.set(0,0); setNub(0,0); };
    const norm=(x,y)=>{ const r=rect(); const cx=r.left+r.width/2, cy=r.top+r.height/2; const dx=(x-cx)/(r.width/2), dy=(y-cy)/(r.height/2); const v=new THREE.Vector2(dx,dy); if(v.length()>1) v.normalize(); return v; };
    function update(x,y){ const v=norm(x,y); input.move.set(v.x, -v.y); setNub(v.x,-v.y); }
    const onTouchStart=e=>{ const t=e.touches[0]; start(t.clientX,t.clientY); lpTimer=setTimeout(()=>{dragMode=true; joy.classList.add('editing');},800) };
    const onTouchMove=e=>{ const t=e.touches[0]; if(dragMode){ joy.style.left=(t.clientX-rect().width/2)+'px'; joy.style.top=(t.clientY-rect().height/2)+'px'; } else move(t.clientX,t.clientY); };
    const onTouchEnd=()=>{ clearTimeout(lpTimer); if(dragMode){ dragMode=false; joy.classList.remove('editing'); store.set('joyPos',{left:joy.style.left,top:joy.style.top}); } else end(); };
    joy.addEventListener('touchstart',onTouchStart,{passive:true}); joy.addEventListener('touchmove',onTouchMove,{passive:true}); joy.addEventListener('touchend',onTouchEnd);

    // Attack & skills with draggable by long press
    function setupBtn(id,flag){ const el=$(id), cd=$(flag); let dragging=false, timer=null, edit=false; el.addEventListener('touchstart',e=>{ timer=setTimeout(()=>{edit=true},800) },{passive:true}); el.addEventListener('touchmove',e=>{ const t=e.touches[0]; if(edit){ el.style.position='absolute'; el.style.right='auto'; el.style.bottom='auto'; el.style.left=(t.clientX-40)+'px'; el.style.top=(t.clientY-40)+'px'; } },{passive:true}); el.addEventListener('touchend',e=>{ if(edit){ edit=false; store.set(id,{left:el.style.left,top:el.style.top,absolute:true}); } else { game && game.heroAttack(id.includes('attack')?'atk': id.includes('skill1')?'s1':'s2'); } }); }
    setupBtn('#attackBtn','#atkCD'); setupBtn('#skill1Btn','#s1CD'); setupBtn('#skill2Btn','#s2CD');

    // Restore positions
    const jp=store.get('joyPos'); if(jp){ joy.style.left=jp.left; joy.style.top=jp.top; }
    ['#attackBtn','#skill1Btn','#skill2Btn'].forEach(id=>{ const p=store.get(id); if(p&&p.absolute){ const el=$(id); el.style.position='absolute'; el.style.left=p.left; el.style.top=p.top; }});
  }

  // ===== UI Bindings =====
  $('#play3').onclick=()=>{ state.queuedMode.size=3; startGameFlow(); };
  $('#play5').onclick=()=>{ state.queuedMode.size=5; startGameFlow(); };
  $('#backToMenu3').onclick=()=>{ show('menuScreen'); };
  $('#replayBtn').onclick=()=>{ startGameFlow(); };

  function startGameFlow(){ show('gameScreen'); $('#classModal').classList.add('show'); }
  $$('.pickClass').forEach(b=>b.onclick=()=>{ state.pickedClass=b.dataset.class; $$('.pickClass').forEach(x=>x.classList.remove('active')); b.classList.add('active'); });
  $('#startMatch').onclick=()=>{ $('#classModal').classList.remove('show'); if(!game) game=new Game(); game.start(state.queuedMode.size); };

  // ===== Minimal click-to-place for dev mode =====
  addEventListener('click',e=>{
    if(!state.dev || !state.placing || !game) return;
    const r=game.renderer.domElement.getBoundingClientRect(); const x=((e.clientX-r.left)/r.width)*2-1; const y=-((e.clientY-r.top)/r.height)*2+1;
    const ray=new THREE.Raycaster(); ray.setFromCamera({x,y},game.camera); const hits=ray.intersectObject(game.ground); if(hits.length){ const p=hits[0].point; const o=game.makeDeco(state.placing); o.position.copy(p); }
  });

  // ===== Cooldown circle text =====
  function cooldownCounter(sel,sec){ const el=$(sel); el.textContent=Math.ceil(sec); const tick=()=>{ sec-=1; if(sec<=0){ el.style.display='none'; el.textContent=''; } else { el.textContent=Math.ceil(sec); setTimeout(tick,1000);} }; setTimeout(tick,1000); }

  // ===== Basic SFX using WebAudio (no external files) =====
  const audioCtx = new (window.AudioContext||window.webkitAudioContext)();
  function beep(freq=440, dur=0.1, type='square', vol=0.05){ const o=audioCtx.createOscillator(); const g=audioCtx.createGain(); o.type=type; o.frequency.value=freq; o.connect(g); g.connect(audioCtx.destination); g.gain.value=vol; o.start(); setTimeout(()=>{o.stop();}, dur*1000); }
  // Hook some events
  document.addEventListener('pointerdown',()=>{ if(audioCtx.state==='suspended') audioCtx.resume(); },{once:true});
})();
</script></body>
</html>
