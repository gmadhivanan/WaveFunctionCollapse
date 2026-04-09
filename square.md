

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
            north:0,
            east:0,
            south:0,
            west:0, 
            tilearray:[],
            getEntropy() { return this.tilearray.length; },
            getTile() { if (this.tilearray.length==1)
            {return this.tilearray[0]}
            else {return 0}
             },
        },
    
]);

return obj;
}
