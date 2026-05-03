# WaveFunctionCollapse
## Explaining the code
## Setup
I initially import 5 different objects into the game.
The welcome text is the opening text to the game. This is mainly a carry over from the basic game file
The square or tile is how the world is segmented
The boat is the player avatar
The sail is a different object initially
The rudder is a different object inititially

Then I start defining initial parameters for the game
The mat size is the number of tiles in the nxn world space
The checkind is used later to determine the direction the player is moving
The sz is the size of each tile
The wave variable for the time of the current wave is created
The startypenum is the amount of unique tile types
The gridOffset is how far offset the origin point is for creating the tiles. This is done such that there are 3 tiles in each direction. So the player doesn't go off the map.
Tile rules are the tiles that can go in the NESW position for each tile type
Wind is used to indicate the direction the wind is blowing
Wind duration is how long the wind stays pointing in given direction
SailH is the height of the sail
Speed_base is the base speed that speed returns to if it needs to change
Speed is the Speed value used
arr is an integer array of the different tile types
## Shader
The main shader used in the game is meant to represent fog during the night with a small light around the boat. There are two inputs for this shader: Time and resolution
The base vec4 is just a bluish overlay where the alpha changes with time in a smooth manner.
Then I get the center of the screen by diving resolution by 2
Distance gets the distance of the pixels from the center
circle smoothsteps between 105 and 100 of the distance so just the center is affected
Then yellow is just the color yellow with a small alpha
mix interpolates between the blue base and yellow color depending on whether its smoothstepped distance in circle
I had originally tried to call the shader as a full screen shader using the usePostEffect function of Kaplay but was having a hard time getting it to look right. Instead, i just add a rect over the entire play area and the shader is applied to that.
But as a result, if the user changes the screensize the shader doesn't update
## Tile Setup
To initially setup the tiles I start with a blank array matCode. The intent is to have a nxn array with each tile. I had errors if they first column (or row_ didn't exist so I loop through and create empty arrays. Then I go through and start to populate the tiles. For each column i pick a random tile and set that as the starter tile which will get an initial tile type. I place the tile in the array and then if it's a starter I give it a random type. 
## Screen Setup
I add the welcome text to the screen.
I add the player with the addboat function.
I add the sail and the rudder to the player.
I add a sword sprite with the rotation value corresponding to wind
The help text is added to the bottom of the screen to tell players where to look for instructions
The score is added to the top right of the screen. A separate value for score is created to keep tract of the value
I set up the allsq variable which grabs all the squares or tiles used
I also have the gold or treasure collision here.
## Update
The score shown on screen is updated to the latest score value
I addded a wave variable marginally increases then scales the player to look like they are moving up and down with the waves
The wind indicator angle is updated to the latest wind value
The sail angle is update to match the wind angle
Variables are created for the  x and y movement
Behaviors are added for the directional keys.
Up and down change the sails setting. It also scales the vertical portion of the sails to make it look like the sails are being raised and lowered
Left and right change the rudder setting as well as changing the rudder angle which is represented visually with the rudder sprite
The different angles are converted to radians and then adjusted so that they fit into typical trig functions
The variable adiff is a difference between the wind and the player heading.
The alignment variable takes the max of either 0.1 or cosine of adiff.
This is such that if the angles are the same (adiff is 0) and cos adiff is 1 but if the angles are more than 90 degrees apart cos(adiff) will be less than one so max will result in only a .1
Alignment is then multiplied by Speed and the sails setting, to get the actual speed value.
Then the X and Y movement values are calculated by multiplying the actual speede times the cos and sin of the player heading respectively.
Additionally this is where we check if the player is colling with the objects titled ground. If they are then speed goes to zero. If they aren't colliding the speed increments back to speed then all the squares move in the opposite direction of speed.
This was done to prevent an issue where the player drove directly into the ground and then bounced back allowing to ground to conitue moving resulting in the player going off screen.

## Infinite World
Still inside the update function, I wanted the world to continue scrolling if the player went to the edges of the screen so it felt like an infinite world.
So here I created 4 variables that pull from matCode, which stores the current tiles. It pulls the center tiles on the top, bottom, left and right.
Then it checks whether the each square has a position too far in each direction. If the square is too far in one direction. The scroll world function is called by passing the direction to update the map and then the allsq variable is updated with the new squares.

## Scroll world
I realized while I was writing the wave function collapse code that any tiles added to the edge of an already collapsed column would just need the column and a new empty column of tiles to extend the world. Then the column on the opposite side could be removed so a consisitent tile matrix size would exist. This function operates on that idea. 
First the function needs the direction that the player is going. rmInd is the index that is to be removed. So if the player is going left the tile at the end of matcode will be removed. If the player is going right, the tiles at the beginining of matcode will be removed.
After the tiles are destroyed, the matCode needs to be adjusted. The shift or pop functions are used to remove the values at the beginning or end. Then the updateIndices function is called to update the matRow and matCol values for each tile. These values are used later for easy indexing.
Next we move on to making the new tiles for a new column. Depending on whether the player is going left or right, the reference column is chosen.
The reference column is the column that will be used to help collapse the new tiles.
The new column index is ncind.
We also need the x position to place the new tiles. So we use the x position of the checkind tile in the reference column to get the new x value. We also need to account for the size of the tile so, size is added or subtracted from that x value.
After this a new array is created to store the new column and we loop through the matrix size to spawn new squares using the spawnSquare function to fill out the new column.
The spawn square function is fairly simply it just takes in the basic inputs to call the addsquare function and sets possible tiles to all options then returns the square. 
Then this new column is pushed or unshifted into the matCode 2d array and then the indices are updated again with the update index function.
Next we get the neighboring tiles for collapsing. I realize I could have just used the refCol again, but this was just to make sure that the latest matrix content was used for collapsing.
Finally, the collapse new tiles function is called with the new column and the refernce column. 
(The function performs a similar set of tasks if the up or down directions are called)

## Collapse New Tiles
This function wraps around a couple others to form the wave function collapse. I had multiple issues where the tiles wouldn't collapse or would result in no possible tiles. This wrapper helped resolve that issue.
The function filters out any elements passed in that aren't tiles or that are null.
Then a while loop is kicked off. In this while loop the good tiles are passed into the updateTiles function. This was my initial wave function collapse function before I decided to attempt the infinite world.
### updateTiles
This functions also takes in a set of tiles.
sqind and sqmin are declared up front. 
Then a for loop is kickedoff, looping between all the different input squares.
To start, if any square has been stripped down to zero possibilities, its tilearray is bumped back to all possibilities. If a square only has one possibility then the loop skips it since it's already collapsed.
Next I go through and get the square at each of the cardinal directions (unless it's on an edge). 
I get either the tileType if there is only one, or the first tile in the array of tiles if there are multiple. 
Then I go to the tileRules and pull out the forbidden tiles for that tile type and that direction. 
Then the square is filtered to remove any of those forbidden tiles. If for some reason this square results in no options, the square is randomized. (probably not the best way of collapsing, but this seems to work).
I get the entropy of each of the squares. I start with a high entropy and get a lower and lower entropy. The lowest entropy so far is sqmin and the index of that square is sqind. If sqind doesn't update then it stays as negative 1.
I either return sqind if there is still a square with entropy greater than one or the length of the tile array if nothing needed to be updated and everything was collapsed.

Back to the collapse new tiles function. If the output is the length of the input tiles then all tiles have collapsed. Then I break the while loop. Otherwise I randomize the tile that didn't collapse, and try again.

Once the tiles are collapsed, the PlaceTile function is called for each tile.

## Place Tile
The place tile takes in a single square input. Based on which tile the square has been collapsed to certain items are placed on that tile.
Each tile has different potential sand bars that are added as children of the square. The sandbars are rects that are sized based on the size of the square. They have area and body which allows them to collide with the player boat. These are tagged as ground.
Each square also has a fish or dolphin that are randomly placed on the tile. This sealife has neither area nor body since the player can't interset withn it. . All of these have a lower opacity so they look underwater.
Additionally, there is a chance that each tile will have a treasure or gold sunken in it. 
This gold has an area but no body since I don't want the player to stop moving when they collide with it. But it does have an isSensor tag in the area so the player can detect when it collides with it.
The treasure also has an amount that is randomized between 0 and 500, and then rounded.

## Dredge
When the button d is pressed, text with an ellipsis pops up on screen. After a 1 second delay, the dredge function is  called. 
This is meant to feel like the time it would take to drege something up from the bottom of the sea. 
I wasn't sure how else to gamify this without adding a bunch of agents which would add more code complexity. So I was inspired by the game Dredge which I would highly recommend.
The dredge function checks whether gold has been found using the goldFound object defined above the update function. This is just an onCollide with gold. If gold was found then the amount is extracted from the object and added to the score.
The text out is concatenated to state the amount, then the object is destroyed.
If the object is null then the text out just reads bupkes.
This is passed back to the function call to write out the text. Then after a second the text is destroyed.

## Help
I got a lot of comments when I demoed the early form of the game that it wasn't clear how to control the ship.
So in addition to having better visual indicators of the state of the ship, I added a button to go to a help menu.
So I added an on keypress to create an object called menu from the help function. And on release the menu object is destroyed.
The help function passes an object that is comprised of a rect and two texts.
The rect just overlays the game with a small offset for aesthetics. 
I don't think a pause function is necessary since there's no real threat to hanging out in the ocean.
There is a text that is just the title of the page and there is a text that includes the actual instructions.



