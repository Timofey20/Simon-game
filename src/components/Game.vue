<template>
  <div id="app">
    <audio
      ref="sound1"
      src="https://raw.githubusercontent.com/kellyk/javascript-simon/master/sounds/1.mp3"
    ></audio>
    <audio
      ref="sound2"
      src="https://raw.githubusercontent.com/kellyk/javascript-simon/master/sounds/2.mp3"
    ></audio>
    <audio
      ref="sound3"
      src="https://raw.githubusercontent.com/kellyk/javascript-simon/master/sounds/3.mp3"
    ></audio>
    <audio
      ref="sound4"
      src="https://raw.githubusercontent.com/kellyk/javascript-simon/master/sounds/4.mp3"
    ></audio>
    <div id="simon">
      <div id="center">
        <div id="title">
          <h1>Simon</h1>
        </div>
        <div id="controls">
          <a id="start" class="btn" @click="start"></a>
          <div id="counter">
            <span
              :class="{ lit: power }"
              v-text="showError ? '! !' : showWin ? '* *' : displayCount"
            ></span>
          </div>
        </div>
        <div id="power">
          <p>Off</p>
          <div id="switch" :class="{ on: power }" @click="togglePower"></div>
          <p>On</p>
        </div>
      </div>
      <div id="buttons">
        <div
          class="button"
          :class="{ highlight: hlGreen }"
          id="green"
          @click="input(1)"
        ></div>
        <div
          class="button"
          :class="{ highlight: hlRed }"
          id="red"
          @click="input(2)"
        ></div>
        <div
          class="button"
          :class="{ highlight: hlYellow }"
          id="yellow"
          @click="input(3)"
        ></div>
        <div
          class="button"
          :class="{ highlight: hlBlue }"
          id="blue"
          @click="input(4)"
        ></div>
      </div>
    </div>
    <div id="speed">
      <div>
        <input
          type="radio"
          id="radio-1"
          class="radio"
          name="speed"
          value="1500"
          v-model="delay"
        />
        <label class="text" for="radio-1">1,5 sec</label>
      </div>
      <div>
        <input
          type="radio"
          id="radio-2"
          class="radio"
          name="speed"
          value="1000"
          v-model="delay"
        />
        <label class="text" for="radio-2">1,0 sec</label>
      </div>
      <div>
        <input
          type="radio"
          id="radio-3"
          class="radio"
          name="speed"
          value="400"
          v-model="delay"
        />
        <label class="text" for="radio-3">0,4 sec</label>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "game",
  data: function () {
    return {
      power: false,
      started: false,
      count: 0,
      series: [],
      playingSeries: false,
      userInput: [],
      hlRed: false,
      hlGreen: false,
      hlYellow: false,
      hlBlue: false,
      allowInput: false,
      showError: false,
      showWin: false,
      delay: 400,
      totalToWin: 20,
    };
  },
  computed: {
    displayCount() {
      if (this.count == 0) return "- -";
      else return this.count;
    },
  },
  methods: {
    reset() {
      this.started = false;
      this.count = 0;
      this.series = [];
      this.userInput = [];
      this.allowInput = false;
      this.playingSeries = false;
      this.showError = false;
      this.showWin = false;
      this.hlGreen = false;
      this.hlRed = false;
      this.hlYellow = false;
      this.hlBlue = false;
      this.delay = 400;
    },
    winner() {
      this.showWin = true;
      let self = this;
      let delay = 400;
      this.hlGreen = true;
      for (let x = 1; x <= 6; x++) {
        setTimeout(function () {
          self.playTone(1);
        }, delay);
        if (x == 6)
          setTimeout(function () {
            self.hlGreen = false;
            self.showWin = false;
            self.reset();
          }, delay + 400);
        delay += 400;
      }
    },
    input(tone) {
      if (!this.allowInput) return;
      this.playTone(tone);
      this.userInput.push(tone);
      if (
        this.userInput[this.userInput.length - 1] !=
        this.series[this.userInput.length - 1]
      ) {
        this.userInput = [];
        this.allowInput = false;
        this.showError = true;
        return;
      }
      if (this.userInput.length == this.series.length) {
        let self = this;
        this.userInput = [];
        if (this.series.length == this.totalToWin) {
          setTimeout(this.winner, 1000);
        } else {
          setTimeout(function () {
            self.addTone();
            self.playSeries();
          }, 1000);
        }
      }
    },
    togglePower() {
      this.power = !this.power;
      if (!this.power) {
        this.reset();
        this.stopTones();
      }
    },

    start() {
      if (this.power && this.count == 0) {
        this.started = true;
        this.addTone();
        this.playSeries();
      }
    },
    addTone() {
      this.count++;
      this.series.push(this.randomTone());
    },
    playSeries() {
      let delay = +this.delay;
      let plusDelay = +this.delay;
      this.allowInput = false;
      this.playingSeries = true;
      let self = this;
      this.series.forEach(function (tone, index, array) {
        if (index == array.length - 1)
          setTimeout(function () {
            if (self.started) {
              self.playTone(tone);
              self.allowInput = true;
              self.playingSeries = false;
            }
          }, delay);
        else
          setTimeout(function () {
            self.playTone(tone);
          }, delay);
        delay += plusDelay;
      });
      this.playingSeries = false;
    },
    clearHighlights() {
      this.hlGreen = false;
      this.hlRed = false;
      this.hlYellow = false;
      this.hlBlue = false;
    },
    playTone(tone) {
      if (!this.power) return;
      switch (tone) {
        case 1:
          this.$refs.sound1.pause();
          this.$refs.sound1.currentTime = 0;
          this.$refs.sound1.play();
          this.hlGreen = true;
          break;
        case 2:
          this.$refs.sound2.pause();
          this.$refs.sound2.currentTime = 0;
          this.$refs.sound2.play();
          this.hlRed = true;
          break;
        case 3:
          this.$refs.sound3.pause();
          this.$refs.sound3.currentTime = 0;
          this.$refs.sound3.play();
          this.hlYellow = true;
          break;
        case 4:
          this.$refs.sound4.pause();
          this.$refs.sound4.currentTime = 0;
          this.$refs.sound4.play();
          this.hlBlue = true;
          break;
      }
      if (!this.showWin) setTimeout(this.clearHighlights, 500);
    },
    stopTones() {
      this.$refs.sound1.pause();
      this.$refs.sound2.pause();
      this.$refs.sound3.pause();
      this.$refs.sound4.pause();
      this.$refs.sound1.currentTime = 0;
      this.$refs.sound2.currentTime = 0;
      this.$refs.sound3.currentTime = 0;
      this.$refs.sound4.currentTime = 0;
    },
    randomTone() {
      return Math.floor(Math.random() * 4) + 1;
    },
  },
};
</script>

