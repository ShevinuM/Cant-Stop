# Taking A Turn
This class involves a lot of heavy logic to determine where the cubes must / can be placed based on the current status of the board since there's a lot of possibilities.

## Structure
- I'm only going to include the neccesary instance variables here.

## Logic
- When the user clicks the start game button, a new instance of the turn class is created. Then the roll dice button is visible and game has started. The text of the start game button also changes to End Turn, so the same button is used to start the game and end the turn.
- When the user clicks the roll dice button, I first check if the roll dice dialog is open. The roll dice dialog simply lets the user select the dice combinations. This prevents the game rolling the die multiple times.
- if the roll dice dialog is not open, I call the rollDice method.
    - First I get the rolled numbers -> `List<Integer> rolledNumbers = turn.roll();`
    - Then i get the combinations -> `Set<List<List<Integer>>> combinations = turn.getCombi(rolledNumbers);`
    - Then I display the combinations and allow the user to select a combination. I extract the values in the list and create string for the button text.
        ```java
        JButton button = new JButton(combinationList.get(0).toString() + " and " + combinationList.get(1).toString());
        ```
        Here the combinationList represents the two pairs we got or in other words the single combination.
    - Now, I need a method to interpret what was selected so I take the text from the button and convert it back to two lists of the two pairs.
        ```java
        String buttonText = button.getText();
        String[] pairsString = buttonText.split(" and ");
        String[] pair1String = pairsString[0].replaceAll("\\[|\\]", "").split(", ");
        List<Integer> pair1List = new ArrayList<>();
        for (String string : pair1String) {
            pair1List.add(Integer.parseInt(string));
        }
        String[] pair2String = pairsString[1].replaceAll("\\[|\\]", "").split(", ");
        List<Integer> pair2List = new ArrayList<>();
        for (String string : pair2String) {
            pair2List.add(Integer.parseInt(string));
        }
        ```
    - Now, I sum each pair.
        ```java
        int firstSum = pair1List.get(0) + pair1List.get(1); 
        int secondSum = pair2List.get(0) + pair2List.get(1);
        ```     
    - Now, the user can confirm or cancel and select another combination.
    - If the user confirms, I call the canMoveCube method of the Turn() class to check if the user has a valid move from the selected combination. The code then tries to move the cube and if it's successful, the cube is moved and the turn ends. 
        - If the user doesn't have a valid move, the invalid combination is removed and user is told to select another combination. If there are no other combinations or in other words, the combination list is empty, the current player is bust.
        - Here's how this works,
            1. First I call canMoveCube method of the turn class passing the two sums, current player, current instance of the game and current instance of whitecubes class is passed. Keep in mind that an instance of the turn class was created when the user clicked the user's turn began
                ```java
                if (turn.canMoveCube(firstSum, secondSum, currentPlayer, game, whiteCube)) {
                 // rest of code
                }
                ```
            2. First i assign values to some instance variables. I update the colour of the current turn by getting the player's colour. Then I update the current instance of the game in the turn to that which we passed. Then the current instance of the whitecubes class is updated to that which we passed. This allows us to access the positions of the whitecubes currently on board
                ```java
                currentColor = currentPlayer.getColour();
    	        currentGame = game;
                whiteCubes = whiteCube;
                ```
            3. Then I call the moveCube method.
                1. This method first gets the positions of the whitecubes from the getter. Then create copy of the positions. The reason I do this is, at the end I compare this copy to the current positions and if they are the same, the cube wasn't moved and I can return false.
                2. Then I check if the user has legal moves with free cubes remaining. This is done by calling hasLegalFreeCubes method and passing the two sums.
                    1. First I get the position of each white cube.
                        ```java
                        cube1Position = positions.get("cube1");
    	                cube2Position = positions.get("cube2");
    	                cube3Position = positions.get("cube3");
                        ```
                    2. Then I calculate the number of free cubes and which cubes are free.
                        ```java
                        numberOfFreeWhiteCubes = 0;
                        /**
                        * the cubePosition is in the form [column, position in column]
                        * if the column is -1 then the cube hasn't been placed in a column yet
                        * first check for how many cubes column is -1. numberOfFreeWhiteCubes store how many cubes are free.
                        */
                        if (cube1Position[0] == -1) {
                            cube1Free = true;
                            numberOfFreeWhiteCubes += 1;
                        } else {
                            cube1Free = false;
                        }
                        if (cube2Position[0] == -1) {
                            cube2Free = true;
                            numberOfFreeWhiteCubes += 1;
                        } else {
                            cube2Free = false;
                        }
                        if (cube3Position[0] == -1) {
                            cube3Free = true;
                            numberOfFreeWhiteCubes += 1;
                        } else {
                            cube3Free = false;
                        }
                        ```
                    3. Then I check if both the columns representing first and second sum are claimed. If they are claimed then the user obviously cannot make any moves.
                        ```java
                        if (columnsClaimedSet.contains(firstSum) && columnsClaimedSet.contains(secondSum)) {
			                return false;
		                }
                        ```
                    4. Then I check if the number of free white cubes is 2 or 3 or 0. If it's 2 or 3, player most certainly can make a move on a free cube. If 3 cubes are free, the player definitely can make a move but if only 2 cubes are free, there is an edge case which must be handled which is if one of the columns is claimed and the cube that is not free is in the other column. In this case we can't advance the free cube. This is handled later on. 
                        ```java
                        if (numberOfFreeWhiteCubes >= 2 ) {
                            return true;
                        } else if (numberOfFreeWhiteCubes == 0) {
                            return false;
                        ```
                    5. If the number of free cubes is 1, however we have to check if the sum of each pair does not fall onto one of the columns the current white cubes that are not free are placed.
                        ```java
                        /**
    		            * If the first sum is equal to the second sum, then we only have one column to move a free cube to so we have check
    		            * if either of the cubes remaining on board falls on that column
    		            */
                        if (firstSum == secondSum) {
                            if (cube1Free) {
                                if (cube2Position[0] == firstSum || cube3Position[0] == firstSum) { return false; }
                            }
                            if (cube2Free) {
                                if (cube1Position[0] == firstSum || cube3Position[0] == firstSum) { return false; }
                            }
                            if (cube3Free) {
                                if (cube2Position[0] == firstSum || cube1Position[0] == firstSum) { return false; }
                            }
                        }
                        if (cube1Free) {
                            /**
                            * There are only two instances where a user has a free cube but cannot move the cube to a column. That is when
                            * the sum of the first pair and sum of second pair falls on the columns representing the cubes already placed.
                            * In this case the only option for the user is to move the existing cubes.
                            */
                            if ((cube2Position[0] == firstSum && cube3Position[0] == secondSum) || 
                                    (cube2Position[0] == secondSum && cube3Position[0] == firstSum)) {
                                return false;
                            }
                        }
                        if (cube2Free) {
                            if ((cube1Position[0] == firstSum && cube3Position[0] == secondSum) || 
                                    (cube1Position[0] == secondSum && cube3Position[0] == firstSum)) {
                                return false;
                            }
                        }
                        if (cube3Free) {
                            if ((cube2Position[0] == firstSum && cube1Position[0] == secondSum) || 
                                    (cube2Position[0] == secondSum && cube1Position[0] == firstSum)) {
                                return false;
                            }
                        }
                        return true;
                        ```

    	




