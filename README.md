from pathlib import Path
import zipfile

root = Path("/mnt/data/terminal-site")
root.mkdir(exist_ok=True)

index_html = r"""<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Terminal Site</title>
<link rel="stylesheet" href="style.css">
</head>
<body>
<canvas id="matrix"></canvas>

<div id="terminal">
  <div id="stats">
    ACTIVE USERS: <span id="users">478</span> |
    TOKENS: <span id="tokens">1829344</span> |
    NODES: <span id="nodes">32</span>
  </div>
</div>

<script src="script.js"></script>
</body>
</html>
"""

style_css = r"""
:root{
--bg:#000;
--text:#00ff66;
}
body{
margin:0;
background:var(--bg);
color:var(--text);
font-family:monospace;
overflow:hidden;
}
#matrix{
position:fixed;
inset:0;
z-index:-1;
opacity:.25;
}
#terminal{
height:100vh;
overflow:auto;
padding:20px;
text-shadow:0 0 5px #00ff66;
}
#terminal::before{
content:"";
position:fixed;
inset:0;
pointer-events:none;
background:repeating-linear-gradient(to bottom,rgba(255,255,255,.03),rgba(255,255,255,.03) 1px,transparent 1px,transparent 3px);
}
"""

script_js = r"""
const terminal=document.getElementById("terminal");

function addLine(text){
 const d=document.createElement("div");
 d.textContent=text;
 terminal.appendChild(d);
 terminal.scrollTop=terminal.scrollHeight;
}

addLine("SYSTEM ONLINE");
addLine("Type help in future command implementation.");

let users=478,tokens=1829344,nodes=32;
setInterval(()=>{
 users+=Math.floor(Math.random()*5)-2;
 tokens+=Math.floor(Math.random()*500);
 if(Math.random()>.95) nodes++;
 usersEl.textContent=users;
 tokensEl.textContent=tokens;
 nodesEl.textContent=nodes;
},1000);

const usersEl=document.getElementById("users");
const tokensEl=document.getElementById("tokens");
const nodesEl=document.getElementById("nodes");

const events=[
"Incoming connection established",
"Node synchronized",
"Packet received",
"Database heartbeat OK",
"Relay online"
];

setInterval(()=>{
 addLine("[NET] "+events[Math.floor(Math.random()*events.length)]);
},1500);

// Matrix
const canvas=document.getElementById("matrix");
const ctx=canvas.getContext("2d");
function resize(){canvas.width=innerWidth;canvas.height=innerHeight;}
resize();
addEventListener("resize",resize);

const chars="ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
const size=14;
let drops=[];
function reset(){
 drops=Array(Math.floor(canvas.width/size)).fill(1);
}
reset();

function draw(){
 ctx.fillStyle="rgba(0,0,0,.05)";
 ctx.fillRect(0,0,canvas.width,canvas.height);
 ctx.fillStyle="#00ff66";
 ctx.font=size+"px monospace";
 for(let i=0;i<drops.length;i++){
   const t=chars[Math.floor(Math.random()*chars.length)];
   ctx.fillText(t,i*size,drops[i]*size);
   if(drops[i]*size>canvas.height && Math.random()>.975) drops[i]=0;
   drops[i]++;
 }
 requestAnimationFrame(draw);
}
draw();
"""

(root / "index.html").write_text(index_html)
(root / "style.css").write_text(style_css)
(root / "script.js").write_text(script_js)

zip_path = "/mnt/data/terminal-site-github-ready.zip"
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for f in root.iterdir():
        z.write(f, f.name)

print(zip_path)
