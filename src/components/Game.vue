<template>
<div>
  <div id="game">
    <!-- 底盘，占用格子 -->
    <div v-for="(num, row) in nums">
      <div v-for="(n, col) in num" class="card"></div>
    </div>
    <span v-for="item in items" v-bind:class="itemClass(item)">{{item.text}}
    </span>
  </div>
  <!-- <el-tabs v-model="activeName" @tab-click="handleClick" id="tabs">
    <el-tab-pane label="我的信息" name="info"> 我的信息
    </el-tab-pane>
    <el-tab-pane label="开始游戏" name="game">
      <div id="game">
        <div v-for="(num, row) in nums">
          <div v-for="(n, col) in num" class="card">
            <div class="box-card" v-bind:class="numClass(row, col)">
              <div class="text item">
                {{ n ? n : '' }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </el-tab-pane>
    <el-tab-pane label="英雄榜" name="ranklist">英雄榜</el-tab-pane>
  </el-tabs> -->
</div>
</template>

<script>
var Item = function(row, col, text) {
  this.text = text || 0;
  this.row = row;
  this.col = col;
  this.oldRow = -1;
  this.oldCol = -1;
  this.id = Item.id++;
};
Item.id = 0;

export default {
  data() {
    return {
      COUNT: 4,
      activeName: 'game',
      nums: [
        [0, 0, 0, 0],
        [0, 0, 0, 0],
        [0, 0, 0, 0],
        [0, 0, 0, 0],
      ],
      items: [],
    };
  },
  beforeCreate() {
    // 插入动画的css代码
    var style = document.createElement('style');
    style.type = 'text/css';
    const COUNT = 4;
    const WIDTH = 100;
    const GAP = 8;
    for (var index = 0; index < COUNT - 1; index++) {
      for (var next = index + 1; next < COUNT; next++) {
        var keyframeName, keyframe, className, gapWidth;
        gapWidth = (next - index) * GAP;

        keyframeName = `row${index}_to_row${next}`;
        keyframe = `@-webkit-keyframes ${keyframeName}{from {left:${index*WIDTH+GAP}px;} to {left:${next*WIDTH + gapWidth}px;}}`;
        style.innerHTML += keyframe;
        style.innerHTML += `.${keyframeName} { animation: ${keyframeName} 100ms linear 1ms 1 normal forwards; }`

        keyframeName = `row${next}_to_row${index}`;
        keyframe = `@-webkit-keyframes ${keyframeName}{from {left:${next*WIDTH + gapWidth}px;} to {left:${index*WIDTH}px;}}`;
        style.innerHTML += keyframe;
        style.innerHTML += `.${keyframeName} { animation: ${keyframeName} 100ms linear 1ms 1 normal forwards; }`

        keyframeName = `col${index}_to_col${next}`;
        keyframe = `@-webkit-keyframes ${keyframeName}{from {top:${index*WIDTH+GAP}px;} to {top:${next*WIDTH + gapWidth}px;}}`;
        style.innerHTML += keyframe;
        style.innerHTML += `.${keyframeName} { animation: ${keyframeName} 100ms linear 1ms 1 normal forwards; }`

        keyframeName = `col${next}_to_col${index}`;
        keyframe = `@-webkit-keyframes ${keyframeName}{from {top:${next*WIDTH + gapWidth}px;} to {top:${index*WIDTH+GAP}px;}}`;
        style.innerHTML += keyframe;
        style.innerHTML += `.${keyframeName} { animation: ${keyframeName} 100ms linear 1ms 1 normal forwards; }`
        console.log(`.${keyframeName} { animation: ${keyframeName} 100ms linear 1ms 1 normal forwards; }`);
      }
    }
    for (var row = 0; row < COUNT; row++) {
      for (var col = 0; col < COUNT; col++) {
        className = `.pos_row${row}_col${col}{ top:${row*WIDTH+(row+1)*GAP}px; left:${col*WIDTH+(col+1)*GAP}px;}`;
        style.innerHTML += className;
      }
    }
    document.getElementsByTagName('head')[0].appendChild(style);
  },
  methods: {
    initNum() {
      this.nums.forEach((n, row) => {
        n.fill(0);
      })
      this.nums.forEach((a, row) => {
        a.forEach((b, col) => {
          this.nums[row][col] = Math.pow(2, parseInt(Math.random() * 11));
          if (this.nums[row][col] >= 2) {
            var item = new Item(row, col, this.nums[row][col]);
            this.items.push(item);
          }
        })
      })
      //   this.updateNums(parseInt(Math.random() * 100) % 4, parseInt(Math.random() * 100) % 4, 2);
      this.updateNums(0, 0, 2);
    },
    updateNums(row, col, newValue) {
      this.$set(this.nums[row], col, newValue);
    },
    randNum() {
      var success = false;
      var emptyPos = [];
      this.nums.forEach((a, row) => {
        a.forEach((b, col) => {
          if (b === 0) {
            emptyPos.push({
              row: row,
              col: col,
            })
          }
        })
      })
      if (emptyPos.length > 0) {
        var index = parseInt(Math.random() * 100) % emptyPos.length
        var pos = emptyPos[index];
        this.updateNums(pos.row, pos.col, (index % 2) ? (2) : (4));
        success = true;
      } else {

      }
      return success;
    },
    handleClick(tab, event) {},
    numClass(r, l) {
      var o = {};
      o['num-' + this.nums[r][l]] = true;
      return o;
    },
    itemClass(item) {
      var classArray = ['item'];
      classArray.push('num-' + item.text);
      classArray.push('pos_row' + item.row + '_col' + item.col);
      return classArray.join(' ');
    },
    left() {
      var nums = this.nums;
      let COUNT = 4;
      for (let row = 0; row < COUNT; row++) {
        let merge = false;
        // 如果有数字相同，先合并
        for (let col = 0; col < COUNT - 1 && (!merge); col++) {
          let curNum = nums[row][col];
          if (curNum > 0) {
            let gap = false;
            for (let next = col + 1; next < COUNT; next++) {
              let nextNum = nums[row][next];
              if (curNum === nextNum && (!gap)) {
                this.updateNums(row, col, curNum * 2);
                this.updateNums(row, next, 0);
                merge = true;
                break;
              } else {
                gap = nextNum > 0;
              }
            }
          }
        }
        // 合并完成，将数字掉落到正确的位置
        for (let col = 0; col < COUNT; col++) {
          let curNum = nums[row][col];
          if (curNum > 0) {
            let dropIndex = -1;
            for (let next = col - 1; next >= 0; next--) {
              let nextNum = nums[row][next];
              if (nextNum === 0) {
                dropIndex = next;
              } else {
                break;
              }
            }
            if (dropIndex >= 0) {
              this.updateNums(row, dropIndex, curNum);
              this.updateNums(row, col, 0);
            }
          }
        }
      }
      this.randNum();
    },
    up() {
      var nums = this.nums;
      var COUNT = 4;
      for (let col = 0; col < COUNT; col++) {
        let merge = false;
        // 如果有数字相同，先合并
        for (let row = 0; row < COUNT && (!merge); row++) {
          let curNum = nums[row][col];
          if (curNum > 0) {
            let gap = false;
            for (let next = row + 1; next < COUNT; next++) {
              let nextNum = nums[next][col];
              if (curNum === nextNum && (!gap)) {
                this.updateNums(next, col, curNum * 2);
                this.updateNums(row, col, 0);
                merge = true;
                break;
              } else {
                gap = nextNum > 0;
              }
            }
          }
        }
        // 合并完成，将数字掉落到正确的位置
        for (let row = 0; row < COUNT; row++) {
          let curNum = nums[row][col];
          if (curNum > 0) {
            let dropIndex = -1;
            for (let next = row - 1; next >= 0; next--) {
              let nextNum = nums[next][col];
              if (nextNum === 0) {
                dropIndex = next;
              } else {
                break;
              }
            }
            if (dropIndex >= 0) {
              this.updateNums(dropIndex, col, curNum);
              this.updateNums(row, col, 0);
            }
          }
        }
      }
      this.randNum();
    },
    right() {
      var nums = this.nums;
      var COUNT = 4;
      for (let row = 0; row < COUNT; row++) {
        let merge = false;
        // 如果有数字相同，先合并
        for (let col = COUNT - 1; col >= 1 && (!merge); col--) {
          let curNum = nums[row][col];
          if (curNum > 0) {
            let gap = false;
            for (let next = col - 1; next >= 0; next--) {
              let nextNum = nums[row][next];
              if (curNum === nextNum && (!gap)) {
                this.updateNums(row, col, curNum * 2);
                this.updateNums(row, next, 0);
                merge = true;
                break;
              } else {
                gap = nextNum > 0;
              }
            }
          }
        }
        // 合并完成，将数字掉落到正确的位置
        for (let col = COUNT - 1; col >= 0; col--) {
          let curNum = nums[row][col];
          if (curNum > 0) {
            let dropIndex = -1;
            for (let next = col + 1; next < COUNT; next++) {
              let nextNum = nums[row][next];
              if (nextNum === 0) {
                dropIndex = next;
              } else {
                break;
              }
            }
            if (dropIndex >= 0) {
              this.updateNums(row, dropIndex, curNum);
              this.updateNums(row, col, 0);
            }
          }
        }
      }
      this.randNum();
    },
    down() {
      var nums = this.nums;
      var COUNT = 4;
      for (let col = 0; col < COUNT; col++) {
        let merge = false;
        // 如果有数字相同，先合并
        for (let row = COUNT - 1; row >= 0 && (!merge); row--) {
          let curNum = nums[row][col];
          if (curNum > 0) {
            let gap = false;
            for (let next = row - 1; next >= 0; next--) {
              let nextNum = nums[next][col];
              if (curNum === nextNum && (!gap)) {
                this.updateNums(next, col, curNum * 2);
                this.updateNums(row, col, 0);
                merge = true;
                break;
              } else {
                gap = nextNum > 0;
              }
            }
          }
        }
        // 合并完成，将数字掉落到正确的位置
        for (let row = 0; row < COUNT; row++) {
          let curNum = nums[row][col];
          if (curNum > 0) {
            let dropIndex = -1;
            for (let next = row + 1; next < COUNT; next++) {
              let nextNum = nums[next][col];
              if (nextNum === 0) {
                dropIndex = next;
              } else {
                break;
              }
            }
            if (dropIndex >= 0) {
              this.updateNums(dropIndex, col, curNum);
              this.updateNums(row, col, 0);
            }
          }
        }
      }
      this.randNum();
    },
  },
  created() {
    this.initNum();
  },
  mounted() {
    document.onkeydown = (e) => {
      var f = {
        '37': this.left,
        '38': this.up,
        '39': this.right,
        '40': this.down,
      };
      this.activeName == 'game' && f[e.keyCode] && f[e.keyCode]();
    }
  },
  computed: {}
}
</script>

<style>
#game {
  border-radius: 6px;
  background-color: RGB(187, 170, 170);
  width: 440px;
  height: 440px;
  margin: 0 auto;
  position: relative;
}

.card {
  width: 100px;
  height: 100px;
  float: left;
  border-radius: 5px;
  background: #CD9B1D;
  margin-left: 8px;
  margin-top: 8px;
  /*animation: col0_to_col2 100ms linear 1ms 1 normal forwards;*/
}

.item {
  height: 100px;
  width: 100px;
  text-align: center;
  line-height: 100px;
  position: absolute;
  border-radius: 5px;
  font-size: 50px;
}

.num-0 {
  background: #CD9B1D;
}

.num-2 {
  background: #FF8C69;
}

.num-4 {
  background: #EEB422;
}

.num-8 {
  background: #EE2C2C;
}

.num-16 {
  background: #CDCD00;
}

.num-32 {
  background: #B0C4DE;
}

.num-64 {
  background: #9AC0CD;
}

.num-128 {
  background: #912CEE;
}

.num-256 {
  background: #8B2323;
}

.num-512 {
  background: #698B22;
}

.num-1024 {
  background: #40E0D0;
  font-size: 40px;
}

.num-2048 {
  background: #228B22;
  font-size: 40px;
}
</style>
