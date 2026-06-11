<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Invertebrate Phylum Guesser</title>

<style>
body {
    font-family: Arial, sans-serif;
    background: #121212;
    color: white;
    text-align: center;
    margin: 0;
    padding: 20px;
}

h1 {
    color: #4CAF50;
}

.container {
    max-width: 1200px;
    margin: auto;
}

select, button {
    padding: 10px;
    font-size: 16px;
    margin: 10px;
    border-radius: 6px;
    border: none;
}

button {
    background: #4CAF50;
    color: white;
    cursor: pointer;
}

button:hover {
    background: #3e8e41;
}

table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 20px;
}

th, td {
    border: 1px solid #333;
    padding: 8px;
}

th {
    background: #222;
}

.match { background: #2e7d32; }
.partial { background: #c8a600; color: black; }
.no-match { background: #b71c1c; }

#message {
    margin-top: 20px;
    font-size: 22px;
    font-weight: bold;
}

#guessCounter {
    margin: 10px;
    font-size: 18px;
}

#stats {
    margin-top: 30px;
    background: #1e1e1e;
    padding: 15px;
    border-radius: 10px;
}
</style>
</head>

<body>

<div class="container">

<h1>Invertebrate Phylum Guesser</h1>

<p>Guess the hidden invertebrate phylum</p>

<p id="guessCounter">Guesses: 0</p>

<select id="guessSelect"></select>

<button onclick="makeGuess()">Guess</button>
<button onclick="startGame()">New Game</button>

<div id="message"></div>

<table>
<thead>
<tr>
<th>Phylum</th>
<th>Level</th>
<th>Symmetry</th>
<th>Segmentation</th>
<th>Digestive</th>
<th>Respiration</th>
<th>Circulation</th>
</tr>
</thead>
<tbody id="tableBody"></tbody>
</table>

<div id="stats">
<h2>📊 Stats</h2>
<p id="gamesPlayed"></p>
<p id="gamesWon"></p>
<p id="winRate"></p>
<p id="currentStreak"></p>
<p id="bestStreak"></p>
<p id="avgGuesses"></p>
</div>

</div>

<script>

const phyla = [
{
name:"Phoronida",
level:"Organ system",
symmetry:"Bilateral",
seg:"No",
digest:"Complete",
resp:"Body surface",
circ:"Closed"
},
{
name:"Bryozoa",
level:"Organ system",
symmetry:"Bilateral",
seg:"No",
digest:"Complete",
resp:"Body surface",
circ:"None"
},
{
name:"Brachiopoda",
level:"Organ system",
symmetry:"Bilateral",
seg:"No",
digest:"Complete/Incomplete",
resp:"Body surface",
circ:"Open"
},
{
name:"Echinodermata",
level:"Organ system",
symmetry:"Radial",
seg:"No",
digest:"Complete",
resp:"Body surface",
circ:"None"
},
{
name:"Hemichordata",
level:"Organ system",
symmetry:"Bilateral",
seg:"Reduced",
digest:"Complete",
resp:"Body surface",
circ:"Part open/Closed"
},
{
name:"Mollusca",
level:"Organ system",
symmetry:"Bilateral",
seg:"No",
digest:"Complete",
resp:"Gills/Lungs",
circ:"Open"
},
{
name:"Arthropoda",
level:"Organ system",
symmetry:"Bilateral",
seg:"Yes",
digest:"Complete",
resp:"Tracheae/Gills/Book lungs",
circ:"Open"
}
];

let answer;
let guesses = 0;

let stats = JSON.parse(localStorage.getItem("stats")) || {
played:0,
won:0,
streak:0,
best:0,
totalGuesses:0
};

function startGame() {

answer = phyla[Math.floor(Math.random()*phyla.length)];

guesses = 0;
document.getElementById("guessCounter").innerText = "Guesses: 0";
document.getElementById("message").innerText = "";
document.getElementById("tableBody").innerHTML = "";

populateDropdown();
updateStats();
}

function populateDropdown() {

let select = document.getElementById("guessSelect");
select.innerHTML = "";

phyla.forEach(p => {
let opt = document.createElement("option");
opt.value = p.name;
opt.textContent = p.name;
select.appendChild(opt);
});
}

function compare(val1, val2) {
if(val1 === val2) return "match";

if(val1.includes("Complete") && val2.includes("Complete"))
return "partial";

if(val1.includes("Body surface") && val2.includes("Body surface"))
return "partial";

if(val1.includes("Part") || val2.includes("Part"))
return "partial";

return "nomatch";
}

function cell(text, type) {
let td = document.createElement("td");
td.textContent = text;

if(type === "match") td.className="match";
else if(type === "partial") td.className="partial";
else td.className="no-match";

return td;
}

function makeGuess() {

let name = document.getElementById("guessSelect").value;
let guess = phyla.find(p => p.name === name);

guesses++;
document.getElementById("guessCounter").innerText =
"Guesses: " + guesses;

let row = document.createElement("tr");

row.appendChild(document.createElement("td")).textContent = guess.name;

row.appendChild(cell(guess.level, compare(guess.level, answer.level)));
row.appendChild(cell(guess.symmetry, compare(guess.symmetry, answer.symmetry)));
row.appendChild(cell(guess.seg, compare(guess.seg, answer.seg)));
row.appendChild(cell(guess.digest, compare(guess.digest, answer.digest)));
row.appendChild(cell(guess.resp, compare(guess.resp, answer.resp)));
row.appendChild(cell(guess.circ, compare(guess.circ, answer.circ)));

document.getElementById("tableBody").appendChild(row);

if(guess.name === answer.name) {
win();
}
}

function win() {

stats.played++;
stats.won++;
stats.streak++;
if(stats.streak > stats.best) stats.best = stats.streak;
stats.totalGuesses += guesses;

localStorage.setItem("stats", JSON.stringify(stats));

document.getElementById("message").innerText =
"🎉 You got it in " + guesses + " guesses! Answer: " + answer.name;

updateStats();
}

function updateStats() {

document.getElementById("gamesPlayed").innerText =
"Games: " + stats.played;

document.getElementById("gamesWon").innerText =
"Wins: " + stats.won;

document.getElementById("winRate").innerText =
"Win %: " + (stats.played ? Math.round(stats.won/stats.played*100) : 0);

document.getElementById("currentStreak").innerText =
"Streak: " + stats.streak;

document.getElementById("bestStreak").innerText =
"Best: " + stats.best;

document.getElementById("avgGuesses").innerText =
"Avg guesses: " + (stats.won ? (stats.totalGuesses/stats.won).toFixed(1) : 0);
}

startGame();

</script>

</body>
</html>
