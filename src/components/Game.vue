<template>
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
            <div>
              <span>ゲームオーバー</span><br>
              <span>{{score}}点</span>
              <a class="twitter-share" :href="twitterShareUrl" target="_blank"><i class="fa fa-twitter"></i>点数をツイート</a>
            </div>
          </div>
        </div>
        <div class="field" :class="{gameover: gameover}">
          <div v-for="(row, key) in printField"  :key="key">
            <div class="field-block-wrap" v-for="(block, key) in row" :key="key">
              <block
                :letter="block.letter"
                :is-cursor="block.isCursor" :is-cleard="block.isCleard"></block>
            </div>
          </div>
        </div>
      </div>
      <div class="stocks-wrap">
        <div class="score">{{score}}点</div>
        <div class="stocks">
          <div class="stocks-title">NEXT</div>
          <div class="stocks-list">
            <block v-for="(letter, key) in printWaitingLetter" :key="key" :letter="letter"></block>
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
      <li>作者：<a href="https://twitter.com/the_minojiro" target="_blank">みのじろー</a></li>
    </ul>
  </div>
</template>

<script>
import Block from './Block'

export default {
  name: 'Game',
  data() {
    return {
      field: [],
      waitingLetters: [],
      score: -1,
      curLetter: '',
      curX: 3,
      curY: 0,
      gameover: false,
      pendingClearBlocks: [],
      unsortedVegetables: [
        'あすぱらがす',
        'いも',
        'うり',
        'かぼちや',
        'きやべつ',
        'くろやなぎてつこ',
        'ごぼう',
        'こまつな',
        'せろり',
        'せり',
        'だいこん',
        'たまねぎ',
        'とまと',
        'なす',
        'にんじん',
        'ねぎ',
        'はくさい',
        'ぱせり',
        'ぴーまん',
        'ぶろつこりー',
        'ほうれんそう',
        'まめ',
        'れたす',
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
    printWaitingLetter() {
      return this.waitingLetters.slice(0,5);
    },
    printField() {
      return this.field.map((row, y) => {
        return row.map((block, x) => {
          if (y === this.curY && x === this.curX) {
            return {
              letter: this.curLetter,
              isCursor: true
            };
          } else {
            return block;
          }
        }).slice(1, row.length - 1);
      }).slice(0, this.field.length - 1);
    },
    isStopped() {
      return !!this.pendingClearBlocks.length || this.gameover;
    },
    vegetables() {
      return this.unsortedVegetables.sort((a, b) => b.length - a.length);
    },
    allLetters() {
      return this.vegetables
        .map(o => o.split(''))
        .reduce((a, b) => a.concat(b))
        .filter((x, i, self) => self.indexOf(x) === i);
    },
    cleard() {
      return this.pendingClearBlocks[0] || null;
    },
    cleardVegetableName() {
      return this.cleard ? this.cleard.vegetable : null;
    },
    twitterShareUrl() {
      return `https://twitter.com/share?url=http://yasai-puyopuyo.s3-website-ap-northeast-1.amazonaws.com/&hashtags=やさいぷよぷよ&text=やさいぷよぷよで、${this.score}点とりました！`
    }
  },
  watch: {
    cleard() {
      if (!this.cleard) return;
      this.cleard.blocks.reduce((a, b) => a.concat(b)).forEach(block => {
        this.field[block.y][block.x].isCleard = true;
      });
      setTimeout(() => {
        this.pendingClearBlocks.shift();
        this.score += 20;
        if (!this.cleard) {
          this.closeEmptyBlocks();
          setTimeout(this.clearSentence, 10);
        }
      }, 500)
    }
  },
  methods: {
    initializeGame() {
      for (let i = 0; i < 2; i++) this.waitingLetters.push(this.generateRandomLetter())
      this.field = Array.from({length: 11}, _ => {
        return Array.from({length: 10}, _ => {
          return {letter: null};
        });
      });
      for(let i = 0; i < 3; i++) this.addLetterStock();
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
    closeEmptyBlocks() {
      const trans = this.arrTranspose(this.field).map(r => {
        r = r.filter(o => !o.isCleard)
        r = Array.from({length: 11 - r.length}, _ => {
          return {letter: null}
        }).concat(r)
        return r;
      });
      this.field = this.arrTranspose(trans);
    },
    arrTranspose(arr) {
      return arr[0].map((_, c) => arr.map(r => r[c]));
    },
    arrSample(arr) {
      return arr[Math.floor(Math.random() * arr.length)];
    },
    arrShuffle(arr) {
      return arr.sort(()=>Math.random() - .5);
    },
    fillEnclosure() {
      const lastKey = this.field.length - 1;
      this.field = this.field.map((row, key) => {
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
        this.field[this.curY][this.curX] = {letter: this.curLetter}
        this.clearSentence();
        this.curY = 0;
        this.curX = 3;
        if (this.field[this.curY][this.curX].letter){
          this.gameover = true;
          this.sendScore();
        } else {
          this.setNewLetter();
        }
      }
    },
    setCur(x, y) {
      if (this.field[y][x].letter) return false;
      this.curX = x;
      this.curY = y;
      return true;
    },
    clearSentence() {
      this.pendingClearBlocks = this.vegetables.map(vegetable => {
        const vegetableSplit = vegetable.split('');
        const blocks = vegetableSplit.map((letter) => {
          return this.field.map((row, y) => {
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
      this.curLetter = this.waitingLetters[0];
      this.waitingLetters.shift();
      if(this.waitingLetters.length < 6) {
        this.addLetterStock();
      }
    },
    addLetterStock() {
      this.waitingLetters = this.waitingLetters.concat(
        this.arrShuffle(
          this.arrSample(this.vegetables).split(''))
      );
    },
    generateRandomLetter() {
      return this.allLetters[Math.floor(Math.random() * this.allLetters.length)];
    },
    sendScore() {
      if(!window.gtag) return;
      gtag('event', 'gameend', {
        'event_category': 'yasai-puyopuyo',
        'event_label': 'gameend',
        'value': this.score,
      });
    },
  }
}
</script>

<style>
.game {
  max-width: 550px;
  margin: 100px auto 0;
  text-align: left;
}

.game-inner {
  display: flex;
  justify-content: space-between;
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

.field-block-wrap {
  width: calc(100% / 8);
}

.field-wrap {
  flex: 1;
  position: relative;
}

.field-over {
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  width: 100%;
  z-index: 1;
}

.field-over-inner {
  position: absolute;
  text-align: center;
  width: 100%;
  font-size: 36px;
  font-weight: bold;
  margin-top: -1.5em;
  top: 50%;
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

.links {
  margin: 0;
  padding: 0;
  margin-top: 40px;
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
  background: -moz-linear-gradient(top, rgba(102,102,102,1) 1%, rgba(50,50,50,1) 100%);
  background: -webkit-linear-gradient(top, rgba(102,102,102,1) 1%,rgba(50,50,50,1) 100%);
  background: linear-gradient(to bottom, rgba(102,102,102,1) 1%,rgba(50,50,50,1) 100%);
  color: #fff;
  list-style: none;
  font-size: 10px;
  margin: 0 1px;
  text-align: center;
  font-size: 20px;
  border-radius: 5px;
  padding: 15px 0 10px;
}

.score {
  font-size: 18px;
  font-weight: bold;
  text-align: center;
}

.twitter-share {
  display: block;
  margin: 0 20px;
  font-size: 15px;
  text-decoration: none;
  background: #1EA1F3;
  color: #fff;
  font-weight: bold;
  padding: 5px;
  border-radius: 5px;
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
