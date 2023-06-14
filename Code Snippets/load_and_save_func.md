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

- The user first selects the file from the file picker in GUI
```java
userInput = fileNameInputField.getText();

String currentDirectory = System.getProperty("user.dir");
            		        		
// Construct the file path using the current working directory and the file name. The file will be saved to the current folder
String filePath = currentDirectory + File.separator + userInput + ".ser";
            		
//Create a file object that represents the location where the game will be saved
File saveFile = new File(filePath);
```