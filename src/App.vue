<script>
export default {
  data() {
    return {
      settings: {
        nMines: 99,
        rows: 16,
        cols: 30,
      },
      stopwatch: 0,
      boom: false,
      won: false,
      grid: [],
    };
  },

  methods: {
    startGame() {
      // reset vars
      this.won = false;
      this.stopwatch = 0;
      this.grid = [];
      this.boom = false;
      let id = 0;

      // mines
      let max = this.settings.rows * this.settings.cols;
      let mines = [];
      while (mines.length < this.settings.nMines && mines.length < max) {
        let row = Math.floor(Math.random() * this.settings.rows);
        let col = Math.floor(Math.random() * this.settings.cols);
        if (this.isMine(mines, [row, col])) {
          continue;
        }
        mines.push([row, col]);
      }

      // grid
      for (let row = 0; row < this.settings.rows; row++) {
        let newRow = [];
        for (let col = 0; col < this.settings.cols; col++) {
          let nMines = this.vecinos([row, col])
            .map(([r, c]) => (this.isMine(mines, [r, c]) ? 1 : 0))
            .reduce((acc, x) => acc + x);
          newRow.push({
            id: id,
            explored: false,
            flagged: false,
            mine: this.isMine(mines, [row, col]),
            nMines: nMines,
          });
          id++;
        }
        this.grid.push(newRow);
      }
      this.startStopwatch();
    },

    startStopwatch() {
      if (!this.won && !this.boom) {
        this.stopwatch++;
        setTimeout(this.startStopwatch, 1000);
      }
    },

    isMine(mines, [row, col]) {
      return mines.some(([mRow, mCol]) => mRow == row && mCol == col);
    },

    vecinos([r, c]) {
      return [
        [r - 1, c],
        [r, c - 1],
        [r + 1, c],
        [r, c + 1],
        [r - 1, c - 1],
        [r - 1, c + 1],
        [r + 1, c - 1],
        [r + 1, c + 1],
      ];
    },

    explore(r, c) {
      let cell = this.grid[r][c];
      if (cell.mine) {
        this.boom = true;
      } else if (!cell.explored) {
        cell.explored = true;
        if (cell.nMines == 0) {
          this.vecinos([r, c]).forEach(([r, c]) => {
            if (
              r >= 0 &&
              r < this.settings.rows &&
              c >= 0 &&
              c < this.settings.cols
            ) {
              let vcell = this.grid[r][c];
              if (!vcell.mine && !vcell.flagged) {
                this.explore(r, c);
              }
            }
          });
        }
      }
    },

    win() {
      for (let r = 0; r < this.settings.rows; r++) {
        for (let c = 0; c < this.settings.cols; c++) {
          let cell = this.grid[r][c];
          if (!cell.mine && !cell.explored) {
            return;
          }
        }
      }
      this.won = true;
    },
  },

  mounted() {
    this.startGame();
  },
};
</script>

<template>
  <main>
    <h1 id="stopwatch">{{ this.stopwatch }}</h1>
    <div
      id="buscaminas"
      @contextmenu.prevent
      :style="`${won ? 'border: 1px solid lightgreen;' : ''} ${
        boom ? 'border: 1px solid #ed3966;' : ''
      }`"
    >
      <template v-for="(row, r) in grid">
        <template v-for="(cell, c) in row" :key="cell.id">
          <div v-if="!boom">
            <button
              v-if="cell.flagged"
              class="cell cell-flagged"
              @click.right="cell.flagged = false"
            ></button>
            <div
              v-else-if="cell.explored"
              :class="`cell cell-explored cell-${cell.nMines}`"
            >
              {{ cell.nMines > 0 ? cell.nMines : "" }}
            </div>
            <button
              v-else
              class="cell cell-unexplored"
              :class="won ? 'cell-mine cell-mine-deactivated' : ''"
              @click="
                explore(r, c);
                win();
              "
              @click.right="cell.flagged = true"
            ></button>
          </div>
          <div v-else>
            <div v-if="cell.mine" class="cell cell-mine"></div>
            <div v-else class="cell cell-explored">
              {{ cell.nMines != 0 ? cell.nMines : "" }}
            </div>
          </div>
        </template>
      </template>
    </div>
    <div id="settings">
      <input v-model="settings.rows" type="number" placeholder="rows" />
      <input v-model="settings.cols" type="number" placeholder="cols" />
      <input v-model="settings.nMines" type="number" placeholder="mines" />
      <button @click="startGame" id="play">play</button>
    </div>
  </main>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  background: black;
  color: white;
  box-sizing: border-box;
  font-family: "Franklin Gothic Medium", "Arial Narrow", Arial, sans-serif;
  font-weight: bold;
  background: #15141a;
}

main {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 10vh;
  background: #15141a;
}

#settings {
  display: flex;
  margin-top: 0.4em;
  height: 2em;
}

#play {
  width: 10em;
  background: #15141a;
  border: 1px #52525e solid;
}

#stopwatch {
  background: #15141a;
}

input {
  text-align: center;
  background: #15141a;
  border: 1px #52525e solid;
  width: 100px;
}

#play:hover {
  background: #2b2a33;
}

#buscaminas {
  margin-top: 0.3em;
  display: grid;
  grid-template-columns: repeat(v-bind("settings.cols"), 1fr);
  grid-template-rows: repeat(v-bind("settings.rows"), 1fr);
  width: min-content;
  height: min-content;
  border: 1px #52525e solid;
}

.cell {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2em;
  height: 2em;
  font-size: initial;
  background: #9f9fad;
  border: 1px #52525e solid;
}

.cell-mine {
  background: #ed3966;
  background-image: url(./assets/bomb.svg);
  background-size: 90%;
  background-repeat: no-repeat;
  background-position: center center;
}

.cell-mine-deactivated {
  background: lightgreen;
  background-image: url(./assets/bomb.svg);
}

.cell-explored {
  background: #15141a;
  border: 1px solid #3a3944;
  color: #52525e;
}

/* .cell-unexplored {
} */

.cell-unexplored:hover {
  border-width: 2px;
}

.cell-flagged {
  background-image: url(./assets/flag.svg);
  background-size: 90%;
  background-repeat: no-repeat;
  background-position: center center;
}

.cell-1 {
  background: lightskyblue;
  color: #15141a;
}

.cell-2 {
  background: lightgreen;
  color: #15141a;
}

.cell-3 {
  background: lightcoral;
  color: #15141a;
}

.cell-4 {
  background: lightseagreen;
  color: #15141a;
}

.cell-5 {
  background: lightyellow;
  color: #15141a;
}

.cell-6 {
  background: lightsalmon;
  color: #15141a;
}

.cell-7 {
  background: lightcyan;
  color: #15141a;
}

.cell-7 {
  background: lightgray;
  color: #15141a;
}
</style>
