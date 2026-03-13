<!DOCTYPE html>
<html>

<head>

<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">

<title>Station Information System</title>

<style>

body{
font-family:Segoe UI;
margin:0;
background:#eef2f7;
}

/* SEARCH AREA */

.searchArea{

padding:20px;

background:white;

box-shadow:0 2px 10px rgba(0,0,0,.1);

position:relative;

}

#search{

width:280px;

padding:10px;

font-size:16px;

border:1px solid #ccc;

border-radius:6px;

}

/* SUGGESTIONS */

#suggestions{

position:absolute;

top:65px;

left:20px;

width:280px;

background:white;

border:1px solid #ddd;

border-radius:6px;

box-shadow:0 5px 15px rgba(0,0,0,.1);

display:none;

max-height:220px;

overflow:auto;

}

.suggestion{

padding:8px 10px;

cursor:pointer;

}

.suggestion:hover{

background:#d1fae5;

}

/* MAIN AREA */

#stationContent{

padding:25px;

}

/* GRID */

.grid{

display:flex;

flex-wrap:wrap;

gap:18px;

}

/* GREEN GLASS CARD */

.card{

background:linear-gradient(
135deg,
rgba(34,197,94,0.18),
rgba(16,185,129,0.25)
);

backdrop-filter:blur(10px);

border:1px solid rgba(34,197,94,0.35);

padding:16px;

border-radius:14px;

box-shadow:
0 8px 25px rgba(0,0,0,0.15),
inset 0 0 8px rgba(255,255,255,0.4);

flex:1 1 420px;

min-width:420px;

transition:all .25s ease;

}

.card:hover{

transform:translateY(-3px);

box-shadow:
0 12px 30px rgba(0,0,0,0.2),
inset 0 0 10px rgba(255,255,255,0.5);

}

.card h3{

margin-top:0;

color:#065f46;

}

/* TABLE */

table{

width:100%;

border-collapse:collapse;

}

td{

border:1px solid rgba(0,0,0,.1);

padding:6px 10px;

font-size:14px;

white-space:nowrap;

}

tr:nth-child(even){

background:rgba(255,255,255,.4);

}

/* MOBILE */

@media (max-width:700px){

.card{
min-width:100%;
}

td{
white-space:normal;
word-break:break-word;
}

}

</style>

</head>

<body>

<div class="searchArea">

<input id="search" placeholder="Search Station Code">

<div id="suggestions"></div>

</div>

<div id="stationContent"></div>

<script>

const sheetID="1y1wPLDcbF1uJNx07cZ4NDGfZoe-Jbkvmo4JZ8AYrsZM";

let stations=[];
let currentIndex=-1;

const search=document.getElementById("search");
const box=document.getElementById("suggestions");


/* LOAD STATIONS */

async function loadStations(){

let url=`https://docs.google.com/spreadsheets/d/${sheetID}/gviz/tq?sheet=INDEX`;

let res=await fetch(url);

let text=await res.text();

let json=JSON.parse(text.substring(47).slice(0,-2));

let rows=json.table.rows;

rows.forEach(r=>{

if(r.c[0]){

stations.push(r.c[0].v.toUpperCase());

}

});

}

loadStations();



/* SEARCH */

search.addEventListener("input",function(){

let value=this.value.toUpperCase();

box.innerHTML="";
currentIndex=-1;

if(value===""){

box.style.display="none";

document.getElementById("stationContent").innerHTML="";

return;

}

stations.forEach(st=>{

if(st.includes(value)){

let div=document.createElement("div");

div.className="suggestion";

div.textContent=st;

div.onclick=()=>{

box.style.display="none";

loadStation(st);

};

box.appendChild(div);

}

});

box.style.display="block";

});



/* KEYBOARD */

search.addEventListener("keydown",function(e){

let items=document.querySelectorAll(".suggestion");

if(e.key==="ArrowDown"){

currentIndex++;

if(currentIndex>=items.length) currentIndex=0;

highlight(items);

}

else if(e.key==="ArrowUp"){

currentIndex--;

if(currentIndex<0) currentIndex=items.length-1;

highlight(items);

}

else if(e.key==="Enter"){

if(currentIndex>-1){

loadStation(items[currentIndex].textContent);

box.style.display="none";

}

else if(items.length>0){

loadStation(items[0].textContent);

box.style.display="none";

}

}

else if(e.key==="Escape"){

box.style.display="none";

}

});


function highlight(items){

items.forEach(i=>i.style.background="");

if(items[currentIndex]){

items[currentIndex].style.background="#bbf7d0";

}

}



/* FETCH RANGE */

async function getRange(sheet,range){

let url=`https://docs.google.com/spreadsheets/d/${sheetID}/gviz/tq?sheet=${sheet}&range=${range}`;

let res=await fetch(url);

let text=await res.text();

let json=JSON.parse(text.substring(47).slice(0,-2));

let rows=json.table.rows;

let data=[];

rows.forEach(r=>{

let row=[];

if(r.c){

r.c.forEach(c=>{
row.push(c && c.v ? c.v : "");
});

}

data.push(row);

});

return data;

}



/* TABLE RENDER */

function renderTable(data){

let html="<table>";

let hasData=false;

data.forEach(row=>{

let empty=row.every(cell=>cell==="");

if(!empty){

hasData=true;

html+="<tr>";

row.forEach(cell=>{

html+="<td>"+cell+"</td>";

});

html+="</tr>";

}

});

html+="</table>";

if(hasData) return html;

return "";

}



/* LOAD STATION */

async function loadStation(st){

let html=`<h2>${st} Station</h2><div class="grid">`;

let officer=await getRange(st,"B3:F7");
let axle=await getRange(st,"B9:C12");
let ufsbi=await getRange(st,"E9:F11");
let ips=await getRange(st,"H9:I13");
let eld=await getRange(st,"B15:C16");
let fire=await getRange(st,"E15:F16");

let sections=[

["Officers",officer],
["Axle Counter",axle],
["UFSBI",ufsbi],
["IPS",ips],
["ELD",eld],
["Fire Alarm",fire]

];

sections.forEach(sec=>{

let table=renderTable(sec[1]);

if(table!=""){

html+=`

<div class="card">

<h3>${sec[0]}</h3>

${table}

</div>

`;

}

});

html+="</div>";

document.getElementById("stationContent").innerHTML=html;

}

</script>

</body>

</html>