<style scoped lang="scss">
$color-black: #000;
$color-dark-gray: #2e2e2e;
$color-med-gray: #666;
$color-light-gray: #ccc;
$color-green: darken(green, 5%);
$color-red: darken(red, 10%);
$color-yellow: darken(#ffd700, 10%);
$color-blue: darken(#0000cd, 5%);
$color-white: #fff;
$color-light-green: lighten(green, 5%);
$color-light-red: lighten(red, 5%);
$color-light-blue: lighten(#0000cd, 5%);
$color-light-yellow: lighten(#ffd700, 5%);
$color-bisque: #ffe4c4;

body {
  font-family: "Open Sans", sans-serif;
  user-select: none;
  background-color: $color-bisque;
}
* {
  margin: 0;
  padding: 0;
  font-family: "Open Sans", sans-serif;
}
h1 {
  font-family: "Russo One", sans-serif;
  font-size: 3rem;
  cursor: default;
  margin: 1rem 0;
}
#simon {
  margin: 2rem auto 0 auto;
  max-width: 680px;
  min-width: 680px;
  max-height: 680px;
  min-height: 680px;
  background-color: $color-dark-gray;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}
#buttons {
  display: flex;
  flex-wrap: wrap;
  max-width: 620px;
  min-width: 620px;
  justify-content: space-between;
}
.button {
  cursor: pointer;
  min-width: 300px;
  min-height: 300px;
  max-width: 300px;
  max-height: 300px;
}
#green {
  background-color: $color-green;
  border-top-left-radius: 300px;
  margin-bottom: 20px;
}
#blue {
  background-color: $color-blue;
  border-bottom-right-radius: 300px;
}
#yellow {
  background-color: $color-yellow;
  border-bottom-left-radius: 300px;
}
#red {
  background-color: $color-red;
  border-top-right-radius: 300px;
  margin-bottom: 20px;
}
#green.highlight {
  background-color: $color-light-green;
}
#red.highlight {
  background-color: $color-light-red;
}
#blue.highlight {
  background-color: $color-light-blue;
}
#yellow.highlight {
  background-color: $color-light-yellow;
}

#center {
  border: 20px solid $color-dark-gray;
  box-sizing: border-box;
  position: absolute;
  min-width: 300px;
  max-width: 300px;
  min-height: 300px;
  max-height: 300px;
  border-radius: 50%;
  background-color: $color-light-gray;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  color: $color-dark-gray;
}
#title {
  margin-top: 20px;
  text-align: center;
  background-color: $color-light-gray;
  width: 200px;
  border-top-left-radius: 50%;
  border-top-right-radius: 50%;
}
#controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  min-width: 180px;
  margin-top: 0.5rem;
}
#counter {
  display: inline-block;
  width: 60px;
  height: 40px;
  color: $color-med-gray;
  background-color: $color-black;
  border: 4px solid $color-dark-gray;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  font-size: 1.6rem;
}
.lit {
  color: $color-red;
}
#power {
  margin-top: 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 110px;
  font-size: 0.875rem;
  text-transform: uppercase;
  font-weight: bold;
}
#switch {
  cursor: pointer;
  display: inline-block;
  width: 50px;
  height: 25px;
  background-color: $color-dark-gray;
  position: relative;
}
#switch:after {
  content: "";
  position: absolute;
  top: 0;
  height: 19px;
  width: 19px;
  border: 3px solid $color-dark-gray;
  background-color: $color-red;
}
#switch.off:after {
  left: 0;
}
#switch.on:after {
  right: 0;
}
.btn {
  cursor: pointer;
  position: relative;
  display: inline-block;
  width: 24px;
  height: 24px;
  background-color: $color-yellow;
  border-radius: 50%;
  border: 4px solid $color-dark-gray;
  margin-top: 1rem;
}
.btn::after {
  position: absolute;
  top: -20px;
  left: -25%;
  font-size: 12px;
  text-transform: uppercase;
  font-weight: bold;
}
.btn:hover {
  background-color: $color-light-yellow;
}
#start.btn {
  background-color: $color-red;
}
#start.btn:hover {
  background-color: $color-light-red;
}
#speed {
  display: flex;
  justify-content: space-around;
  align-items: center;
  flex-direction: column;
  border: 5px solid #2e2e2e;
  border-radius: 10px;
  max-width: 500px;
  margin: 30px auto;
}
#speed > div {
  margin: 0.5rem 0px;
}

.text {
  font-family: "Russo One", sans-serif;
  font-size: 1.3rem;
  cursor: default;
  margin: 1rem 0;
  color: $color-dark-gray;
}
#start::after {
  content: "Start";
}
</style>
