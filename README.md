<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <title>لعبة المتاهة العشوائية</title>
  <style>
    body {
      margin: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      height: 100vh;
      background-color: #222;
      font-family: Arial, sans-serif;
    }
    canvas {
      border: 2px solid white;
      background-color: #111;
    }
    .controls {
      margin-top: 20px;
      display: grid;
      grid-template-areas:
        ". up ."
        "left . right"
        ". down .";
      grid-gap: 10px;
    }
    .controls button {
      width: 50px;
      height: 50px;
      font-size: 20px;
      background-color: rgba(255, 255, 255, 0.2);
      color: white;
      border: 1px solid white;
      border-radius: 10px;
      cursor: pointer;
    }
    #up { grid-area: up; }
    #down { grid-area: down; }
    #left { grid-area: left; }
    #right { grid-area: right; }

    /* مربع الفوز صغير وذو زاوية */
    #winPopup {
      position: absolute;
      top: 20px;
      right: 20px;
      width: 180px;
      height: auto;
      background-color: rgba(255, 255, 255, 0.85);
      display: flex;
      justify-content: center;
      align-items: center;
      border-radius: 15px;
      padding: 15px;
      flex-direction: column;
      z-index: 10;
      box-shadow: 0 4px 10px rgba(0,0,0,0.3);
    }
    #winBox h1 {
      font-size: 18px;
      margin: 5px 0;
    }
    #winBox button {
      font-size: 14px;
      padding: 5px 10px;
      margin-top: 5px;
      cursor: pointer;
      border-radius: 8px;
      border: 1px solid #444;
      background-color: #28a745;
      color: white;
    }
    .hidden { display: none; }
  </style>
</head>
<body>
  <canvas id="mazeCanvas" width="800" height="800"></canvas>

  <!-- أزرار التحكم الشفافة -->
  <div class="controls">
    <button id="up">↑</button>
    <button id="left">←</button>
    <button id="down">↓</button>
    <button id="right">→</button>
  </div>

  <!-- مربع الفوز -->
  <div id="winPopup" class="hidden">
    <div id="winBox">
      <h1>🎉 مبروك! 🎉</h1>
      <button id="restartBtn">إعادة اللعب</button>
    </div>
  </div>

  <script>
    const canvas = document.getElementById("mazeCanvas");
    const ctx = canvas.getContext("2d");

    const winPopup = document.getElementById("winPopup");
    const restartBtn = document.getElementById("restartBtn");

    const cellSize = 50;
    const rows = 15, cols = 15;
    let maze, player, exit;

    function generateMaze() {
      maze = Array.from({length: rows}, () => Array(cols).fill(1));

      function carve(x, y) {
        const dirs = [[0,-2],[0,2],[-2,0],[2,0]].sort(() => Math.random()-0.5);
        for (const [dx,dy] of dirs) {
          const nx = x + dx;
          const ny = y + dy;
          if(nx>0 && nx<cols && ny>0 && ny<rows && maze[ny][nx]===1){
            maze[y + dy/2][x + dx/2] = 0;
            maze[ny][nx] = 0;
            carve(nx, ny);
          }
        }
      }

      const startX = 1, startY = rows-2;
      maze[startY][startX] = 0;
      carve(startX, startY);
      player = {x:startX, y:startY, color:"yellow"};

      exit = {x: cols-2, y: 1, color:"green"};
      maze[exit.y][exit.x] = 0;
    }

    function drawMaze() {
      for(let y=0; y<rows; y++){
        for(let x=0; x<cols; x++){
          ctx.fillStyle = maze[y][x]===1 ? "white" : "#111";
          ctx.fillRect(x*cellSize, y*cellSize, cellSize, cellSize);
        }
      }
    }

    function drawPlayer() {
      ctx.fillStyle = player.color;
      ctx.fillRect(player.x*cellSize+10, player.y*cellSize+10, cellSize-20, cellSize-20);

      ctx.fillStyle = exit.color;
      ctx.fillRect(exit.x*cellSize+10, exit.y*cellSize+10, cellSize-20, cellSize-20);
    }

    function drawGame() {
      ctx.clearRect(0,0,canvas.width,canvas.height);
      drawMaze();
      drawPlayer();
    }

    function movePlayer(dx,dy){
      const newX = player.x + dx;
      const newY = player.y + dy;
      if(maze[newY][newX]===0){
        player.x = newX;
        player.y = newY;
        checkWin();
      }
      drawGame();
    }

    function checkWin(){
      if(player.x===exit.x && player.y===exit.y){
        winPopup.classList.remove("hidden");
      }
    }

    document.getElementById("up").addEventListener("click", ()=>movePlayer(0,-1));
    document.getElementById("down").addEventListener("click", ()=>movePlayer(0,1));
    document.getElementById("left").addEventListener("click", ()=>movePlayer(-1,0));
    document.getElementById("right").addEventListener("click", ()=>movePlayer(1,0));

    document.addEventListener("keydown",(e)=>{
      switch(e.key){
        case "ArrowUp": movePlayer(0,-1); break;
        case "ArrowDown": movePlayer(0,1); break;
        case "ArrowLeft": movePlayer(-1,0); break;
        case "ArrowRight": movePlayer(1,0); break;
      }
    });

    restartBtn.addEventListener("click", ()=>{
      winPopup.classList.add("hidden");
      generateMaze();
      drawGame();
    });

    // بدء اللعبة لأول مرة
    generateMaze();
    drawGame();
  </script>
</body>
</html>
