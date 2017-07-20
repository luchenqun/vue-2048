<template>
<div>
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
  <div id="game">
    <!-- 底盘，占用格子 -->
    <div v-for="(rowNums, row) in board.nums">
      <div v-for="(item, col) in rowNums" class="card"></div>
    </div>
    <span v-for="item in board.items.filter((item) => item.num > 0)" v-bind:class="itemClass(item)">{{item.num}}
    </span>
  </div>
</div>
</template>

<script>
//////////////////////////////////////Item//////////////////////////////////////
var Item = function(num, row, col) {
  this.num = num || 0;
  this.row = (row === undefined) ? (-1) : (row);
  this.col = (col === undefined) ? (-1) : (col);
  this.oldRow = -1;
  this.oldCol = -1;
  this.merge = null;
  this.id = Item.id++;
};
Item.id = 0;

// 判断该元素是否是首次出现
Item.prototype.firstShow = function() {
  return ((this.oldRow < 0) || (this.oldCol < 0)) && (!this.merge);
}

Item.prototype.updatePos = function(row, col) {
  this.oldRow = this.row;
  this.oldCol = this.col;
  this.row = row;
  this.col = col;
}

Item.prototype.needMove = function() {
  return (this.merge) || (this.fromRow() != -1 && (this.fromRow() != this.toRow() || this.fromCol() != this.toCol()));
}

Item.prototype.fromRow = function() {
  return this.merge ? this.row : this.oldRow;
}

Item.prototype.toRow = function() {
  return this.merge ? this.merge.row : this.row;
}

Item.prototype.fromCol = function() {
  return this.merge ? this.col : this.oldCol;
}

Item.prototype.toCol = function() {
  return this.merge ? this.merge.col : this.col;
}


//////////////////////////////////////Board/////////////////////////////////////
var Board = function() {
  this.items = []; // 用于来移动的数字
  this.nums = []; // 用于来做逻辑数字



  // 初始化棋盘
  this.nums = new Array(Board.SIZE);
  for (var row = 0; row < this.nums.length; row++) {
    this.nums[row] = new Array(Board.SIZE);
    for (var col = 0; col < this.nums[row].length; col++) {
      var num = Math.pow(2, parseInt(Math.random() * 6) + 1);
      this.nums[row][col] = this.addItem(num, row, col);
    }
  }

  // this.nums[0] = [this.addItem(4, 0, 0), this.addItem(4, 0, 1), this.addItem(0, 0, 2), this.addItem(2, 0, 3)];
  // 初始化一个数字
  // this.addRandNum();
  this.win = false; // 胜利标志

  // 一些常量
  this.SIZE = this.size = 4; // 数字盘大小
  // this.UP = 3;
  // this.DOWN = 1;
  // this.LEFT = 0;
  // this.RIGHT = 2;
  this.MAXNUM = 2048;
}

Board.SIZE = Board.size = 4; // 数字盘大小
Board.MAXNUM = 2048;

Board.prototype.addItem = function(num, row, col) {
  var item = new Item(num, row, col);
  this.items.push(item);
  return item;
}

Board.prototype.setPositions = function() {
  this.nums.forEach((rowNums, row) => {
    rowNums.forEach((item, col) => {
      item.oldRow = item.row;
      item.oldCol = item.col;
      item.row = row;
      item.col = col;
    })
  })
}

Board.prototype.randNum = function() {
  return Math.random() < 0.1 ? 4 : 2;
}

Board.prototype.addRandNum = function() {
  var emptyPos = [];
  this.nums.forEach((rowNums, row) => {
    emptyPos = emptyPos.concat(rowNums.filter((item) => item.num === 0));
  });
  if (emptyPos.length > 0) {
    var pos = emptyPos[~~(Math.random() * emptyPos.length)];
    var item = this.addItem(this.randNum(), pos.row, pos.col);
    this.nums[pos.row][pos.col] = item;
  }
}

// 对数字旋转n次，使上，右，下操作都转为左操作
Board.prototype.rotateNums = function(n) {
  n = n % 4;
  if (n === 0) return;
  var numsTemp = [];
  numsTemp = new Array(Board.SIZE);
  for (var row = 0; row < numsTemp.length; row++) {
    numsTemp[row] = new Array(Board.SIZE);
  }
  // 旋转90°
  // 1 2 3       7 4 1
  // 4 5 6  -->  8 5 2
  // 7 8 9       9 6 3
  var dst = Board.SIZE - 1; // 这里我们从目标矩阵的最后一列开始存放数据
  for (var row = 0; row < Board.SIZE; row++, dst--)
    for (var col = 0; col < Board.SIZE; col++)
      numsTemp[col][dst] = this.nums[row][col];

  // 拷贝回原数组
  for (var row = 0; row < Board.SIZE; row++)
    for (var col = 0; col < Board.SIZE; col++)
      this.nums[row][col] = numsTemp[row][col];

  // 继续旋转
  this.rotateNums(n - 1);
}

