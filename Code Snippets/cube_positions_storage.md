# Storing The Cube Positions During A Game
There are two types of cube positions that should be saved, the coloured cubes and white cubes. White cubes are the cubes that get advanced during a user's turn and coloured cubes are the permanant cubes that are unique to each player that are placed on the board. So there are two classes
    1. CubePositions - to store the coloured cube positions
    2. WhiteCubePositions - to store the white cube positions


## Storing White Cube Positions

### Structure
- Instance Variables
    - `whiteCubePositions` - HashMap mapping the white cube to an array of the form [column number, position in the column] for each white cube.
- Constructor
    - Adds the three white cubes to the hashmap and initialises the positions to -1 indicating that there's no cubes placed in any of the columns
- Getter
    - `getPositions` - returns the HashMap containing the white cube positions
- Setter
    - `setPosition` - takes a hashmap of the form `HashMap<String, int[]> newPositions)` and updates the cube positions.
    - Here's how i set the position of the first white cube.

### Code
- Here's how I set the positions of the white cubes.
    ```java
    int[] cube1Position = newPositions.get("cube1");
    if (cube1Position[0] != -2) { cubePositions.put("cube1", cube1Position); }
    ```
    -2 here indicates that the position remains unchanged.
- The rest of the code consists of basic constructors and setters.


## Storing Coloured Cube Positions

### Structure
- Each coloured cube has a seperate static class to store it's position. This is because each coloured cube has different positions on the board for each player.
    - Instance Variables
       `redcubePositions` - HashMap mapping the column number to the position in the column the cube is placed. The keys of the hashmap are the column names and the values are the positions.
    - Constructor
        - Adds the columns to the hashmap and initialises the positions to -1 indicating that there's no cubes placed in any of the columns
    - Getter
        `getPosition` - Takes in a column number as a parameter and returns the position of the cube in that column.
    - `cubeExistsInColumn` - Takes in a column number as a parameter and returns true if a cube of the colour cube exists in that column.
    - Setter
        `setPosition` - Takes in a column number and a position as parameters and updates the cube position.
    - `returnRedCubePositions()` - returns the HashMap containing the red cube positions. This is used for saving the game.

### Code
- This the HashMap which is used to stored coloured cube positions.
    ```java
    HashMap<String, Integer> redCubePositions
    ```
- This is how i set the position of a coloured cube. It checks if the column is valid and then sets the position. It handles illegal arguments by throwing an exception.
    ```java
    if (column.matches("col(1[0-2]|[2-9])")) {
        redCubePositions.put(column, position);
    } else {
        throw new IllegalArgumentException("The column is not between 2 and 12.");
    }
    ```
- This is how `getPosition` is implemented when a column number is taken as a parameter.
    ```java
    String getterString = "col" + columnNumber;
	return redCubePositions.get(getterString);
    ```
- Similarly i check if a cube exists by checking if the position is not -1.
    ```java
    if (redCubePositions.get(getterString) != -1) {
	    return true;
	}
    ```
