# Predictions-EV
EV tracker for Predictions on Real - Sports
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Daily Recap Generator</title>

<style>
body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: #f0f4f8;
    color: #1a1a1a;
    padding: 20px;
}

h2 {
    color: #2c3e50;
}

table {
    border-collapse: collapse;
    width: 100%;
    margin-bottom: 15px;
    background-color: #ffffff;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}

th, td {
    border: 1px solid #ddd;
    padding: 10px;
    text-align: center;
}

th {
    background-color: #4a90e2;
    color: white;
    font-size: 14px;
}

input {
    width: 90px;
    padding: 5px;
    border-radius: 4px;
    border: 1px solid #ccc;
}

button {
    padding: 10px 16px;
    margin-right: 10px;
    border-radius: 6px;
    border: none;
    background-color: #4a90e2;
    color: white;
    font-weight: bold;
    cursor: pointer;
    transition: background-color 0.2s;
}

button:hover {
    background-color: #357ABD;
}

.output {
    white-space: pre-line;
    font-family: monospace;
    background-color: #e8f0fe;
    padding: 15px;
    border-radius: 8px;
    border: 1px solid #bbb;
    margin-top: 15px;
}
</style>
</head>

<body>
<h2>Daily Recap Generator</h2>

<table id="bets">
<tr>
  <th>Team</th>
  <th>Real %</th>
  <th>Market Odds</th>
  <th>RAX</th>
  <th>Payout (optional)</th>
  <th>Result (W/L)</th>
</tr>
<tr>
  <td><input placeholder="Team"></td>
  <td><input placeholder="Real %"></td>
  <td><input placeholder="Odds"></td>
  <td><input placeholder="RAX"></td>
  <td><input placeholder="Payout"></td>
  <td><input value="W"></td>
</tr>
</table>

<button onclick="addRow()">Add Bet</button>
<button onclick="generate()">Generate Recap</button>
<button onclick="copyOutput()">Copy All</button>

<div class="output" id="output"></div>

<script>
function addRow() {
  const table = document.getElementById("bets");
  const row = table.insertRow();
  for (let i = 0; i < 6; i++) {
    const cell = row.insertCell();
    const input = document.createElement("input");
    if(i===5) input.value="W";
    if(i===0) input.placeholder="Team";
    if(i===1) input.placeholder="Real %";
    if(i===2) input.placeholder="Odds";
    if(i===3) input.placeholder="RAX";
    if(i===4) input.placeholder="Payout";
    cell.appendChild(input);
  }
}

// Convert odds to implied market %
function oddsToPercent(odds) {
  odds = Number(odds);
  if (odds < 0) return Math.abs(odds) / (Math.abs(odds)+100) * 100;
  return 100 / (odds+100) * 100;
}

function generate() {
  const table = document.getElementById("bets");
  let totalRisk = 0, totalProfit = 0, totalEdge = 0;
  let wins = 0, losses = 0;

  let betBlocks = "";
  let recapLines = "";

  for (let i = 1; i < table.rows.length; i++) {
    const cells = table.rows[i].cells;
    const team = cells[0].children[0].value;
    const real = Number(cells[1].children[0].value);
    const odds = Number(cells[2].children[0].value);
    const rax = Number(cells[3].children[0].value);
    const payoutInput = cells[4].children[0].value;
    const result = cells[5].children[0].value.toUpperCase();

    if (!team) continue;

    const market = oddsToPercent(odds);
    const edge = real - market; // Edge = Real % - Market %
    const buyMax = market - 5;

    let profit = 0;
    if (result === "W") {
      if(payoutInput) {
        profit = Number(payoutInput) - rax;
      } else {
        profit = (odds < 0 ? rax * (100/Math.abs(odds)) : rax * (odds/100));
      }
      wins++;
    } else {
      profit = -rax;
      losses++;
    }

    totalRisk += rax;
    totalProfit += profit;
    totalEdge += edge;

    // Per-bet block
    betBlocks +=
`Team: ${team}
Bought: ${rax.toLocaleString()} RAX
Real: (${real}%)
Market: (${odds}) (${market.toFixed(1)}%)
Edge: (+${edge.toFixed(1)}%)
Buy ≤ (${buyMax.toFixed(1)}%)
Result: ${result === "W" ? "✅" : "❌"}

`;

    recapLines += `${team} ML (${odds}): ${result === "W" ? "✅" : "❌"}\n`;
  }

  const actualReturn = (totalProfit/totalRisk)*100;
  const luck = actualReturn - totalEdge;

  document.getElementById("output").textContent =
betBlocks +
"-----------------\n" +
recapLines +
"-----------------\n" +
`Record: ${wins}-${losses}
Total Risk: ${totalRisk.toLocaleString()} RAX
Profit: ${totalProfit.toFixed(0)} RAX
Expected Return: +${totalEdge.toFixed(1)}%
Actual Return: ${actualReturn.toFixed(1)}%
Luck: ${luck.toFixed(1)}%`;
}

// Copy to clipboard
function copyOutput() {
  const output = document.getElementById("output");
  navigator.clipboard.writeText(output.textContent).then(() => {
    alert("Daily Recap copied to clipboard!");
  });
}
</script>

</body>
</html>
