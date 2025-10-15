# i
LiveOlympiad Mock Test Portal is an interactive web-based exam simulator designed for CBSE students of Classes 1 to 10. It provides subject-wise mock tests in Maths, Science, and English. Perfect for schools and students to practice online tests for the LiveOlympiad.

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>LiveOlympiad | Mock Test Portal</title>
<style>
:root {
  --blue:#0B3D91;
  --orange:#f97316;
  --yellow:#facc15;
  --light:#ffffff;
  --dark:#111827;
  --border:#e5e7eb;
}

body {
  font-family:'Helvetica', sans-serif;
  background:var(--light);
  color:var(--dark);
  margin:0;
  padding:0;
}

h1 {
  text-align:center;
  color:var(--blue);
  font-size:22px;
  margin:12px 0;
}

.test-box {
  background:#fff;
  padding:20px;
  border-radius:10px;
  border-left:5px solid var(--orange);
  box-shadow:0 2px 6px rgba(0,0,0,.08);
  margin-bottom:18px;
}

/* 👇 Question text larger */
.test-box b {
  font-size:18px;
}

.test-box br + div,
.test-box div.options {
  font-size:18px; /* 👈 increased question text */
  line-height:1.6;
  margin-top:10px;
}

/* 👇 Options styling */
.options label {
  display:block;
  margin:12px 0;
  padding:12px;
  border:1px solid #d1d5db;
  border-radius:8px;
  background:#f9fafb;
  cursor:pointer;
  font-size:17px; /* 👈 increased option text */
  transition:background 0.2s;
}

.options label:hover {
  background:#fff8f1;
}

.options input {
  margin-right:10px;
  transform:scale(1.3); /* 👈 larger radio buttons */
}

/* 👇 Buttons vertically stacked (previous below next) */
.nav-controls {
  display:flex;
  flex-direction:column;
  gap:10px;
  margin-top:20px;
}

