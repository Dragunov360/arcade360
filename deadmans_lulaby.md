<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>ARCADE 360</title>
  <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
  <style>
    body {
      margin: 0;
      font-family: 'Press Start 2P', monospace;
      background: linear-gradient(to bottom, #001f3f, #0074D9);
      color: white;
      text-align: center;
    }

    h1, #score, #level, .overlay-text, #audio-message {
      animation: glitch 1s infinite;
      text-shadow: 2px 2px #ff00ff, -2px -2px #00ffff;
    }

    h1 {
      margin: 20px;
      font-size: 32px;
    }

    #game {
      position: relative;
      margin: 0 auto;
      width: 600px;
      height: 400px;
      overflow: hidden;
      border: 5px solid #fff;
      background: #111;
    }

    #frog {
      position: absolute;
      bottom: 0;
      left: 50px;
      width: 40px;
      height: 40px;
      background-image: url('rfami.png');
      background-size: contain;
      background-repeat: no-repeat;
      background-position: center;
    }

    .obstacle {
      position: absolute;
      width: 40px;
      height: 40px;
      background-color: red;
      background-size: cover;
      background-position: center;
    }

    #score, #level {
      font-size: 14px;
      margin-top: 10px;
    }

    .overlay-text {
      display: none;
      position: absolute;
      width: 100%;
      text-align: center;
      font-size: 24px;
      top: 40%;
      z-index: 10;
      color: yellow;
    }

    #pause-text {
      color: cyan;
    }

    #audio-message {
      margin-top: 10px;
      font-size: 14px;
      color: #ffcc00;
    }

    @keyframes glitch {
      0% { transform: translate(0); opacity: 1; }
      20% { transform: translate(1px, -1px); }
      40% { transform: translate(-1px, 1px); }
      60% { transform: translate(1px, 1px); opacity: 0.9; }
      80% { transform: translate(-1px, -1px); opacity: 1; }
      100% { transform: translate(0); }
    }
  </style>
</head>
<body>

<h1>ARCADE 360</h1>

<audio id="bg-music" loop autoplay muted>
  <source src="Call of Duty_ Black Ops - Dead Ops Arcade song _Clockwork Squares_ James McCawley.mp3" type="audio/mpeg">
</audio>

<div id="score">PUNTUACIÓN: 0</div>
<div id="level">NIVEL: 1</div>
<div id="audio-message">🎵 Haz clic o presiona una tecla para activar el sonido</div>

<div id="game">
  <div id="frog"></div>
  <div id="game-over" class="overlay-text">¡GAME OVER!</div>
  <div id="pause-text" class="overlay-text">⏸️ PAUSA</div>
</div>

<script>
  const frog = document.getElementById("frog");
  const game = document.getElementById("game");
  const gameOverText = document.getElementById("game-over");
  const pauseText = document.getElementById("pause-text");
  const scoreDisplay = document.getElementById("score");
  const levelDisplay = document.getElementById("level");
  const bgMusic = document.getElementById("bg-music");
  const audioMessage = document.getElementById("audio-message");

  let score = 0;
  let level = 1;
  const lanes = 5;
  let laneHeight = game.clientHeight / lanes;
  let currentLane = 0;
  let spawnInterval;
  let gameRunning = true;
  let gamePaused = false;
  let obstacles = [];

  function resetGame() {
    score = 0;
    level = 1;
    currentLane = 0;
    frog.style.top = `${game.clientHeight - laneHeight}px`;
    scoreDisplay.textContent = `PUNTUACIÓN: ${score}`;
    levelDisplay.textContent = `NIVEL: ${level}`;
    gameOverText.style.display = "none";
    pauseText.style.display = "none";
    gameRunning = true;
    gamePaused = false;
    clearObstacles();
    startSpawning();
  }

  function clearObstacles() {
    obstacles.forEach(obj => clearInterval(obj.interval));
    document.querySelectorAll(".obstacle").forEach(ob => ob.remove());
    obstacles = [];
  }

  function moveFrogUp() {
    if (currentLane < lanes - 1) {
      currentLane++;
      frog.style.top = `${game.clientHeight - laneHeight * (currentLane + 1)}px`;
    }
  }

  function moveFrogDown() {
    if (currentLane > 0) {
      currentLane--;
      frog.style.top = `${game.clientHeight - laneHeight * (currentLane + 1)}px`;
    }
  }

  function togglePause() {
    if (!gameRunning) return;
    gamePaused = !gamePaused;
    pauseText.style.display = gamePaused ? "block" : "none";

    if (gamePaused) {
      clearInterval(spawnInterval);
      obstacles.forEach(obj => clearInterval(obj.interval));
    } else {
      startSpawning();
      obstacles.forEach(obj => startObstacleMovement(obj));
    }
  }

  function spawnObstacle() {
    if (!gameRunning || gamePaused) return;

    const obstacle = document.createElement("div");
    obstacle.classList.add("obstacle");

    const randomLane = Math.floor(Math.random() * lanes);
    const topPosition = game.clientHeight - laneHeight * (randomLane + 1);
    obstacle.style.top = `${topPosition}px`;
    game.appendChild(obstacle);

    const objData = { element: obstacle, left: game.clientWidth, interval: null };
    obstacles.push(objData);
    startObstacleMovement(objData);
  }

  function startObstacleMovement(obj) {
    const speed = 3 + level * 1.5;
    obj.interval = setInterval(() => {
      if (!gameRunning || gamePaused) return;

      obj.left -= speed;
      obj.element.style.left = `${obj.left}px`;

      const frogRect = frog.getBoundingClientRect();
      const obsRect = obj.element.getBoundingClientRect();
      if (
        obsRect.left < frogRect.right &&
        obsRect.right > frogRect.left &&
        obsRect.top < frogRect.bottom &&
        obsRect.bottom > frogRect.top
      ) {
        gameOverText.style.display = "block";
        pauseText.style.display = "none";
        gameRunning = false;
        clearInterval(spawnInterval);
        clearInterval(obj.interval);
        setTimeout(resetGame, 3000);
        return;
      }

      if (obj.left + obj.element.offsetWidth < 0) {
        obj.element.remove();
        clearInterval(obj.interval);
        obstacles = obstacles.filter(o => o !== obj);
        if (gameRunning) {
          score++;
          scoreDisplay.textContent = `PUNTUACIÓN: ${score}`;
          if (score % 10 === 0) {
            level++;
            levelDisplay.textContent = `NIVEL: ${level}`;
          }
        }
      }
    }, 20);
  }

  function startSpawning() {
    spawnInterval = setInterval(spawnObstacle, Math.max(500, 1800 - level * 100));
  }

  function tryPlayMusic() {
    if (bgMusic.muted) {
      bgMusic.muted = false;
      bgMusic.play().catch(() => {});
      audioMessage.style.display = "none";
    }
  }

  document.addEventListener("keydown", e => {
    tryPlayMusic();
    if (e.code === "ArrowUp" && gameRunning && !gamePaused) moveFrogUp();
    else if (e.code === "ArrowDown" && gameRunning && !gamePaused) moveFrogDown();
    else if (e.code === "KeyP") togglePause();
    else if (e.code === "KeyR") resetGame();
  });

  window.addEventListener("click", tryPlayMusic);

  frog.style.top = `${game.clientHeight - laneHeight}px`;
  startSpawning();
</script>

</body>
</html>


