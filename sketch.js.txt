let cube;
let dim = 3;
let len = 100;
var _text;
let allMoves = [];
let drawing = true;
let moves = [];
let timer=300;
let viewx=0
let viewy=0
let viewz=0
let myFont;
let numberofturns=0;
let candisplay=false;


function setup() {
  createCanvas(windowWidth, windowHeight, WEBGL);
 
  cube = new Cube(dim);
  initMoves();
  resetcube();





}

function draw() {
  background(600);
  rotateX(viewx);
  rotateY(viewy);
  rotateZ(viewz);
  numberofturns=numberofturns+1;
  //cube.show();
  if(candisplay){
    cube.show();
  }
  

}

function resetcube(){
    timer=300;
    while(numberofturns<=200){
      cube.doMove(allMoves[9]);
      cube.show();
      cube.doMove(allMoves[6]);
      cube.show();
      cube.doMove(allMoves[5]);
      cube.show();
      cube.doMove(allMoves[2]);
      cube.show();
      numberofturns=numberofturns+1;
  }
  candisplay=true;
}
function keyPressed(char) {
  if (cube.isAnimating()) {
    return;
  }

  let dir = Config.CC;
  if (char.key.toLowerCase() == char.key) {
    dir = Config.CW;
  }

  if (["u", "d", "r", "l", "f", "b"].indexOf(char.key.toLowerCase()) != -1) {
    let move = new Move(char.key.toLowerCase(), dir);
    cube.doMove(move);
  }
  
  if (keyCode==LEFT_ARROW){
    viewy=viewy-0.5;
    rotateY(viewy);
    
  } else if(keyCode==RIGHT_ARROW){
    viewy=viewy+0.5;
    rotateY(viewy+0.1);
  }
  
  if (keyCode==UP_ARROW){
    viewx=viewx-0.5;
    rotateX(viewx);
    
  } else if(keyCode==DOWN_ARROW){
    viewx=viewx+0.5;
    rotateX(viewx+0.1);
  }
  
  if (keyCode==ENTER){
    viewz=viewz-0.5;
    rotateZ(viewz);
    
  } else if(keyCode==CONTROL){
    viewz=viewz+0.5;
    rotateZ(viewz+0.1);
  }

}



function initMoves() {
  allMoves[0] = new Move("u", Config.CW);
  allMoves[1] = new Move("d", Config.CW);
  allMoves[2] = new Move("r", Config.CW);
  allMoves[3] = new Move("l", Config.CW);
  allMoves[4] = new Move("f", Config.CW);
  allMoves[5] = new Move("b", Config.CW);
  allMoves[6] = new Move("u", Config.CC);
  allMoves[7] = new Move("d", Config.CC);
  allMoves[8] = new Move("r", Config.CC);
  allMoves[9] = new Move("l", Config.CC);
  allMoves[10] = new Move("f", Config.CC);
  allMoves[11] = new Move("b", Config.CC);
  


}

class Cube {
  constructor(dimension) {
    this.dimension = dimension;
    this.cube = [];
    for (let i = 0; i < this.dimension; i++) {
      this.cube[i] = [];
      for (let j = 0; j < this.dimension; j++) {
        this.cube[i][j] = [];
        for (let k = 0; k < this.dimension; k++) {
          let offset = ((this.dimension - 1) * len) / 2;
          let x = len * i - offset;
          let y = len * j - offset;
          let z = len * k - offset;
          this.cube[i][j][k] = new Cubie(x, y, z, len);
        }
      }
    }
    this.currentMove = null;
  }

  show() {
    if (this.currentMove) {
      this.currentMove.update();
    }

    for (let i = 0; i < this.dimension; i++) {
      for (let j = 0; j < this.dimension; j++) {
        for (let k = 0; k < this.dimension; k++) {
          if (this.currentMove) {
            if (this.currentMove.shouldRotateX(i)) {
              push();
              rotateX(this.currentMove.angle);
              this.cube[i][j][k].show();
              pop();
            } else if (this.currentMove.shouldRotateY(j)) {
              push();
              rotateY(this.currentMove.angle);
              this.cube[i][j][k].show();
              pop();
            } else if (this.currentMove.shouldRotateZ(k)) {
              push();
              rotateZ(this.currentMove.angle);
              this.cube[i][j][k].show();
              pop();
            } else {
              this.cube[i][j][k].show();
            }
          } else {
            this.cube[i][j][k].show();
          }
        }
      }
    }
  }

