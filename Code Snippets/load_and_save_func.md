# Loading and Saving A Game

Below I'm including my functionality to load/save the game. I use Java Serialization to save / load the game

## Save Game

### Code
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
- After everything if the file is created