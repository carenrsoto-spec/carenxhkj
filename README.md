<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Software Classification Game</title>
  <style>
    * {
      box-sizing: border-box;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }

    body {
      margin: 0;
      padding: 0;
      background: linear-gradient(135deg, #1e293b, #0f172a);
      color: #f8fafc;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      overflow: hidden;
    }

    #game-container {
      position: relative;
      width: 800px;
      height: 600px;
      background-color: #020617;
      border: 3px solid #38bdf8;
      border-radius: 12px;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
      overflow: hidden;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
    }

    /* Header & Stats */
    #header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 15px 25px;
      background: rgba(30, 41, 59, 0.8);
      border-bottom: 2px solid #1e293b;
      z-index: 10;
    }

    .stat-box {
      font-size: 1.1rem;
      font-weight: bold;
    }

    .stat-value {
      color: #38bdf8;
    }

    /* Play Area */
    #play-area {
      position: relative;
      flex-grow: 1;
      width: 100%;
    }

    /* Falling Card */
    .falling-card {
      position: absolute;
      width: 220px;
      padding: 15px;
      background: #1e293b;
      border: 2px solid #38bdf8;
      border-radius: 8px;
      text-align: center;
      box-shadow: 0 4px 12px rgba(56, 189, 248, 0.3);
      left: 50%;
      transform: translateX(-50%);
      transition: top 0.05s linear;
    }

    .card-title {
      font-weight: bold;
      font-size: 1.1rem;
      color: #f1f5f9;
      margin-bottom: 5px;
    }

    .card-desc {
      font-size: 0.85rem;
      color: #94a3b8;
    }

    /* Buckets Footer */
    #buckets-container {
      display: flex;
      justify-content: space-around;
      padding: 15px;
      background: rgba(15, 23, 42, 0.9);
      border-top: 2px solid #1e293b;
      z-index: 10;
    }

    .bucket-btn {
      flex: 1;
      margin: 0 8px;
      padding: 15px 10px;
      background: #0f172a;
      border: 2px solid #334155;
      border-radius: 8px;
      color: #f8fafc;
      font-size: 0.95rem;
      font-weight: bold;
      cursor: pointer;
      transition: all 0.2s ease;
      text-align: center;
    }

    .bucket-btn:hover {
      background: #1e293b;
      border-color: #38bdf8;
      transform: translateY(-2px);
    }

    .bucket-btn:active {
      transform: translateY(0);
    }

    /* Overlay Screens */
    .overlay {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(2, 6, 23, 0.92);
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      text-align: center;
      padding: 30px;
      z-index: 20;
    }

    .overlay h1 {
      font-size: 2.2rem;
      color: #38bdf8;
      margin-bottom: 15px;
    }

    .overlay p {
      font-size: 1rem;
      color: #cbd5e1;
      max-width: 500px;
      line-height: 1.5;
      margin-bottom: 25px;
    }

    .start-btn {
      padding: 12px 30px;
      font-size: 1.1rem;
      font-weight: bold;
      color: #0f172a;
      background-color: #38bdf8;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      transition: background 0.2s;
    }

    .start-btn:hover {
      background-color: #7dd3fc;
    }

    .hidden {
      display: none !important;
    }
  </style>
