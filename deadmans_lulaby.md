<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>ARCADE 360</title>
  <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
  <style>
    body {
      margin: 0;
      padding: 0;
      font-family: 'Press Start 2P', monospace;
      background: linear-gradient(to bottom, #001f3f, #0074D9);
      color: white;
      text-align: center;
    }

    h1, #score, #level, .overlay-text {
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
      background-color: limegreen;
      border-radius: 50%;
      background-image: url('FROG_SPRITE_URL'); /* Reemplaza por tu sprite */
      background-size: cover;
    }

    .obstacle {
      position: absolute;
      width: 40px;
      height: 40px;
      background-color: red;
      background-image: url('CAR_SPRITE_URL'); /* Reemplaza por tu sprite */
      background-size: cover;
      left: 600px;
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
  Tu navegador no soporta el audio.
</audio>

<div id="score">PUNTUACIÓN: 0</div>
<div id="level">NIVEL: 1</div>

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

  let score = 0;
  let level = 1;
  let lanes = 5;
  let laneHeight = game.clientHeight / 10;
  let currentLane = 0;
  let spawnInterval;
  let gameRunning = true;
  let gamePaused = false;
  let obstacles = [];

  function updateLaneBackground() {
    let laneColors = [];
    for (let i = 0; i < lanes; i++) {
      const percentTop = ((i + 1) / lanes) * 100;
      const color = i < 5 ? "#111" : "#FFD700";
      laneColors.push(`${color} ${100 - percentTop}%`);
    }
    game.style.background = `linear-gradient(to top, ${laneColors.join(", ")})`;
  }

  function resetGame() {
    score = 0;
    level = 1;
    lanes = 5;
    currentLane = 0;
    frog.style.top = `${game.clientHeight - laneHeight}px`;
    scoreDisplay.textContent = `PUNTUACIÓN: ${score}`;
    levelDisplay.textContent = `NIVEL: ${level}`;
    gameOverText.style.display = "none";
    pauseText.style.display = "none";
    gameRunning = true;
    gamePaused = false;
    clearObstacles();
    updateLaneBackground();
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

    if (gamePaused) {
      pauseText.style.display = "block";
      clearInterval(spawnInterval);
      obstacles.forEach(obj => clearInterval(obj.interval));
    } else {
      pauseText.style.display = "none";
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
          if (score % 10 === 0 && lanes < 10) {
            level++;
            lanes++;
            levelDisplay.textContent = `NIVEL: ${level}`;
            updateLaneBackground();
          }
        }
      }
    }, 20);
  }

  function startSpawning() {
    spawnInterval = setInterval(spawnObstacle, Math.max(500, 1800 - level * 100));
  }

  // Iniciar música tras interacción del usuario
  function tryPlayMusic() {
    bgMusic.muted = false;
    bgMusic.play().catch(() => {});
  }

  document.addEventListener("keydown", e => {
    tryPlayMusic();

    if (e.code === "ArrowUp" && gameRunning && !gamePaused) moveFrogUp();
    else if (e.code === "ArrowDown" && gameRunning && !gamePaused) moveFrogDown();
    else if (e.code === "KeyP") togglePause();
    else if (e.code === "KeyR") resetGame();
  });

  window.addEventListener("click", tryPlayMusic);

  // Inicio
  frog.style.top = `${game.clientHeight - laneHeight}px`;
  updateLaneBackground();
  startSpawning();
</script>

</body>
</html>
