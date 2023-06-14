# Loading A Game
Below I'm including my functionality to loading the game. I'm using Java Deserialization to load the game.

## Structure
- I have a LoadGameGUI class which allows the user to select the file from the file picker and load the game. The Game object obtained from the file is passed as a parameter to the loadGame method of the Game class. Then the cube positions are set using setPosition method of CubePositions class.

## Backend
- The selected file is deserialized and the Game object is obtained and passed as a parameter to the loadGame method of the Game class.
    ```java
    try (FileInputStream fileIn = new FileInputStream(loadedFile);
        ObjectInputStream objectIn = new ObjectInputStream(fileIn)) {
    		Object obj = objectIn.readObject();
            if (obj instanceof Game) {
                Game selectedGame = (Game) obj;
                game.loadGame(selectedGame);
                JOptionPane.showMessageDialog(null, "Game loaded successfully");
                loadGameFrame.dispose();
            }
        }
    ```
- I also handled cases where the file is not loaded properly.
    - If the object loaded is not an instance of the Game class, the user is informed.
    - If the file doesn't get loaded at all I catch the IOException and ClassNotFoundException.
- Going to the loadGame method of the Game class, I load all the instance variables such as players, player settings, display options, difficulty. This is done simply assigning the instance variables of the game class we deserialized and assigning them to the current instance variables.
- Loading the coloured cube positions is done by a seperate method. Each game class has an instance variable corresponding to each coloured cube position and this variable is a HashMap mapping the column name to the position in the column the cube is placed. Taking each hashmap, I assign the column number to it's column position, using the setPosition method of the data storage class used to store the cube positions. Here's how I'm saving the red cube positions,
    ```java
    for (Map.Entry<String, Integer> entry : game.redCubePositions.entrySet()) {
		String key = entry.getKey();
		Integer value = entry.getValue();
		CubePositions.RedCubePositions.setPosition(key, value);
    }

## Frontend
- I also made the GUI for loading the game. This was done with a Swing JFrame.