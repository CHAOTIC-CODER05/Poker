[p0k3r.html](https://github.com/user-attachments/files/28978196/p0k3r.html)


<!DOCTYPE html>
<html>
<head>
<title>Poker Draw Game</title>
<style>
    body {
        background:#1b1b1b;
        color:white;
        font-family:Arial;
        text-align:center;
        padding:20px;
    }
    #gameBox {
        width:600px;
        margin:auto;
        background:#333;
        padding:20px;
        border-radius:10px;
    }
    .card {
        width:80px;
        height:120px;
        background:white;
        color:black;
        border-radius:8px;
        display:flex;
        justify-content:center;
        align-items:center;
        font-size:24px;
        cursor:pointer;
        margin:5px;
        border:3px solid transparent;
    }
    .held {
        border:3px solid gold;
    }
    #cards {
        display:flex;
        justify-content:center;
        margin:20px 0;
    }
    button {
        padding:10px 20px;
        margin:10px;
        font-size:18px;
        border:none;
        border-radius:6px;
        cursor:pointer;
    }
    .shopBtn { background:#2196f3; color:white; }
    .drawBtn { background:#4caf50; color:white; }
</style>
</head>
<body>

<h1>Poker Draw Game</h1>

<div id="gameBox">
    <p id="stats"></p>
    <div id="cards"></div>
    <button class="drawBtn" onclick="drawCards()">Draw</button>
    <button class="shopBtn" onclick="openShop()">Shop</button>
    <p id="message"></p>
</div>

<script>
let coins = 50;
let luck = 1;
let level = 1;

let deck = [];
let hand = [];
let held = [false, false, false, false, false];
let drawPhase = 1;

function updateStats() {
    document.getElementById("stats").innerHTML =
        "Coins: " + coins + "<br>" +
        "Luck Level: " + level + "<br>" +
        "Luck Multiplier: x" + luck;
}

function createDeck() {
    let suits = ["♠","♥","♦","♣"];
    let values = ["A","2","3","4","5","6","7","8","9","10","J","Q","K"];
    deck = [];
    for (let s of suits) {
        for (let v of values) {
            deck.push({value:v, suit:s});
        }
    }
}

function shuffle() {
    for (let i = deck.length - 1; i > 0; i--) {
        let j = Math.floor(Math.random() * (i + 1));
        [deck[i], deck[j]] = [deck[j], deck[i]];
    }
}

function dealHand() {
    hand = [];
    for (let i = 0; i < 5; i++) {
        hand.push(deck.pop());
    }
}

function renderHand() {
    let box = document.getElementById("cards");
    box.innerHTML = "";
    for (let i = 0; i < 5; i++) {
        let c = document.createElement("div");
        c.className = "card";
        if (held[i]) c.classList.add("held");
        c.innerHTML = hand[i].value + "<br>" + hand[i].suit;
        c.onclick = () => toggleHold(i);
        box.appendChild(c);
    }
}

function toggleHold(i) {
    if (drawPhase === 2) return;
    held[i] = !held[i];
    renderHand();
}

function drawCards() {
    if (drawPhase === 1) {
        createDeck();
        shuffle();
        dealHand();
        renderHand();
        drawPhase = 2;
        document.getElementById("message").innerHTML = "Select cards to hold, then press Draw again.";
    } else {
        for (let i = 0; i < 5; i++) {
            if (!held[i]) hand[i] = deck.pop();
        }
        renderHand();
        scoreHand();
        held = [false, false, false, false, false];
        drawPhase = 1;
    }
}

function scoreHand() {
    let values = hand.map(c => c.value);
    let suits = hand.map(c => c.suit);

    let counts = {};
    for (let v of values) counts[v] = (counts[v] || 0) + 1;

    let isFlush = suits.every(s => s === suits[0]);

    let order = ["A","2","3","4","5","6","7","8","9","10","J","Q","K"];
    let idx = values.map(v => order.indexOf(v)).sort((a,b)=>a-b);
    let isStraight = idx.every((v,i)=> i===0 || v === idx[i-1]+1);

    let reward = 0;
    let msg = "";

    let pairs = Object.values(counts).filter(c => c === 2).length;
    let threes = Object.values(counts).includes(3);
    let fours = Object.values(counts).includes(4);

    if (isStraight && isFlush) { reward = 50; msg = "Straight Flush!"; }
    else if (fours) { reward = 25; msg = "Four of a Kind!"; }
    else if (threes && pairs === 1) { reward = 15; msg = "Full House!"; }
    else if (isFlush) { reward = 10; msg = "Flush!"; }
    else if (isStraight) { reward = 8; msg = "Straight!"; }
    else if (threes) { reward = 5; msg = "Three of a Kind!"; }
    else if (pairs === 2) { reward = 3; msg = "Two Pair!"; }
    else if (pairs === 1) { reward = 1; msg = "One Pair!"; }
    else { msg = "No hand."; }

    reward *= luck;
    coins += reward;

    document.getElementById("message").innerHTML =
        msg + " You earned " + reward + " coins.";

    updateStats();
}

function openShop() {
    let cost = level * 30;
    if (coins >= cost) {
        coins -= cost;
        level++;
        luck++;
        document.getElementById("message").innerHTML =
            "🛒 Luck upgraded! Your winnings are now multiplied more.";
    } else {
        document.getElementById("message").innerHTML =
            "❌ Need " + cost + " coins to upgrade.";
    }
    updateStats();
}

updateStats();
</script>

</body>
</html>
