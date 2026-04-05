

export function addsquare(x,y,sz) {
    const border= 2;
const obj = add([
    
    //rect(sz, sz), // Draw this object as a rectangle
    //pos(x, y), // Position this object in X: 10 and Y: 20
    //"shape", // Classify this object as "shape"
    //color("black"),
    rect(sz-border, sz-border), // Draw this object as a rectangle
    pos(x, y), // Position this object in X: 10 and Y: 20
    "square", // Classify this object as "shape"
    color("blue"),
     {
            entropy: 2,
            tiletype: Math.floor(Math.random() * 4) ,
            north:0,
            east:0,
            south:0,
            west:0, 
        },
    
]);
return obj;
}
