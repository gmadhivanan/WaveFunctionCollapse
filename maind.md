import { addWelcomeText } from "../objects/welcomeText.js";
import { addsquare } from "../objects/square.js"

var matsiz = 5;
var sz = 200;
var starttypenum = 7;
var tileRules = [
  [[2, 3, 4, 5, 7], [2, 5, 6, 7],       [3, 4, 6],          [2, 4, 6, 7]],       // tile 1
  [[3, 4, 5, 7],    [1, 3, 4, 5],        [1, 3, 4, 6],       [1, 3, 4, 5]],       // tile 2
  [[1, 2, 4, 6],    [2, 5, 6, 7],        [1, 2, 4, 5, 7],    [2, 4, 6, 7]],       // tile 3 fixed
  [[1, 2, 3, 4, 6], [1, 2, 3, 4, 5],    [1, 2, 3, 4, 5, 6, 7], [2, 4, 6, 7]],    // tile 4
  [[3, 4, 5, 7],    [2, 5, 6, 7],        [1, 2, 5, 7],       [1, 2, 3, 4, 5]],    // tile 5
  [[1, 2, 4, 6],    [1, 3, 4],           [3, 4, 6],          [1, 3, 5]],          // tile 6
  [[3, 4, 5, 7],    [1, 3, 4],           [1, 2, 5, 7],       [1, 3, 5]]           // tile 7
];

var arr = Array.from({ length: starttypenum }, (_, i) => i + 1);