</head>
<body>

  <div id="game-container">
    <div id="header">
      <div class="stat-box">Score: <span id="score-val" class="stat-value">0</span></div>
      <div class="stat-box">Lives: <span id="lives-val" class="stat-value">3</span></div>
    </div>

    <div id="play-area">
      </div>

    <div id="buckets-container">
      <button class="bucket-btn" onclick="checkAnswer('System')">
        1. System Software
      </button>
      <button class="bucket-btn" onclick="checkAnswer('Application')">
        2. Application Software
      </button>
      <button class="bucket-btn" onclick="checkAnswer('Specialized')">
        3. Specialized Software
      </button>
    </div>

    <div id="start-screen" class="overlay">
      <h1>Software Type Catcher</h1>
      <p>Items representing different types of software will drop from the top. Click the correct category bucket at the bottom before they hit the ground!</p>
      <button class="start-btn" onclick="startGame()">Start Game</button>
    </div>

    <div id="game-over-screen" class="overlay hidden">
      <h1>Game Over!</h1>
      <p>Final Score: <span id="final-score" style="color: #38bdf8; font-weight: bold;">0</span></p>
      <button class="start-btn" onclick="startGame()">Play Again</button>
    </div>
  </div>

  <script>
    // Game Data based on provided PDF module
    const softwareItems = [
      // System Software
      { name: "Operating System (OS)", category: "System", desc: "Manages hardware resources like CPU, memory, and storage." },
      { name: "Device Drivers", category: "System", desc: "Allows OS to communicate with printers and graphics cards." },
      { name: "Language Processors", category: "System", desc: "Converts high-level code into machine code (e.g. Java, C++)." },
      { name: "Firmware", category: "System", desc: "Embedded software controlling hardware in routers/cameras." },

      // Application Software
      { name: "Desktop Apps (Word, Excel)", category: "Application", desc: "Programs installed locally for end-user productivity." },
      { name: "Mobile Apps (Instagram)", category: "Application", desc: "Apps designed specifically for smartphones and tablets." },
      { name: "Web Apps (Google Docs)", category: "Application", desc: "Software accessed directly via web browsers." },
      { name: "Enterprise Software (ERP)", category: "Application", desc: "Supports large organizational operations and CRM." },
      { name: "Cloud-Based SaaS (Slack)", category: "Application", desc: "Accessible online without local installation." },

      // Specialized Software
      { name: "Programming Software (IDEs)", category: "Specialized", desc: "Compilers and IDEs (VS Code) for writing/testing code." },
      { name: "DBMS (MySQL, Oracle)", category: "Specialized", desc: "Dedicated software for storing and managing data." },
      { name: "Embedded Systems", category: "Specialized", desc: "Software integrated into washing machines or automotive systems." },
      { name: "AI Software", category: "Specialized", desc: "Programs running machine learning and NLP." },
      { name: "Simulation Software", category: "Specialized", desc: "Virtual models used for training in aviation and healthcare." }
    ];

    let score = 0;
    let lives = 3;
    let currentItem = null;
    let gameInterval = null;
    let cardPosY = 0;
    let fallSpeed = 2; // Speed multiplier
    const maxDropHeight = 380; // Height threshold before hitting bottom

    function startGame() {
      score = 0;
      lives = 3;
      fallSpeed = 2;
      updateUI();

      document.getElementById('start-screen').classList.add('hidden');
      document.getElementById('game-over-screen').classList.add('hidden');

      spawnCard();
      if (gameInterval) clearInterval(gameInterval);
      gameInterval = setInterval(updateGame, 20);
    }

    function spawnCard() {
      const playArea = document.getElementById('play-area');
      playArea.innerHTML = ''; // Clear existing card

      // Pick random item
      const randomIndex = Math.floor(Math.random() * softwareItems.length);
      currentItem = softwareItems[randomIndex];

      cardPosY = 0;

      const card = document.createElement('div');
      card.className = 'falling-card';
      card.id = 'active-card';
      card.style.top = cardPosY + 'px';
      card.innerHTML = `
        <div class="card-title">${currentItem.name}</div>
        <div class="card-desc">${currentItem.desc}</div>
      `;

      playArea.appendChild(card);
    }

    function updateGame() {
      const card = document.getElementById('active-card');
      if (!card) return;

      cardPosY += fallSpeed;
      card.style.top = cardPosY + 'px';

      // Check if card hit bottom
      if (cardPosY >= maxDropHeight) {
        handleMiss();
      }
    }

    function checkAnswer(selectedCategory) {
      if (!currentItem) return;

      if (selectedCategory === currentItem.category) {
        score += 10;
        // Slightly increase speed as score increases to ramp up difficulty
        fallSpeed += 0.2; 
      } else {
        lives--;
      }

      updateUI();

      if (lives <= 0) {
        endGame();
      } else {
        spawnCard();
      }
    }

    function handleMiss() {
      lives--;
      updateUI();

      if (lives <= 0) {
        endGame();
      } else {
        spawnCard();
      }
    }

    function updateUI() {
      document.getElementById('score-val').innerText = score;
      document.getElementById('lives-val').innerText = lives;
    }

    function endGame() {
      clearInterval(gameInterval);
      document.getElementById('play-area').innerHTML = '';
      document.getElementById('final-score').innerText = score;
      document.getElementById('game-over-screen').classList.remove('hidden');
    }
  </script>
</body>
</html>
