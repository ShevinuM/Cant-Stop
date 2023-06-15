# Taking A Turn
This class involves a lot of heavy logic to determine where the cubes must / can be placed based on the current status of the board since there's a lot of possibilities.

## Structure
- I'm only going to include the neccesary instance variables here.

## Logic
- When the user clicks the start game button, a new instance of the turn class is created. Then we update the board if user has any coloured cubes in the column. This is required if we load a game. Here's how I'm doing this,
    - First, I call the updateColumnsClaimed method in Turn class passing the current player and current game as input parameters.
    - columnsClaimed is a HashSet unique to each instance of the game. It stores the columns that have been claimed so that non of the users will be able to make a move.