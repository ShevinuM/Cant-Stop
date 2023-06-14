# Turn
This is by far in my opinion the most complex part of the project.Turn class is responsible for the game logic and the game flow. Dice class is represents the DiceRolls. Determining the positions of the cubes was pretty tough and the logic was pretty complex. Initially there were a lot of bugs as well. 

## Structure
- Instance Variables. I am only including the most important instance variables here since there's more.
    - `Set<List<List<Integer>>> combinations` - So in this game, the user rolls four dice. The user can choose any 2 combinations of the four dice. The inner-most list contains a single pair, the middle list contains two pairs and the outer set contains all possible combinations.

## Code
- To store the combinations, I use a set to avoid duplication of combinations. A question that can arise is, there can be multiple combinations that are the same but in a different order. For example, `[[1, 2],[3,4]]` and `[[2, 1],[4,3]]` are the same two combinations but in a different order. How does the set handle this. To avoid this, each pair is sorted to ensure that the pairs are in ascending order.
    ```java
    Collections.sort(pair1);
    ```
### Rolling the 4 Dice
- The dice rolls are simulated through `randInt` method of the `Random()` class.
- The first step is to form the combinations properly. To do this,
    - I create a clone of the diceRolls list - `duplicatedDiceRolls`.
    - I then remove the first element of the clone and iterate over the rest of the list.
        - I create a new list and clone the `duplicateDiceRolls` list again.
        - The first element is first paired with the second element, then third and then fourth and each of these is.

            ```java
            for (Integer roll : duplicatedDiceRolls) {
                // rest of code
                int indexToRemove = list2.indexOf(Integer.valueOf(roll));
                int elementToRemove = list2.remove(indexToRemove);
                // rest of code
            }
            ```
        