Board.prototype.left = function() {
  var hasChanged = false;
  for (var row = 0; row < Board.SIZE; ++row) {
    var curRow = this.nums[row].filter(item => item.num >= 2);
    var newRow = [];
    for (var col = 0; col < Board.SIZE; ++col) {
      var curItem = curRow.length ? curRow.shift() : this.addItem(0, row, col);
      var nextItem = (curRow.length && curRow[0]) || null;

      // 满足合并条件
      if (nextItem && curItem.num == nextItem.num) {
        var mergeItem = this.addItem(curItem.num * 2);

        var itemMove1 = curItem;
        itemMove1.merge = mergeItem;

        var itemMove2 = curRow.shift();
        itemMove2.merge = mergeItem;

        curItem = mergeItem;
      }

      newRow[col] = curItem;

      this.win = (curItem.num === Board.MAXNUM);
      hasChanged = (curItem.num != this.nums[row][col].num);
    }
    this.nums[row] = newRow;
  }
  return hasChanged;
}
Board.prototype.print = function() {
  this.nums.forEach((rowNums, row) => {
    var nums = [];
    rowNums.forEach((item) => {
      nums.push(item.num)
    })
    console.log(row + ":" + JSON.stringify(nums));
  })
  console.log('');
}

Board.prototype.move = function(direction) {
  this.print();
  this.rotateNums(direction); // 旋转，为左操作准备
  this.left();
  // this.addRandNum();
  this.rotateNums(4 - (direction % 4)) // 操作完成之后，要旋转回来
  this.setPositions();
  this.print();
}

export default {
  data() {
    return {
      COUNT: 4,
      activeName: 'game',
      board: new Board(),
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
        var keyframeName, keyframe, className;
        var begin = index * (WIDTH + GAP) + GAP;
        var end = next * (WIDTH + GAP) + GAP;
        var duration = 1000;

        keyframeName = `r${index}_to_r${next}`;
        keyframe = `@-webkit-keyframes ${keyframeName}{from {top:${begin}px;} to {top:${end}px;}}`;
        style.innerHTML += keyframe;
        style.innerHTML += `.${keyframeName} { animation: ${keyframeName} ${duration}ms linear 1ms 1 normal forwards; }`

        keyframeName = `r${next}_to_r${index}`;
        keyframe = `@-webkit-keyframes ${keyframeName}{from {top:${end}px;} to {top:${begin}px;}}`;
        style.innerHTML += keyframe;
        style.innerHTML += `.${keyframeName} { animation: ${keyframeName} ${duration}ms linear 1ms 1 normal forwards; }`

        keyframeName = `c${index}_to_c${next}`;
        keyframe = `@-webkit-keyframes ${keyframeName}{from {left:${begin}px;} to {left:${end}px;}}`;
        style.innerHTML += keyframe;
        style.innerHTML += `.${keyframeName} { animation: ${keyframeName} ${duration}ms linear 1ms 1 normal forwards; }`

        keyframeName = `c${next}_to_c${index}`;
        keyframe = `@-webkit-keyframes ${keyframeName}{from {left:${end}px;} to {left:${begin}px;}}`;
        style.innerHTML += keyframe;
        style.innerHTML += `.${keyframeName} { animation: ${keyframeName} ${duration}ms linear 1ms 1 normal forwards; }`
      }
    }
    for (var row = 0; row < COUNT; row++) {
      for (var col = 0; col < COUNT; col++) {
        className = `.pos${row}${col}{ top:${row*WIDTH+(row+1)*GAP}px; left:${col*WIDTH+(col+1)*GAP}px;}`;
        style.innerHTML += className;
      }
    }
    document.getElementsByTagName('head')[0].appendChild(style);
  },
  methods: {
    initNum() {},
    updateNums(row, col, newValue) {
      this.$set(this.nums[row], col, newValue);
    },
    randNum() {
      var success = false;
      return success;
    },
    handleClick(tab, event) {},
    itemClass(item) {
      console.log('row:' + item.row, 'col:' + item.col, 'oldRow:' + item.oldRow, 'oldCol:' + item.oldCol, 'num:' + item.num, item.merge, item.firstShow());
      var classArray = ['item'];
      classArray.push('num-' + item.num); // 数字
      classArray.push('pos' + item.row + item.col); // 位置
      if (item.firstShow()) {
        classArray.push('first_show'); // 位置
      }

      if (item.needMove()) {
        classArray.push(`r${item.fromRow()}_to_r${item.toRow()}`); // 位置
        classArray.push(`c${item.fromCol()}_to_c${item.toCol()}`); // 位置
      }
      return classArray.join(' ');
    },
    keyDown(event) {
      if (event.keyCode >= 37 && event.keyCode <= 40) {
        // event.preventDefault();
        var direction = event.keyCode - 37;
        var directionMap = {};
        directionMap['37'] = 0;
        directionMap['38'] = 3;
        directionMap['39'] = 2;
        directionMap['40'] = 1;
        this.board.move(directionMap[event.keyCode])
        // this.board.nums = JSON.parse(JSON.stringify(this.board.nums));
      }
    },
  },
  created() {
    this.initNum();
  },
  beforeDestroy() {
    window.removeEventListener('keydown', this.keyDown.bind(this));
  },
  mounted() {
    window.addEventListener('keydown', this.keyDown.bind(this));
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

.item.first_show {
  animation: new_item 300ms linear 1000ms 1 normal forwards;
  transform: scale(0);
}

@keyframes new_item {
  from {
    transform: scale(0);
  }
  to {
    transform: scale(1);
  }
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
