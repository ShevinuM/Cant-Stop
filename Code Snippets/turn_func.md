# Taking A Turn
This functionality involves a lot of heavy logic to determine where the cubes must / can be placed based on the current status of the board since there's a lot of possibilities.

## Structure
- There's a lot of methods, instance variables and classes involved in this functionality. I'll try to explain the structure as best as I can but I'm not including all of them here. I'll only include the most important ones. If you run the game, there might be some bugs related to the colours of the board updating later than expected but this is a GUI bug and I have throughly tested the backend and my data structures are updating properly. The functionality also works as expected.

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
                3. Now if the user has legal moves with free cubes remaining, i call the moveFreeCubes method. This method has a lot of logic so I'm not explaining every line of code. I'll explain the general idea.
                    1. The method initially handles some exceptions, then create a hashmap to store the new positions so we can update the positions of the white cubes later.
                    2. The number of free white cubes is an instance variable of the turn class. I first check if it's 3. If so I use the following logic to move the cubes.
                        ```java
                        // if the firstSum is equal to the secondSum then we can only move one cube and we make two moves in that column
                        if (firstSum == secondSum) {
                            // Increments cube1 2 steps in the firstSum column.
                            incrementFreeCubesTwoSteps(firstSum, "cube1");
                        } else {
                            setPositionOfFreeCubes(firstSum, "cube1");
                            if (!columnsClaimedSet.contains(firstSum)) {
                                cubeToMove = "cube2";
                            } else {
                                cubeToMove = "cube1";
                            }
                            setPositionOfFreeCubes(secondSum, cubeToMove);
                        }
                        ```
                    3. If the number of free white cubes is 2, I use the following logic to move the cubes.
                        ```java
                        int cube1Position = positions.get("cube1")[1];
                        int columnToUse; // variable to keep track of which column to add the free cube if the cube already on board falls on 1st or 2nd sum
                        if (firstSum == secondSum) {
                            /**
                            *  check if the cube that is not free (which is cube 1 because we add cubes to the board in an orderly manner) is in
                            *  firstSum column
                            */
                            if (positions.get("cube1")[0] == firstSum) {
                                /**
                                * Check if the cube is about to be claimed which means it's in the square below the last. If it is, we don't have to
                                * move the white cube 2 spaces up
                                */
                                if (isAboutToBeClaimed(firstSum, cube1Position)) {
                                    setPositionOfExistingWhiteCubes(firstSum, "cube1", 1);
                                } else {
                                    setPositionOfExistingWhiteCubes(firstSum, "cube1", 2);
                                }
                            } else {
                                incrementFreeCubesTwoSteps(firstSum, "cube2");
                            }
                        } else {
                            if (positions.get("cube1")[0] == firstSum) {
                                newPositions.put("cube1", new int[] {firstSum, cube1Position+1});
                                whiteCubes.setPosition(newPositions);
                                columnToUse = secondSum;
                                setPositionOfFreeCubes(secondSum, "cube2");  
                            } else if (positions.get("cube1")[0] == secondSum) {
                                newPositions.put("cube1", new int[] {secondSum, cube1Position+2});
                                whiteCubes.setPosition(newPositions);
                                columnToUse = firstSum; 
                                /**
                                * We know that the cube 1 is in the column represented by the secondSum so we have to move cube 2 to the column 
                                * represented by the first sum
                                */
                                setPositionOfFreeCubes(firstSum, "cube2");
                            } else {
                                setPositionOfFreeCubes(firstSum, "cube2");
                                if (!columnsClaimedSet.contains(firstSum)) {
                                    cubeToMove = "cube2";
                                } else {
                                    cubeToMove = "cube1";
                                }
                                setPositionOfFreeCubes(firstSum, cubeToMove);
                                
                            }
                                
                        }
                        ```
                    4. If the number of free white cubes is 1, I use the following logic.
                        ```java
                        int cube1Position = positions.get("cube1")[1];
                        int cube2Position = positions.get("cube2")[1];
                        if (firstSum == secondSum) {

                            /**
                            *  check if cube 1 is in firstSum column
                            */
                            if (positions.get("cube1")[0] == firstSum) {
                                /**
                                * Check if the cube is about to be claimed which means it's in the square below the last. If it is, we don't have to
                                * move the white cube 2 spaces up
                                */
                                if (isAboutToBeClaimed(firstSum, cube1Position)) {
                                    setPositionOfExistingWhiteCubes(firstSum, "cube1", 1);
                                } else {
                                    setPositionOfExistingWhiteCubes(firstSum, "cube1", 2);
                                }
                            } else if (positions.get("cube2")[0] == firstSum) {
                                /**
                                * Check if the cube is about to be claimed which means it's in the square below the last. If it is, we don't have to
                                * move the white cube 2 spaces up
                                */
                                if (isAboutToBeClaimed(firstSum, cube2Position)) {
                                    setPositionOfExistingWhiteCubes(firstSum, "cube2", 1);
                                } else {
                                    setPositionOfExistingWhiteCubes(firstSum, "cube2", 2);
                                }
                            } else {
                                incrementFreeCubesTwoSteps(firstSum, "cube3");
                            }
                                    
                        } else {
                            
                            if (positions.get("cube1")[0] == firstSum) {
                                setPositionOfExistingWhiteCubes(firstSum, "cube1", 1);
                                
                                if (positions.get("cube2")[0] == secondSum) {
                                    setPositionOfExistingWhiteCubes(secondSum, "cube2", 1);
                                    
                                } else {
                                    setPositionOfFreeCubes(secondSum, "cube3");
                                }
                                
                            } else {
                                if (positions.get("cube1")[0] == secondSum) {
                                    setPositionOfExistingWhiteCubes(secondSum, "cube1", 1);
                                    
                                    if (positions.get("cube2")[0] == firstSum) {
                                        setPositionOfExistingWhiteCubes(firstSum, "cube2", 1);
                                    } else {
                                        setPositionOfFreeCubes(firstSum, "cube3");
                                    }
                                } else {
                                    if (positions.get("cube2")[0] == firstSum) {
                                        setPositionOfExistingWhiteCubes(firstSum, "cube2", 1);
                                        setPositionOfFreeCubes(secondSum, "cube3");
                                    } else {
                                        if (positions.get("cube2")[0] == secondSum) {
                                            setPositionOfExistingWhiteCubes(secondSum, "cube2", 1);
                                            setPositionOfFreeCubes(firstSum, "cube3");
                                        } else {
                                            int selectedSum = SumDialogBox.showSumDialogBox(firstSum, secondSum);
                                            if (selectedSum == firstSum) {
                                                setPositionOfFreeCubes(firstSum, "cube3");
                                            } else if (selectedSum == secondSum){
                                                setPositionOfFreeCubes(secondSum, "cube3");
                                            }
                                        }
                                    }
                                }
                                
                            }
                        }
                    5. The code I included above involves a lot of logic since there's multiple possible combinations. I have debugged the code with various test cases and it resulted in the correct output. If you read the logic carefully, all of the possible combinations are covered.
                    6. The above method uses other methods to set the position of free cubes and I'll explain those here.
                        1. setPositionOfFreeCubes is used to set the position of free cubes. It takes in a cube number and a sum (which corresponds to the column) and the sets the position of the cube. It also checks if the user has any coloured cubes placed in the column from previous turns. If so we set it one above the coloured cubes. Here's how I set it if for example the player's colour was red.
                            ```java
                            if (!columnsClaimedSet.contains(sum)) {
                                if (CubePositions.RedCubePositions.cubeExistsInColumn(sum)) {
                                columnPosition = CubePositions.RedCubePositions.getPosition(sum);
                                System.out.println(columnPosition + 1);
                                newPositions.put(cubeNumber, new int[] {sum, columnPosition + 1});
                                } else {
                                    newPositions.put(cubeNumber, new int[] {sum, 1});
                                }
                            }
                            whiteCubes.setPosition(newPositions);
                            ```
                        2. setPositionOfExistingWhiteCubes is used to set the position of white cubes already on the board. It takes in the column number, cube number and number of steps (1 or 2 steps). 
                            ```java
                            int cubePosition = positions.get(cubeNumber)[1];
                            newPositions.put(cubeNumber, new int[] {sum, cubePosition + numberOfPositions});
                            whiteCubes.setPosition(newPositions);
                            ```
                        3. incrementFreeCubesTwoSteps is used to increment a free cube two steps. It takes in the column number and cube number. It handles edge cases such as instances where the move is 2 steps but the column will be claimed in 1 step. This is just one edge case that's handled, there's more in the code. For example, here's how I increment a free cube by two steps if the player is red.
                            ```java
                            // Below line checks if the user already has a red cube in the column
                            if (CubePositions.RedCubePositions.cubeExistsInColumn(sum)) {
                                columnPosition = CubePositions.RedCubePositions.getPosition(sum);
                                /**
                                *  Checks if the column is about to be claimed which means the red cube is on the square below the last. If so
                                *  we don't increment the columnPosition by 2
                                */
                                if (isAboutToBeClaimed(sum, columnPosition)) {
                                    newPositions.put(cubeNumber, new int[] {sum, columnPosition + 1});
                                } else {
                                    newPositions.put(cubeNumber, new int[] {sum, columnPosition + 2});
                                }
                            } else {
                                newPositions.put(cubeNumber, new int[] {sum, 2});
                            }
                            whiteCubes.setPosition(newPositions);
                            ```
                        4. isAboutToBeClaimed is used to check if a column is about to be claimed (1 step away from getting claimed). If so it returns true. I do this by comparing the number of rows a column has with the current position.
                            ```java
                            if (columnNumber == 2 || columnNumber == 12) {
                                if (columnPosition == 2) {
                                    return true;
                                } else {
                                    return false;
                                }
                            } else if (columnNumber == 3 || columnNumber == 11 ) {
                                if (columnPosition == 4) {
                                    return true;
                                } else {
                                    return false;
                                }
                            } else if (columnNumber == 4 || columnNumber == 10) {
                                if (columnPosition == 6) {
                                    return true;
                                } else {
                                    return false;
                                }
                            } else if (columnNumber == 5 || columnNumber == 9) {
                                if (columnPosition == 8) {
                                    return true;
                                } else {
                                    return false;
                                }
                            } else if (columnNumber == 6 || columnNumber == 8) {
                                if (columnPosition == 10) {
                                    return true;
                                } else {
                                    return false;
                                }
                            } else if (columnNumber == 7) {
                                if (columnPosition == 12) {
                                    return true;
                                } else {
                                    return false;
                                }
                            } else {
                                return false;
                            }
                            ```
                        5. cubeExistsInColumn is a method of the cube storage class which given a column returns if a cube of that colour exists or not.
                4. If the player doesn't have free cubes to move, then i call the method moveCubesOnBoard which moves cubes on the board. This is the logic behind moving cubes of board which handles all edge cases.
                    ```java
                    if (firstSum == secondSum) {
                        if (positions.get("cube1")[0] == firstSum) {
                            cubeToMove = "cube1";
                        } else if (positions.get("cube2")[0] == firstSum) {
                            cubeToMove = "cube2";
                        } else if (positions.get("cube3")[0] == firstSum) {
                            cubeToMove = "cube3";
                        } else {
                            cubeToMove = "noCube";
                        }
                        if (!cubeToMove.equals("noCube")) {
                            int cubePosition = positions.get(cubeToMove)[1];
                            if (isAboutToBeClaimed(firstSum, cubePosition)) {
                                    setPositionOfExistingWhiteCubes(firstSum, cubeToMove, 1);
                            } else {
                                if (!isClaimed(firstSum, cubePosition)) {
                                    setPositionOfExistingWhiteCubes(firstSum, cubeToMove, 2);
                                }
                            }
                        }
                    } else {
                        if (positions.get("cube1")[0] == firstSum) {
                            if(!isClaimed(firstSum, positions.get("cube1")[1])) {
                                setPositionOfExistingWhiteCubes(firstSum, "cube1", 1);
                            }
                            if (positions.get("cube2")[0] == secondSum) {
                                if (!isClaimed(secondSum, positions.get("cube2")[1])) {
                                    setPositionOfExistingWhiteCubes(secondSum, "cube2", 1);
                                }
                            } else if (positions.get("cube3")[0] == secondSum) {
                                if (!isClaimed(secondSum, positions.get("cube3")[1])) {
                                    setPositionOfExistingWhiteCubes(secondSum, "cube3", 1);
                                }
                            }
                        } else if (positions.get("cube2")[0] == firstSum) {
                            if (!isClaimed(firstSum, positions.get("cube2")[1])) {
                                setPositionOfExistingWhiteCubes(firstSum, "cube2", 1);
                            }
                            if (positions.get("cube1")[0] == secondSum) {
                                if (!isClaimed(secondSum, positions.get("cube1")[1])) {
                                    setPositionOfExistingWhiteCubes(secondSum, "cube1", 1);
                                }
                            } else if (positions.get("cube3")[0] == secondSum) {
                                if (!isClaimed(secondSum, positions.get("cube3")[1])) {
                                    setPositionOfExistingWhiteCubes(secondSum, "cube3", 1);
                                }
                            }
                        } else if (positions.get("cube3")[0] == firstSum) {
                            if (!isClaimed(firstSum, positions.get("cube3")[1])) {
                                setPositionOfExistingWhiteCubes(firstSum, "cube3", 1);
                            }
                            if (positions.get("cube2")[0] == secondSum) {
                                if (!isClaimed(secondSum, positions.get("cube2")[1])) {
                                    setPositionOfExistingWhiteCubes(secondSum, "cube2", 1);
                                }
                            } else if (positions.get("cube1")[0] == secondSum) {
                                if (!isClaimed(secondSum, positions.get("cube1")[1])) {
                                    setPositionOfExistingWhiteCubes(secondSum, "cube1", 1);
                                }
                            }
                        } else {
                            if (positions.get("cube1")[0] == secondSum) {
                                if (!isClaimed(secondSum, positions.get("cube1")[1])) {
                                    setPositionOfExistingWhiteCubes(secondSum, "cube1", 1);
                                }
                            } else if (positions.get("cube2")[0] == secondSum) {
                                if (!isClaimed(secondSum, positions.get("cube2")[1])) {
                                    setPositionOfExistingWhiteCubes(secondSum, "cube2", 1);
                                }
                            } else if (positions.get("cube3")[0] == secondSum) {
                                if (!isClaimed(secondSum, positions.get("cube3")[1])) {
                                    setPositionOfExistingWhiteCubes(secondSum, "cube3", 1);
                                }
                            }
                        }
                    }
                5. After the cubes have moved, it's time to check if there is a winner. To do this, I get the columns claimed by each player. 
                    ```java
                    HashMap<String, HashSet<Integer>> columnsClaimedWithPlayers = currentGame.getColumnsClaimed();
                    ```
                    This is a hashmap mapping the player name to a set which contains the columns that the player has claimed.
                    ```
                6. I then check if any of the players have claimed 3 columns. If so, I pass the name to the winnerGUI and end the game
                    ```java
                    for (Map.Entry<String, HashSet<Integer>> entry : columnsClaimedWithPlayers.entrySet()) {
                        String key = entry.getKey();
                        HashSet<Integer> value = entry.getValue();
                        int size = value.size();
                        if (size == 3) {
                            // end game
                            WinnerDialog winnerDialog = new WinnerDialog();
                            winnerDialog.show(key);
                            try{
                                Thread.sleep(5000);
                            }
                            catch (InterruptedException ex){
                                ex.printStackTrace();
                            }
                            gameGUI.dispose();
                        }
                    }
                    ```

            4. Now finally I check if a cube has moved. If so, I return true. The white cubes have moved in the backend and my colleague implemented the frontend to show the movement of the cube.
            5. I finally call the setPositions method of the turn class to change the positions of the coloured cubes. Here's how it's done.
                1. First the setPositions method determines the column each white cube is located at and calls the setColouredCubePositions method passing the column number and position.
                    ```java
                    int[] cube1 = whiteCubes.getPositions().get("cube1");
                    int[] cube2 = whiteCubes.getPositions().get("cube2");
                    int[] cube3 = whiteCubes.getPositions().get("cube3");
                    int[][]cubes = {cube1, cube2, cube3};
                    int[] columnPosition;
                    for (int i=0 ; i < cubes.length; i++) {
                        columnPosition = cubes[i];
                        if (columnPosition[0] == 2) {
                            setColoredCubePositions("col2", columnPosition[1]);
                        } else if (columnPosition[0] == 3) {
                            setColoredCubePositions("col3", columnPosition[1]);
                        } else if (columnPosition[0] == 4) {
                            setColoredCubePositions("col4", columnPosition[1]);
                        } else if (columnPosition[0] == 5) {
                            setColoredCubePositions("col5", columnPosition[1]);
                        } else if (columnPosition[0] == 6) {
                            setColoredCubePositions("col6", columnPosition[1]);
                        } else if (columnPosition[0] == 7) {
                            setColoredCubePositions("col7", columnPosition[1]);
                        } else if (columnPosition[0] == 8) {
                            setColoredCubePositions("col8", columnPosition[1]);
                        } else if (columnPosition[0] == 9) {
                            setColoredCubePositions("col9", columnPosition[1]);
                        } else if (columnPosition[0] == 10) {
                            setColoredCubePositions("col10", columnPosition[1]);
                        } else if (columnPosition[0] == 11) {
                            setColoredCubePositions("col11", columnPosition[1]);
                        } else if (columnPosition[0] == 12) {
                            setColoredCubePositions("col12", columnPosition[1]);
                        }
                    }
                    ```
                2. The setColoredCubePositions determines the current colour and sets the position accordingly. For example, here's how it sets if the colour is red.
                    ```java
                    if (currentColor.equals(Color.RED)) {
                        CubePositions.RedCubePositions.setPosition(column, columnPosition);
                    }
                    ```
- When the dice has rolled. The user can either continue until the player get busted or the player can end the turn. If the player ends the turn the turn, then the columns claimed during the turn gets updated. Here's how it's done.
    - First I call the updateColumnsClaimed method of the turn class. This method takes in the current player and the game as parameters.
        ```java
        turn.updateColumnsClaimed(currentPlayer, game);
        ```       
    - Then this method call addToColumnsClaimed method
        ```java
        String playerName = currentPlayer.getName();
		if (currentPlayer.getColour().equals(Color.RED)) {
			for (Map.Entry<String, Integer> entry : CubePositions.RedCubePositions.returnRedCubePositions().entrySet()) {
				addToColumnsClaimed(entry, playerName, gameInProgress);
			}
		}
        ```
    -  This method checks if the column has been claimed by the player and calls the addToColumnsClaimed method of the Game class.
        ```java
        String key = entry.getKey();
	    Integer value = entry.getValue();
	    int colNumber = Integer.parseInt(key.substring(3));
	    if (isClaimed(colNumber, value)) {
	    	gameInProgress.addToColumnsClaimed(playerName, colNumber);
	    }
        ```
    - This method updates the HashMap mapping each player to the Set consisting of the columns each player has claimed. This is an instance variable and this variable exists untill the Game ends providing a way to keep track of the columns claimed by the players throughout the game.
        ```java
        HashSet<Integer> set = columnsClaimed.get(name);
    	set.add(column);
    	columnsClaimed.put(name, set);
        ```
- If the player gets busted, the cube positions are reset back to their initial position before the turn began. The way I do this is using the instance variable which stores the state of the cubes before the turn begins. After the turn ends, I reset the cube Positions in the storage class to what I have in the instance variables.
    ```java
    if (getPlayer().getColour().equals(Color.RED)) {
        for (Map.Entry<String, Integer> entry : redCubePositions.entrySet()) {
            String key = entry.getKey();
            Integer value = entry.getValue();
            CubePositions.RedCubePositions.setPosition(key, value);
        }
    }
    ```


    	




