<template>
  <div class="game-container">
    <canvas ref="gameCanvas"></canvas>

    <div class="ui">
      <h2>Score: {{ score }}</h2>
    </div>

    <div v-if="gameOver" class="game-over">
      <h1>Game Over</h1>

      <button @click="restartGame">
        Restart
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "JumpRunner",

  data() {
    return {
      canvas: null,
      ctx: null,

      player: {
        x: 100,
        y: 0,
        width: 60,
        height: 60,
        velocityY: 0,
        jumping: false,
      },

      obstacle: {
        x: 1000,
        y: 0,
        width: 50,
        height: 80,
        speed: 8,
      },

      gravity: 0.8,
      groundY: 0,

      score: 0,
      gameOver: false,

      animationId: null,

      jumpSound: null,
      scoreSound: null,
      bgMusic: null,
    }
  },

  mounted() {
    this.canvas = this.$refs.gameCanvas

    this.ctx = this.canvas.getContext("2d")

    this.resizeCanvas()

    window.addEventListener("resize", this.resizeCanvas)

    window.addEventListener("keydown", this.handleKey)

    window.addEventListener("touchstart", this.jump)

    this.setupSounds()

    this.startGame()
  },

  beforeUnmount() {
    cancelAnimationFrame(this.animationId)

    window.removeEventListener("resize", this.resizeCanvas)

    window.removeEventListener("keydown", this.handleKey)
  },

  methods: {

    resizeCanvas() {
      this.canvas.width = window.innerWidth
      this.canvas.height = window.innerHeight

      this.groundY = this.canvas.height - 120

      this.player.y = this.groundY - this.player.height

      this.obstacle.y = this.groundY - this.obstacle.height
    },

    setupSounds() {

      this.jumpSound = new Audio(
        "public/sounds/jump.m4a"
      )

      this.scoreSound = new Audio(
        "public/sounds/score.m4a"
      )

      this.bgMusic = new Audio(
        "public/sounds/forest.m4a"
      )

      this.bgMusic.loop = true
      this.bgMusic.volume = 0.3

      this.bgMusic.play()
    },

    handleKey(e) {
      if (e.code === "Space") {
        this.jump()
      }
    },

    jump() {

      if (this.player.jumping || this.gameOver) return

      this.player.velocityY = -18

      this.player.jumping = true

      this.jumpSound.currentTime = 0
      this.jumpSound.play()
    },

    drawBackground() {

      const ctx = this.ctx

      // Sky
      const gradient = ctx.createLinearGradient(
        0,
        0,
        0,
        this.canvas.height
      )

      gradient.addColorStop(0, "#87CEEB")
      gradient.addColorStop(1, "#dff6ff")

      ctx.fillStyle = gradient

      ctx.fillRect(
        0,
        0,
        this.canvas.width,
        this.canvas.height
      )

      // Ground
      ctx.fillStyle = "#228B22"

      ctx.fillRect(
        0,
        this.groundY,
        this.canvas.width,
        200
      )

      // Trees
      for (let i = 0; i < 25; i++) {

        const x = i * 150

        // trunk
        ctx.fillStyle = "#6B4423"

        ctx.fillRect(
          x + 40,
          this.groundY - 120,
          20,
          120
        )

        // leaves
        ctx.beginPath()

        ctx.fillStyle = "#0b6623"

        ctx.arc(
          x + 50,
          this.groundY - 140,
          50,
          0,
          Math.PI * 2
        )

        ctx.fill()
      }
    },

    drawPlayer() {

      const ctx = this.ctx
      const p = this.player

      // body
      ctx.fillStyle = "#ff4444"

      ctx.fillRect(
        p.x,
        p.y,
        p.width,
        p.height
      )

      // eyes
      ctx.fillStyle = "white"

      ctx.fillRect(p.x + 12, p.y + 15, 10, 10)
      ctx.fillRect(p.x + 38, p.y + 15, 10, 10)

      // legs
      ctx.fillStyle = "black"

      ctx.fillRect(p.x + 10, p.y + 50, 10, 15)
      ctx.fillRect(p.x + 40, p.y + 50, 10, 15)
    },

    drawObstacle() {

      const ctx = this.ctx
      const o = this.obstacle

      ctx.fillStyle = "#333"

      ctx.fillRect(
        o.x,
        o.y,
        o.width,
        o.height
      )

      // spikes
      ctx.fillStyle = "red"

      for (let i = 0; i < 5; i++) {

        ctx.beginPath()

        ctx.moveTo(o.x + i * 10, o.y)

        ctx.lineTo(o.x + 5 + i * 10, o.y - 20)

        ctx.lineTo(o.x + 10 + i * 10, o.y)

        ctx.fill()
      }
    },

    update() {

      if (this.gameOver) return

      const p = this.player
      const o = this.obstacle

      // gravity
      p.velocityY += this.gravity

      p.y += p.velocityY

      if (p.y >= this.groundY - p.height) {

        p.y = this.groundY - p.height

        p.jumping = false
      }

      // move obstacle
      o.x -= o.speed

      if (o.x < -100) {

        o.x = this.canvas.width + 200

        this.score++

        this.scoreSound.currentTime = 0
        this.scoreSound.play()
      }

      // collision
      if (
        p.x < o.x + o.width &&
        p.x + p.width > o.x &&
        p.y < o.y + o.height &&
        p.y + p.height > o.y
      ) {

        this.gameOver = true

        this.bgMusic.pause()
      }
    },

    gameLoop() {

      this.ctx.clearRect(
        0,
        0,
        this.canvas.width,
        this.canvas.height
      )

      this.drawBackground()

      this.update()

      this.drawPlayer()

      this.drawObstacle()

      this.animationId = requestAnimationFrame(
        this.gameLoop
      )
    },

    startGame() {
      this.gameLoop = this.gameLoop.bind(this)

      this.gameLoop()
    },

    restartGame() {
      location.reload()
    }
  }
}
</script>

<style>
*{
  margin:0;
  padding:0;
  box-sizing:border-box;
}

body{
  overflow:hidden;
  font-family:Arial;
}

.game-container{
  width:100%;
  height:100vh;
  position:relative;
  overflow:hidden;
}

canvas{
  display:block;
  width:100%;
  height:100%;
}

.ui{
  position:absolute;
  top:20px;
  left:20px;

  color:white;

  z-index:10;

  font-size:24px;
}

.game-over{
  position:absolute;

  top:50%;
  left:50%;

  transform:translate(-50%, -50%);

  text-align:center;

  color:white;

  z-index:20;
}

.game-over h1{
  font-size:70px;
  margin-bottom:20px;
}

.game-over button{
  padding:15px 30px;

  border:none;

  font-size:22px;

  border-radius:10px;

  cursor:pointer;
}
</style>
```
