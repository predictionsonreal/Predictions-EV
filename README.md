# Predictions-EV
EV tracker for Predictions on Real - Sports
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Real App Daily Recap</title>

<style>
body { font-family: Arial, sans-serif; padding: 20px; }
table { border-collapse: collapse; margin-bottom: 15px; }
th, td { border: 1px solid #ccc; padding: 6px; text-align: center; }
input { width: 90px; }
button { padding: 8px 12px; margin-right: 8px; }
.output { white-space: pre-line; font-family: monospace; background: #f4f4f4; padding: 12px; margin-top: 15px; }
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
  <td><input></td>
  <td><input></td>
  <td><input></td>
  <td><input></td>
  <td><input></td>
  <td><input></td>
</tr>
</table>

<button onclick="addRow()">Add Bet</button>
<button onclick="generate()">Generate</button>

<div class="output" id="output"></div>

<script>
function addRow() {
  const table = document.getElementById("bets");
  const row = table.insertRow();
  for (let i = 0; i < 6; i++) {
    const cell = row.insertCell();
    const input = document.createElement("input");
    if(i===5) input.value="W";
    cell.appendChild(input);
  }
}

// Converts odds to implied market % (like Real app)
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
    const edge = market - real;
    const buyMax = market - 5;

    let profit = 0;
    if (result === "W") {
      if(payoutInput) {
        profit = Number(payoutInput) - rax;
      } else {
        // Calculate automatically
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

    // Per-bet format
    betBlocks +=
`Team: ${team}
Bought: ${(rax/1000).toFixed(0)}k
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
</script>

</body>
</html>
