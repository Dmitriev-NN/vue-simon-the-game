<template >
  <div class="wrapper">
    <div class="game__field">
      <div class="button__block">
        <div class="top button__row">
          <div
            id="blueButton"
            class="blue simonBtn"
            :class="{ active: opacityValue.blue }"
            @click="activateButton('blue')"
          ></div>
          <div
            id="redButton"
            class="red simonBtn"
            :class="{ active: opacityValue.red }"
            @click="activateButton('red')"
          ></div>
        </div>
        <div class="bottom button__row">
          <div
            id="yellowButton"
            class="yellow simonBtn"
            :class="{ active: opacityValue.yellow }"
            @click="activateButton('yellow')"
          ></div>
          <div
            id="greenButton"
            class="green simonBtn"
            :class="{ active: opacityValue.green }"
            @click="activateButton('green')"
          ></div>
        </div>
      </div>
    </div>
    <div class="menu">
      <h2 class="gameEnd" v-if="gameOver && start">Вы проиграли!</h2>
      <h1>Score: {{ score }}</h1>
      <button v-if="!start" @click="startClick">Start</button>
      <hr />
      <h2>Выберите сложность</h2>
      <div class="selectLevel">
        <div>
          <input
            type="radio"
            id="easy"
            name="level"
            value="easyLevel"
            v-model="currentLevel"
          />
          <label for="easy">Легкий</label>
        </div>
        <div>
          <input
            type="radio"
            id="medium"
            name="level"
            value="mediumLevel"
            v-model="currentLevel"
          />
          <label for="medium">Средний</label>
        </div>
        <div>
          <input
            type="radio"
            id="hard"
            name="level"
            value="hardLevel"
            v-model="currentLevel"
          />
          <label for="hard">Сложный</label>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      score: 0,
      gameArray: [],
      playerArray: [],
      start: false,
      gameOver: false,
      baseParamColor: {
        1: "blue",
        2: "red",
        3: "yellow",
        4: "green",
      },
      currentColor: "",
      opacityValue: {
        blue: false,
        red: false,
        yellow: false,
        green: false,
      },
      difficulty: {
        easyLevel: 1500,
        mediumLevel: 1000,
        hardLevel: 400,
      },
      currentLevel: "mediumLevel",
      isClickable: false,
    };
  },
  methods: {
    startClick() {
      this.start = true;
      this.gameOver = false;
      this.runSequence(this.getLevel);
    },
    // randomInteger(min, max) {
    randomInteger() {
      return Math.round(1 + Math.random() * (4 - 1));
    },
    runSequence(level) {
      setTimeout(() => {
        this.gameArray.push(this.randomInteger());
        this.gameArray.forEach((el, index) => {
          setTimeout(() => {
            let color = this.baseParamColor[el];
            this.opacityValue[color] = true; // Подсвечивание активной клавиши
            setTimeout(() => {
              // Позволяет адекватно воспроизводить последовательность нажатий клавиш (не все разом)
              this.opacityValue[color] = false;
              this.audioOnClick(color);
            }, 200);
          }, level * (index + 1)); // index + 1 так как массив начинается с нуля
        });
        // Весь блок обёрнут в ещё один setTimeout для добавления паузы после повторения игроком последовательности за компьютером
      }, 1000);
    },
    activateButton(color) {
      // Подсветка клавиши при нажатии, воспроизведение звука и проверка последовательности
      this.opacityValue[color] = true;
      this.isClickable = true;
      this.currentColor = color;
      setTimeout(() => {
        this.opacityValue[color] = false;
        this.isClickable = false;
        this.audioOnClick(color);
      }, 200);
    },
    getKeyByValue(object, value) {
      // Получение ключа (цвета) из объекта (baseParamColor)
      return Object.keys(object).find((key) => object[key] === value);
    },
    checkUserArray() {
      this.playerArray.forEach((el, index) => {
        if (el !== this.gameArray[index]) {
          this.gameOver = true;
          return;
        }
      });

      if (this.playerArray.length === this.gameArray.length && !this.gameOver) {
        this.playerArray.length = 0;
        this.score++;
        this.startClick();
      }
    },
    audioOnClick(color) {
      // Звуки при нажатии на один из кубиков
      let audio;
      switch (color) {
        case "blue":
          audio = new Audio(
            "https://s3.amazonaws.com/freecodecamp/simonSound1.mp3"
          );
          break;
        case "red":
          audio = new Audio(
            "https://s3.amazonaws.com/freecodecamp/simonSound2.mp3"
          );
          break;
        case "yellow":
          audio = new Audio(
            "https://s3.amazonaws.com/freecodecamp/simonSound3.mp3"
          );
          audio.play();
          break;
        case "green":
          audio = new Audio(
            "https://s3.amazonaws.com/freecodecamp/simonSound4.mp3"
          );
          break;
      }
      audio.play();
    },
  },
  watch: {
    currentLevel() {
      // Обнуление при изменении уровня сложности
      this.start = false;
      this.score = 0;
      this.gameArray.length = 0;
      this.playerArray.length = 0;
    },
    isClickable() {
      // Следим за кликами пользователя
      if (this.isClickable) {
        let value = this.getKeyByValue(this.baseParamColor, this.currentColor);
        this.playerArray.push(Number(value));
        this.checkUserArray();
      }
    },
    gameOver() {
      if (this.gameOver) {
        // Обнуление при проигрыше
        this.start = false;
        this.gameArray.length = 0;
        this.playerArray.length = 0;
        this.score = 0;
      }
    },
  },
  computed: {
    getLevel() {
      // Получение уровня из чекбоксов
      return this.difficulty[this.currentLevel];
    },
  },
  // Возможность играть на клавишах вместо мышки
  mounted() {
    const bBtn = document.getElementById("blueButton"),
      rBtn = document.getElementById("redButton"),
      yBtn = document.getElementById("yellowButton"),
      gBtn = document.getElementById("greenButton");

    document.addEventListener("keyup", (e) => {
      switch (e.code) {
        case "ArrowUp":
          bBtn.click();
          break;
        case "ArrowDown":
          gBtn.click();
          break;
        case "ArrowLeft":
          yBtn.click();
          break;
        case "ArrowRight":
          rBtn.click();
          break;

        default:
          break;
      }
    });
  },
};
</script>

