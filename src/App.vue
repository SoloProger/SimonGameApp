<template>
  <div id="app">
    <div :class="{ shake: isWrong, grow: isWon }">
      <div class="game" :class="{ fullSize: playing }">
        <div
          class="square"
          @click="clicked(1)"
          :class="{ lit: isLit[1] }"
        ></div>
        <div
          class="square"
          @click="clicked(2)"
          :class="{ lit: isLit[2] }"
        ></div>
        <div
          class="square"
          @click="clicked(3)"
          :class="{ lit: isLit[3] }"
        ></div>
        <div
          class="square"
          @click="clicked(4)"
          :class="{ lit: isLit[4] }"
        ></div>
        <div class="start" @click="startGame">
          {{ centerButton }}
          <span v-show="playing">{{ showScore }}</span>
        </div>
      </div>
    </div>
    <div v-show="playing" class="bottom-bar">
      <p>Mode:</p>
      <button :class="{ active: !strict }" @click="strict = false">
        Normal
      </button>
      <button :class="{ active: strict }" @click="strict = true">Strict</button>
    </div>
  </div>
</template>

<script>
import { Howl } from "howler";

export default {
  name: "App",
  components: {},
  data() {
    return {
      centerButton: "START",
      playing: false,
      isClickable: false,
      isWon: false,
      isWrong: false,
      strict: false,
      score: 0,
      sequence: [],
      sequenceInterval: null,
      playerSequence: [],
      sounds: [
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound1.mp3"],
        }),
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound2.mp3"],
        }),
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound3.mp3"],
        }),
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound4.mp3"],
        }),
      ],
      isLit: {
        1: false,
        2: false,
        3: false,
        4: false,
      },
    };
  },
  computed: {
    showScore() {
      if (this.isWon) {
        return "Play Again?";
      }
      return "Score: " + this.score;
    },
  },
  watch: {
    // if strict changes in middle of game, reset it
    strict() {
      this.startGame();
    },
  },
  methods: {
    startGame() {
      this.playing = true;
      this.sequence = [];
      this.playerSequence = [];
      this.centerButton = "RESET";
      this.isWon = false;
      this.isWrong = false;
      this.score = 0;
      clearInterval(this.sequenceInterval);
      this.showSequence();
    },
    clicked(tile) {
      if (this.isClickable) {
        this.playSound(tile);
        this.lightUp(tile);
        this.playerSequence.push(tile);
        this.checkWinLose();
      }
    },
    checkWinLose() {
      // check for incorrect
      for (let i = 0; i < this.playerSequence.length; i++) {
        if (this.playerSequence[i] !== this.sequence[i]) {
          if (this.strict) {
            this.centerButton = "Wrong!";
            this.isWrong = true;
            this.isClickable = false;
            return setTimeout(() => {
              this.startGame();
            }, 1000);
          }
          this.playerSequence = [];
          this.centerButton = "Wrong!";
          this.isWrong = true;
          setTimeout(() => {
            this.centerButton = "RESET";
            this.isWrong = false;
          }, 1000);
          this.showSequence(true);
        }
      }
      // if all correct and same length , continue
      if (this.playerSequence.length === this.sequence.length) {
        this.playerSequence = [];
        this.score++;
        // if win condition, show win, dont continue.
        if (this.score === 20) {
          this.centerButton = "Winner!";
          this.isClickable = false;
          this.isWon = true;
        } else {
          this.showSequence();
        }
      }
    },
    lightUp(tile) {
      this.isLit[tile] = true;
      setTimeout(() => {
        this.isLit[tile] = false;
      }, 300);
    },
    playSound(tile) {
      this.sounds[tile - 1].play();
    },
    showSequence(redo) {
      let currentIndex = 0;
      let speed = this.sequence.length === 0 ? 1000 : 700;
      this.isClickable = false;
      if (!redo) {
        // dont add number on incorrect answers
        this.sequence.push(Math.floor(Math.random() * 4 + 1));
      }
      this.sequenceInterval = setInterval(() => {
        if (currentIndex >= this.sequence.length) {
          clearInterval(this.sequenceInterval);
          return (this.isClickable = true);
        }
        this.playSound(this.sequence[currentIndex]);
        this.lightUp(this.sequence[currentIndex]);
        currentIndex++;
      }, speed);
    },
  },
};
</script>

<style lang="scss">
$bg-color: #343434;
$red: #aa2525;
$white: #fff;
$yellow: #aaaa25;
$green: #45aa25;
$blue: #2525aa;

@mixin center-align {
  align-items: center;
  display: flex;
  justify-content: center;
}

body {
  @include center-align();
  background: $bg-color;
  font-family: "Lato";
  font-weight: 900;
  font-size: 1.4em;
  height: 100vh;
  overflow: hidden;
}

.game {
  align-content: space-around;
  background: darken($bg-color, 8%);
  border-radius: 3%;
  display: flex;
  flex-wrap: wrap;
  height: 0;
  justify-content: space-around;
  width: 0;
  .start {
    @include center-align();
    align-content: center;
    flex-wrap: wrap;
    background: $white;
    border: 20px solid darken($bg-color, 5%);
    border-radius: 50%;
    cursor: pointer;
    position: absolute;
    height: 100px;
    left: 50%;
    top: 50%;
    user-select: none;
    transform: translate(-50%, -50%); // center origin
    width: 100px;
    span {
      font-size: 0.7em;
    }
  }
  .square {
    cursor: pointer;
    height: 40%;
    width: 40%;
    &:nth-of-type(1) {
      background: $green;
      &.lit {
        background: lighten($green, 25%);
      }
    }
    &:nth-of-type(2) {
      background: $red;
      &.lit {
        background: lighten($red, 25%);
      }
    }
    &:nth-of-type(3) {
      background: $yellow;
      &.lit {
        background: lighten($yellow, 25%);
      }
    }
    &:nth-of-type(4) {
      background: $blue;
      &.lit {
        background: lighten($blue, 25%);
      }
    }
  }
}

.bottom-bar {
  animation: moveup 1s ease-in forwards;
  background: darken($bg-color, 10%);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0.3em 0;
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;

  button {
    background: lighten($bg-color, 10%);
    color: $white;
    border: none;
    border-radius: 5px;
    outline: none;
    font-size: 0.6em;
    padding: 0.4em;
    &.active {
      background: $green;
    }
    &:nth-of-type(1) {
      margin-right: 0.3em;
    }
  }
  p {
    color: $white;
    margin: 0;
    font-size: 0.6em;
    display: inline;
    padding-right: 1em;
  }
}

// Animations
.shake {
  animation: shake 500ms linear forwards;
}

.fullSize {
  animation: fullSize 1s ease-out forwards;
}

.grow {
  animation: grow 1s linear infinite;
}

@keyframes fullSize {
  from {
    transform: rotate(-360deg);
  }
  to {
    height: 325px;
    width: 325px;
  }
}

@keyframes shake {
  0% {
    transform: rotate(-5deg);
  }
  25% {
    transform: rotate(5deg);
  }
  75% {
    transform: rotate(-5deg);
  }
  100% {
    transform: rotate(0deg);
  }
}

@keyframes grow {
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes moveup {
  from {
    transform: translateY(100px);
  }
  to {
  }
}

@media (max-width: 500px) {
  .game {
    transform: scale(0.8);
  }
}
</style>
