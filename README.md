<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Space Invaders</title>
</head>
<body style="margin:0; overflow:hidden; background:black;">

<canvas id="gameCanvas"></canvas>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

let score = 0;
let player = {
    x: 0,
    y: 0,
    width: 50,
    height: 20,
    speed: 7
};
let keys = {};
let bullets = [];
let enemyBullets = [];
let gameOver = false;
let enemies = [];
let enemyRows = 4;
let enemyCols = 8;
let enemySpeed = 1;
let enemyDirection = 1;
let lastShotTime = 0;

function resizeCanvas() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    player.x = Math.max(0, Math.min(player.x, canvas.width - player.width));
    player.y = canvas.height - 80;
}
resizeCanvas();
window.addEventListener("resize", resizeCanvas);

document.addEventListener("keydown", e => keys[e.key] = true);
document.addEventListener("keyup", e => keys[e.key] = false);

document.addEventListener("keydown", e => {
    if (e.key === " " && Date.now() - lastShotTime > 200) {
        bullets.push({
            x: player.x + player.width / 2 - 3,
            y: player.y,
            width: 6,
            height: 15,
            speed: 8
        });
        lastShotTime = Date.now();
    }
});

function createEnemies() {
    enemies = [];
    const enemyWidth = 40;
    const enemyHeight = 30;
    const gapX = 20;
    const gapY = 20;
    const formationWidth = enemyCols * enemyWidth + (enemyCols - 1) * gapX;
    const startX = Math.max(20, (canvas.width - formationWidth) / 2);
    const startY = 60;

    for (let r = 0; r < enemyRows; r++) {
        for (let c = 0; c < enemyCols; c++) {
            enemies.push({
                x: startX + c * (enemyWidth + gapX),
                y: startY + r * (enemyHeight + gapY),
                width: enemyWidth,
                height: enemyHeight,
                alive: true
            });
        }
    }
}
createEnemies();

function resetWave() {
    enemyRows = Math.min(enemyRows + 1, 6);
    enemySpeed += 0.5;
    enemyDirection = 1;
    bullets = [];
    enemyBullets = [];
    createEnemies();
}

function shootEnemy(enemy) {
    enemyBullets.push({
        x: enemy.x + enemy.width / 2 - 3,
        y: enemy.y + enemy.height,
        width: 6,
        height: 12,
        speed: 4
    });
}

function maybeEnemyFire(aliveEnemies) {
    const fireChance = 0.008 + enemyRows * 0.002;
    if (aliveEnemies.length > 0 && Math.random() < fireChance) {
        const shooter = aliveEnemies[Math.floor(Math.random() * aliveEnemies.length)];
        shootEnemy(shooter);
    }
}

function endGame() {
    if (gameOver) return;
    gameOver = true;
    setTimeout(() => alert("GAME OVER! Score: " + score), 10);
    setTimeout(() => location.reload(), 60);
}

function gameLoop() {
    if (gameOver) return;
    ctx.fillStyle = "black";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    ctx.fillStyle = "white";
    ctx.font = "28px Arial";
    ctx.fillText("Score: " + score, 20, 30);

    if (keys["ArrowRight"] || keys["d"]) player.x += player.speed;
    if (keys["ArrowLeft"] || keys["a"]) player.x -= player.speed;
    player.x = Math.max(0, Math.min(canvas.width - player.width, player.x));

    ctx.fillStyle = "cyan";
    ctx.fillRect(player.x, player.y, player.width, player.height);

    ctx.fillStyle = "yellow";
    for (let i = bullets.length - 1; i >= 0; i--) {
        const b = bullets[i];
        b.y -= b.speed;
        ctx.fillRect(b.x, b.y, b.width, b.height);
        if (b.y + b.height < 0) bullets.splice(i, 1);
    }

    let hitEdge = false;
    enemies.forEach(e => {
        if (!e.alive) return;
        e.x += enemySpeed * enemyDirection;
        if (e.x + e.width > canvas.width - 20 || e.x < 20) hitEdge = true;
    });
    if (hitEdge) {
        enemyDirection *= -1;
        enemies.forEach(e => { if (e.alive) e.y += 20; });
    }

    const aliveEnemies = enemies.filter(e => e.alive);
    maybeEnemyFire(aliveEnemies);

    ctx.fillStyle = "red";
    aliveEnemies.forEach(e => ctx.fillRect(e.x, e.y, e.width, e.height));

    for (let i = bullets.length - 1; i >= 0; i--) {
        const b = bullets[i];
        for (const e of aliveEnemies) {
            if (
                b.x < e.x + e.width &&
                b.x + b.width > e.x &&
                b.y < e.y + e.height &&
                b.y + b.height > e.y
            ) {
                e.alive = false;
                bullets.splice(i, 1);
                score += 10;
                break;
            }
        }
    }

    ctx.fillStyle = "magenta";
    for (let i = enemyBullets.length - 1; i >= 0; i--) {
        const b = enemyBullets[i];
        b.y += b.speed;
        ctx.fillRect(b.x, b.y, b.width, b.height);
        if (b.y > canvas.height) {
            enemyBullets.splice(i, 1);
            continue;
        }
        if (
            b.x < player.x + player.width &&
            b.x + b.width > player.x &&
            b.y < player.y + player.height &&
            b.y + b.height > player.y
        ) {
            enemyBullets.splice(i, 1);
            endGame();
            return;
        }
    }

    if (aliveEnemies.length === 0) {
        resetWave();
    }

    for (const e of aliveEnemies) {
        if (e.y + e.height >= player.y) {
            endGame();
            return;
        }
    }

    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

</body>
</html>
