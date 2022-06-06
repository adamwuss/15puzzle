<template>
  <div>
    <div class="board">
      <div
        v-for="(tile, index) in actualConfiguration"
        :key="index"
        class="tile d-inline-flex align-items-center justify-content-center"
        :class="{ empty: !tile, 'cursor-pointer': tileNextToEmpty(tile), 'grey-background': index + 1 === tile }"
        @click="moveTile(tile)"
      >
        {{ tile }}
      </div>
    </div>
    <div class="mt-3">
      <div
        class="d-flex justify-content-between"
        style="margin-left: 5px"
      >
        <div>
          <button
            class="btn btn-primary me-2"
            @click="shuffle"
          >
            New game
          </button>
        </div>
        <div
          class="d-flex align-items-center"
          style="font-size: 25px"
        >
          Moves: {{ moves }}
        </div>
        <div
          class="d-inline-flex align-items-center"
          style="margin-right: 5px; font-size: 25px"
        >
          {{ time }}
        </div>
      </div>
    </div>
    <div class="mt-3">
      <ol>
        <li
          v-for="(result, i) in sortedResults"
          :key="i"
        >
          {{ result.name }}: {{ result.time }}
        </li>
      </ol>
    </div>
  </div>
</template>

<script>
const SOLUTION = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 0];

export default {
  name: 'Board',
  data: () => ({
    actualConfiguration: [],
    moves: 0,
    results: [],
    time: '00:00:00.000',
    running: false,
    timeBegan: null,
    started: null,
  }),
  mounted() {
    this.actualConfiguration = JSON.parse(JSON.stringify(this.randomNumber(SOLUTION)));
    this.start();
  },
  methods: {
    moveTile(tile) {
      const indexTile = this.actualConfiguration.indexOf(tile);
      const indexEmpty = this.actualConfiguration.indexOf(0);

      if (this.tileNextToEmpty(tile)) {
        this.moves += 1;
        this.$set(this.actualConfiguration, indexTile, 0);
        this.$set(this.actualConfiguration, indexEmpty, tile);

        if (JSON.stringify(this.actualConfiguration) === JSON.stringify(SOLUTION)) {
          this.stop();
          const name = window.prompt('Congratulation! You solve the puzzle! Name:') || 'User';
          this.results.push({ name, time: this.time });
        }
      }
    },
    tileNextToEmpty(tile) {
      const indexTile = this.actualConfiguration.indexOf(tile);
      const indexZero = this.actualConfiguration.indexOf(0);

      if (indexTile === (indexZero - 1) && ![0, 4, 8, 12].includes(indexZero)) {
        return true;
      } else if (indexTile === (indexZero + 1) && ![3, 7, 11, 15].includes(indexZero)) {
        return true;
      } else if (indexTile === (indexZero - 4)) {
        return true;
      } else if (indexTile === (indexZero + 4)) {
        return true;
      }
      return false;
    },
    randomNumber(array) {
      const copyArray = JSON.parse(JSON.stringify(array));
      let currentIndex = 1000;

      while (currentIndex !== 0) {
        const indexZero = copyArray.indexOf(0);
        const availableIndexes = [...this.findAvailableIndexes(indexZero)];
        const randomIndex = Math.floor(Math.random() * availableIndexes.length);
        const indexAvailable = availableIndexes[randomIndex];

        [copyArray[indexZero], copyArray[indexAvailable]] = [copyArray[indexAvailable], copyArray[indexZero]];

        currentIndex--;
      }

      return copyArray;
    },
    shuffle() {
      this.actualConfiguration = this.randomNumber(this.actualConfiguration);
      this.reset()
      this.start();
      this.moves = 0;
    },
    start() {
      if (this.running) return;

      if (this.timeBegan === null) {
        this.timeBegan = new Date();
      }

      this.started = setInterval(this.clockRunning, 10);
      this.running = true;
    },
    stop() {
      this.running = false;
      clearInterval(this.started);
    },
    reset() {
      this.running = false;
      clearInterval(this.started);
      this.timeBegan = null;
      this.time = "00:00:00.000";
    },
    clockRunning() {
      const currentTime = new Date()
      const timeElapsed = new Date(currentTime - this.timeBegan)
      const hour = timeElapsed.getUTCHours()
      const min = timeElapsed.getUTCMinutes()
      const sec = timeElapsed.getUTCSeconds()
      const ms = timeElapsed.getUTCMilliseconds();

      this.time = this.zeroPrefix(hour, 2) + ":" + this.zeroPrefix(min, 2) + ":" + this.zeroPrefix(sec, 2) + "." + this.zeroPrefix(ms, 3);
    },
    zeroPrefix(num, digit) {
      let zero = '';
      for (let i = 0; i < digit; i++) {
        zero += '0';
      }
      return (zero + num).slice(-digit);
    },
    findAvailableIndexes(indexZero) {
      if (indexZero === 0) {
        return [indexZero + 1, indexZero + 4];
      }
      if (indexZero === 3) {
        return [indexZero - 1, indexZero + 4];
      }
      if (indexZero === 12) {
        return [indexZero + 1, indexZero - 4];
      }
      if (indexZero === 15) {
        return [indexZero - 1, indexZero - 4];
      }
      if (indexZero < 3) {
        return [indexZero - 1, indexZero + 1, indexZero + 4];
      }
      if (indexZero > 12) {
        return [indexZero - 1, indexZero + 1, indexZero - 4];
      }
      if ([7, 11].includes(indexZero)) {
        return [indexZero - 1, indexZero - 4, indexZero + 4];
      }
      if ([4, 8].includes(indexZero)) {
        return [indexZero + 1, indexZero - 4, indexZero + 4];
      }
      return [indexZero + 1, indexZero - 1, indexZero - 4, indexZero + 4];
    },
  },
  computed: {
    sortedResults() {
      const copyArr = JSON.parse(JSON.stringify(this.results));
      copyArr.sort((a, b) => Number(a.time.split(':').join('')) - Number(b.time.split(':').join('')));
      return copyArr;
    },
  },
};
</script>

<style scoped>
  .board {
    width: 440px;
    height: 440px;
  }
 .tile {
   width: 100px;
   height: 100px;
   background-color: #f4db87;
   border-radius: 5px;
   font-size: 25px;
   margin: 5px;
   user-select: none;
 }
 .empty {
   opacity: 0;
 }
 .cursor-pointer {
   cursor: pointer;
 }
 .grey-background {
   background-color: #adadad;
 }
</style>