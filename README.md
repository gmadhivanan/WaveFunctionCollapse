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
The gridOffset was used to create a gap in between the tiles. It was present in earlier versions but is now there just in case the tiles need to be debugged where i can see the borders
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





