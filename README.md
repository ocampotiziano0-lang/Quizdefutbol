
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>QuizDelFutbol</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#071b13;color:white;font-family:Arial,sans-serif}
header{text-align:center;padding:28px 12px;background:linear-gradient(130deg,#08763f,#06351f)}
header h1{font-size:34px;margin:5px}
header p{color:#d1f5df}
main{max-width:850px;margin:auto;padding:18px}
.stats{display:flex;justify-content:space-around;background:#123b28;padding:15px;border-radius:14px;margin-bottom:20px}
h2{text-align:center}
.modes{display:grid;grid-template-columns:repeat(2,1fr);gap:12px}
button{cursor:pointer;border:0;border-radius:12px;padding:15px;font-size:15px;font-weight:bold}
.mode{background:#174b32;color:white;border:1px solid #347451}
.mode:active{transform:scale(.97)}
.panel{background:#103523;padding:20px;border-radius:18px;margin-top:15px}
.answer{display:block;width:100%;margin:10px 0;background:#20553a;color:white;text-align:left}
.answer.correct{background:#16864c}
.answer.wrong{background:#a63232}
.primary{width:100%;background:#4be28b;color:#062416;margin-top:15px}
#game{display:none}
#feedback{text-align:center;min-height:25px;margin:15px}
.small{color:#b8d5c3;text-align:center}
footer{text-align:center;color:#86a895;padding:25px}
</style>
</head>
<body>
<header>
<div style="font-size:45px">⚽</div>
<h1>QUIZ DEL FÚTBOL</h1>
<p>¿Sos un verdadero experto?</p>
</header>
<main>
<div class="stats">
<div>⭐ Puntos: <b id="points">0</b></div>
<div>🏆 Récord: <b id="record">0</b></div>
</div>

<section id="home">
<h2>Elegí tu desafío</h2>
<div class="modes" id="modes"></div>
</section>

<section id="game" class="panel">
<p class="small" id="modeName"></p>
<div class="stats">
<span id="counter">1/5</span>
<span id="timer">⏱️ 20</span>
</div>
<h2 id="question"></h2>
<div id="answers"></div>
<p id="feedback"></p>
<button class="primary" id="next" onclick="next()" disabled>Siguiente →</button>
<button class="mode" style="width:100%;margin-top:10px" onclick="home()">Volver al menú</button>
</section>

<section id="result" class="panel" style="display:none;text-align:center">
<div style="font-size:55px">🏆</div>
<h2>¡Desafío terminado!</h2>
<h1 id="final"></h1>
<p id="message"></p>
<button class="primary" onclick="home()">Volver al menú</button>
</section>
</main>
<footer>QUIZ DEL FÚTBOL · Demostrá lo que sabés</footer>

<script>
const modes=[
"🧠 Trivia","🕵️ Adiviná el jugador","🛡️ Adiviná el escudo",
"👕 Adiviná la camiseta","⏱️ Contra reloj","📚 Historia",
"🔄 Carrera del jugador","⚖️ Verdadero o falso",
"🎯 Adiviná el resultado","⭐ Equipo ideal",
"🏆 Desafío de campeones","🔥 Desafío diario"
];

const bank=[
["¿Qué selección ganó el Mundial 2022?",["Argentina","Francia","Brasil","Alemania"],0],
["¿Cuántos jugadores tiene un equipo en cancha?",["9","10","11","12"],2],
["¿Qué país ganó más Mundiales masculinos?",["Argentina","Italia","Brasil","Alemania"],2],
["¿En qué club jugó Messi durante gran parte de su carrera?",["Barcelona","Chelsea","Milan","Boca"],0],
["¿Qué selección ganó el Mundial 2018?",["Croacia","Francia","Argentina","España"],1],
["¿Qué equipo argentino juega en La Bombonera?",["River","Racing","Boca","Independiente"],2],
["¿Cuánto dura un partido sin descuento?",["80 minutos","90 minutos","100 minutos","120 minutos"],1],
["¿Qué país organizó el Mundial 2014?",["Rusia","Brasil","Qatar","Sudáfrica"],1],
["¿Qué jugador es conocido como CR7?",["Neymar","Cristiano Ronaldo","Mbappé","Suárez"],1],
["¿Qué selección ganó el Mundial 1986?",["Argentina","Alemania","Brasil","Italia"],0]
];

const trueFalse=[
["Argentina ganó el Mundial 2022.",["Verdadero","Falso"],0],
["Un partido normal dura 80 minutos.",["Verdadero","Falso"],1],
["Brasil ganó cinco Mundiales masculinos.",["Verdadero","Falso"],0],
["Messi nació en Brasil.",["Verdadero","Falso"],1],
["Francia ganó el Mundial 2018.",["Verdadero","Falso"],0]
];

const players=[
["Ganó ocho Balones de Oro y es argentino.","Lionel Messi"],
["Es portugués y lleva el número 7 como símbolo.","Cristiano Ronaldo"],
["Es francés y fue campeón del mundo en 2018.","Kylian Mbappé"],
["Es brasileño y se hizo famoso por sus regates.","Neymar"]
];

const clubs=[
["Su estadio es La Bombonera.","Boca Juniors"],
["Su estadio es el Monumental.","River Plate"],
["Su apodo es el Rojo.","Independiente"],
["Su apodo es la Academia.","Racing Club"]
];

const results=[
["¿Qué resultado tuvo la final Argentina-Francia del Mundial 2022 al terminar el tiempo extra?",["2-1","3-3","1-1","4-2"],1],
["¿Cuántos goles hizo Argentina en esa final?",["2","3","4","1"],1],
["¿Quién ganó el Mundial 2014?",["Argentina","Brasil","Alemania","Francia"],2]
];

let selected=[],index=0,score=0,locked=false,clock=null,seconds=20;
let mode="";
let record=Number(localStorage.getItem("quizRecord")||0);
document.getElementById("record").textContent=record;

const modeBox=document.getElementById("modes");
modes.forEach((m,i)=>{
let b=document.createElement("button");
b.className="mode";b.textContent=m;
b.onclick=()=>start(i);
modeBox.appendChild(b);
});

function start(n){
mode=modes[n];score=0;index=0;locked=false;
if(clock)clearInterval(clock);
let source=bank;
if(n===1)source=players.map(x=>[x[0],players.map(p=>p[1]),players.indexOf(x)]);
if(n===2||n===3)source=clubs.map(x=>[x[0],clubs.map(c=>c[1]),clubs.indexOf(x)]);
if(n===7)source=trueFalse;
if(n===8)source=results;
if(n===5||n===10||n===11)source=bank.slice().reverse();
if(n===6)source=players;
if(n===9)source=bank.slice(0,5);
selected=[...source].sort(()=>Math.random()-.5).slice(0,5);
if(n===4)seconds=30;else seconds=20;
document.getElementById("home").style.display="none";
document.getElementById("result").style.display="none";
document.getElementById("game").style.display="block";
document.getElementById("modeName").textContent=mode;
show();
}

function show(){
locked=false;
let q=selected[index];
document.getElementById("counter").textContent=(index+1)+"/"+selected.length;
document.getElementById("question").textContent=q[0];
document.getElementById("feedback").textContent="";
document.getElementById("next").disabled=true;
document.getElementById("answers").innerHTML="";
q[1].forEach((a,i)=>{
let b=document.createElement("button");
b.className="answer";b.textContent=a;
b.onclick=()=>choose(i,b);
document.getElementById("answers").appendChild(b);
});
if(clock)clearInterval(clock);
if(mode.includes("reloj")){
seconds=30;
document.getElementById("timer").textContent="⏱️ "+seconds;
clock=setInterval(()=>{
seconds--;
document.getElementById("timer").textContent="⏱️ "+seconds;
if(seconds<=0){
clearInterval(clock);
if(!locked)choose(-1,null);
}
},1000);
}else{
document.getElementById("timer").textContent="⚽ ¡A jugar!";
}
}

function choose(i,button){
if(locked)return;
locked=true;
if(clock)clearInterval(clock);
let q=selected[index];
let correct=i===q[2];
let buttons=document.querySelectorAll(".answer");
buttons.forEach((b,j)=>{
b.disabled=true;
if(j===q[2])b.classList.add("correct");
});
if(correct){
score+=100;
document.getElementById("feedback").textContent="¡CORRECTO! +100 puntos 🎉";
}else{
if(button)button.classList.add("wrong");
document.getElementById("feedback").textContent="La respuesta era: "+q[1][q[2]];
}
document.getElementById("points").textContent=score;
document.getElementById("next").disabled=false;
}

function next(){
if(!locked)return;
index++;
if(index<selected.length)show();
else finish();
}

function finish(){
  if(clock) clearInterval(clock);

  document.getElementById("game").style.display="none";
  document.getElementById("result").style.display="block";
  document.getElementById("final").textContent=score+" puntos";

  document.getElementById("message").textContent=
    score===500 ? "¡INCREÍBLE! Sos una leyenda 🐐" :
    score>=300 ? "¡Muy buen jugador! ⚽" :
    "¡Seguí practicando! 🔥";

  if(score>record){
    record=score;
    localStorage.setItem("quizRecord",record);
    document.getElementById("record").textContent=record;
  }
}

function home(){
if(clock)clearInterval(clock);
document.getElementById("game").style.display="none";
document.getElementById("result").style.display="none";
document.getElementById("home").style.display="block";
}
</script>
</body>
</html>
