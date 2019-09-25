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

## Main WHILE Loop

I decided to go with a while loop because we can theoretically continue to run this simulation until a stop condition is met.

```
status = ClearArray("previousSeed")
	
tmp = ArrayToCSVString(seedArray, ",") 
status = ArrayFromCSVString(tmp, ",", "", "T", "previousSeed")
```

Once we go into the loop we immediately cache the current seed array into a previous seed value. Because LIMS Basic won't let you copy an array with the assignment operator, a quick shorthand way to perform an array duplication is to convert the array to a csv string with a known-safe delimiter (such as | or \`) In this case I just went with a simple comma since our seed array is an integer-based data structure.

We then can re-convert the csv string back to an array and wa-la, array duplication achieved. With a cached previous seed we can check directly if the newly generated seed array is exactly the same as the previous, ending the simulation.

```
htmlOut = ""
htmlOut = htmlOut + "<h3>Current Generation: " + seedCount + "</h3>" + CRLF
htmlOut = htmlOut + "<table width={dblQt}100%{dblQt} height={dblQt}300px{dblQt} border={dblQt}1px{dblQt}>" + CRLF
```
 At this point we can start building the HTML structure itself to display the array output to the user. I decided to inline css because I really didn't want to use an external CSS file for this exercise.
 
```
neighbors = 0
curVal = previousSeed[i,j] 

IF (previousSeed[i - 1,j] > 0) AND (NotEmpty(previousSeed[i - 1,j])) THEN
	neighbors = neighbors + 1
ENDIF 

IF (previousSeed[i - 1, j + 1] > 0) AND (NotEmpty(previousSeed[i - 1, j + 1])) THEN
	neighbors = neighbors + 1
ENDIF 

IF (previousSeed[i - 1, j - 1] > 0) AND (NotEmpty(previousSeed[i - 1, j - 1])) THEN
	neighbors = neighbors + 1
ENDIF 

IF (previousSeed[i, j + 1] > 0) AND (NotEmpty(previousSeed[i, j + 1])) THEN
	neighbors = neighbors + 1
ENDIF 

IF (previousSeed[i,j - 1] > 0) AND (NotEmpty(previousSeed[i,j - 1])) THEN
	neighbors = neighbors + 1
ENDIF 

IF (previousSeed[i + 1,j] > 0) AND (NotEmpty(previousSeed[i + 1,j])) THEN
	neighbors = neighbors + 1
ENDIF 

IF (previousSeed[i + 1, j - 1] > 0) AND (NotEmpty(previousSeed[i + 1, j - 1])) THEN
	neighbors = neighbors + 1
ENDIF 

IF (previousSeed[i + 1, j + 1] > 0) AND (NotEmpty(previousSeed[i + 1, j + 1])) THEN
	neighbors = neighbors + 1
ENDIF 
```
The core of the loop here checks the current cell's neighbors as part of the rules of Conway's Game of Life, namely:

- If a Live Cell has less than 2 neighbors, it dies via starvation
- If a Live Cell has more than 3 neighbors, it dies from overcrowding
- If a dead Cell has exactly 3 neighbors, it will be born

A simple +- 1 on both the X and Y coordinates will easily get us the surrounding 8 neighbors. We use the "Not Empty" check for the array boundaries exclusively (When an out-of-index check happens, the result is EMPTY but when compared mathematically takes the string length of EMPTY aka 5 which is > 0. A rather annoying but actually nice part of Arrays in LIMS Basic as I don't have to deal with runtime errors for out-of-bounds index checks)

```
nextVal = 0

SELECTCASE curVal 
CASE 1
	IF (neighbors = 2) OR (neighbors = 3) THEN
		nextVal = 1
	ELSE 
		nextVal = 0
	ENDIF 
CASE 0
	IF (neighbors = 3) THEN 
		nextVal = 1
	ENDIF 
CASE ELSE 				
ENDSELECT

IF (nextVal = 1) AND (Not(hasSurvivors)) THEN
	hasSurvivors = TRUE
ENDIF

seedArray[i,j] = nextVal
```

Simple fall through Case Statement really. I suppose you can use IF()THEN but this looks a lot nicer and allows us in the future to expand to more values in the seed array (I did make a clone of this called Conway's Game of Thrones that implements this).

We also can quickly determine if this seed has any survivors. No extra O(N^2) cost involved.

```
tmp = "000000"

SELECTCASE nextVal
CASE 1
	tmp = liveColor 
CASE 0
	tmp = deadColor 
CASE ELSE 
ENDSELECT 

htmlOut = htmlOut + "<td bgcolor={dblQt}#{tmp}{dblQt}> </td>" + CRLF
htmlOut = StringReplace(htmlOut, "{tmp}", tmp , "T")
```

Finally we wrap up this loops iteration of the HTML Table Cell. I was lazy and did a string replace but it's probably a lot more clean and performant to inline the string mutations

```
htmlOut = htmlOut + "</table>" + CRLF

htmlOut = StringReplace(htmlOut, "{dblQt}", dblQt , "T") 

MsgboxHtml(htmlOut) 

IF (seedCount >= 25) THEN 
	continueSimulation = FALSE 
ENDIF 
' ====================================================================================
' If no Survivors, end simulation
IF (Not(hasSurvivors)) THEN
	continueSimulation = FALSE 
ENDIF 
' ====================================================================================
seedCount = seedCount + 1
```

Once we're out of the main nested FOR loop we'll just display the current seed to the player. If we have no survivors we'll end the simulation. I hard coded a limit of 25 simulations as there is no guaranteed User-Input simulation exit function, though this can easily be done with another HTML command.