  doMove(move) {
    this.currentMove = move;
    move.start();
  }

  isAnimating() {
    return this.currentMove && this.currentMove.animating;
  }

  rotateSide(move) {
    let side = move.side;
    let dir = move.dir;

    let index = 0;
    let rotateReverse = false;
    if (["d", "r", "f"].indexOf(side) != -1) {
      index = this.dimension - 1;
      rotateReverse = true;
    }
    if (dir == Config.CC) {
      rotateReverse = !rotateReverse;
    }

    let newConfigs = [];
    if (side == "u" || side == "d") {
      for (let i = 0; i < this.dimension; i++) {
        for (let j = 0; j < this.dimension; j++) {
          this.cube[i][index][j].config.rotate(side, dir);
          let nextJ = rotateReverse ? this.dimension - 1 - i : i;
          let nextI = rotateReverse ? j : this.dimension - 1 - j;
          newConfigs.push({
            i: nextI,
            j: nextJ,
            config: this.cube[i][index][j].config.copy(),
          });
        }
      }

      for (let k = 0; k < newConfigs.length; k++) {
        this.cube[newConfigs[k].i][index][newConfigs[k].j].config =
          newConfigs[k].config;
      }
    }
    if (side == "l" || side == "r") {
      for (let i = 0; i < this.dimension; i++) {
        for (let j = 0; j < this.dimension; j++) {
          this.cube[index][i][j].config.rotate(side, dir);
          let nextJ = rotateReverse ? i : this.dimension - 1 - i;
          let nextI = rotateReverse ? this.dimension - 1 - j : j;
          newConfigs.push({
            i: nextI,
            j: nextJ,
            config: this.cube[index][i][j].config.copy(),
          });
        }
      }
      for (let k = 0; k < newConfigs.length; k++) {
        this.cube[index][newConfigs[k].i][newConfigs[k].j].config =
          newConfigs[k].config;
      }
    }
    if (side == "f" || side == "b") {
      for (let i = 0; i < this.dimension; i++) {
        for (let j = 0; j < this.dimension; j++) {
          this.cube[i][j][index].config.rotate(side, dir);
          let nextJ = rotateReverse ? i : this.dimension - 1 - i;
          let nextI = rotateReverse ? this.dimension - 1 - j : j;
          newConfigs.push({
            i: nextI,
            j: nextJ,
            config: this.cube[i][j][index].config.copy(),
          });
        }
      }
      for (let k = 0; k < newConfigs.length; k++) {
        this.cube[newConfigs[k].i][newConfigs[k].j][index].config =
          newConfigs[k].config;
      }
    }
  }
}

class Cubie {
  constructor(x, y, z, len) {
    this.pos = createVector(x, y, z);
    this.len = len;
    this.config = new Config();
  }

  show() {
    stroke(0);
    strokeWeight(8);
    let r = this.len / 2;
    push();

    translate(this.pos.x, this.pos.y, this.pos.z);

    //UP
    fill(this.config.get("u"));
    beginShape();
    vertex(-r, -r, -r);
    vertex(r, -r, -r);
    vertex(r, -r, r);
    vertex(-r, -r, r);
    endShape(CLOSE);

    //DOWN
    fill(this.config.get("d"));
    beginShape();
    vertex(-r, r, -r);
    vertex(r, r, -r);
    vertex(r, r, r);
    vertex(-r, r, r);
    endShape(CLOSE);

    //BACK
    fill(this.config.get("b"));
    beginShape();
    vertex(-r, -r, -r);
    vertex(r, -r, -r);
    vertex(r, r, -r);
    vertex(-r, r, -r);
    endShape(CLOSE);

    //FRONT
    fill(this.config.get("f"));
    beginShape();
    vertex(-r, -r, r);
    vertex(r, -r, r);
    vertex(r, r, r);
    vertex(-r, r, r);
    endShape(CLOSE);

    //LEFT
    fill(this.config.get("l"));
    beginShape();
    vertex(-r, -r, -r);
    vertex(-r, r, -r);
    vertex(-r, r, r);
    vertex(-r, -r, r);
    endShape(CLOSE);

    //RIGHT
    fill(this.config.get("r"));
    beginShape();
    vertex(r, -r, -r);
    vertex(r, r, -r);
    vertex(r, r, r);
    vertex(r, -r, r);
    endShape(CLOSE);
    


    pop();
  }
}

