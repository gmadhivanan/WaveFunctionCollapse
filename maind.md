import { addWelcomeText } from "../objects/welcomeText.js";
import { addsquare } from "../objects/square.js"

const matsiz=9
const sz =100;
function getSquareAt(x, y) {
    return get("square").find(sq => sq.pos.x === x && sq.pos.y === y);
}
scene("game", () => {
    


  for(let h = 0; h <matsiz; h++){ 
    const h1=h*sz
    for(let w = 0; w < matsiz; w++){
    const w1= w*sz
    addsquare(w1, h1, sz);
  }
  }
    const Welcome = addWelcomeText();

    add([
        sprite("mark"),
        pos(center()),
        scale(1),
        anchor("center"),
        rotate(0),
    ]);
    const pressEvent = onKeyPress((key) => {
        Welcome.destroy()
        
        
    });
const dimlim = (matsiz-1)*sz
const allsq= get("square");
for (let s=0; s<allsq.length; s++){
const sq = allsq[s]

if (sq.pos.y>0){
const sqnorth =getSquareAt(sq.pos.x,sq.pos.y-sz)
sq.north=sqnorth.tiletype

}
if (sq.pos.y<dimlim){
const sqsouth =getSquareAt(sq.pos.x,sq.pos.y+sz)
sq.south = sqsouth.tiletype;

}
if (sq.pos.x>0){
const sqwest =getSquareAt(sq.pos.x-sz,sq.pos.y)
sq.west=sqwest.tiletype

}
if (sq.pos.x<dimlim){
const sqeast=getSquareAt(sq.pos.x+sz,sq.pos.y)
sq.east=sqeast.tiletype

}

console.log(sq.north,sq.east,sq.south,sq.west)
 sq.add([
        text(sq.tiletype),
        anchor("center"),
        pos(sq.width / 2, sq.height / 2), // center it within the square
        color("white"),
    ]);
}












});
