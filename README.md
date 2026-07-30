<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Elicarlos // Terminal Snake</title>
<style>
  :root {
    --green: #00FF41;
    --green-dim: #0a5c1e;
    --bg: #0d1117;
  }
  * { box-sizing: border-box; }
  html, body {
    margin: 0;
    height: 100%;
    background: var(--bg);
    font-family: "Fira Code", "Courier New", monospace;
    color: var(--green);
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .wrap {
    text-align: center;
    padding: 20px;
    width: 100%;
    max-width: 520px;
  }
  h1 {
    font-size: 1.1rem;
    letter-spacing: 2px;
    margin: 0 0 4px;
    text-shadow: 0 0 8px var(--green);
  }
  .sub {
    font-size: 0.8rem;
    opacity: 0.7;
    margin-bottom: 16px;
  }
  .hud {
    display: flex;
    justify-content: space-between;
    font-size: 0.9rem;
    margin-bottom: 8px;
    padding: 0 4px;
  }
  #board {
    background: #000;
    border: 2px solid var(--green);
    box-shadow: 0 0 20px rgba(0, 255, 65, 0.35);
    image-rendering: pixelated;
    width: 100%;
    max-width: 480px;
    height: auto;
    aspect-ratio: 1 / 1;
  }
  .msg {
    margin-top: 14px;
    min-height: 1.4em;
    font-size: 0.95rem;
  }
  .keys {
    margin-top: 10px;
    font-size: 0.75rem;
    opacity: 0.6;
  }
  button {
    margin-top: 14px;
    background: transparent;
    color: var(--green);
    border: 1px solid var(--green);
    padding: 8px 18px;
    font-family: inherit;
    font-size: 0.85rem;
    cursor: pointer;
    border-radius: 4px;
  }
  button:hover {
    background: var(--green);
    color: #000;
  }
  .touch-controls {
    display: grid;
    grid-template-columns: repeat(3, 48px);
    grid-template-rows: repeat(2, 48px);
    gap: 6px;
    justify-content: center;
    margin-top: 16px;
  }
  .touch-controls button {
    margin: 0;
    padding: 0;
    font-size: 1.1rem;
  }
  @media (min-width: 620px) {
    .touch-controls { display: none; }
  }
  footer {
    margin-top: 20px;
    font-size: 0.7rem;
    opacity: 0.5;
  }
  a { color: var(--green); }
</style>
</head>
<body>
<div class="wrap">
  <h1>&lt; ELICARLOS // TERMINAL SNAKE /&gt;</h1>
  <div class="sub">transformando café em código, uma cobra de cada vez</div>

  <div class="hud">
    <span>SCORE: <b id="score">0</b></span>
    <span>BEST: <b id="best">0</b></span>
  </div>

  <canvas id="board" width="24" height="24"></canvas>

  <div class="msg" id="msg">pressione uma seta ou WASD para começar</div>

  <div>
    <button id="restart">reiniciar</button>
  </div>

  <div class="touch-controls">
    <span></span>
    <button data-dir="up">▲</button>
    <span></span>
    <button data-dir="left">◀</button>
    <button data-dir="down">▼</button>
    <button data-dir="right">▶</button>
  </div>

  <div class="keys">setas / WASD para mover · espaço para pausar</div>

  <footer>
    feito em HTML + CSS + JS puro ·
    <a href="./README.md">voltar ao perfil</a>
  </footer>
</div>

