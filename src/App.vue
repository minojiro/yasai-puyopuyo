<template>
  <div id="app">
    <div class="game">
      <div class="game-inner">
        <div class="field-wrap">
          <div class="field-over" v-if="cleardVegetableName">
            <div class="field-over-inner">
              <span>{{cleardVegetableName}}</span>
            </div>
          </div>
          <div class="field-over" v-if="gameover">
            <div class="field-over-inner">
              <span>ゲームオーバー</span>
            </div>
          </div>
          <div class="field" :class="{gameover: gameover}">
            <div v-for="(row, key) in printBlocks"  :key="key">
              <block v-for="(block, key) in row" :key="key" :letter="block.letter" :cur="block.isCur" :cleard="block.cleard"></block>
            </div>
          </div>
        </div>
        <div class="stocks-wrap">
          <div class="score">score:{{score}}</div>
          <div class="stocks">
            <div class="stocks-title">NEXT</div>
            <div class="stocks-list">
              <block v-for="(letter, key) in letterStock" :key="key" :letter="letter" full="true"></block>
            </div>
          </div>
        </div>
      </div>
      <ul class="controller">
        <li @click="moveCursor({y:0, x:-1 })"><i class="fa fa-angle-left"></i></li>
        <li @click="moveCursor({y:1, x:0 })"><i class="fa fa-angle-down"></i></li>
        <li @click="moveCursor({y:0, x:1 })"><i class="fa fa-angle-right"></i></li>
      </ul>
      <ul class="links">
        <li>元ネタ：<a href="https://twitter.com/jagarikin/status/1032816709088858113" target="_blank">じゃがりきん(@jagarikin)さんのツイート</a></li>
      </ul>
    </div>
  </div>
</template>

<script>
import Block from './components/Block'

export default {
  name: 'App',
  data() {
    return {
      blocks: [],
      letterStock: ['た', 'ま', 'ね', 'ぎ', ], // todo: set empty
      score: -1,
      curLetter: '',
      curX: 3,
      curY: 0,
      gameover: false,
      cleardStock: [],
      vegetables: [
        'たまねぎ',
        'ぴーまん',
        'きゆうり',
        'くろやなぎてつこ',
        'はくさい',
        'きやべつ',
        'じやがいも',
      ],
    }
  },
  components: {
    Block
  },
  mounted() {
    this.allocateKeyboardEvents();
    this.initializeGame();
  },
  computed: {
    printBlocks() {
      return this.blocks.map((row, y) => {
        return row.map((o, x) => {
          if (y === this.curY && x === this.curX) {
            return {
              letter: this.curLetter,
              isCur: true
            };
          } else {
            return o;
          }
        }).slice(1, row.length - 1);
      }).slice(0, this.blocks.length - 1);
    },
    isStopped() {
      return !!this.cleardStock.length || this.gameover;
    },
    allLetters() {
      return this.vegetables
        .map(o => o.split(''))
        .reduce((a, b) => a.concat(b))
        .filter((x, i, self) => self.indexOf(x) === i);
    },
    cleard() {
      return this.cleardStock[0] || null;
    },
    cleardVegetableName() {
      return this.cleard ? this.cleard.vegetable : null;
    },
  },
  watch: {
    cleard() {
      if (!this.cleard) return;
      this.cleard.blocks.reduce((a, b) => a.concat(b)).forEach(block => {
        this.blocks[block.y][block.x].cleard = true;
      });
      setTimeout(() => {
        this.cleardStock.shift();
        this.score += 20;
        if (!this.cleard) {
          this.fillEmptyBlocks();
          setTimeout(this.clearSentence, 10);
        }
      }, 500)
    }
  },
  methods: {
    initializeGame() {
      for (let i = 0; i < 2; i++) this.letterStock.push(this.generateRandomLetter())
      this.blocks = Array.from({length: 11}, _ => {
        return Array.from({length: 10}, _ => {
          return {letter: null};
        });
      });
      this.setNewLetter();
      this.fillEnclosure();
      this.fallTimer();
    },
    allocateKeyboardEvents() {
      const keymap = {
        '39': { x: 1, y: 0},
        '37': { x: -1, y: 0},
        '40': { x: 0, y: 1},
      };
      window.onkeydown = (kc) => {
        if (keymap[kc.keyCode]) this.moveCursor(keymap[kc.keyCode]);
      };
    },
    fillEmptyBlocks() {
      const trans = this.arrTranspose(this.blocks).map(r => {
        r = r.filter(o => !o.cleard)
        r = Array.from({length: 11 - r.length}, _ => {
          return {letter: null}
        }).concat(r)
        return r;
      });
      this.blocks = this.arrTranspose(trans);
    },
    arrTranspose(a) {
      return a[0].map((_, c) => a.map(r => r[c]));
    },
    fillEnclosure() {
      const lastKey = this.blocks.length - 1;
      this.blocks = this.blocks.map((row, key) => {
        if (key === lastKey) {
          return Array.from({length: 10}, (v, k) => {
            return {letter: '/'};
          });
        } else {
          row[0] = row[9] = {letter: '/'};
          return row;
        }
      })
    },
    moveCursor(dir) {
      if (!this.isStopped && !this.setCur(this.curX + dir.x, this.curY + dir.y) && dir.y > 0) {
        this.blocks[this.curY][this.curX] = {letter: this.curLetter}
        this.clearSentence();
        this.curY = 0;
        this.curX = 3;
        if (this.blocks[this.curY][this.curX].letter) this.gameover = true;
        this.setNewLetter();
      }
    },
    setCur(x, y) {
      if (this.blocks[y][x].letter) return false;
      this.curX = x;
      this.curY = y;
      return true;
    },
    clearSentence() {
      this.cleardStock = this.vegetables.map(vegetable => {
        const vegetableSplit = vegetable.split('');
        const blocks = vegetableSplit.map((letter) => {
          return this.blocks.map((row, y) => {
            return row.map((col, x) => {
              return col.letter === letter ? [{x, y}] : null;
            })
          }).reduce((a, b) => a.concat(b)).filter(o => !!o);
        }).reduce((pLets, nLets, idx) => {
          let r = [];
          nLets.forEach(nLet => {
            pLets.forEach(pLet => {
              const lastNLet = nLet[nLet.length - 1];
              const lastPLet = pLet[pLet.length - 1];
              const isClose = Math.abs(lastPLet.x - lastNLet.x) + Math.abs(lastPLet.y - lastNLet.y) === 1;
              if (isClose) {
                r.push(pLet.concat({x: lastNLet.x, y: lastNLet.y}));
              }
            });
          });
          return r;
        })
        return {vegetable, blocks};
      }).filter(o => !!o.blocks.length);
      return !!this.cleard;
    },
    fallTimer() {
      this.moveCursor({y: 1, x: 0});
      setTimeout(this.fallTimer, 500);
    },
    setNewLetter() {
      this.score++;
      this.curLetter = this.letterStock[0];
      this.letterStock.shift();
      this.letterStock.push(this.generateRandomLetter());
    },
    generateRandomLetter() {
      return this.allLetters[Math.floor(Math.random() * this.allLetters.length)];
    },
  }
}
</script>

