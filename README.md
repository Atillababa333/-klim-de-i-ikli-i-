# -klim-de-i-ikli-i-
 4 portal var. WASD ile hareket ediyoruz. Bir adadayız. Yaşlı bir bilge var. onla E'ye basarak konuşuyoruz. O bize diyor ki "bu dört portalın içinde labirentler var. 1.'sinde fabrikaları bulup filtrelemen gerekiyor. (E tuşuyla) 2.'sinde açık muslukları bulman gerekiyor. 3.'sünde açık kalan lambaları kapatman 4.'sünde arabaları toplu taşıma araçlarına çevirmen gerekiyor" der. Labirent 12x12 olacak. Arabalar, Fabrikalar, musluklar ve lambalar rastgele hareket edecek. Labirentlerde görevleri tamamladığımız. Zaman çıkış açılacak. Bütün labirentleri bitirdiğimiz zaman dünyayı kurtardınız yazısı çıkacak.
<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>İklim Kahramanı</title>
  
  <script src="https://cdn.tailwindcss.com/3.4.17"></script>
  <link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;700&display=swap" rel="stylesheet">
  
  <style>
    body {
      margin: 0;
      overflow: hidden;
      font-family: 'DM Sans', sans-serif;
      background: #1a1a2e;
    }
    
    canvas {
      display: block;
      margin: auto;
    }

    #hud {
      position: fixed;
      top: 10px;
      left: 10px;
      color: #fff;
      font-size: 14px;
      z-index: 10;
      background: rgba(0, 0, 0, 0.6);
      padding: 8px 12px;
      border-radius: 8px;
    }

    #dialog {
      position: fixed;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      background: rgba(0, 0, 0, 0.85);
      color: #fff;
      padding: 16px 24px;
      border-radius: 12px;
      max-width: 600px;
      width: 90%;
      display: none;
      z-index: 20;
      font-size: 15px;
      line-height: 1.5;
    }

    #victory {
      position: fixed;
      inset: 0;
      display: none;
      z-index: 30;
      background: rgba(0, 0, 0, 0.8);
      align-items: center;
      justify-content: center;
      flex-direction: column;
    }

    #victory h1 {
      color: #4ade80;
      font-size: 48px;
      font-weight: 700;
    }

    #victory p {
      color: #fff;
      font-size: 20px;
      margin-top: 12px;
    }
  </style>
