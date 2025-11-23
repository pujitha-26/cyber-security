<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Secure Communication</title>
<style>
  :root{
    --panel-bg: #ffffff;
    --panel-border: #8a8a8a; /* DARKER BORDER */
    --page-bg: linear-gradient(135deg,#f0f4f8,#e8f1fb);
    --header-bg: #f7fafc;
    --accent: #006b5e;
    --text: #000000;
    --muted: #6b7280;
  }
  *{box-sizing:border-box;font-family:Inter,Segoe UI,Roboto,Arial,sans-serif}
  html,body{height:100%;margin:0;color:var(--text);background:var(--page-bg);}

  #home{
    height:100vh;
        width:100%;
        display:flex;
        justify-content:center;
        align-items:center;
        flex-direction:column;
        text-align:center;
        background: linear-gradient(135deg,#1e3c72,#2a5298,#4b79a1,#283e51);
        background-size:400% 400%;
        animation: gradientShift 10s ease infinite;
        color:white;
  }
@keyframes gradientShift {
        0%{background-position:0% 50%;}
        50%{background-position:100% 50%;}
        100%{background-position:0% 50%;}
    }
  h1{font-size:40px;margin-bottom:10px;letter-spacing:1px;}
    p{opacity:0.9; margin-bottom:25px;}
    #startBtn{
        padding:15px 45px;
        background:white;
        border:none;
        border-radius:30px;
        font-size:18px;
        font-weight:600;
        cursor:pointer;
        color:#003c89;
        box-shadow:0 4px 15px rgba(255,255,255,0.4);
        transition:.3s;
  }
  #startBtn:hover{transform:translateY(-3px);transition:.18s ease}

  #app{display:none;padding:22px;height:100vh;background:lightgreen}
  .container{display:flex;gap:20px;align-items:flex-start;justify-content:center;height:100%}

  .panel{
    width:32%;min-width:300px;height:92%;background:var(--panel-bg);
    border-radius:18px;border:2px solid var(--panel-border); /* THICKER BORDER */
    display:flex;flex-direction:column;box-shadow:0 12px 30px rgba(10,20,40,0.06);
  }

  .panel .header{
    height:64px;background:var(--header-bg);display:flex;align-items:center;padding:12px 16px;
    gap:10px;font-weight:700;color:var(--text);border-bottom:2px solid var(--panel-border);
  }
  .dot{width:12px;height:12px;border-radius:50%;background:#34d399}

  .chat-area{
    flex:1;padding:14px;overflow:auto;background:linear-gradient(180deg,#fafafa,#ffffff);
  }

  .bubble{max-width:78%;padding:10px 12px;border-radius:12px;margin:8px 0;font-size:14px;color:var(--text)}
  .sent{background:#e6f7f2;margin-left:auto;border:1px solid #d6efe7}
  .received{background:#ffffff;border:1px solid #efefef}
  .time{display:inline-block;margin-left:8px;font-size:11px;color:var(--muted)}

  .input-row{display:flex;gap:10px;padding:12px;background:transparent;border-top:2px solid var(--panel-border)}
  .input-row input{
    flex:1;padding:10px 12px;border-radius:12px;border:2px solid var(--panel-border);
    background:#fff;color:var(--text);font-size:14px;outline:none;
  }

  /* UPDATED SEND BUTTON ARROW */
  .send-btn{
    width:44px;height:44px;border-radius:10px;border:2px solid var(--panel-border);
    background:linear-gradient(180deg,#ffffff,#f3f6f8);cursor:pointer;font-weight:900;
    font-size:20px;color:#000; /* BLACK & BOLD */
  }

  #cipherBox{
    background:#fcfffb;border:2px solid var(--panel-border);padding:12px;border-radius:10px;
    color:var(--text);font-family: ui-monospace, "Roboto Mono", monospace;font-size:13px;
    white-space:pre-wrap;
  }

  @media (max-width:1100px){
    .container{flex-direction:column;align-items:center;padding-bottom:40px}
    .panel{width:92%;height:70vh}
  }
</style>
</head>
<body>

<div id="home">
  <h1>Secure Communication System</h1>
  <p class="lead">Click START to open the demo→End to End Encryption </p>
  <button id="startBtn">START</button>
</div>

<div id="app" aria-hidden="true">
  <div class="container">

    <div class="panel" id="panel-client">
      <div class="header"><div class="dot"></div> Client (Sender)</div>
      <div class="chat-area" id="clientChat">
        <div class="bubble received" style="border:1px dashed #c1c1c1;color:var(--muted)">Compose your message below</div>
      </div>
      <div class="input-row">
        <input id="messageInput" placeholder="Type message..." />
        <button class="send-btn" id="sendBtn">➤</button>
      </div>
    </div>

    <div class="panel" id="panel-encrypt">
      <div class="header"><div class="dot" style="background:#f59e0b"></div> Encryption</div>
      <div class="chat-area"><div id="cipherBox">Encrypted output will appear here</div></div>
      <div style="padding:12px;border-top:2px solid var(--panel-border);"><small style="color:var(--muted)">Encryption: Base64 (demo)</small></div>
    </div>

    <div class="panel" id="panel-receiver">
      <div class="header"><div class="dot" style="background:#60a5fa"></div> Receiver (Output)</div>
      <div class="chat-area" id="receiverChat">
        <div class="bubble received">Waiting for messages...</div>
      </div>
      <div style="padding:12px;border-top:2px solid var(--panel-border);"><small style="color:var(--muted)">Delivery indicator: ✔✔</small></div>
    </div>

  </div>
</div>

<script>
const startBtn=document.getElementById('startBtn');
const home=document.getElementById('home');
const app=document.getElementById('app');
startBtn.addEventListener('click',()=>{home.style.display='none';app.style.display='block';});

const el=id=>document.getElementById(id);
const encode=s=>btoa(unescape(encodeURIComponent(s)));
const decode=s=>decodeURIComponent(escape(atob(s)));

const clientChat=el('clientChat');
const receiverChat=el('receiverChat');
const cipherBox=el('cipherBox');
const sendBtn=el('sendBtn');
const input=el('messageInput');

function addClientBubble(text){
  const d=document.createElement('div');
  d.className='bubble sent';
  const time=new Date().toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
  d.innerText=text+' ';
  const tm=document.createElement('span');tm.className='time';tm.innerText=time;
  d.appendChild(tm);clientChat.appendChild(d);
  clientChat.scrollTop=clientChat.scrollHeight;
}

function addReceiverBubble(text){
  if(receiverChat.children.length===1 && receiverChat.children[0].innerText.includes('Waiting')){
    receiverChat.innerHTML='';
  }
  const d=document.createElement('div');
  d.className='bubble received';
  const time=new Date().toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
  d.innerText=text+' ';
  const tm=document.createElement('span');tm.className='time';tm.innerText=time;
  const ticks=document.createElement('span');
  ticks.className='time';ticks.style.marginLeft='8px';ticks.innerHTML='✔✔';
  d.appendChild(tm);d.appendChild(ticks);receiverChat.appendChild(d);
  receiverChat.scrollTop=receiverChat.scrollHeight;
  setTimeout(()=>{ticks.style.color='#1d4ed8';},1500);
}

function sendMessage(){
  const txt=input.value.trim();
  if(!txt)return;
  addClientBubble(txt);
  const cipher=encode(txt);
  cipherBox.innerText=cipher;
  try{addReceiverBubble(decode(cipher));}catch(e){addReceiverBubble('Decryption failed');}
  input.value='';input.focus();
}

sendBtn.addEventListener('click',sendMessage);
input.addEventListener('keydown',(e)=>{if(e.key==='Enter')sendMessage();});
</script>
</body>
</html>