scene("game", () => {
  let starttype = 0;
  for (let h = 0; h < matsiz; h++) {
    const h1 = h * sz;
    let starter = Math.floor(rand() * matsiz);
    for (let w = 0; w < matsiz; w++) {
      const w1 = w * sz;
      const sq0 = addsquare(w1, h1, sz, starttype);
      if (w == starter) {
        sq0.tilearray = [Math.floor(rand() * starttypenum)+1];
      } else {
        sq0.tilearray = [...arr];
      }
      const label = sq0.add([
        //text(sq0.getEntropy()),
        text(sq0.getTile()),
        anchor("center"),
       
        pos(sq0.width / 2, sq0.height / 2),
        // center it within the square
        color("white")
      ]);
      sq0.label = label;
    }
  }
  const Welcome = addWelcomeText();
  const player= add([
    sprite("mark"),
    pos(center()),
    area(),
    body(),
    scale(1),
    anchor("center"),
    rotate(0)
  ]);
  const SPEED = 200;
const diagSpeedfract=1.4
onUpdate(() => {
    if (isKeyDown("up") && isKeyDown("right")) {
        player.move(SPEED/diagSpeedfract, -SPEED/diagSpeedfract);
    } else if (isKeyDown("up") && isKeyDown("left")) {
        player.move(-SPEED/diagSpeedfract, -SPEED/diagSpeedfract);
    } else if (isKeyDown("down") && isKeyDown("right")) {
        player.move(SPEED/diagSpeedfract, SPEED/diagSpeedfract);
    } else if (isKeyDown("down") && isKeyDown("left")) {
        player.move(-SPEED/diagSpeedfract, SPEED/diagSpeedfract);
    } else if (isKeyDown("up")) {
        player.move(0, -SPEED);
    } else if (isKeyDown("down")) {
        player.move(0, SPEED);
    } else if (isKeyDown("left")) {
        player.move(-SPEED, 0);
    } else if (isKeyDown("right")) {
        player.move(SPEED, 0);
    }
});
loadShader("fogOfWar", null, `
    uniform vec2 u_playerPos;
    uniform float u_radius;
    uniform float u_softness;

    vec4 frag(vec2 pos, vec2 uv, vec4 color, sampler2D tex) {
        float dist = distance(pos, u_playerPos);
        float fog = smoothstep(u_radius, u_radius + u_softness, dist);
        
        return vec4(0.0, 0.0, 0.0, fog);
    }
`);

const fogOverlay = add([
    rect(width(), height()),
    pos(0, 0),
    opacity(1),
    shader("fogOfWar", () => ({
        u_playerPos: vec2(player.pos.x, player.pos.y),
        u_radius: 150.0,
        u_softness: 100.0,
    })),
    z(100),
]);


  const pressEvent = onKeyPress((key) => {
    Welcome.destroy();
  });
  const dimlim = (matsiz - 1) * sz;
  const allsq = get("square");
  const sqMap = {};
allsq.forEach(sq => {
    sqMap[`${sq.pos.x},${sq.pos.y}`] = sq;
});
function getSquareAt(x, y) {
    return sqMap[`${x},${y}`];
}
  function updateTiles(allsq2) {
    let sqind = -1;
    let sqmin = 99;
    for (let s = 0; s < allsq2.length; s++) {
      const sq = allsq2[s];
      
      let forbiddentiles = [];
      if (sq.pos.y > 0) {
        const sqnorth = getSquareAt(sq.pos.x, sq.pos.y - sz);
        sq.north = sqnorth.getTile();
        if (sq.north > 0) {
          forbiddentiles = tileRules[sq.north - 1][0];
          sq.tilearray = sq.tilearray.filter((t) => !forbiddentiles.includes(t));
        }
      }
      if (sq.pos.y < dimlim) {
        const sqsouth = getSquareAt(sq.pos.x, sq.pos.y + sz);
        sq.south = sqsouth.getTile();
        if (sq.south > 0) {
          forbiddentiles = tileRules[sq.south - 1][2];
          sq.tilearray = sq.tilearray.filter((t) => !forbiddentiles.includes(t));
        }
      }
      if (sq.pos.x > 0) {
        const sqwest = getSquareAt(sq.pos.x - sz, sq.pos.y);
        sq.west = sqwest.getTile();
        if (sq.west > 0) {
          forbiddentiles = tileRules[sq.west - 1][3];
          sq.tilearray = sq.tilearray.filter((t) => !forbiddentiles.includes(t));
        }
      }
      if (sq.pos.x < dimlim) {
        const sqeast = getSquareAt(sq.pos.x + sz, sq.pos.y);
        sq.east = sqeast.getTile();
        if (sq.east > 0) {
          forbiddentiles = tileRules[sq.east - 1][1];
          sq.tilearray = sq.tilearray.filter((t) => !forbiddentiles.includes(t));
        }
      }
      if (sq.getEntropy() < sqmin && sq.getEntropy() > 1) {
        sqmin = sq.getEntropy();
        sqind = s;
      }
      //sq.label.text = String(sq.getEntropy());
      sq.label.text = String(sq.getTile());
      if (sq.tilearray.length === 0) {
    //console.log("Contradiction at", sq.pos.x, sq.pos.y, "north:", sq.north, "south:", sq.south, "east:", sq.east, "west:", sq.west);
}
else if (sq.tilearray.length === 1){placeTile(sq)}
      
    }
    
    if (sqind === -1) return allsq.length
    else{return sqind;}
    
  }
  function randomizeTile(sqi) {
    const len = sqi.tilearray.length;
    const randind = Math.floor(rand() * len);
    sqi.tilearray = [sqi.tilearray[randind]];
  }
  let sqind1 = updateTiles(allsq);
  let x = allsq[sqind1].getEntropy();
  let i = 0;
function placeTile(sqin){
   if (sqin.placed) return;
    sqin.placed = true;
let n = sqin.getEntropy();
if (n==1){
let tile=sqin.getTile();
let spr="mark"
console.log(tile)
if (tile==1){spr="4way"}
else if (tile==2){spr="NShall"}
else if (tile==3){spr="EWhall"}
else if (tile==4){spr="NEhall"}
else if (tile==5){spr="SWhall"}
else if (tile==6){spr="Nent"}
else if (tile==7){spr="Sent"}

  sqin.add([
    sprite(spr),
    scale(1),
    anchor("center"),
    //area(),
    //body({ isStatic: true }),
    pos(sqin.width / 2, sqin.height / 2),
    rotate(0),
    spr, 
  ]);
addCollision(sqin, tile);

}


}
function addCollision(sqin, tile) {
    const s = sz;
    const w = 10;  // wall thickness
    const gap = s / 2;      // doorway width
    const side = (s - gap) / 2; // wall on each side of doorway

    if (tile == 1) {
        // 4 way intersection - walls on all 4 corners
        sqin.add([rect(w, side), pos(0, 0),         area(), body({ isStatic: true })]);
        sqin.add([rect(w, side), pos(s-w, 0),       area(), body({ isStatic: true })]);
        sqin.add([rect(w, side), pos(0, s-side),    area(), body({ isStatic: true })]);
        sqin.add([rect(w, side), pos(s-w, s-side),  area(), body({ isStatic: true })]);

    } else if (tile == 2) {
        // north south hallway - walls on left and right
        sqin.add([rect(w, s), pos(0, 0),   area(), body({ isStatic: true })]);
        sqin.add([rect(w, s), pos(s-w, 0), area(), body({ isStatic: true })]);

    } else if (tile == 3) {
        // east west hallway - walls on top and bottom
        sqin.add([rect(s, w), pos(0, 0),   area(), body({ isStatic: true })]);
        sqin.add([rect(s, w), pos(0, s-w), area(), body({ isStatic: true })]);

    } else if (tile == 4) {
        // north east - open north and east
        sqin.add([rect(w, side),   pos(0, 0),       area(), body({ isStatic: true })]);  // left top
        sqin.add([rect(w, side),   pos(0, s-side),  area(), body({ isStatic: true })]);  // left bottom
        sqin.add([rect(side, w),   pos(0, s-w),     area(), body({ isStatic: true })]);  // bottom left
        sqin.add([rect(side, w),   pos(s-side, s-w),area(), body({ isStatic: true })]);  // bottom right

    } else if (tile == 5) {
        // south west - open south and west
        sqin.add([rect(side, w),   pos(0, 0),       area(), body({ isStatic: true })]);  // top left
        sqin.add([rect(side, w),   pos(s-side, 0),  area(), body({ isStatic: true })]);  // top right
        sqin.add([rect(w, side),   pos(s-w, 0),     area(), body({ isStatic: true })]);  // right top
        sqin.add([rect(w, side),   pos(s-w, s-side),area(), body({ isStatic: true })]);  // right bottom

    } else if (tile == 6) {
        // room with north entrance
        sqin.add([rect(side, w),   pos(0, 0),       area(), body({ isStatic: true })]);  // top left
        sqin.add([rect(side, w),   pos(s-side, 0),  area(), body({ isStatic: true })]);  // top right
        sqin.add([rect(w, s),      pos(0, 0),       area(), body({ isStatic: true })]);  // left wall
        sqin.add([rect(w, s),      pos(s-w, 0),     area(), body({ isStatic: true })]);  // right wall
        sqin.add([rect(s, w),      pos(0, s-w),     area(), body({ isStatic: true })]);  // bottom wall

    } else if (tile == 7) {
        // room with south entrance
        sqin.add([rect(s, w),      pos(0, 0),       area(), body({ isStatic: true })]);  // top wall
        sqin.add([rect(w, s),      pos(0, 0),       area(), body({ isStatic: true })]);  // left wall
        sqin.add([rect(w, s),      pos(s-w, 0),     area(), body({ isStatic: true })]);  // right wall
        sqin.add([rect(side, w),   pos(0, s-w),     area(), body({ isStatic: true })]);  // bottom left
        sqin.add([rect(side, w),   pos(s-side, s-w),area(), body({ isStatic: true })]);  // bottom right
    }
}
onKeyPress("b", () => {
    //console.log(allsq[sqind1])
});
onKeyPress("space", () => {
    while (true) {
        sqind1 = updateTiles(allsq);
        if (sqind1 >= allsq.length) break; 
        const x = allsq[sqind1].getEntropy();
        if (x <= 1) break;
        randomizeTile(allsq[sqind1]);
    }
});


onMousePress(() => {
    const mousepos = mousePos();
    const x = Math.floor(mousepos.x / sz) * sz;
    const y = Math.floor(mousepos.y / sz) * sz;
    const sq = getSquareAt(x, y);
    if (sq) {
        console.log("Position:", sq.pos.x, sq.pos.y);
        console.log("Tilearray:", sq.tilearray);
        console.log("Entropy:", sq.getEntropy());
        console.log("North:", sq.north, "South:", sq.south, "East:", sq.east, "West:", sq.west);
    }
});
});
