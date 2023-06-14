# Saving A Game

Below I'm including my functionality to load/save the game. I use Java Serialization to save / load the game

## Structure 
- I have a SaveGameGUI class which allows the user to enter the filename and create a file which is passed as a parameter into the saveGame method of the Game class.

## Backend
   ```java
   public void saveGame(File file){
         // TODO: Write the code to save the difficulty, the current status of the game, display options
         saveCubePositions();
         saveDifficiculty();
         try {
            //Create an output stream to write the game data to the file that needs to be saved
            FileOutputStream fileOut = new FileOutputStream(file);
            
            //Write the game data to the output stream
            ObjectOutputStream out = new ObjectOutputStream(fileOut);
               out.writeObject(this);
               
               //Close the output stream
               out.close();
               fileOut.close();
            } catch (IOException i) {
               i.printStackTrace();
            }
      }
   ```

- The user first types the filename and then the file is saved in the current directory. The reason I did this was so that
the user won't forget where the files are saved if I allowed to use a file picker. 
   ```java
   userInput = fileNameInputField.getText();

   String currentDirectory = System.getProperty("user.dir");
                                 
   // Construct the file path using the current working directory and the file name. The file will be saved to the current folder
   String filePath = currentDirectory + File.separator + userInput + ".ser";
                     
   //Create a file object that represents the location where the game will be saved
   File saveFile = new File(filePath);
   ```
- I also handled cases where the file is not saved properly.
   - First is catching the exception because all of the code related to saving the file is inside a `try` block. The 
   following statement displays an error to the user.
      ```java
      catch (IOException ex) {
         JOptionPane.showMessageDialog(saveGameFrame, "An error occurred while saving the game: " + ex.getMessage());
      }
      ```
   - I also handled the case where the file name is either invalid or it exists. 
      ```java
      boolean created = saveFile.createNewFile();
      if (created) {
         
      } else {
         JOptionPane.showMessageDialog(saveGameFrame, "The specified filename is invalid or already exists.");
      }
      ```
- After everything if the file is created, I pass the created file as an argument to the saveGame method.
- I have another file which shows and explains the data storage class used to store cube positions while the game is going on.
- To permenantly save the cube positions, I use a hashmap for each color of the cube to map each column to the position of the cube in that column. This set of hashmaps are instance variables so they correspond to the current instance of the game.
- I first save the current cube positions.
   - This is done by first getting the cube positions from the data storage class and assigning them to the instance variables. 
   - For example, here's how I'm saving the red cube positions
      ```java
      this.redCubePositions = CubePositions.RedCubePositions.returnRedCubePositions();
      ```
- I don't need to manually save the rest of the settings for the game because the game already has instance variables for the list of players, player settings, display options, difficulty.
- Finally i save the game as an object through serialization.
   ```java
   ObjectOutputStream out = new ObjectOutputStream(fileOut);
   out.writeObject(this);
   ```
- Serialization allowed to save the entire instance of the game to a file and later on retrieve that entire instance if I want to load the game. I checked the file size and it was only 2 KB which is very small and I can also further extend the 
current game to a networked online multiplayer version by sending the serialized files back and forth since it's pretty small.

## Frontend
- I also made the GUI for saving the game. This was done with a Swing JFrame.