<style scoped>
/* =================================> ИГРОВОЕ ПОЛЕ*/
.game__field {
  background-color: black;
  border-radius: 100%;
  padding: 30px;
}
.button__block {
  display: flex;
  flex-direction: column;
  transform: rotate(45deg);
}
.button__row {
  display: flex;
  flex-direction: row;
}

.simonBtn {
  width: 300px;
  height: 300px;
  opacity: 0.3;
  cursor: pointer;
  border: 0 solid;
  border-radius: 20px;
  margin: 15px;
}

/* =================================> КНОПКИ*/
.blue {
  background-color: blue;
  border-top-left-radius: 300px;
  order: 1;
}
.red {
  background-color: red;
  border-top-right-radius: 300px;
  order: 2;
}
.yellow {
  background-color: yellow;
  border-bottom-left-radius: 300px;
  order: 3;
}
.green {
  background-color: green;
  border-bottom-right-radius: 300px;
  order: 4;
}

/* =================================> АКТИВАЦИЯ КЛАВИШ */
/* =================================> ИНТЕРАКТИВНЫЕ СОБЫТИЯ */
.active {
  opacity: 1;
  transition: 200ms;
}

.gameEnd {
  color: red;
  font-size: 30px;
}

/* =================================> ВЁРСТКА ИГРОВОГО ПОЛЯ И МЕНЮ*/

.wrapper {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
}

.menu {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.levelSelect {
  display: flex;
}

h1 {
  font-size: 50px;
}

button {
  font-size: 40px;
  width: 200px;
  height: 200px;
  cursor: pointer;
}
</style>