<script>
(function () {
  "use strict";

  var GRID = 24;
  var canvas = document.getElementById("board");
  var ctx = canvas.getContext("2d");
  var scoreEl = document.getElementById("score");
  var bestEl = document.getElementById("best");
  var msgEl = document.getElementById("msg");
  var restartBtn = document.getElementById("restart");

  var BEST_KEY = "elicarlos_snake_best";
  var best = 0;
  try {
    best = parseInt(window.localStorage.getItem(BEST_KEY), 10) || 0;
  } catch (e) {
    best = 0;
  }
  bestEl.textContent = best;

  var snake, dir, nextDir, food, score, alive, paused, tickMs, timer;

  function randCell() {
    return {
      x: Math.floor(Math.random() * GRID),
      y: Math.floor(Math.random() * GRID)
    };
  }

  function placeFood() {
    var cell;
    var onSnake;
    do {
      cell = randCell();
      onSnake = snake.some(function (s) { return s.x === cell.x && s.y === cell.y; });
    } while (onSnake);
    return cell;
  }

  function reset() {
    snake = [
      { x: 12, y: 12 },
      { x: 11, y: 12 },
      { x: 10, y: 12 }
    ];
    dir = { x: 1, y: 0 };
    nextDir = { x: 1, y: 0 };
    score = 0;
    alive = true;
    paused = false;
    tickMs = 130;
    food = placeFood();
    scoreEl.textContent = score;
    msgEl.textContent = "pressione uma seta ou WASD para começar";
    draw();
  }

  function setDir(x, y) {
    // impede reverter diretamente sobre o próprio corpo
    if (dir.x === -x && dir.y === -y) return;
    nextDir = { x: x, y: y };
  }

  function step() {
    if (!alive || paused) return;

    dir = nextDir;
    var head = { x: snake[0].x + dir.x, y: snake[0].y + dir.y };

    var hitWall = head.x < 0 || head.y < 0 || head.x >= GRID || head.y >= GRID;
    var hitSelf = snake.some(function (s) { return s.x === head.x && s.y === head.y; });

    if (hitWall || hitSelf) {
      gameOver();
      return;
    }

    snake.unshift(head);

    if (head.x === food.x && head.y === food.y) {
      score += 10;
      scoreEl.textContent = score;
      food = placeFood();
      tickMs = Math.max(60, tickMs - 2);
      restartLoop();
    } else {
      snake.pop();
    }

    draw();
  }

  function gameOver() {
    alive = false;
    if (score > best) {
      best = score;
      bestEl.textContent = best;
      try {
        window.localStorage.setItem(BEST_KEY, String(best));
      } catch (e) {}
      msgEl.textContent = "game over — novo recorde! " + score + " pts 🏆";
    } else {
      msgEl.textContent = "game over — score: " + score + " pts";
    }
    clearInterval(timer);
    draw();
  }

  function draw() {
    var cell = canvas.width / GRID;

    ctx.fillStyle = "#000";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.strokeStyle = "rgba(0,255,65,0.08)";
    ctx.lineWidth = 1;
    for (var i = 1; i < GRID; i++) {
      ctx.beginPath();
      ctx.moveTo(i * cell, 0);
      ctx.lineTo(i * cell, canvas.height);
      ctx.stroke();
      ctx.beginPath();
      ctx.moveTo(0, i * cell);
      ctx.lineTo(canvas.width, i * cell);
      ctx.stroke();
    }

    ctx.fillStyle = "#ff4757";
    ctx.shadowColor = "#ff4757";
    ctx.shadowBlur = 6;
    ctx.fillRect(food.x * cell + 1, food.y * cell + 1, cell - 2, cell - 2);
    ctx.shadowBlur = 0;

    for (var s = 0; s < snake.length; s++) {
      var seg = snake[s];
      ctx.fillStyle = s === 0 ? "#00FF41" : "#0adb35";
      ctx.shadowColor = "#00FF41";
      ctx.shadowBlur = s === 0 ? 8 : 0;
      ctx.fillRect(seg.x * cell + 1, seg.y * cell + 1, cell - 2, cell - 2);
    }
    ctx.shadowBlur = 0;

    if (paused && alive) {
      ctx.fillStyle = "rgba(0,0,0,0.6)";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = "#00FF41";
      ctx.font = (canvas.width / 10) + "px monospace";
      ctx.textAlign = "center";
      ctx.fillText("PAUSADO", canvas.width / 2, canvas.height / 2);
    }
  }

  function restartLoop() {
    clearInterval(timer);
    timer = setInterval(step, tickMs);
  }

  function togglePause() {
    if (!alive) return;
    paused = !paused;
    msgEl.textContent = paused ? "pausado — espaço para continuar" : "";
    draw();
  }

  // canvas em pixels reais para nitidez do grid
  canvas.width = 480;
  canvas.height = 480;
  GRID = 24;

  document.addEventListener("keydown", function (e) {
    switch (e.key) {
      case "ArrowUp":
      case "w":
      case "W":
        e.preventDefault();
        setDir(0, -1);
        break;
      case "ArrowDown":
      case "s":
      case "S":
        e.preventDefault();
        setDir(0, 1);
        break;
      case "ArrowLeft":
      case "a":
      case "A":
        e.preventDefault();
        setDir(-1, 0);
        break;
      case "ArrowRight":
      case "d":
      case "D":
        e.preventDefault();
        setDir(1, 0);
        break;
      case " ":
        e.preventDefault();
        togglePause();
        break;
    }
    if (!timer && alive) {
      restartLoop();
      msgEl.textContent = "";
    }
  });

  document.querySelectorAll(".touch-controls button").forEach(function (btn) {
    btn.addEventListener("click", function () {
      var d = btn.getAttribute("data-dir");
      if (d === "up") setDir(0, -1);
      if (d === "down") setDir(0, 1);
      if (d === "left") setDir(-1, 0);
      if (d === "right") setDir(1, 0);
      if (!timer && alive) {
        restartLoop();
        msgEl.textContent = "";
      }
    });
  });

  restartBtn.addEventListener("click", function () {
    clearInterval(timer);
    reset();
  });

  reset();
})();
</script>
</body>
</html>