class Config {
  static colors = {
    white: "#FFFFFF",
    yellow: "#FFFF00",
    red: "#FF0000",
    orange: "#FFA500",
    green: "#00FF00",
    blue: "#0000FF",
  };

  //clockwise rotations
  rotations = {
    u: ["u", "d", "b", "f", "r", "l"],
    d: ["u", "d", "f", "b", "l", "r"],
    r: ["f", "b", "r", "l", "d", "u"],
    l: ["b", "f", "r", "l", "u", "d"],
    f: ["l", "r", "u", "d", "f", "b"],
    b: ["r", "l", "d", "u", "f", "b"],
  };

  static CW = 1;
  static CC = -1;

  constructor(sides) {
    this.sides = sides || {
      u: Config.colors.white,
      d: Config.colors.yellow,
      r: Config.colors.red,
      l: Config.colors.orange,
      f: Config.colors.green,
      b: Config.colors.blue,
    };
  }

  copy() {
    return new Config(this.sides);
  }

  get(side) {
    return this.sides[side];
  }

  rotate(side, dir) {
    if (dir == Config.CC) {
      switch (side) {
        case "u":
          side = "d";
          break;
        case "d":
          side = "u";
          break;
        case "r":
          side = "l";
          break;
        case "l":
          side = "r";
          break;
        case "f":
          side = "b";
          break;
        case "b":
          side = "f";
          break;
      }
    }

    let temp = JSON.parse(JSON.stringify(this.sides));
    this.sides.u = temp[this.rotations[side][0]];
    this.sides.d = temp[this.rotations[side][1]];
    this.sides.r = temp[this.rotations[side][2]];
    this.sides.l = temp[this.rotations[side][3]];
    this.sides.f = temp[this.rotations[side][4]];
    this.sides.b = temp[this.rotations[side][5]];
  }
}

class Move {
  constructor(side, dir) {
    this.side = side;
    this.dir = dir;
    this.angle = 0;
    this.animating = false;
  }

  start() {
    this.animating = true;
  }

  update() {
    if (this.animating) {
      let dir = this.dir;
      if (["d", "r", "f"].indexOf(this.side) != -1) {
        dir = -1 * dir;
      }
      this.angle -= dir * 0.7;

      if (Math.abs(this.angle) >= HALF_PI) {
        this.angle = 0;
        cube.rotateSide(this);
        this.animating = false;
      }
    }
  }

  shouldRotateX(index) {
    if (!this.animating) {
      return false;
    }

    if (["l", "r"].indexOf(this.side) != -1) {
      if (index == 0 && this.side == "l") {
        return true;
      }
      if (index == cube.dimension - 1 && this.side == "r") {
        return true;
      }
    }
    return false;
  }

  shouldRotateY(index) {
    if (!this.animating) {
      return false;
    }

    if (["u", "d"].indexOf(this.side) != -1) {
      if (index == 0 && this.side == "u") {
        return true;
      }
      if (index == cube.dimension - 1 && this.side == "d") {
        return true;
      }
    }
    return false;
  }

  shouldRotateZ(index) {
    if (!this.animating) {
      return false;
    }

    if (["f", "b"].indexOf(this.side) != -1) {
      if (index == 0 && this.side == "b") {
        return true;
      }
      if (index == cube.dimension - 1 && this.side == "f") {
        return true;
      }
    }
    return false;
  }

  reverse() {
    return new Move(this.side, -1 * this.dir);
  }
}