.nav-controls button {
  width:100%;
  padding:14px;
  font-size:16px;
  border-radius:8px;
  background:var(--blue);
  color:white;
  font-weight:600;
  border:none;
  cursor:pointer;
  transition:0.3s;
}
.nav-controls button:hover { background:#062c6a; }

/* 👇 Navigator buttons remain same */
.navigator {
  background:#fff;
  border:2px solid var(--orange);
  border-radius:10px;
  box-shadow:0 2px 6px rgba(0,0,0,.08);
  padding:10px;
  margin-top:22px;
  display:flex;
  flex-wrap:wrap;
  justify-content:center;
  gap:6px;
}
.nav-btn {
  width:36px;
  height:36px;
  line-height:36px;
  text-align:center;
  border-radius:50%;
  border:1px solid var(--border);
  font-weight:600;
  cursor:pointer;
  font-size:15px;
}
.nav-btn.current{background:var(--orange);color:#fff;}
.nav-btn.answered{background:var(--blue);color:#fff;}
.nav-btn.seen{background:var(--yellow);color:#000;}
.nav-btn.unanswered{background:#f3f4f6;color:#333;}

/* 👇 Responsive for portrait screens */
@media(max-width:600px){
  h1 { font-size:20px; }
  .test-box { padding:16px; }
  .options label { font-size:16px; padding:10px; }
  .test-box b { font-size:17px; }
}
</style>

</head>
<body>

<h1>LiveOlympiad — Mock Test</h1>

<div class="card" id="selectCard">
  <select id="classSelect"><option value="">Select Class</option></select>
  <select id="subjectSelect">
    <option value="">Select Subject</option>
    <option value="Maths">Maths</option>
    <option value="Science">Science</option>
    <option value="English">English</option>
  </select>
  <select id="mockSelect"><option value="">Select Mock Test</option></select>
  <button onclick="startTest()">Start Test</button>
</div>

<div class="container" id="main" style="display:none;">
  <div id="timer">Time Left: 30:00</div>
  <div id="test"></div>
  <div class="navigator" id="navigator"></div>
</div>

<script>
// Dropdown setup
for(let i=1;i<=10;i++){
  classSelect.innerHTML+=`<option value="${i}">Class ${i}</option>`;
  mockSelect.innerHTML+=`<option value="${i}">Mock Test ${i}</option>`;
}

// RNG & helpers
function makeRNG(seed){let t=seed>>>0;return()=>{t+=0x6D2B79F5;let r=Math.imul(t^(t>>>15),1|t);r^=r+Math.imul(r^(r>>>7),r|61);return((r^(r>>>14))>>>0)/4294967296;}}
function hashSeed(s){let h=2166136261>>>0;for(let i=0;i<s.length;i++){h^=s.charCodeAt(i);h=Math.imul(h,16777619);}return h>>>0;}
function shuffle(a,r){let arr=a.slice();for(let i=arr.length-1;i>0;i--){let j=Math.floor(r()*(i+1));[arr[i],arr[j]]=[arr[j],arr[i]];}return arr;}
function rand(min,max,r){return Math.floor(r()*(max-min+1))+min;}

// Paragraphs
const paragraphs=[
  {text:`Once upon a time, a farmer had a goose that laid one golden egg each day. The farmer became greedy and decided to take all the eggs at once. He killed the goose, but found no gold inside. He lost his source of wealth forever.`,
   qs:[["What did the goose lay each day?","A golden egg"],["Why did the farmer kill the goose?","He was greedy"],["Moral of the story?","Greed leads to loss"]]},
  {text:`Plastic pollution is one of the biggest problems today. It harms oceans, animals, and even enters our food. Reducing plastic use and recycling can help save our planet.`,
   qs:[["Main topic of paragraph?","Plastic pollution"],["What does plastic harm?","Oceans and animals"],["How can we solve it?","By reducing and recycling plastic"]]}
];

// Generators
function genMath(c,r){const type=rand(1,5,r);let q,a,d;
  if(type===1){let x=rand(5,30,r),y=rand(5,30,r);q=`Simplify: ${x}×${y}+${y}`;a=(x*y+y)+'';d=[a-5,a+5,a+10];}
  else if(type===2){let p=rand(10,90,r),t=rand(50,200,r);q=`Find ${p}% of ${t}`;a=Math.round(p*t/100)+'';d=[a-10,a+10,a+20];}
  else if(type===3){let a1=rand(10,80,r),a2=rand(10,80,r),a3=rand(10,80,r);q=`Average of ${a1}, ${a2}, ${a3}`;a=((a1+a2+a3)/3).toFixed(1);d=[(+a+2).toFixed(1),(+a-2).toFixed(1),(+a+3).toFixed(1)];}
  else if(type===4){let x=rand(2,10,r),y=rand(1,10,r);q=`If x=${x}, find 3x − ${y}`;a=(3*x-y)+'';d=[a-3,a+3,a+5];}
  else{let a1=rand(10,60,r),a2=rand(10,60,r);let per=Math.round(((a2-a1)/a1)*100);q=`Value changes from ${a1} to ${a2}, % change?`;a=per+'';d=[per+5,per-5,per+10];}
  return {q,options:shuffle([a,...d.map(String)],r),answer:a};
}
function genSci(c,r){const pool=[
  ["Which law explains action and reaction?","Newton's Third Law"],
  ["Which metal is liquid at room temperature?","Mercury"],
  ["Unit of electric current?","Ampere"],
  ["Gas produced during photosynthesis?","Oxygen"],
  ["Which organ purifies blood?","Kidney"],
  ["Chemical symbol of Iron?","Fe"]
];const q=pool[rand(0,pool.length-1,r)];
let wrong=shuffle(pool.map(x=>x[1]).filter(x=>x!==q[1]),r).slice(0,3);
return {q:q[0],options:shuffle([q[1],...wrong],r),answer:q[1]};}
function genEng(c,r){const type=rand(1,4,r);let q,a,d;
  if(type===1){const pair=[['go','went'],['run','ran'],['come','came'],['see','saw']][rand(0,3,r)];
    q=`Past tense of '${pair[0]}' is __`;a=pair[1];d=['goed',pair[0]+'ed',pair[0]+'en'];}
  else if(type===2){const syn=[['happy','joyful'],['big','large'],['smart','clever'],['sad','unhappy']][rand(0,3,r)];
    q=`Synonym of '${syn[0]}'`;a=syn[1];d=['angry','tiny','fast','lazy'];}
  else if(type===3){const ant=[['hot','cold'],['light','dark'],['fast','slow'],['up','down']][rand(0,3,r)];
    q=`Antonym of '${ant[0]}'`;a=ant[1];d=['big','small','high','near'];}
  else{const arr=[{q:'He is interested ___ science.',a:'in',d:['on','at','for']},
                 {q:'They arrived ___ the airport early.',a:'at',d:['in','to','on']}];
    const pick=arr[rand(0,arr.length-1,r)];q=pick.q;a=pick.a;d=pick.d;}
  return {q,options:shuffle([a,...d],r),answer:a};}
function genParaQ(r){const para=paragraphs[rand(0,paragraphs.length-1,r)];
const out=[];para.qs.forEach((item,i)=>{
let opt=shuffle([item[1],'None','All','Not given'],r);
let text=i===0?`<p class='para'>${para.text}</p>${item[0]}`:item[0];
out.push({q:text,options:opt,answer:item[1]});
});return out;}
function makeQs(c,s,m){const seed=hashSeed(`C${c}|${s}|M${m}`),r=makeRNG(seed);
let qList=[];for(let i=0;i<30;i++){
if(s==='Maths')qList.push(genMath(c,r));
else if(s==='Science')qList.push(genSci(c,r));
else qList.push(genEng(c,r));}
if(s==='English' && c>=6){const para=genParaQ(r);
qList.splice(0,para.length,...para);}return qList;}

// Logic
let qs=[],sec=1800,timerId,current=0,answers={},seen=new Set();
function startTest(){
  const c=classSelect.value,s=subjectSelect.value,m=mockSelect.value;
  if(!c||!s||!m){alert("Please select Class, Subject & Mock Test");return;}
  qs=makeQs(c,s,m);
  selectCard.style.display='none';main.style.display='block';
  current=0;sec=1800;updateTimer();
  clearInterval(timerId);
  timerId=setInterval(()=>{sec--;updateTimer();if(sec<=0){clearInterval(timerId);submit();}},1000);
  renderNav();showQuestion();
}
function updateTimer(){const m=Math.floor(sec/60),s=String(sec%60).padStart(2,'0');
timer.textContent=`Time Left: ${m}:${s}`;}
function saveAnswer(){const sel=document.querySelector(`input[name='q${current}']:checked`);
if(sel)answers[current]=sel.value;}
function getSaved(i){return answers[i]||"";}
function showQuestion(){
  seen.add(current);
  const q=qs[current];
  let html=`<div class='test-box'><b>Q${current+1}/${qs.length}</b><br>${q.q}<div class='options'>`;
  q.options.forEach(opt=>{
    html+=`<label><input type='radio' name='q${current}' value='${opt}' ${getSaved(current)==opt?"checked":""}> ${opt}</label>`;
  });
  html+=`</div><div class='nav-controls'>`;
  if(current>0)html+=`<button onclick='prevQ()'>Previous</button>`;
  if(current<qs.length-1)html+=`<button onclick='nextQ()'>Next</button>`;
  else html+=`<button onclick='submit()'>Submit Test</button>`;
  html+=`</div></div>`;
  test.innerHTML=html;
  renderNav();
}
function nextQ(){saveAnswer();if(current<qs.length-1){current++;showQuestion();}}
function prevQ(){saveAnswer();if(current>0){current--;showQuestion();}}
function gotoQ(i){saveAnswer();current=i;showQuestion();}
function renderNav(){
  const nav=document.getElementById("navigator");
  nav.innerHTML="";
  for(let i=0;i<qs.length;i++){
    const btn=document.createElement("div");
    btn.textContent=i+1;
    let cls="nav-btn ";
    if(i===current)cls+="current";
    else if(answers[i])cls+="answered";
    else if(seen.has(i))cls+="seen";
    else cls+="unanswered";
    btn.className=cls;
    btn.onclick=()=>gotoQ(i);
    nav.appendChild(btn);
  }
}
function submit(){
  saveAnswer();clearInterval(timerId);
  let correct=0;
  qs.forEach((q,i)=>{if(answers[i]===q.answer)correct++;});
  const total=qs.length,wrong=total-correct,p=((correct/total)*100).toFixed(2);
  test.innerHTML=`<div class='test-box result'>✅ Correct: ${correct}<br>❌ Wrong: ${wrong}<br>📊 Score: ${p}%<br><br><button onclick='location.reload()'>Back</button></div>`;
  document.getElementById("navigator").style.display="none";
  timer.textContent='Test Finished';
}
</script>
</body>
</html>
