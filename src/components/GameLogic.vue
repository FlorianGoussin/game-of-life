<template>
  <div @mousedown="startDraw" @mousemove="draw" @mouseup="endDraw">
    <canvas></canvas>
    <!-- <label>Cell size:</label><input type="number"> -->
    <button class="ui toggle button" @click="seed">Seed</button>
    <button class="ui button" @click="generate">Start</button>
    <button class="ui button" @click="endGenerate">Stop</button>
    <button class="ui button" @click="setup">Reset</button>
  </div>
</template>

<script>
let cellSize = 8,
  width = 0,
  height = 0,
  canvas = null,
  ctx = null,
  grid = [],
  intervalId = null,
  mousePressed = false;

class Cell {
  constructor(x, y, randomize) {
    this.x = x;
    this.y = y;
    this.alive = randomize ? Math.random() >= 0.7 : false;
  }

  draw() {
    ctx.beginPath();
    ctx.rect(this.x, this.y, cellSize, cellSize);
    if (this.alive) {
      ctx.fillStyle = "rgba(255,255,255,0)";
    } else {
      ctx.fillStyle = "rgba(0,0,0,1)";
    }
    ctx.fill();
  }
}

export default {
  name: "GameOfLife",
  data() {
    return {
      queue: [],
      randomize: false
    };
  },
  props: ["image", "topMargin"],
  methods: {
    startDraw() {
      mousePressed = true;
    },
    draw($event) {
      const cursorRadius = 10;
      if (mousePressed) {
        for (let x = 0; x < width; x += cellSize) {
          for (let y = 0; y < height; y += cellSize) {
            const cell = grid[x][y];
            const dx = Math.abs(cell.x - $event.pageX);
            const dy = Math.abs(cell.y - $event.pageY + this.topMargin);
            if (dx < cursorRadius && dy < cursorRadius) {
              cell.alive = true;
            }
          }
        }
        drawGrid();
      }
    },
    endDraw() {
      mousePressed = false;
    },
    generate() {
      intervalId = setInterval(applyRules, 50);
    },
    endGenerate() {
      if (intervalId) {
        clearInterval(intervalId);
      }
    },
    seed() {
      this.randomize = !this.randomize;
      setupCanvas(this.randomize);
    },
    setup() {
      // width = this.image.clientWidth;
      // height = this.image.clientHeight;
      width = 782;
      height = 604;
      setupCanvas();
    }
  },
  mounted: function() {
    this.setup();
    // window.addEventListener("load", this.setup);
  },
  beforeDestroy: function() {
    // window.removeEventListener("load", this.setup);
  }
};

function drawGrid() {
  ctx.clearRect(0, 0, width, height);

  for (let x = 0; x < width; x += cellSize) {
    for (let y = 0; y < height; y += cellSize) {
      grid[x][y].draw();
    }
  }
}

function setupCanvas(randomize) {
  canvas = document.querySelector("canvas");
  canvas.width = width;
  canvas.height = height;
  ctx = canvas.getContext("2d");

  grid = []; // clear
  ctx.clearRect(0, 0, width, height);
  for (let x = 0; x < width; x += cellSize) {
    grid[x] = [];
    for (let y = 0; y < height; y += cellSize) {
      grid[x][y] = new Cell(x, y, randomize);
    }
  }

  drawGrid();
}

// http://www.conwaylife.com/wiki/Conway%27s_Game_of_Life#Rules
function applyRules() {
  let newGenGrid = [];
  for (let x = 0; x < width; x += cellSize) {
    newGenGrid[x] = [];
    for (let y = 0; y < height; y += cellSize) {
      newGenGrid[x][y] = new Cell(x, y);
    }
  }

  for (let x = 0; x < width; x += cellSize) {
    for (let y = 0; y < height; y += cellSize) {
      const cell = grid[x][y];
      const aliveNeighbors = getAliveNeighbors(cell);

      // Any live cell with fewer than two live neighbours dies
      // Any live cell with more than three live neighbours dies
      if (cell.alive && (aliveNeighbors < 2 || aliveNeighbors > 3)) {
        newGenGrid[x][y].alive = false;
      }

      // Any live cell with two or three live neighbours lives, unchanged, to the next generation
      if (cell.alive && (aliveNeighbors === 2 || aliveNeighbors === 3)) {
        newGenGrid[x][y].alive = true;
      }

      // Any dead cell with exactly three live neighbours will come to life
      if (!cell.alive && aliveNeighbors === 3) {
        newGenGrid[x][y].alive = true;
      }
    }
  }

  grid = newGenGrid;
  drawGrid();
}

function getAliveNeighbors(cell) {
  const x = cell.x,
    y = cell.y;
  let neighbors = 0;

  if (x > 0) {
    if (y > 0 && grid[x - cellSize][y - cellSize].alive) {
      neighbors++;
    }

    if (grid[x - cellSize][y].alive) {
      neighbors++;
    }

    if (y < height - cellSize && grid[x - cellSize][y + cellSize].alive) {
      neighbors++;
    }
  }

  if (x < width - cellSize) {
    if (y > 0 && grid[x + cellSize][y - cellSize].alive) {
      neighbors++;
    }

    if (grid[x + cellSize][y].alive) {
      neighbors++;
    }

    if (y < height - cellSize && grid[x + cellSize][y + cellSize].alive) {
      neighbors++;
    }
  }

  if (y > 0 && grid[x][y - cellSize].alive) {
    neighbors++;
  }

  if (y < height - cellSize && grid[x][y + cellSize].alive) {
    neighbors++;
  }

  return neighbors;
}
</script>

<style>
canvas {
  position: absolute;
  top: var(--top-margin);
  left: 0;
}
</style>