<style>
body {
  background: #D5D6D9;
  padding: 10px;
}

.field {
  margin: 0;
  padding: 0;
  list-style: none;
  background: #A8A291;
}

.field>div {
  display: flex;
}

.field.gameover {
  opacity: 0.5;
  -webkit-filter: grayscale(100%);
  -moz-filter: grayscale(100%);
  -ms-filter: grayscale(100%);
  -o-filter: grayscale(100%);
  filter: grayscale(100%);
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

.game {
  max-width: 550px;
  margin: 100px auto 0;
  text-align: left;
}

.game-inner {
  display: flex;
  justify-content: space-between;
}

.field-wrap {
  flex: 1;
  position: relative;
}

.stocks-wrap {
  width: 20%;
  margin-left: 20px;
}

.stocks-title {
  text-align: center;
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 20px;
}

.stocks-list {
  width: 50px;
  margin: 0 auto;
}

.field-over {
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  width: 100%;
}

.field-over-inner {
  position: absolute;
  text-align: center;
  width: 100%;
  font-size: 36px;
  font-weight: bold;
  margin-top: -0.5em;
  top: 50%;
}

.links {
  margin: 0;
  padding: 0;
  margin-top: 30px;
}

.links>li {
  list-style: none;
  padding-left: 1.5em;
  margin: 0;
  font-size: 12px;
  position: relative;
}

.links>li:before {
  font-family: FontAwesome;
  content: "\f0c1";
  left: 0;
  top: 0;
  position: absolute;
}

.controller {
  display: none;
  margin-top: 20px;
  margin: 0;
  padding: 0;
}

.controller>li {
  flex: 1;
  background: #444;
  color: #fff;
  list-style: none;
  font-size: 10px;
  margin: 0 1px;
  text-align: center;
  font-size: 20px;
  padding: 10px;
}

.score {
  font-size: 18px;
  font-weight: bold;
  text-align: center;
}

@media screen and (max-width: 500px) {
  .game {
    margin-top: 0;
  }
  .game-inner {
    display: block;
  }
  .stocks-wrap {
    width: 100%;
    margin-left: 0;
  }
  .stocks {
    display: flex;
    align-items: center;
  }
  .stocks-title {
    margin-bottom: 0;
    margin-right: 10px;
  }
  .stocks-list {
    width: 100%;
    padding: 5px;
    display: flex;
  }
  .controller {
    display: flex;
  }
}
</style>
