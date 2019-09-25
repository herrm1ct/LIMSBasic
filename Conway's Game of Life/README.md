# Conway's Game of Life in LIMS Basic

### Overview

Brief history of Conway's Game of Life found [here](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life)

### Design / Architecture

Note: Variables in LIMS Basic subroutines should almost always be local variables. Context Variables (behaviior similar to public variables in most other languages) exist for the execution life cycle of the code and thus should not be "littered" around carelessly. Local variables will be released from memory once execution is done.

```
arrayBounds = 10
liveCellCutoff = 0.75

liveColor = "FF0000"
deadColor = "FFFFFF"
```

The above section is simply the parameters for the game, we are building a square 2 dimensional array, and defining the colors to assign to "live" and "dead" blocks within the array. We also have a cutoff value we'll use with a random number generator to seed the initial starting array and determie which cells start out alive or dead.

```
FOR i = 1 TO arrayBounds STEP 1 
	FOR j = 1 TO arrayBounds STEP 1

		seedArray[i,j] = Rnd(1) 

		IF (seedArray[i,j] >= liveCellCutoff) THEN 
			seedArray[i,j] = 1
		ELSE 
			seedArray[i,j] = 0
		ENDIF

	NEXT 
NEXT 
```

The above section we're simply building the initial seed array using a random number generator. The parameter for live cell cutoff kicks in to ensure whether or not our array position is alive (1) or not (0)
