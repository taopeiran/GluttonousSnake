<template>
  <div class="container">
    <header>
      <p style="color: white;font-size:60px">贪吃蛇</p>
      <p style="color: white;font-size:40px"><span>难度：{{difficults[difficlutMod]}}</span>&nbsp;&nbsp;&nbsp;&nbsp;<span>分数：{{ count }}</span></p>
      <p class="difficult-change">
        <button class="btn" @click="changeMod(0)">简单</button>
        <button class="btn" @click="changeMod(1)">困难</button>
        <button class="btn" @click="changeMod(2)">地域</button>
      </p>
    </header>
    <main class="gamebox-container">
      <div class="gamebox">
        <span
          v-for="(item, index) in map"
          :key="index"
          :class="{
            isFood: item.isFood,
            isShen: item.isShen,
            isTou: item.isTou,
          }"
        ></span>
      </div>
    </main>
    <footer>
      <button class="btn play-button" @click="play">play</button>
      <button class="btn reset-button" @click="reset">reset</button>
    </footer>
  </div>
</template>

<script>
export default {
  data() {
    return {
      count: 0,
      difficults:["简单","困难","地狱"],
      difficlutMod:0,
      map: [],
      snake: [],
      direction: "right",
      timer: null,
      isPlaying: false,
      food: new Set(),
      speeds:[200,200,100],
      foodNums:[5,7,7]
    };
  },
  created() {
    this.initMap();
  },
  mounted() {
    this.renderSnake();
    this.addEvent();
  },
  unmounted(){
    this.removeEvent()
  },
  methods: {
    initMap() {
      this.map = [];
      this.food = new Set();
      for (let i = 0; i < 600; i++) {
        this.map.push({ isTou: false, isShen: false, isFood: false });
      }
      this.snake = [51, 52, 53, 54, 55, 56];
    },
    clearMap() {
      this.map.forEach((x) => {
        (x.isTou = false), (x.isShen = false), (x.isFood = false);
      });
    },
    renderSnake() {
      this.map[this.snake[this.snake.length - 1]].isTou = true;
      for (let i = 0; i < this.snake.length - 1; i++) {
        this.map[this.snake[i]].isShen = true;
      }
    },
    renderFood() {
      if (this.food.size == 0) {
        for (let i = 0; i < this.foodNums[this.difficlutMod]; i++) {
          this.makeFood();
        }
      } else {
        this.food.forEach((x) => {
          this.map[x].isFood = true;
        });
      }
    },
    makeFood() {
      let newFood = Math.floor(Math.random() * 600);
      while (this.snake.includes(newFood)) {
        newFood = Math.floor(Math.random() * 600);
      }
      this.food.add(newFood);
    },
    play() {
      if (this.isPlaying) {
        return;
      } else {
        this.isPlaying = true;
      }
      this.timer = setInterval(() => {
        let isMove = true;
        const Tou = this.snake[this.snake.length - 1];
        const judgeArr = [
          this.direction == "right" && (Tou + 1) % 40 == 0,
          this.direction == "up" && Tou - 40 < 0,
          this.direction == "down" && Tou + 40 > 600,
          this.direction == "left" && Tou % 40 == 0,
          this.snake.findIndex((value) => value == Tou) !=
            this.snake.length - 1,
        ];

        if (judgeArr.includes(true)) {
          isMove = false;
          this.clearTimer();
          this.isPlaying = false;
          alert("Gamer Over!\n"+"你的分数是："+this.count)
        }

        if (isMove) {
          this.clearMap();
          let nextTou = Tou;

          if (this.direction == "right") {
            nextTou = Tou + 1;
          } else if (this.direction == "left") {
            nextTou = Tou - 1;
          } else if (this.direction == "up") {
            nextTou = Tou - 40;
          } else if (this.direction == "down") {
            nextTou = Tou + 40;
          }

          this.snake.push(nextTou);
          if (this.food.has(nextTou)) {
            this.food.delete(nextTou);
            this.count = this.count + 1;
          } else {
            this.snake.splice(0, 1);
          }

          this.renderSnake();
          this.renderFood();
        }

        this.$forceUpdate();
      }, this.speeds[this.difficlutMod]);
    },
    clearTimer() {
      if (this.timer) {
        clearInterval(this.timer);
        this.timer = null;
      }
    },
    addEvent() {
      document.addEventListener("keydown", (e) => {
        // ←37|65，↑38|87，→39|68，↓40|83
        switch (e.keyCode) {
          case 37:
            if (this.direction == "right") break;
            this.direction = "left";
            break;
          case 38:
            if (this.direction == "down") break;
            this.direction = "up";
            break;
          case 39:
            if (this.direction == "left") break;
            this.direction = "right";
            break;
          case 40:
            if (this.direction == "up") break;
            this.direction = "down";
            break;
        }
      });
    },
    removeEvent() {
      document.removeEventListener("keydown");
    },
    reset() {
      if (this.isPlaying) {
        return;
      } else {
        this.initMap();
        this.renderSnake();
        this.direction = "right";
        this.count = 0
      }
    },
    changeMod(mod){
      if (this.isPlaying) {
        return;
      } else {
        this.difficlutMod = mod
      }
    }
  },
};
</script>

<style scoped>
.gamebox-container {
  display: flex;
  justify-content: center;
  margin-top: 50px;
}
.gamebox {
  width: 1200px;
  height: 450px;
  border: 1px solid rgba(255, 255, 255, 0.45);
}
.gamebox {
  display: grid;
  grid-template-columns: repeat(40, 1fr);
}
.gamebox span {
  width: 30px;
  height: 30px;
  box-shadow: 1px 1px 1px rgba(255, 255, 255, 0.25);
}
.isTou::after {
  content: "";
  display: inline-block;
  background: red;
  width: 30px;
  height: 30px;
  z-index: 4;
}
.isShen::after {
  content: "";
  display: inline-block;
  background: green;
  width: 30px;
  height: 30px;
  z-index: 3;
}
.isFood::after {
  content: "";
  display: inline-block;
  background: yellow;
  width: 30px;
  height: 30px;
  z-index: 2;
}
footer {
  margin-top: 40px;
}
.btn {
  width: 120px;
  height: 60px;
  border-radius: 20px;
  font-size: 20px;
  cursor: pointer;
  background: #fff;
}
.btn:active {
  background: #eee;
}
.play-button {
  margin-right: 50px;
}
.difficult-change{
  text-align: left;
  padding-left: 200px;
  padding-top: 40px;
}
.difficult-change .btn{
  margin-right: 40px;
}
.difficult-change .btn:nth-child(1){
  background: green;
}
.difficult-change .btn:nth-child(1):active{
  background: rgb(3, 115, 3);
}
.difficult-change .btn:nth-child(2){
  background: red;
}
.difficult-change .btn:nth-child(2):active{
  background: rgb(222, 4, 4);
}
.difficult-change .btn:nth-child(3){
  background: purple;
}
.difficult-change .btn:nth-child(3):active{
  background: rgb(108, 2, 108);
}
</style>
