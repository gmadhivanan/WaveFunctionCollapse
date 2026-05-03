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