</head>
<body>
  <div id="hud"></div>
  <div id="dialog"></div>
  <div id="victory">
    <h1>🌍 Dünyayı Kurtardınız!</h1>
    <p>Tüm görevleri tamamladınız. Tebrikler!</p>
  </div>
  
  <canvas id="gameCanvas"></canvas>

  <script>
    // --- SABİTLER (CONSTANTS) ---
    const GRID_SIZE = 12;
    const MOVE_COOLDOWN_TIME = 8;
    const TARGET_MOVE_DELAY = 30;
    
    const ELDER_POS = { x: 6, y: 4 };
    const EXIT_POS = { x: 11, y: 11 };

    // Portal verilerini tek bir yapıda topladık
    const PORTALS = [
      { id: 0, x: 3, y: 2, color: '#ef4444', name: 'Fabrika', task: 'Fabrikaları bul ve filtrele (E)', emoji: '🏭', fixedEmoji: '✅' },
      { id: 1, x: 9, y: 2, color: '#3b82f6', name: 'Musluk', task: 'Açık muslukları kapat (E)', emoji: '🚰', fixedEmoji: '✅' },
      { id: 2, x: 3, y: 6, color: '#eab308', name: 'Lamba', task: 'Açık lambaları kapat (E)', emoji: '💡', fixedEmoji: '✅' },
      { id: 3, x: 9, y: 6, color: '#22c55e', name: 'Araba', task: 'Arabaları toplu taşımaya çevir (E)', emoji: '🚗', fixedEmoji: '🚌' }
    ];

    const MAZES = [
      // Labirent 0 (Fabrika)
      [
        [0,1,0,0,0,1,0,1,0,0,0,0], [0,1,0,1,0,1,0,1,0,1,1,0], [0,0,0,1,0,0,0,0,0,0,1,0], [1,1,0,1,1,1,0,1,1,0,1,0],
        [0,0,0,0,0,0,0,0,0,0,0,0], [0,1,1,1,0,1,1,1,0,1,1,0], [0,0,0,0,0,0,0,1,0,0,0,0], [1,1,0,1,1,0,1,1,1,0,1,1],
        [0,0,0,0,0,0,0,0,0,0,0,0], [0,1,1,0,1,1,1,0,1,1,0,1], [0,0,0,0,0,0,0,0,0,0,0,0], [1,1,1,1,0,1,1,1,0,1,1,0]
      ],
      // Labirent 1 (Musluk)
      [
        [0,0,0,1,0,0,0,0,0,1,0,0], [1,1,0,1,0,1,1,1,0,1,1,0], [0,0,0,0,0,0,0,0,0,0,0,0], [0,1,1,1,0,1,0,1,1,1,0,1],
        [0,0,0,0,0,1,0,0,0,0,0,0], [1,1,0,1,1,1,1,1,0,1,1,0], [0,0,0,0,0,0,0,0,0,0,0,0], [0,1,1,0,1,0,1,1,1,0,1,1],
        [0,0,0,0,1,0,0,0,0,0,0,0], [1,1,0,1,1,1,0,1,1,1,0,1], [0,0,0,0,0,0,0,0,0,0,0,0], [0,1,1,1,0,1,1,1,0,1,1,0]
      ],
      // Labirent 2 (Lamba)
      [
        [0,0,0,0,0,1,0,0,0,0,0,0], [1,1,0,1,0,1,1,1,0,1,1,1], [0,0,0,1,0,0,0,0,0,0,0,0], [0,1,1,1,1,0,1,1,1,0,1,0],
        [0,0,0,0,0,0,0,0,0,0,0,0], [1,0,1,1,1,0,1,0,1,1,0,1], [0,0,0,0,0,0,1,0,0,0,0,0], [0,1,1,0,1,1,1,1,0,1,1,1],
        [0,0,0,0,0,0,0,0,0,0,0,0], [1,1,0,1,0,1,1,1,0,1,0,1], [0,0,0,1,0,0,0,0,0,0,0,0], [0,1,1,1,1,1,0,1,1,1,1,0]
      ],
      // Labirent 3 (Araba)
      [
        [0,0,0,0,0,0,0,1,0,0,0,0], [1,1,0,1,1,0,1,1,1,0,1,0], [0,0,0,0,0,0,0,0,0,0,1,0], [0,1,1,1,0,1,1,1,0,1,1,0],
        [0,0,0,0,0,0,0,0,0,0,0,0], [1,1,0,1,1,1,0,1,1,1,0,1], [0,0,0,0,0,0,0,0,0,0,0,0], [0,1,1,0,1,0,1,1,1,0,1,1],
        [0,0,0,0,1,0,0,0,0,0,0,0], [1,1,0,1,1,1,0,1,1,1,0,1], [0,0,0,0,0,0,0,0,0,0,0,0], [0,1,1,1,0,1,1,1,0,1,1,0]
      ]
    ];

    // --- DOM ELEMENTLERİ VE CANVAS KURULUMU ---
    const canvasElement = document.getElementById('gameCanvas');
    const ctx = canvasElement.getContext('2d');
    const hudElement = document.getElementById('hud');
    const dialogElement = document.getElementById('dialog');
    const victoryElement = document.getElementById('victory');

    let canvasWidth, canvasHeight, tileSize;
    
    // --- OYUN DURUMU (GAME STATE) ---
    const gameState = {
      mode: 'island', // 'island' veya 'maze'
      activePortalId: -1,
      completedPortals: [false, false, false, false],
      player: { x: 6, y: 8 },
      moveCooldown: 0,
      activeMazeLayout: [],
      targets: [],
      fixedTargetCount: 0,
      totalTargets: 4,
      isExitOpen: false,
      targetMoveTimer: 0
    };

    // --- GİRDİ (INPUT) YÖNETİMİ ---
    const keys = {};
    document.addEventListener('keydown', e => {
      const key = e.key.toLowerCase();
      keys[key] = true;
      if (key === 'e') handleInteraction();
    });
    
    document.addEventListener('keyup', e => {
      keys[e.key.toLowerCase()] = false;
    });

    // --- TEMEL FONKSİYONLAR ---
    function handleResize() {
      canvasWidth = window.innerWidth;
      canvasHeight = window.innerHeight;
      canvasElement.width = canvasWidth;
      canvasElement.height = canvasHeight;
      tileSize = Math.floor(Math.min(canvasWidth, canvasHeight) / (GRID_SIZE + 2));
    }

    function showDialog(text) {
      dialogElement.style.display = 'block';
      dialogElement.textContent = text;
    }

    function hideDialog() {
      dialogElement.style.display = 'none';
    }

    // --- OYUN MEKANİKLERİ ---
    function enterMaze(portalId) {
      gameState.mode = 'maze';
      gameState.activePortalId = portalId;
      gameState.player = { x: 0, y: 0 };
      gameState.activeMazeLayout = MAZES[portalId];
      gameState.isExitOpen = false;
      spawnTargets();
    }

    function spawnTargets() {
      gameState.targets = [];
      gameState.fixedTargetCount = 0;
      
      for (let i = 0; i < gameState.totalTargets; i++) {
        let x, y;
        do {
          x = Math.floor(Math.random() * GRID_SIZE);
          y = Math.floor(Math.random() * GRID_SIZE);
        } while (
          gameState.activeMazeLayout[y][x] === 1 || 
          (x === 0 && y === 0)
        );
        gameState.targets.push({ x, y, fixed: false });
      }
    }

    function moveTargetsRandomly() {
      for (const target of gameState.targets) {
        if (target.fixed) continue;
        
        const directions = [ {dx: -1, dy: 0}, {dx: 0, dy: -1}, {dx: 1, dy: 0}, {dx: 0, dy: 1} ];
        const randomDir = directions[Math.floor(Math.random() * directions.length)];
        
        const nx = target.x + randomDir.dx;
        const ny = target.y + randomDir.dy;
        
        if (nx >= 0 && nx < GRID_SIZE && ny >= 0 && ny < GRID_SIZE && gameState.activeMazeLayout[ny][nx] === 0) {
          target.x = nx;
          target.y = ny;
        }
      }
    }

    function handleInteraction() {
      const { player } = gameState;

      if (gameState.mode === 'island') {
        // Bilge ile etkileşim
        if (Math.abs(player.x - ELDER_POS.x) <= 1 && Math.abs(player.y - ELDER_POS.y) <= 1) {
          showDialog("Yaşlı Bilge: \"Bu dört portalın içinde labirentler var. 1.'sinde fabrikaları bulup filtrelemen, 2.'sinde açık muslukları kapatman, 3.'sünde açık lambaları kapatman, 4.'sünde arabaları toplu taşımaya çevirmen gerekiyor. Haydi, dünyayı kurtar!\"");
          return;
        }
        
        // Portallara giriş
        for (const portal of PORTALS) {
          if (player.x === portal.x && player.y === portal.y && !gameState.completedPortals[portal.id]) {
            enterMaze(portal.id);
            hideDialog();
            return;
          }
        }
      } else if (gameState.mode === 'maze') {
        // Hedeflerle (Sorunlarla) etkileşim
        for (const target of gameState.targets) {
          if (!target.fixed && Math.abs(player.x - target.x) <= 1 && Math.abs(player.y - target.y) <= 1) {
            target.fixed = true;
            gameState.fixedTargetCount++;
            
            if (gameState.fixedTargetCount >= gameState.totalTargets) {
              gameState.isExitOpen = true;
              showDialog('Çıkış açıldı! Sağ alt köşeye git.');
            }
            return;
          }
        }
        
        // Çıkış portalı
        if (gameState.isExitOpen && player.x === EXIT_POS.x && player.y === EXIT_POS.y) {
          completeMaze();
        }
      }
    }

    function completeMaze() {
      const activeId = gameState.activePortalId;
      gameState.completedPortals[activeId] = true;
      gameState.mode = 'island';
      gameState.player = { x: PORTALS[activeId].x, y: PORTALS[activeId].y };
      
      if (gameState.completedPortals.every(completed => completed)) {
        victoryElement.style.display = 'flex';
      }
    }

    // --- GÜNCELLEME DÖNGÜSÜ (UPDATE) ---
    function updateLogic() {
      gameState.moveCooldown--;
      
      if (gameState.moveCooldown <= 0) {
        let dx = 0, dy = 0;
        
        if (keys['w'] || keys['arrowup']) dy = -1;
        if (keys['s'] || keys['arrowdown']) dy = 1;
        if (keys['a'] || keys['arrowleft']) dx = -1;
        if (keys['d'] || keys['arrowright']) dx = 1;
        
        if (dx !== 0 || dy !== 0) {
          const nx = gameState.player.x + dx;
          const ny = gameState.player.y + dy;
          const isWithinBounds = nx >= 0 && nx < GRID_SIZE && ny >= 0 && ny < GRID_SIZE;
          
          if (gameState.mode === 'island' && isWithinBounds) {
            gameState.player.x = nx;
            gameState.player.y = ny;
            hideDialog();
          } else if (gameState.mode === 'maze' && isWithinBounds && gameState.activeMazeLayout[ny][nx] === 0) {
            gameState.player.x = nx;
            gameState.player.y = ny;
            hideDialog();
          }
          gameState.moveCooldown = MOVE_COOLDOWN_TIME;
        }
      }

      // Hedeflerin hareket etmesi
      if (gameState.mode === 'maze') {
        gameState.targetMoveTimer++;
        if (gameState.targetMoveTimer > TARGET_MOVE_DELAY) {
          moveTargetsRandomly();
          gameState.targetMoveTimer = 0;
        }
      }
    }

    // --- ÇİZİM (RENDER) İŞLEMLERİ ---
    function drawIsland(offsetX, offsetY) {
      // Ada zemini
      ctx.fillStyle = '#065f46';
      ctx.fillRect(offsetX, offsetY, GRID_SIZE * tileSize, GRID_SIZE * tileSize);
      
      // Bilge
      ctx.font = `${tileSize}px serif`;
      ctx.fillText('🧙', offsetX + ELDER_POS.x * tileSize, offsetY + (ELDER_POS.y + 1) * tileSize);
      
      // Portallar
      for (const portal of PORTALS) {
        const isCompleted = gameState.completedPortals[portal.id];
        ctx.fillStyle = isCompleted ? '#4b5563' : portal.color;
        ctx.fillRect(offsetX + portal.x * tileSize, offsetY + portal.y * tileSize, tileSize, tileSize);
        
        ctx.fillStyle = '#fff';
        ctx.font = `${tileSize * 0.5}px sans-serif`;
        ctx.fillText(
          isCompleted ? '✓' : (portal.id + 1), 
          offsetX + portal.x * tileSize + tileSize * 0.3, 
          offsetY + portal.y * tileSize + tileSize * 0.65
        );
      }
      
      hudElement.textContent = 'Ada | WASD: Hareket | E: Etkileşim';
    }

    function drawMaze(offsetX, offsetY) {
      // Labirent duvarları ve yolları
      for (let y = 0; y < GRID_SIZE; y++) {
        for (let x = 0; x < GRID_SIZE; x++) {
          const isWall = gameState.activeMazeLayout[y][x] === 1;
          ctx.fillStyle = isWall ? '#374151' : '#d1d5db';
          ctx.fillRect(offsetX + x * tileSize, offsetY + y * tileSize, tileSize - 1, tileSize - 1);
        }
      }
      
      // Çıkış Noktası
      ctx.fillStyle = gameState.isExitOpen ? '#22c55e' : '#991b1b';
      ctx.fillRect(offsetX + EXIT_POS.x * tileSize, offsetY + EXIT_POS.y * tileSize, tileSize - 1, tileSize - 1);
      
      // Hedefler
      const currentPortalInfo = PORTALS[gameState.activePortalId];
      ctx.font = `${tileSize * 0.8}px serif`;
      
      for (const target of gameState.targets) {
        const emojiToDraw = target.fixed ? currentPortalInfo.fixedEmoji : currentPortalInfo.emoji;
        ctx.fillText(emojiToDraw, offsetX + target.x * tileSize, offsetY + (target.y + 0.85) * tileSize);
      }
      
      hudElement.textContent = `Portal ${currentPortalInfo.id + 1}: ${currentPortalInfo.task} | Kalan: ${gameState.totalTargets - gameState.fixedTargetCount}`;
    }

    function drawGame() {
      ctx.clearRect(0, 0, canvasWidth, canvasHeight);
      
      // Izgarayı ekranın ortasına hizalamak için offset hesaplaması
      const offsetX = (canvasWidth - GRID_SIZE * tileSize) / 2;
      const offsetY = (canvasHeight - GRID_SIZE * tileSize) / 2;

      if (gameState.mode === 'island') {
        drawIsland(offsetX, offsetY);
      } else {
        drawMaze(offsetX, offsetY);
      }

      // Oyuncuyu çiz
      ctx.font = `${tileSize}px serif`;
      ctx.fillText('🧑', offsetX + gameState.player.x * tileSize, offsetY + (gameState.player.y + 1) * tileSize);
    }

    // --- OYUN DÖNGÜSÜ (GAME LOOP) ---
    function gameLoop() {
      updateLogic();
      drawGame();
      requestAnimationFrame(gameLoop);
    }

    // --- BAŞLATMA (INIT) ---
    window.addEventListener('resize', handleResize);
    handleResize(); // İlk boyutlandırma
    gameLoop();     // Döngüyü başlat
  </script>
</body>
</html>
