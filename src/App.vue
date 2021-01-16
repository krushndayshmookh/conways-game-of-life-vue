<template>
  <div id="app" :style="cssVars">
    <div id="dashboard">
      <h1>Conway's Game of Life</h1>

      <p>
        Created by:
        <a href="https://krushndayshmookh.github.io">Krushn Dayshmookh</a>
      </p>

      <div class="controls">
        <button @click="start" :disabled="state">start</button>
        <button @click="stop" :disabled="!state">stop</button>
        <label for="showNeighbors">
          <input type="checkbox" v-model="showNeighbors" id="showNeighbors" />
          Show Neighbors
        </label>
        <label for="nightMode">
          <input type="checkbox" v-model="nightMode" id="nightMode" />
          Night Mode
        </label>

        <label for="width">
          Width:
          <input
            type="number"
            v-model="width"
            id="width"
            min="10"
            @input="createGrid"
          />
        </label>
        <label for="height">
          Height:
          <input
            type="number"
            v-model="height"
            id="height"
            min="10"
            @input="createGrid"
          />
        </label>

        <label for="size">
          Size:
          <input type="number" v-model="cellSize" id="size" />
        </label>
      </div>
      <div class="status">
        <span>Iteration: {{ iteration }}</span>
        <span>Live: {{ liveCount }}</span>
        <span>Stable: {{ hasStabilized }}</span>
      </div>
    </div>
    <div class="grid">
      <div class="row" v-for="(row, idr) in grid" :key="idr">
        <div
          class="cell"
          v-for="(cell, idc) in row"
          :key="idc"
          :class="cell.live ? 'black' : 'grey'"
          @click="cell.live = !cell.live"
        >
          <span v-if="showNeighbors">{{ countLiveNeighbors(idr, idc) }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import _ from 'lodash'

const THEME = {
  day: {
    page: '#ffffff',
    cellLive: '#000000',
    cellDead: '#aaaaaa',
    text: '#000000',
  },
  night: {
    page: '#000000',
    cellLive: '#ffff00',
    cellDead: '#000000',
    text: '#ffffff',
  },
}

export default {
  name: 'App',
  data() {
    return {
      width: 60,
      height: 30,

      updateInterval: 100,

      grid: [],

      timer: null,
      state: false,
      iteration: 0,

      showNeighbors: false,
      nightMode: false,

      cellSize: 16,
    }
  },

  computed: {
    theme() {
      return this.nightMode ? THEME.night : THEME.day
    },

    cssVars() {
      return {
        '--cell-size': this.cellSize + 'px',
        '--bg-page': this.theme.page,
        '--text-color': this.theme.text,
        '--cell-dead': this.theme.cellDead,
        '--cell-live': this.theme.cellLive,
      }
    },

    liveCount() {
      let flattendGrid = this.grid.flat()
      return flattendGrid.filter((c) => c.live).length
    },

    nextState() {
      let newGrid = []
      for (let i = 0; i < this.height; i++) {
        let row = []
        for (let j = 0; j < this.width; j++) {
          let n = this.countLiveNeighbors(i, j)
          let currCell = this.grid[i][j]
          if (currCell.live && (n == 3 || n == 2)) {
            row.push({ live: true })
          } else if (n == 3) {
            row.push({ live: true })
          } else {
            row.push({ live: false })
          }
        }
        newGrid.push(row)
      }

      return newGrid
    },

    hasStabilized() {
      if (this.grid.length && this.nextState.length)
        return _(this.grid).differenceWith(this.nextState, _.isEqual).isEmpty()
      return false
    },
  },

  watch: {
    liveCount(c) {
      if (!c) this.stop()
    },

    hasStabilized(v) {
      if (this.state && v) {
        this.stop()
      }
    },
  },

  created() {
    this.createGrid()
  },

  methods: {
    createGrid() {
      this.grid = []
      for (let i = 0; i < this.height; i++) {
        let row = []
        for (let j = 0; j < this.width; j++) {
          row.push({ live: false })
        }
        this.grid.push(row)
      }
    },

    updateGrid() {
      this.grid = this.nextState
      this.iteration++
    },

    countLiveNeighbors(i, j) {
      let g = this.grid
      let neighbors = [
        g[i - 1] ? g[i - 1][j - 1] : false,
        g[i - 1] ? g[i - 1][j] : false,
        g[i - 1] ? g[i - 1][j + 1] : false,
        g[i] ? g[i][j - 1] : false,
        g[i] ? g[i][j + 1] : false,
        g[i + 1] ? g[i + 1][j - 1] : false,
        g[i + 1] ? g[i + 1][j] : false,
        g[i + 1] ? g[i + 1][j + 1] : false,
      ]

      return neighbors.filter((n) => n && n.live).length
    },

    start() {
      this.iteration = 0
      this.timer = setInterval(this.updateGrid, this.updateInterval)
      this.state = true
    },

    stop() {
      clearInterval(this.timer)
      this.timer = null
      this.state = false
    },
  },
}
</script>

<style lang="scss">
#app {
  margin: 0;
  padding: 0;
  background: var(--bg-page);
  color: var(--text-color);

  font-family: 'Courier New', Courier, monospace;

  #dashboard {
    padding: 0 8px;

    h1 {
      margin: 0;
      padding-bottom: 0;
    }
    p {
      margin: 0;
      padding-bottom: 4px;
    }

    .controls {
      margin-bottom: 4px;

      input[type='number'] {
        width: 60px;
      }
    }

    button,
    span {
      margin-right: 16px;
    }
  }

  .grid {
    padding: 8px;

    .row {
      height: calc(var(--cell-size) + 2px);

      .cell {
        margin: 0px;
        border: 1px solid var(--bg-page);

        width: var(--cell-size);
        height: var(--cell-size);

        display: inline-block;

        &.black {
          background: var(--cell-live);
        }

        &.grey {
          background: var(--cell-dead);
        }
      }
    }
  }
}
</style>
