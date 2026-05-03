import { addWelcomeText } from "../objects/welcomeText.js";
import { addsquare } from "../objects/square.js";
import { addboat } from "../objects/boat.js";
import { addsail } from "../objects/sail.js"
import { addrudder } from "../objects/rudder.js"

let matsiz = 7;
const checkind = Math.round(matsiz / 2);
let sz = 700;
let wave=0
let starttypenum = 5;
let gridOffset = 3 * sz
let tileRules = [
  [[2, 3, 4, 5], [2, 5], [3, 4], [2, 4]],
  [[3, 4, 5], [1, 3, 4, 5], [1, 3, 4], [1, 3, 4, 5]],
  [[1, 2, 4], [2, 5], [1, 2, 4, 5], [2, 4]],
  [[1, 2, 3, 4], [1, 2, 3, 4, 5], [1, 2, 3, 4, 5], [2, 4]],
  [[3, 4, 5], [2, 5], [1, 2, 5], [1, 2, 3, 4, 5]],

];
let wind = 180
let wind_duration = 30
let sailH=0.05;
const SPEED_base =4;
let SPEED = 4;
let arr = Array.from({ length: starttypenum }, (_, i) => i + 1);
scene("game", () => {
  let started = false;
loadShader("fog", null, `
    uniform float time;
    uniform vec2 res;

    vec4 frag(vec2 pos, vec2 uv, vec4 color, sampler2D tex) {
      vec4 base =def_frag() * vec4(.11, .2, 1, .4*sin(time)*cos(time));
      vec2 center = res / 2.0;
        float dist = distance(pos, center);
        float circle = smoothstep(105.0, 100.0, dist);
        vec4 yellow = vec4(.3, .3, 0.1, 0.00001);
      
      return mix(base, yellow, circle);
       
    }
`);

add([
    rect(width(), height()),
    pos(0, 0),
    fixed(),
    shader("fog", () => ({
        time: time(),
        res: vec2(width(), height()),
    })),
    z(150),
]);
  let matCode = []
  for (let h = 0; h < matsiz; h++) { matCode[h] = []; }
  for (let h = 0; h < matsiz; h++) {
    const h1 = (h * sz) - gridOffset;

    let starter = Math.floor(Math.random() * matsiz);
    for (let w = 0; w < matsiz; w++) {
      const w1 = (w * sz) - gridOffset;
      const sq0 = addsquare(w1, h1, sz, w, h);

      matCode[w][h] = sq0;

      if (w == starter) {
        sq0.tilearray = [Math.floor(Math.random() * starttypenum) + 1];
      } else {
        sq0.tilearray = [...arr];
      }
      //const label = sq0.add([
        //text(sq0.getEntropy()),
        //text(sq0.getTile()),
        //anchor("center"),

       //pos(sq0.width / 2, sq0.height / 2),

        //color("white")
      //]);
      //sq0.label = label;

      //console.log(sq0.matRow,sq0.matCol)
    }
  }
  //console.log(matCode)
  const Welcome = addWelcomeText();
  
  const player = addboat()
  const sail = player.add(addsail());
  const rud = player.add(addrudder());
  const wind_ind = add([
    sprite("sword"),
    pos(50, height() - 50),
    scale(1),
    anchor("center"),
    rotate((wind + 270) % 360),
    z(200)
  ])
add([
        text("h for help", { 
          size: 16,
          align: "center",
           width:700 }),
        pos(width()-80,height()-25),
        anchor("center"),
        z(150)
    ]);
let score=0
 let scoreText=add([
        text("Score: "+score, { 
          size: 16,
          align: "center",
           width:700 }),
        pos(width()-80,25),
        anchor("center"),
        z(150)
    ]);   
  let allsq = get("square");
  let goldFound=null
player.onCollide("gold", (obj) => {
    goldFound = obj;
});

player.onCollideEnd("gold", () => {
    goldFound = null;
});

  onUpdate(() => {
    scoreText.text="Score:"+score
    wave+=1/10
    player.scaleTo(.2+(Math.sin(wave%360*Math.PI/180)*0.04))
    //sail.scaleTo(.05+(Math.sin(wave%360*Math.PI/180)*0.02))
    //console.log(dt());
    wind_ind.angle = (wind + 270) % 360
    let pang = (player.angle + 270) % 360
    sail.angle = (wind + 270) % 360 - player.angle;
    let moveX = 0;
    let moveY = 0;
    if (isKeyDown("up")) {
      player.setSails(player.sails + 1)
      sailH=clamp(sailH+0.01,0.05,.4)
      sail.scaleTo(0.3,sailH)
    } else if (isKeyDown("down")) {
      player.setSails(player.sails - 1)
      sailH=clamp(sailH-0.01,0.05,.4)
      sail.scaleTo(0.3,sailH)
    } else if (isKeyDown("left")) {
      player.setRudder(player.rudder - 1)
     rud.angle= clamp(rud.angle+1,-30,30)
    } else if (isKeyDown("right")) {
      player.setRudder(player.rudder + 1)
      rud.angle= clamp(rud.angle-1,-30,30)
    }
    let wrad = ((wind + 180) % 360) * Math.PI / 180;


    player.heading = player.heading + (player.rudder / 90)
    player.angle = (player.heading)
    let prad = ((player.angle + 270) % 360) * Math.PI / 180;

    let adiff = wrad - prad;
    let alignment = Math.max(0.1, Math.cos(adiff));
    let speedact = player.sails * SPEED * alignment;
    moveX = Math.cos(prad) * speedact;
    moveY = Math.sin(prad) * speedact;

    if (!player.isColliding("ground")) {
      SPEED=clamp(SPEED+0.1,0,SPEED_base)
        allsq.forEach(sq => {
        sq.move(-moveX, -moveY);
        });
    }
    else{SPEED=0}
   

    const sqL = matCode[0][checkind];
    const sqR = matCode[matsiz - 1][checkind];
    const sqU = matCode[checkind][0];
    const sqD = matCode[checkind][matsiz - 1];


    if (sqL.pos.x < -gridOffset) { //console.log("too far East"); 
      scrollWorld("left")
      allsq = get("square")
    }
    if (sqU.pos.y < -gridOffset) { //console.log("too far South"); 
      scrollWorld("up")
      allsq = get("square")

    }
    if (sqR.pos.x > (matsiz - 2) * sz) { //console.log("too far West");
      scrollWorld("right")
      allsq = get("square")

    }
    if (sqD.pos.y > (matsiz - 2) * sz) { //console.log("too far North"); 
      scrollWorld("down")
      allsq = get("square")

    }

  });



  onKeyPress((key) => {

    if (started == false) {
      Welcome.destroy();
      collapseNewTiles(allsq)
      started=true
    }
  });

  

  function check_wind() {
    let sign = randi(2)
    let changeval = randi(90)
    if (sign == 0) { wind = wind - changeval }
    if (sign == 1) { wind = wind + changeval }
  }
  loop(wind_duration, () => {
    check_wind();
  });
  function updateTiles(allsq2) {
    let sqind = -1;
    let sqmin = 99;
    for (let s = 0; s < allsq2.length; s++) {
      
      const sq = allsq2[s];
      if (sq.tilearray.length === 0) {
        sq.tilearray = [...arr];
    }
    if (sq.tilearray.length === 1) continue; 
      let forbiddentiles = [];
      if (sq.matRow > 1) {
        const sqnorth = matCode[sq.matCol][sq.matRow - 1];
        sq.north = sqnorth.getTile();
        if (sq.north > 0) {
          forbiddentiles = tileRules[sq.north - 1][0];
          sq.tilearray = sq.tilearray.filter((t) => !forbiddentiles.includes(t));
          if (sq.tilearray.length === 0) {
            randomizeTile(sq);
          }
        }
      }
      if (sq.matRow < matsiz - 2) {

        const sqsouth = matCode[sq.matCol][sq.matRow + 1];;
        sq.south = sqsouth.getTile();
        if (sq.south > 0) {
          forbiddentiles = tileRules[sq.south - 1][2];
          sq.tilearray = sq.tilearray.filter((t) => !forbiddentiles.includes(t));
          if (sq.tilearray.length === 0) {
            randomizeTile(sq);
          }
        }
      }
      if (sq.matCol > 0) {
        const sqwest = matCode[sq.matCol - 1][sq.matRow];
        sq.west = sqwest.getTile();
        if (sq.west > 0) {
          forbiddentiles = tileRules[sq.west - 1][3];
          sq.tilearray = sq.tilearray.filter((t) => !forbiddentiles.includes(t));
          if (sq.tilearray.length === 0) {
            randomizeTile(sq);
          }
        }
      }
      if (sq.matCol < matsiz - 2) {

        const sqeast = matCode[sq.matCol + 1][sq.matRow];
        sq.east = sqeast.getTile();
        if (sq.east > 0) {
          forbiddentiles = tileRules[sq.east - 1][1];
          sq.tilearray = sq.tilearray.filter((t) => !forbiddentiles.includes(t));
          if (sq.tilearray.length === 0) {
             randomizeTile(sq);
          }
        }
      }
      //console.log(sqmin)
      if (sq.getEntropy() < sqmin && sq.getEntropy() > 1) {
        sqmin = sq.getEntropy();
        
        sqind = s;
      }
      //sq.label.text = String(sq.getEntropy());
      //sq.label.text = String(sq.getTile());
      if (sq.tilearray.length === 0) {

      }
      //else if (sq.tilearray.length === 1){placeTile(sq)}

    }

    if (sqind === -1) return allsq2.length
    else { return sqind; }

  }
  function randomizeTile(sqi) {
    const len = sqi.tilearray.length;
    const randind = Math.floor(Math.random() * len);
    sqi.tilearray = [sqi.tilearray[randind]];
  }
  

let treasures = [];
  function placeTile(sqin) {
    if (sqin.placed) return;
    let tile1 = sqin.getTile();
    //if (tile1 == 0) return;
    sqin.placed = true;

    //sqin.label.text=sqin.getTile()+", "+sqin.getEntropy()
    let n = sqin.getEntropy();
    let sc = 1;
    let rt = Math.random() * 360
    let fishrandx=(Math.random()*sz/2)-sz/4
    let fishrandy=(Math.random()*sz/2)-sz/4
    let isTreasure=Math.random()
    const clr =[235, 181, 73]
    
      let tile = sqin.getTile();
      let spr = "rudder"
        if (tile == 1) {
        spr = "fish_05"
        sc = .4
        
        sqin.add([
        rect(sqin.width/10, sqin.height/10),
        pos(0, 0),
        color(clr),
        area(),
        body({ isStatic: true }),
        opacity(1),
        //z(99),
        "ground"
        ]);
        sqin.add([
        rect(sqin.width/10, sqin.height/10),
        pos(0, sqin.height-(sqin.height/10)),
        color(clr),
        area(),
        body({ isStatic: true }),
        opacity(1),
        //z(99),
        "ground"
        ]);
        sqin.add([
        rect(sqin.width/10, sqin.height/10),
        pos(sqin.width-(sqin.width/10), 0),
        color(clr),
        area(),
        body({ isStatic: true }),
        opacity(1),
        //z(99),
        "ground"
        ]);
        sqin.add([
        rect(sqin.width/10, sqin.height/10),
        pos(sqin.width-(sqin.width/10), sqin.height-(sqin.height/10)),
        color(clr),
        area(),
        body({ isStatic: true }),
        opacity(1),
       // z(99),
        "ground"
        ]);
      }

      else if (tile == 2) {
        spr = "dolphin_01",
        
        sc=.1
        sqin.add([
        rect(sqin.width/10, sqin.height),
        pos(0, 0),
        area(),
        body({ isStatic: true }),
        color(clr),
        opacity(1),
        //z(99),
        "ground"
        ]);
        sqin.add([
        rect(sqin.width/10, sqin.height),
        pos(sqin.width-(sqin.width/10), 0),
        area(),
        body({ isStatic: true }),
        color(clr),
        opacity(1),
        //z(99),
        "ground"
        ]);
      }
      else if (tile == 3) {
        spr = "dolphin_02",
        
        sc = .1,
        sqin.add([
        rect(sqin.width, sqin.height/10),
        area(),
        body({ isStatic: true }),
        pos(0, 0),
        color(clr),
        opacity(1),
        //z(99),
        "ground"
        ]);
        sqin.add([
        rect(sqin.width, sqin.height/10),
        area(),
        body({ isStatic: true }),
        pos(0, sqin.height-(sqin.height/10)),
        color(clr),
        opacity(1),
        //z(99),
        "ground"
        ]);
      }
      else if (tile == 4) {
        spr = "fish_03"
        
        sc = .4
        sqin.add([
        rect(sqin.width, sqin.height/10),
        area(),
        body({ isStatic: true }),
        pos(0, sqin.height-(sqin.height/10)),
        color(clr),
        opacity(1),
        //z(99),
        "ground"
        ]);
        sqin.add([
        rect(sqin.width/10, sqin.height),
        pos(0, 0),
        area(),
        body({ isStatic: true }),
        color(clr),
        opacity(1),
        //z(99),
        "ground"
        ]);
      }
      else if (tile == 5) {
        spr = "fish_04"
        
        sc = .4
        sqin.add([
        rect(sqin.width/10, sqin.height),
        pos(sqin.width-(sqin.width/10), 0),
        area(),
        body({ isStatic: true }),
        color(clr),
        opacity(1),
        //z(99),
        "ground"
        ]);
        sqin.add([
        rect(sqin.width, sqin.height/10),
        area(),
        body({ isStatic: true }),
        pos(0, 0),
        color(clr),
        opacity(1),
        //z(99),
        "ground"
        ]);
      }


      //console.log(tile)

      sqin.add([
        sprite(spr),
        scale(sc),
        anchor("center"),
        color(150, 0, 255),
        opacity(0.5),
        //area(),
        //body({ isStatic: true }),
        pos(((sqin.width / 2)+fishrandx) , ((sqin.height / 2)+fishrandy) ),
        rotate(rt),
        //z(50),
        "sealife",
      ]);
      if (isTreasure>=0.2){
        const  treasure=sqin.add([
        sprite("treasure"),
        scale(.2),
        anchor("center"),
        color(150, 0, 255),
        opacity(0.2),
        area({isSensor:true}),
        //body({ isStatic: false }),
        pos(((sqin.width / 2)-fishrandx) , ((sqin.height / 2)-fishrandy) ),
        rotate(rt),
        //z(50),
        "gold",
        {amount:Math.round(Math.random()*500)}
      ]);
      //treasures.push(treasure);



      }


  }
  //let log = false
  let menu
  //onKeyPress("b", () => {
    //player.collisionIgnore = ["ground"];
    //SPEED=SPEED/2;
    //console.log("dragging the boat")

  //});
  //onKeyRelease("b",()=>{
    //player.collisionIgnore = [];
    //SPEED=SPEED_base;

  //})
    onKeyPress("w", () => {
    check_wind();
    //console.log("dragging the boat")

  });
  onKeyPress("h", () => {
    menu=help();
    //console.log("dragging the boat")

  });
  
  onKeyPress("d",()=>{ let timer=1
        const dtext = add([
        text("...", { 
            size: 24,
            align: "center",
            width: width()
        }),
        pos(width()/2, height()/4),
        anchor("center"),
        fixed(),
        z(150)
    ]);
    wait(timer, () => {
        dtext.text=dredge();

        
    });
        wait(timer+1, () => {
        dtext.destroy();
   
    });
    
  });
  onKeyRelease("h",()=>{
  menu.destroy()


  })
  function help(){
    const ofs=30
    const obj=add([
  
    rect(width()-ofs, height()-ofs),
    pos(ofs/2, ofs/2),
    fixed(),
    z(200),
    color(199, 165, 105),
    opacity(.99)
    ])
  obj.add([
    text("Help", {
        size: 48,
        font: "monospace",
        align: "center",
        width: 200,
    }),
    pos(obj.width / 2, obj.height / 18),
    anchor("top"),
    color(0, 0, 0),
    fixed(),
    z(201),
]);
  obj.add([
    text("Use up and down arrows to raise or lower the sails.\n Use the right and left arrows to move the rudder.\n Press w to wait for a change of winds.\n Press d to dredge up treasure.\n", {
        size: 24,
        font: "monospace",
        align: "center",
        width: obj.width ,
    }),
    pos(obj.width / 2, obj.height / 4),
    anchor("top"),
    color(0, 0, 0),
    fixed(),
    z(201),
]);
    return obj
  }
 function dredge(){    
  let textOut
  if (goldFound) {//console.log("found treasure!",goldFound.amount);
score=score+goldFound.amount
textOut="found "+goldFound.amount+" gold!"
goldFound.destroy()
}
else{//console.log("bupkes")
textOut="bupkes"}
return textOut
 }
function spawnSquare(x, y, col, row) {
      const sq0 = addsquare(x, y, sz, col, row);
      sq0.tilearray = [...arr];
      //sq0.label = sq0.add([
       // text(sq0.getTile()), 
        //anchor("center"),
       // pos(sq0.width / 2, sq0.height / 2), 
       // color("white")
     // ]);
      return sq0;
    }
function updateIndices() {
      for (let c = 0; c < matCode.length; c++) {
        for (let r = 0; r < matCode[c].length; r++) {
          if (matCode[c][r]) {
            matCode[c][r].matCol = c;
            matCode[c][r].matRow = r;
          }
        }
      }
    }

  function scrollWorld(direction) {

    if (direction == "left" || direction == "right") {
      let rmInd
      let goingLeft
      if (direction=="left")
      {goingLeft=true}
      else
      {goingLeft=false}
      if (goingLeft==true) 
      { rmInd = 0 } 
      else 
      { rmInd = matCode.length - 1 }

      
      for (let h = 0; h < matsiz; h++) {
        if (matCode[rmInd][h]) 
        {destroy(matCode[rmInd][h])}
      }
      if (goingLeft==true) 
      { matCode.shift() } 
      else { matCode.pop() }
      updateIndices();

     
      let refCol
      let ncind
      if (goingLeft==true) {
        refCol = matCode[matCode.length - 1]
        ncind = matCode.length
      }
      else {
        refCol = matCode[0]
        ncind = 0
      }
      let newX = refCol[checkind].pos.x
      if (goingLeft==true) { newX += sz }
      else { newX -= sz }

      const newCol = [];
      for (let h = 0; h < matsiz; h++) {
        newCol.push(spawnSquare(newX, refCol[h].pos.y, ncind, h));
      }
      if (goingLeft==true) { matCode.push(newCol) } else { matCode.unshift(newCol) }
      updateIndices();

      let neighbour
      if (goingLeft==true) { neighbour = matCode[matCode.length - 2] } else { neighbour = matCode[1] }
      collapseNewTiles([...newCol, ...neighbour]);

    } else {
      let rmInd
      let goingUp 
      if(direction == "up")
        {goingUp=true}
      else
        {goingUp=false}
      if (goingUp==true) 
        { rmInd = 0 } 
      else 
        { rmInd = matCode[0].length - 1 }

      for (let c = 0; c < matCode.length; c++) {
        if (matCode[c][rmInd]) 
        {destroy(matCode[c][rmInd])}
        if (goingUp==true) 
        { matCode[c].shift() } 
        else 
        { matCode[c].pop() }
      }
      updateIndices();

      
      let refRowind
      if (goingUp==true) 
      { refRowind = matCode[checkind].length - 1 } 
      else 
      { refRowind = 0 }
      let newY = matCode[checkind][refRowind].pos.y
      if (goingUp==true) 
      { newY += sz } 
      else 
      { newY -= sz }

      const newRow = [];
      for (let c = 0; c < matCode.length; c++) {
        let newRowind
        if (goingUp==true) 
        { newRowind = matCode[c].length } 
        else { newRowind = 0 }
        const sq = spawnSquare(matCode[c][refRowind].pos.x, newY, c, newRowind);
        if (goingUp==true) 
        { matCode[c].push(sq) } 
        else { matCode[c].unshift(sq) }
        newRow.push(sq);
      }
      updateIndices();

      const neighbourRow = []
      for (let c = 0; c < matCode.length; c++) {
        if (goingUp==true) 
        { neighbourRow.push(matCode[c][matCode[c].length - 2]) }
        else 
        { neighbourRow.push(matCode[c][1]) }
      }
      collapseNewTiles([...newRow, ...neighbourRow]);
    }
  }
  function collapseNewTiles(tiles) {
    let goodtiles = []
    let badtiles=[]
    for (let i = 0; i < tiles.length; i++) {
      if (tiles[i]) { goodtiles.push(tiles[i]) }
    }
    
    while (true) {
      let ind = updateTiles(goodtiles);
      if (ind >= goodtiles.length) {
        //console.log(ind, goodtiles.length);
        break};
      let sq2=goodtiles[ind];
      let entropy = sq2.getEntropy()
      if (entropy == 1) {
        placeTile(sq2);
        //sq2.label.text=sq2.getTile()+","+sq2.getEntropy()
        break}
      randomizeTile(sq2);
      placeTile(sq2)
    }
        for (let i = 0; i < goodtiles.length; i++) {
        if (goodtiles[i].getEntropy() == 1) {
            placeTile(goodtiles[i]);
        }
     }
  }

});
