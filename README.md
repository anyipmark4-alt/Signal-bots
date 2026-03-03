<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PowerSignal Institutional Trading Bot</title>

<script src="https://cdn.jsdelivr.net/npm/technicalindicators@3.0.0/dist/browser.js"></script>
<script src="https://s3.tradingview.com/tv.js"></script>

<style>
body{
    margin:0;
    font-family:Arial;
    background:#0f172a;
    color:white;
}

header{
    background:#111827;
    padding:15px;
    text-align:center;
    font-size:22px;
    font-weight:bold;
}

.container{padding:20px;}

.card{
    background:#1e293b;
    padding:20px;
    border-radius:10px;
    margin-bottom:20px;
}

.buy{color:#00ff88;font-weight:bold;}
.sell{color:#ff4d4d;font-weight:bold;}
.hold{color:#facc15;font-weight:bold;}

button{
    padding:10px 15px;
    background:#2563eb;
    border:none;
    color:white;
    border-radius:5px;
    cursor:pointer;
}

button:hover{background:#1d4ed8;}
</style>
</head>

<body>

<header>
🚀 PowerSignal Institutional AI Bot
</header>

<div class="container">

<div class="card">
<h2>Market Selection</h2>

<select id="market">
<option value="BTCUSDT">BTCUSDT (Crypto)</option>
<option value="ETHUSDT">ETHUSDT (Crypto)</option>
<option value="BNBUSDT">BNBUSDT (Crypto)</option>
</select>

<button onclick="analyze()">Analyze Market</button>
<button onclick="toggleAuto()">Toggle Auto Trading</button>

<p id="result">Waiting for analysis...</p>
</div>

<div id="chart"></div>

</div>

<script>

// TradingView Chart
let widget = new TradingView.widget({
    "width": "100%",
    "height": 500,
    "symbol": "BINANCE:BTCUSDT",
    "interval": "15",
    "theme": "dark",
    "container_id": "chart"
});

if (Notification.permission !== "granted") {
    Notification.requestPermission();
}

let autoTrading = false;

function toggleAuto(){
    autoTrading = !autoTrading;
    alert("Auto Trading: " + (autoTrading ? "ON" : "OFF"));
}

async function analyze(){

    const symbol = document.getElementById("market").value;

    const response = await fetch(
        `https://api.binance.com/api/v3/klines?symbol=${symbol}&interval=15m&limit=150`
    );

    const data = await response.json();
    const closes = data.map(c => parseFloat(c[4]));

    const rsi = window.technicalindicators.RSI.calculate({
        values: closes,
        period: 14
    });

    const emaFast = window.technicalindicators.EMA.calculate({
        values: closes,
        period: 9
    });

    const emaSlow = window.technicalindicators.EMA.calculate({
        values: closes,
        period: 21
    });

    const macd = window.technicalindicators.MACD.calculate({
        values: closes,
        fastPeriod: 12,
        slowPeriod: 26,
        signalPeriod: 9,
        SimpleMAOscillator: false,
        SimpleMASignal: false
    });

    const lastPrice = closes[closes.length-1];
    const lastRSI = rsi[rsi.length-1];
    const lastFast = emaFast[emaFast.length-1];
    const lastSlow = emaSlow[emaSlow.length-1];
    const lastMACD = macd[macd.length-1];

    let signal = "HOLD";

    if(lastRSI < 30 && lastFast > lastSlow && lastMACD.histogram > 0){
        signal = "BUY";
    }
    else if(lastRSI > 70 && lastFast < lastSlow && lastMACD.histogram < 0){
        signal = "SELL";
    }

    let sl = signal==="BUY" ? lastPrice*0.995 : lastPrice*1.005;
    let tp = signal==="BUY" ? lastPrice*1.01 : lastPrice*0.99;

    let className = signal.toLowerCase();

    document.getElementById("result").innerHTML =
        `<span class="${className}">${signal}</span>
        <br>Entry: ${lastPrice.toFixed(2)}
        <br>Stop Loss: ${sl.toFixed(2)}
        <br>Take Profit: ${tp.toFixed(2)}
        <br>RSI: ${lastRSI.toFixed(2)}`;

    if(signal !== "HOLD"){
        new Notification("New Signal!", {
            body: `${signal} ${symbol} at ${lastPrice}`
        });

        if(autoTrading){
            simulateTrade(signal, lastPrice, sl, tp);
        }
    }
}

function simulateTrade(signal, entry, sl, tp){
    console.log("Auto Trade Executed:");
    console.log(signal, entry, sl, tp);
}

setInterval(() => {
    if(autoTrading){
        analyze();
    }
}, 900000);

</script>

</body>
</html>
