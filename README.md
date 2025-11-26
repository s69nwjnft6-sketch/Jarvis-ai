# jarvis ai
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jarvis AI</title>
<style>
/* --- Dark futuristic style --- */
body {margin:0;padding:0;background:#02040a;font-family:Arial;color:#e0faff;overflow:hidden;}
#particles {position:fixed;top:0;left:0;width:100%;height:100%;z-index:-2;}
#boot {position:fixed;top:0;left:0;width:100%;height:100%;background:black;display:flex;justify-content:center;align-items:center;font-size:28px;color:cyan;letter-spacing:3px;z-index:10;animation:bootFade 3.5s forwards;}
@keyframes bootFade {0%{opacity:1;}80%{opacity:1;}100%{opacity:0;display:none;}}
.jarvis-core {position:fixed;top:40px;left:50%;transform:translateX(-50%);width:150px;height:150px;border-radius:50%;border:4px solid cyan;box-shadow:0 0 35px cyan;animation:pulse 2.2s infinite linear;z-index:2;}
@keyframes pulse {0%{box-shadow:0 0 25px cyan;}50%{box-shadow:0 0 80px cyan;}100%{box-shadow:0 0 25px cyan;}}
.container {display:flex;flex-direction:column;height:100vh;}
.header {padding:10px;text-align:center;margin-top:160px;font-size:34px;letter-spacing:4px;font-weight:bold;color:cyan;text-shadow:0 0 10px cyan;}
.chat-box {flex:1;padding:20px;overflow-y:auto;}
.msg {max-width:70%;padding:14px 18px;margin-bottom:20px;border-radius:12px;font-size:17px;}
.user {background: rgba(0,255,255,0.15); border:1px solid cyan; box-shadow:0 0 12px cyan;margin-left:auto;}
.bot {background: rgba(0,120,150,0.25); border:1px solid cyan; box-shadow:0 0 22px cyan;backdrop-filter: blur(3px);animation: hologram 0.6s ease;}
@keyframes hologram {from {opacity:0; transform:scale(0.9);} to {opacity:1; transform:scale(1);}}
.typing {width:22px;height:22px;border-radius:50%;border:3px solid cyan;border-top-color:transparent;animation:spin 0.6s infinite linear;margin-bottom:20px;}
@keyframes spin {to {transform: rotate(360deg);}}
.input-area {display:flex;padding:20px;background: rgba(0,0,0,0.7);border-top:1px solid rgba(0,255,255,0.3);}
.input-area input {flex:1;padding:14px;background:#0b1320;border-radius:10px;border:1px solid cyan;color:white;font-size:16px;}
.input-area button {margin-left:10px;padding:14px;background:cyan;color:black;border:none;border-radius:10px;font-weight:bold;cursor:pointer;box-shadow:0 0 15px cyan;}
</style>
</head>
<body>
<div id="boot">INITIALIZING JARVIS...</div>
<canvas id="particles"></canvas>
<div class="jarvis-core"></div>

<div class="container">
  <div class="header">J A R V I S</div>
  <div class="chat-box" id="chatBox"></div>

  <div class="input-area">
    <input id="input" type="text" placeholder="Speak to Jarvis..." />
    <button onclick="sendMessage()">Send</button>
    <button onclick="startVoice()">🎤</button>
  </div>
</div>

<script>
// --- Particles background ---
const canvas = document.getElementById("particles");
const ctx = canvas.getContext("2d");
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;
let particles = [];
for (let i=0;i<90;i++){particles.push({x:Math.random()*canvas.width,y:Math.random()*canvas.height,r:Math.random()*2+1,s:Math.random()*2});}
function animateParticles(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    particles.forEach(p=>{
        ctx.beginPath();
        ctx.arc(p.x,p.y,p.r,0,Math.PI*2);
        ctx.fillStyle="cyan";
        ctx.fill();
        p.y-=p.s;
        if(p.y<0)p.y=canvas.height;
    });
    requestAnimationFrame(animateParticles);
}
animateParticles();

// --- Jarvis voice selection ---
function speakJarvis(text) {
    const synth = window.speechSynthesis;
    let voices = synth.getVoices();
    if(!voices.length){ // for iOS delay
        synth.onvoiceschanged = ()=>{voices = synth.getVoices(); speakJarvis(text);};
        return;
    }
    const jarvisVoice = voices.find(v => 
        v.name.includes("Alex") || v.name.includes("Siri") || v.name.includes("Daniel")
    ) || voices[0]; // fallback
    const utter = new SpeechSynthesisUtterance(text);
    utter.voice = jarvisVoice;
    utter.pitch = 1.05;
    utter.rate = 1;
    synth.speak(utter);
}

// --- Chat / AI ---
async function sendMessage(){
    const input = document.getElementById("input");
    const chatBox = document.getElementById("chatBox");
    const text = input.value.trim();
    if(!text) return;

    const userMsg = document.createElement("div");
    userMsg.className="msg user";
    userMsg.textContent=text;
    chatBox.appendChild(userMsg);
    input.value="";

    const typing=document.createElement("div");
    typing.className="typing";
    chatBox.appendChild(typing);
    chatBox.scrollTop=chatBox.scrollHeight;

    const response = await fetch("https://api.openai.com/v1/chat/completions",{
        method:"POST",
        headers:{
            "Content-Type":"application/json",
            "Authorization": `Bearer ${process.env.OPENAI_API_KEY}`
        },
        body: JSON.stringify({
            model:"gpt-5-mini",
            messages:[{role:"user",content:text}]
        })
    });

    const data = await response.json();
    typing.remove();
    const botText = data.choices[0].message.content;

    const botMsg=document.createElement("div");
    botMsg.className="msg bot";
    botMsg.textContent=botText;
    chatBox.appendChild(botMsg);
    chatBox.scrollTop=chatBox.scrollHeight;

    speakJarvis(botText);
}

// --- Voice input ---
function startVoice(){
    const recog = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
    recog.lang="en-US";
    recog.onresult=e=>{
        document.getElementById("input").value = e.results[0][0].transcript;
        sendMessage();
    };
    recog.start();
}
</script>
</body>
</html>
