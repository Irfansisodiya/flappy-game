<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Flappy Bird Game</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }
        body {
            background-color: #1a1a1a;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            font-family: sans-serif;
        }
        canvas {
            border: 2px solid #fff;
            background-color: #70c5ce;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
            max-width: 100vw;
            max-height: 100vh;
        }
    </style>
</head>
<body>

<canvas id="gameCanvas" width="360" height="600"></canvas>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

let frames = 0;
let score = 0;
let highScore = 0;
let gameState = "START"; // START, PLAY, GAMEOVER

// चिड़िया (Bird)
const bird = {
    x: 60,
    y: 250,
    radius: 14,
    gravity: 0.28,
    jump: -5.8,
    velocity: 0,
    draw() {
        ctx.fillStyle = "#FFD700";
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
        ctx.fill();
        ctx.strokeStyle = "#000";
        ctx.lineWidth = 2;
        ctx.stroke();

        // आँख
        ctx.fillStyle = "#fff";
        ctx.beginPath();
        ctx.arc(this.x + 5, this.y - 4, 4, 0, Math.PI * 2);
        ctx.fill();
        ctx.fillStyle = "#000";
        ctx.beginPath();
        ctx.arc(this.x + 7, this.y - 4, 2, 0, Math.PI * 2);
        ctx.fill();

        // चोंच
        ctx.fillStyle = "#FF4500";
        ctx.beginPath();
        ctx.moveTo(this.x + 10, this.y);
        ctx.lineTo(this.x + 18, this.y + 4);
        ctx.lineTo(this.x + 10, this.y + 8);
        ctx.fill();
    },
    update() {
        if (gameState === "PLAY") {
            this.velocity += this.gravity;
            this.y += this.velocity;

            // जमीन से टकराना
            if (this.y + this.radius >= canvas.height - 50) {
                this.y = canvas.height - 50 - this.radius;
                gameOver();
            }
            // छत से बाहर न जाए
            if (this.y - this.radius <= 0) {
                this.y = this.radius;
                this.velocity = 0;
            }
        }
    },
    flap() {
        this.velocity = this.jump;
    },
    reset() {
        this.y = 250;
        this.velocity = 0;
    }
};

// पाइप्स (Obstacles)
const pipes = {
    items: [],
    width: 52,
    gap: 130,
    dx: 2.2,
    draw() {
        for (let p of this.items) {
            ctx.fillStyle = "#2ecc71";
            ctx.strokeStyle = "#27ae60";
            ctx.lineWidth = 3;

            // ऊपर का पाइप
            ctx.fillRect(p.x, 0, this.width, p.top);
            ctx.strokeRect(p.x, 0, this.width, p.top);

            // नीचे का पाइप
            let bottomY = p.top + this.gap;
            let bottomHeight = canvas.height - 50 - bottomY;
            ctx.fillRect(p.x, bottomY, this.width, bottomHeight);
            ctx.strokeRect(p.x, bottomY, this.width, bottomHeight);
        }
    },
    update() {
        if (gameState !== "PLAY") return;

        // हर 110 फ्रेम पर नया पाइप
        if (frames % 110 === 0) {
            let maxTop = canvas.height - 50 - this.gap - 60;
            let topHeight = Math.floor(Math.random() * (maxTop - 50)) + 50;
            this.items.push({
                x: canvas.width,
                top: topHeight,
                passed: false
            });
        }

        for (let i = 0; i < this.items.length; i++) {
            let p = this.items[i];
            p.x -= this.dx;

            // टक्कर चेक (Collision Detection)
            let birdRight = bird.x + bird.radius;
            let birdLeft = bird.x - bird.radius;
            let birdTop = bird.y - bird.radius;
            let birdBottom = bird.y + bird.radius;

            let pipeRight = p.x + this.width;
            let bottomPipeY = p.top + this.gap;

            if (birdRight > p.x && birdLeft < pipeRight) {
                if (birdTop < p.top || birdBottom > bottomPipeY) {
                    gameOver();
                }
            }

            // स्कोर बढ़ना
            if (p.x + this.width < bird.x && !p.passed) {
                score++;
                p.passed = true;
            }

            // स्क्रीन से बाहर गए पाइप हटाना
            if (p.x + this.width < 0) {
                this.items.splice(i, 1);
                i--;
            }
        }
    },
    reset() {
        this.items = [];
    }
};

// जमीन (Ground)
function drawGround() {
    ctx.fillStyle = "#ded895";
    ctx.fillRect(0, canvas.height - 50, canvas.width, 50);
    ctx.fillStyle = "#73bf2e";
    ctx.fillRect(0, canvas.height - 50, canvas.width, 10);
}

// टेक्स्ट और स्कोर UI
function drawUI() {
    ctx.fillStyle = "#fff";
    ctx.font = "bold 26px sans-serif";
    ctx.textAlign = "center";

    if (gameState === "START") {
        ctx.fillText("TAP TO FLY", canvas.width / 2, canvas.height / 2 - 20);
        ctx.font = "16px sans-serif";
        ctx.fillText("Touch or Press Space", canvas.width / 2, canvas.height / 2 + 20);
    } else if (gameState === "PLAY") {
        ctx.fillText(score, canvas.width / 2, 60);
    } else if (gameState === "GAMEOVER") {
        ctx.fillStyle = "#e74c3c";
        ctx.fillText("GAME OVER", canvas.width / 2, canvas.height / 2 - 40);
        ctx.fillStyle = "#fff";
        ctx.font = "20px sans-serif";
        ctx.fillText("Score: " + score, canvas.width / 2, canvas.height / 2);
        ctx.fillText("Best: " + highScore, canvas.width / 2, canvas.height / 2 + 35);
        ctx.font = "16px sans-serif";
        ctx.fillText("Tap to Restart", canvas.width / 2, canvas.height / 2 + 80);
    }
}

function gameOver() {
    gameState = "GAMEOVER";
    if (score > highScore) {
        highScore = score;
    }
}

function handleAction() {
    if (gameState === "START") {
        gameState = "PLAY";
        bird.flap();
    } else if (gameState === "PLAY") {
        bird.flap();
    } else if (gameState === "GAMEOVER") {
        bird.reset();
        pipes.reset();
        score = 0;
        frames = 0;
        gameState = "PLAY";
    }
}

// इनपुट लिसनर (टच और कीबोर्ड)
window.addEventListener("keydown", (e) => {
    if (e.code === "Space") {
        e.preventDefault();
        handleAction();
    }
});
canvas.addEventListener("touchstart", (e) => {
    e.preventDefault();
    handleAction();
}, { passive: false });
canvas.addEventListener("mousedown", () => handleAction());

// मुख्य गेम लूप
function loop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    pipes.update();
    pipes.draw();

    drawGround();

    bird.update();
    bird.draw();

    drawUI();

    frames++;
    requestAnimationFrame(loop);
}

loop();
</script>
</body>
</html>
