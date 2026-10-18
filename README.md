<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Chess Tamasha Online</title>

<!-- PeerJS Library for P2P Game + Video Call -->
<script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>

<style>
:root{--bg:#fff3d6;--fg:#2b2118;--l:#f3d9a4;--d:#c98b4d;--card:#fff;--bd:#e0c48f;--s:min(11.4vw,54px);box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media(prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#1d1a16;--fg:#f5e9d6;--l:#b8935a;--d:#7a5430;--card:#2a2520;--bd:#4a3f33}}
:root[data-theme="dark"]{--bg:#1d1a16;--fg:#f5e9d6;--l:#b8935a;--d:#7a5430;--card:#2a2520;--bd:#4a3f33}
html,body{margin:0}
body{background:var(--bg);color:var(--fg);font-family:system-ui,-apple-system,"Segoe UI",sans-serif;display:flex;flex-direction:column;align-items:center;padding:10px 8px 24px}
h1{margin:4px 0 0;font-size:22px}.sub{margin:0 0 8px;font-size:13px;opacity:.75;text-align:center}
.box{width:100%;max-width:calc(var(--s)*8 + 8px);box-sizing:border-box}
.card{background:var(--card);border:2px solid var(--bd);border-radius:14px;padding:10px 12px;margin-top:10px;font-size:15px}
input{font:inherit;font-size:15px;padding:9px 12px;border-radius:10px;border:2px solid var(--bd);background:var(--card);color:var(--fg);box-sizing:border-box;width:100%;margin-top:8px}
#bd{display:grid;grid-template-columns:repeat(8,var(--s));border:4px solid var(--d);border-radius:8px;overflow:hidden}
.q{width:var(--s);height:var(--s);display:flex;align-items:center;justify-content:center;font-size:calc(var(--s)*.78);position:relative;cursor:pointer;user-select:none;line-height:1}
.q.l{background:var(--l)}.q.d{background:var(--d)}
.q.sel{background:#ffe45e!important}.q.last{box-shadow:inset 0 0 0 3px #ff7043}
.q.mv::after{content:"";position:absolute;width:30%;height:30%;border-radius:50%;background:rgba(40,160,70,.7)}
.q.mv.c::after{width:86%;height:86%;background:none;border:4px solid rgba(220,50,50,.8)}
.w{color:#fff;text-shadow:0 0 2px #000,0 0 2px #000,0 0 3px #000}.k{color:#111}
.q.last span{display:inline-block;animation:wob .6s}
@keyframes wob{0%,100%{transform:rotate(0)}25%{transform:rotate(-18deg) scale(1.2)}75%{transform:rotate(18deg) scale(1.2)}}
#chat{height:120px;overflow-y:auto;font-size:14px;border-radius:8px;padding:2px 0}
#chat div{margin:3px 0;word-wrap:break-word}#chat .me{text-align:right}
#chat b{font-size:12px;opacity:.7;display:block}
.row{display:flex;gap:6px;margin-top:8px;flex-wrap:wrap}
button{font:inherit;font-size:14px;padding:8px 14px;border-radius:20px;border:2px solid var(--d);background:var(--card);color:var(--fg);cursor:pointer}
button.main{background:var(--d);color:#fff}
.hide{display:none}

/* Video Call CSS */
.video-container{display:flex;gap:8px;justify-content:center;margin-top:10px}
.vid-box{position:relative;width:48%;background:#000;border-radius:12px;overflow:hidden;aspect-ratio:4/3;border:2px solid var(--bd)}
.vid-box video{width:100%;height:100%;object-fit:cover}
.vid-box .label{position:absolute;bottom:4px;left:6px;background:rgba(0,0,0,0.6);color:#fff;font-size:11px;padding:2px 6px;border-radius:6px}
</style>
</head>
<body>

<h1>♞ Chess Tamasha Online 🤡</h1>
<p class="sub">Do dost, ek room code, aur full gappe-shappe</p>

<div id="lobby" class="box card">
<b>Room mein aao</b>
<input id="nm" placeholder="Tumhara naam" maxlength="14">
<input id="rc" placeholder="Room code (dost ko bhi yahi batao)" maxlength="10">
<div class="row">
  <button class="main" id="go">▶️ Join / Create</button>
  <button id="rnd">🎲 Random code</button>
</div>
<div id="err" style="margin-top:8px;font-size:13px;color:#d32f2f"></div>
</div>

<div id="game" class="box hide">
<div class="card" id="pl" style="margin-top:0;text-align:center;font-weight:600"></div>

<!-- Video Call Layout -->
<div class="video-container">
  <div class="vid-box">
    <video id="myVid" autoplay playsinline muted></video>
    <div class="label" id="myVidLbl">Tum</div>
  </div>
  <div class="vid-box">
    <video id="oppVid" autoplay playsinline></video>
    <div class="label" id="oppVidLbl">Dost</div>
  </div>
</div>

<div id="bd" style="margin-top:8px"></div>
<div class="card"><b>🎙️ Commentator:</b> <span id="msg"></span></div>
<div class="card"><b>💬 Chat</b>
<div id="chat"></div>
<div class="row"><button class="em">😂</button><button class="em">🤡</button><button class="em">Jaldi chal!</button><button class="em">Chai?</button><button class="em">Wah!</button></div>
<div class="row" style="flex-wrap:nowrap"><input id="tx" placeholder="Kuch likho..." maxlength="200" style="margin-top:0"><button class="main" id="sd">Send</button></div></div>
<div class="row" style="justify-content:center"><button id="rm">🔁 Rematch</button><button id="cp">📋 Code copy</button></div>
</div>

<script>
const G={k:"♚",q:"♛",r:"♜",b:"♝",n:"♞",p:"♟"},V={p:1,n:3,b:3,r:5,q:9,k:100},N={p:"Pyaada",n:"Ghoda",b:"Oont",r:"Haathi",q:"Rani",k:"Raja"};
const pick=a=>a[Math.floor(Math.random()*a.length)],$=id=>document.getElementById(id);

function start(){const B="rnbqkbnr",a=Array(64).fill(null);for(let f=0;f<8;f++){a[f]="b"+B[f];a[8+f]="bp";a[48+f]="wp";a[56+f]="w"+B[f]}return a}

function gen(b,i){const p=b[i],c=p[0],t=p[1],r=i>>3,f=i&7,o=[];
const add=(R,F,m)=>{if(R<0||R>7||F<0||F>7)return false;const q=b[R*8+F];if(!q){if(m!==2)o.push(R*8+F);return true}if(q[0]!=c&&m!==0)o.push(R*8+F);return false};
if(t=="p"){const d=c=="w"?-1:1;add(r+d,f,0);if(o.length&&r==(c=="w"?6:1))add(r+2*d,f,0);add(r+d,f-1,2);add(r+d,f+1,2)}
else if(t=="n")[[1,2],[2,1],[-1,2],[-2,1],[1,-2],[2,-1],[-1,-2],[-2,-1]].forEach(([a,e])=>add(r+a,f+e));
else if(t=="k")[[1,0],[-1,0],[0,1],[0,-1],[1,1],[1,-1],[-1,1],[-1,-1]].forEach(([a,e])=>add(r+a,f+e));
else{const D=[];if(t!="b")D.push([1,0],[-1,0],[0,1],[0,-1]);if(t!="r")D.push([1,1],[1,-1],[-1,1],[-1,-1]);
D.forEach(([a,e])=>{for(let k=1;k<8;k++)if(!add(r+a*k,f+e*k))break})}
return o}

const inCheck=(b,c)=>{const k=b.indexOf(c+"k");return b.some((p,i)=>p&&p[0]!=c&&gen(b,i).includes(k))};
function mv(b,a,z){const n=b.slice();n[z]=n[a];n[a]=null;if(n[z][1]=="p"&&(z>>3==0||z>>3==7))n[z]=n[z][0]+"q";return n}
const legal=(b,i)=>gen(b,i).filter(z=>!inCheck(mv(b,i,z),b[i][0]));
function any(b,c){return b.some((p,i)=>p&&p[0]==c&&legal(b,i).length)}

let peer, conn, myStream, room, myName, oppName="", b=start(), st={turn:"w",over:"",last:-1,msg:"Khel shuru! White pehle chalega 😎"}, color=null, sel=null, ls=[];
const Q=n=>pick([`${n} ne chaal chali, computer nahi, insaan hai bhai! 🧠`,`${n} ki chaal dekh ke samose thande ho gaye 🥟`,`${n} sochta hai ki wo Anand hai 😎`,`${n} ne pyaada aise hilaya jaise Sharma ji ka beta 🎓`]);
const C=(n,v)=>pick([`${n} ne ${v} ko uda diya! 💥 Shok sabha kal 9 baje 🕯️`,`${v} bol raha: “${n} bhai, mujhe kyun maara?” 😭`,`${n} ne ${v} utha liya, jaise Mumbai local mein seat! 🚆`]);

function boot(){
  try{
    $("nm").value=localStorage.getItem("cn")||"";
    $("rc").value=localStorage.getItem("cr")||"";
  }catch(e){}
}

$("rnd").onclick=()=>{$("rc").value=pick(["CHAI","SAMOSA","GHODA","RANI","JALEBI","PYAADA"])+Math.floor(Math.random()*90+10)};

async function startCamera() {
  if (myStream) return true;
  try {
    myStream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
    $("myVid").srcObject = myStream;
    $("myVid").play().catch(e=>console.log(e));
    return true;
  } catch(e) {
    console.log("Camera Error: ", e);
    // Camera fail hone par bhi game band na ho, bina camera ke chale
    return true; 
  }
}

function handleIncomingCall(call) {
  call.answer(myStream);
  call.on('stream', (remoteStream) => {
    $("oppVid").srcObject = remoteStream;
    $("oppVid").play().catch(e=>console.log(e));
  });
}

function callPeer(targetPeerId) {
  if (!myStream || !peer) return;
  const call = peer.call(targetPeerId, myStream);
  if (call) {
    call.on('stream', (remoteStream) => {
      $("oppVid").srcObject = remoteStream;
      $("oppVid").play().catch(e=>console.log(e));
    });
  }
}

$("go").onclick=async ()=>{
  const n=$("nm").value.trim().slice(0,14), c=$("rc").value.trim().toUpperCase().replace(/[^A-Z0-9]/g,"");
  if(!n||!c){$("err").textContent="Naam aur room code dono daalo!";return}
  $("err").textContent="Connecting to room...";
  myName=n; room=c;
  try{localStorage.setItem("cn",n);localStorage.setItem("cr",c)}catch(e){}

  await startCamera();

  const hostPeerId = "chesstamasha-" + room;
  const peerConfig = {
    config: {
      iceServers: [
        { urls: 'stun:stun.l.google.com:19302' },
        { urls: 'stun:stun1.l.google.com:19302' },
        { urls: 'stun:stun2.l.google.com:19302' }
      ]
    }
  };

  peer = new Peer(peerConfig);

  peer.on('error', (err) => {
    console.log("Peer error:", err);
    if(err.type === 'unavailable-id') {
      // Already host, ignore
    }
  });

  peer.on('open', (id) => {
    peer.on('call', handleIncomingCall);

    // Pehle Client ban kar connect karne ki koshish karo
    let testConn = peer.connect(hostPeerId, { reliable: true });
    let connected = false;

    testConn.on('open', () => {
      connected = true;
      conn = testConn;
      color = "b";
      setupConn();
      conn.send({type:'JOIN', name: myName, myPeerId: id});
      callPeer(hostPeerId);
      showGame();
    });

    testConn.on('error', (e) => {
      console.log("Connection err:", e);
    });

    // Agar 2 second me connection nahi bana, matlab room pehle se nahi hai -> Host ban jao
    setTimeout(() => {
      if (!connected) {
        testConn.close();
        peer.destroy();
        
        peer = new Peer(hostPeerId, peerConfig);
        peer.on('open', () => {
          peer.on('call', handleIncomingCall);
          color = "w";
          showGame();
          $("msg").textContent = "Room ban gaya! Dost ka wait kar rahe hain...";
        });
        
        peer.on('connection', (c) => {
          conn = c;
          setupConn();
        });
      }
    }, 2000);
  });
};

function setupConn() {
  conn.on('data', (data) => {
    if(data.type === 'JOIN') {
      oppName = data.name;
      conn.send({type: 'INIT', name: MyNameSafe(), board: JSON.stringify(b), st, hostPeerId: peer.id});
      updatePlayerInfo();
      $("msg").textContent = `${oppName} aa gaya! Ab asli maza shuru 🔥`;
      if(data.myPeerId) callPeer(data.myPeerId);
    } else if(data.type === 'INIT') {
      oppName = data.name;
      b = JSON.parse(data.board);
      st = data.st;
      updatePlayerInfo();
      render();
      if(data.hostPeerId) callPeer(data.hostPeerId);
    } else if(data.type === 'MOVE') {
      b = JSON.parse(data.board);
      st = data.st;
      sel = null; ls = [];
      render();
      $("msg").textContent = st.msg;
      updatePlayerInfo();
    } else if(data.type === 'CHAT') {
      addChatMessage(data.name, data.msg, false);
    } else if(data.type === 'REMATCH') {
      b = start();
      st = {turn:"w", over:"", last:-1, msg:"Rematch shuru! Badla lene ka mauka 😤"};
      sel = null; ls = [];
      render();
      updatePlayerInfo();
      $("msg").textContent = st.msg;
    }
  });
}

function MyNameSafe() { return myName; }

function showGame() {
  $("err").textContent = "";
  $("lobby").classList.add("hide");
  $("game").classList.remove("hide");
  updatePlayerInfo();
  render();
}

function updatePlayerInfo() {
  const wName = color === "w" ? myName : (oppName || "wait...");
  const bName = color === "b" ? myName : (oppName || "wait...");
  $("pl").textContent = `⚪ ${wName} vs ⚫ ${bName} | ${st.over ? "Game over" : (st.turn=="w"?"⚪":"⚫")+" ki baari"} | Tum: ${color=="w"?"⚪":"⚫"}`;
  $("myVidLbl").textContent = myName || "Tum";
  $("oppVidLbl").textContent = oppName || "Dost";
}

function render(){
  const idx=[...Array(64).keys()];
  if(color=="b") idx.reverse();
  $("bd").innerHTML=idx.map(i=>{
    const p=b[i], r=i>>3, f=i&7;
    let c="q "+((r+f)%2?"d":"l");
    if(i==sel) c+=" sel";
    if(st && i==st.last) c+=" last";
    if(ls.includes(i)) c+=" mv"+(p?" c":"");
    return `<div class="${c}" data-i="${i}">${p?`<span class="${p[0]}">${G[p[1]]}\uFE0E</span>`:""}</div>`;
  }).join("");
}

$("bd").onclick=e=>{
  const q=e.target.closest(".q"); if(!q||!st)return;
  const i=+q.dataset.i;
  if(st.over||!color)return;
  if(!conn)return void($("msg").textContent="Dost ka wait karo, abhi akele ho! 🦗");
  if(st.turn!=color)return void($("msg").textContent="Sabr karo, dost ki baari hai! ⏳");

  if(sel!==null && ls.includes(i)){
    const cap=b[i], nb=mv(b,sel,i), nx=color=="w"?"b":"w", nm=myName;
    let over="", msg=cap?C(nm,N[cap[1]]):Q(nm);
    const chk=inCheck(nb,nx);
    if(!any(nb,nx)){
      over=chk?"mate":"draw";
      msg=chk?`CHECKMATE! 🏆🎉 ${nm} jeet gaya!`:"Draw! 🤝 Game samaapt.";
    } else if(chk){
      msg+=" ⚠️ CHECK! Raja ghabra gaya 😱";
    }
    b = nb;
    st = {turn: nx, last: i, over, msg};
    sel=null; ls=[];
    render();
    updatePlayerInfo();
    $("msg").textContent = msg;
    conn.send({type: 'MOVE', board: JSON.stringify(b), st});
  } else if(b[i] && b[i][0]==color){
    sel=i; ls=legal(b,i); render();
    if(!ls.length)$("msg").textContent=`${N[b[i][1]]} bol raha: “Main hil hi nahi sakta!” 😭`;
  } else {
    sel=null; ls=[]; render();
  }
};

function addChatMessage(name, text, isMe) {
  const el=$("chat"), m=document.createElement("div"), n=document.createElement("b"), t=document.createElement("span");
  if(isMe) m.className="me";
  n.textContent=name; t.textContent=text; m.append(n,t); el.append(m);
  el.scrollTop=el.scrollHeight;
}

function send(t){
  t=(t||"").trim(); if(!t)return;
  addChatMessage(myName, t, true);
  if(conn) conn.send({type: 'CHAT', name: myName, msg: t});
}

$("sd").onclick=()=>{send($("tx").value);$("tx").value=""};
$("tx").onkeydown=e=>{if(e.key=="Enter"){send($("tx").value);$("tx").value=""}};
document.querySelectorAll(".em").forEach(x=>x.onclick=()=>send(x.textContent));

$("rm").onclick=()=>{
  b = start();
  st = {turn:"w", over:"", last:-1, msg:"Rematch shuru! Badla lene ka mauka 😤"};
  sel = null; ls = [];
  render();
  updatePlayerInfo();
  $("msg").textContent = st.msg;
  if(conn) conn.send({type: 'REMATCH'});
};

$("cp").onclick=()=>{
  try{navigator.clipboard.writeText(room)}catch(e){}
  $("msg").textContent="Room code: "+room+" (dost ko bhejo)";
};

boot();
</script>
</body>
</html>
