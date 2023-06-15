# Rolling the 4 Dice

## Structure
- Constructor - Creates a new instance of the Random class.
- Getters
    - `getNumber()` - Returns the result of rolling a single die.
    - `getDiceRolls()` - Rolls four dice and returns the 4 numbers obtained by rolling each die as a list.
    - `getCombi(List<Integer> diceRolls)` - Returns a list of all possible combinations of pairs of numbers obtained by rolling four dice

## Logic
- The dice rolls are simulated through `randInt` method of the `Random()` class.
- The first step is to form the combinations properly. To do this,
    - I create a clone of the diceRolls list - `duplicatedDiceRolls`.
    - I then remove the first element of the clone and iterate over the rest of the list.
        - I create a new list - `list2` and clone the `duplicateDiceRolls` list again.
        - The for-loop is iterating over the 3 elements except the first element. In each iteration, I pair 1 of the 3 elements with the first element. Now, the index of that element `roll` is different in list2 and duplicateDiceRolls. So, I find the index of the element in list2 and remove it from list2.
            ```java
            for (Integer roll : duplicatedDiceRolls) {
                // rest of code
                int indexToRemove = list2.indexOf(Integer.valueOf(roll));
                int elementToRemove = list2.remove(indexToRemove);
                // rest of code
            }
            ```
        - After this i create a new list and add the first element and the elementToRemove to the pair.
        - `list2` now contains the remaining two elements.
        - These two elements are sorted to ensure the hashset handles the duplications properly.
        - So, I have the two pairs now. I add the two pairs to a new list forming a new combination.
        - Now, the other issue when forming the combination is that the pairs can be in different order. So, I sort the pairs to ensure that the pairs are in ascending order.
        ```java
        if (pair1.get(0) + pair1.get(1) > pair2.get(0) + pair2.get(1)) {
    		combination.add(pair1);
        	combination.add(pair2);
    	} else {
    		combination.add(pair2);
    		combination.add(pair1);
    	}
    	combinations.add(combination);
        ```
    - A question which someone would have is I'm only pairing the first element with the rest of the elements. What about the other elements. Well, the answer is that, the first element will always be paired with another
    element in the combination in all possible combinations. If i repeat the process for other elements, it'll
    involve duplicated combinations so I don't have to do that. It doesn't matter if i use the first element or not, I just need to pick an element and then use it to pair it with the rest.
- To store the combinations, I use a set to avoid duplication of combinations. Since we sorted each pair and then each combination, the set will remove duplications because the combinations are in ascending order.





