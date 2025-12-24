# agar<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Agar.io with AI</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background: #111;
        }
        canvas {
            display: block;
        }
    </style>
</head>
<body>
<canvas id="game"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// ================= PLAYER =================
const player = {
    x: canvas.width / 2,
    y: canvas.height / 2,
    radius: 20,
    speed: 3,
    color: "lime"
};

// ================= MOUSE =================
const mouse = { x: player.x, y: player.y };
window.addEventListener("mousemove", e => {
    mouse.x = e.clientX;
    mouse.y = e.clientY;
});

// ================= FOOD =================
const foods = [];
const FOOD_COUNT = 250;

function createFood() {
    return {
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        radius: 4,
        color: "white"
    };
}

for (let i = 0; i < FOOD_COUNT; i++) foods.push(createFood());

// ================= ENEMIES =================
const enemies = [];
const ENEMY_COUNT = 5;

function createEnemy() {
    return {
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        radius: 20 + Math.random() * 20,
        speed: 1.5,
        dx: Math.random() * 2 - 1,
        dy: Math.random() * 2 - 1,
        color: "red"
    };
}

for (let i = 0; i < ENEMY_COUNT; i++) enemies.push(createEnemy());

// ================= UTILS =================
function distance(a, b, c, d) {
    return Math.hypot(c - a, d - b);
}

function drawCircle(x, y, r, color) {
    ctx.beginPath();
    ctx.arc(x, y, r, 0, Math.PI * 2);
    ctx.fillStyle = color;
    ctx.fill();
}

// ================= LOGIC =================
function movePlayer() {
    const dx = mouse.x - player.x;
    const dy = mouse.y - player.y;
    const dist = Math.hypot(dx, dy);

    if (dist > 1) {
        player.x += (dx / dist) * player.speed;
        player.y += (dy / dist) * player.speed;
    }
}

function moveEnemies() {
    enemies.forEach(e => {
        e.x += e.dx * e.speed;
        e.y += e.dy * e.speed;

        if (Math.random() < 0.01) {
            e.dx = Math.random() * 2 - 1;
            e.dy = Math.random() * 2 - 1;
        }

        // Wall bounce
        if (e.x < 0 || e.x > canvas.width) e.dx *= -1;
        if (e.y < 0 || e.y > canvas.height) e.dy *= -1;
    });
}

function eatFood(entity) {
    for (let i = foods.length - 1; i >= 0; i--) {
        const f = foods[i];
        if (distance(entity.x, entity.y, f.x, f.y) < entity.radius) {
            foods.splice(i, 1);
            entity.radius += 0.3;
            foods.push(createFood());
        }
    }
}

function enemyVsPlayer() {
    enemies.forEach((e, i) => {
        const d = distance(player.x, player.y, e.x, e.y);

        if (d < player.radius && player.radius > e.radius) {
            player.radius += e.radius * 0.5;
            enemies.splice(i, 1);
            enemies.push(createEnemy());
        }

        if (d < e.radius && e.radius > player.radius) {
            alert("Game Over!");
            location.reload();
        }
    });
}

// ================= GAME LOOP =================
function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    movePlayer();
    moveEnemies();

    eatFood(player);
    enemies.forEach(e => eatFood(e));

    enemyVsPlayer();

    foods.forEach(f => drawCircle(f.x, f.y, f.radius, f.color));
    enemies.forEach(e => drawCircle(e.x, e.y, e.radius, e.color));
    drawCircle(player.x, player.y, player.radius, player.color);

    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>
</body>
</html>
