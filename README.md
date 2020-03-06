# a2-1

### PART 1 : IJK

#### 1. Deterministic part
We have solved given problem using minimax algorithm logic for two player game. We also used alpha-beta prunning to get results faster by searching the game tree at deeper level.

#### 2. Non-deterministic part
We used logic of expectiminimax algorithm in Stochastic Two-Player. Here a chance node move is introduced after each move of player1 or player2.

#### Description of program working
- We have considered the AI player as Max Player. Thus the next_move function gives the best move for Max player (AI) by calling best_max_move function for deterministic game and by calling best_max_move_nondet function for non-deterministic game.
- For Deterministic game :
  - As AI is assumed as Max player it assumes the other player as min player who plays optimally to not lose the game.
  - The function best_max_move calculates the best move for Max player by choosing the best possible move which has maximum   
    heuristic value. The function best_min_move calculates the best move for Min player by choosing the best possible move 
    which minimizes the heuristic value.
  - The game tree is searched till a particular depth (currently kept at 6) and the best moves at each level by each player is 
    calculated and finally the best move is returned to the next_move function.
  - We have also used the alpha-beta prunning to efficiently search the game tree.
  
- For Non-deterministic game :
  - Here after every move of one player there is a chance node which means the new 'A' or 'a' can be added at any of the empty     locations on the board. 
  - The function best_max_move_nondet calculates the best move for Max player by choosing the best possible move which has    
    maximum heuristic value. The function best_min_move_nondet calculates the best move for Min player by choosing the best 
    possible move which minimizes the heuristic value. The function chance calculates the all possible locations at which new 
    'A' or 'a' after the player move can be added. There is equal probability for each of the empty locations. Thus we take 
    summation products of probability of each stage and its heuristic. In code this is simply achieved by taking sum of all 
    the heuristic of possible stages after adding 'A' or 'a' and then dividing it by the count of all possible states. 
  - The game tree is searched till a particular depth (currently kept at 3) and the best moves at each level by each player is 
    calculated and finally the best move is returned to the next_move function. Here game moves from Max-player -> chance -> 
    Min-player -> chance -> Max-player -> chance ... and so on.
    
#### Problems faced, assumptions and design decisions
design decisions -
- In the given program the makeMove function in logic_IJK makes a given move, swiches the player and adds extra 'A' or 'a' based on the next player at a certain position. Here add_piece function adds the new 'A' or 'a' based on the deterministic or non-deterministic game. In non-deterministic game, the chance node should be able to calculate the empty positions where the new 'A' or 'a' will be added. Using the given logic makeMove already adds this 'A' or 'a', but we need the board structure before this addition.
- As we did not want to change the functions in logic_IJK, we made a new function makeMove_nondet in ai_IJK which just makes the move and switches the player but doesn't add the new 'A' or 'a'. This will be added in the chance node.

assumptions -
- Piazza post @291 mentions that the depth level should be decided based on the allowed time limit per move. We were under impression that the time limit per move will be 60 seconds (as it is given in IJK.py that timeout=60).
- Thus we were not able to modify the code accordingly that the depth will change according to the timeout given
- As it is mentioned in the same post that the time limit per move would be 3 seconds during the tournament, we have chosen the depth limit in such a way that each move will complete within 3 seconds. We tested this on burrow and we have kept depth limit as 4 for deterministic part and 3 for non-deterministic part.

heuristic -
- We tried multiple heuristics. Our main focus while designing the heuristic was 
  1. There should be the Maximum tile present by the AI player on the board at each time.
  2. To achieve logic in one, the heuristic value contributed by larger tiles must be greater than the value contributed by   
      two smaller tile. for example - C contributes 7 while B contributes 3. Thus it is better to have one C on the board than 
      2 B.
  3. Another consideration is if there is a option between combining the characters of same format and different formats, 
      always favor the combination made by different formats. for example if having two options, combine 'b' and 'B' to make a 
      'C' than combining 'B' and 'B'. This will ensure to increase the heuristic value for capital letters and will also 
      decrese the contribution of small letters
  4. The AI can be playing with capital letters or small letters thus the heuristuc adjust accordingly.

#### references :
https://www.geeksforgeeks.org/minimax-algorithm-in-game-theory-set-4-alpha-beta-pruning/ 
https://stackabuse.com/minimax-and-alpha-beta-pruning-in-python/


# a2-2


### PART 2 : HORIZON FINDING

#### 1. Using individual Bayes' Nets for each column


This part is solved using an individual bayes net for each column that estimates the most-probable row (index) for that particular column given a vector of image gradient values. The probability for a particular row index x given the vector w will be

P(x|w) = P(w|x)P(x)/P(w)

To simplify this equation, we can directly eliminate P(x) and P(w) since here we are talking only about a single variable, P(w) will be the same for all possible x in the column. Also, P(w|x) will be 1 since it is a representation of the probability of x being in that column which, obviously, is 1. So the above equation reduces to

P(x|w) ~= P(x)

which is again, direct proportional to the image gradient values. So the probability distribution over a column can be modeled as

P(x_i) = x_i / sum(w)

Then, the probability of the index x being on the horizon is simply the $argmax$ over the column vector w.

#### 2. Using a single Bayes' Net for the entire image

This part is solved by using a Hidden Markov Model modeled on observed variables given by the image gradient values and hidden variables as the indices in each column. The dynamic programming - based Viterbi algorithm simplifies the task of computation.

#### 3. Incorporating Human Feedback in Part 2

A pair of coordinates (x,y) supplied by a human agent improves outputs produced by Part 2.

#### Design decisions, challenges and workarounds

1. Multiplication Factor:

A multiplicative factor of 10 has been applied to emission and transition probability matrices to counter the numeric underflow caused by multiplying very small values. Taking Logarithm was an alternative solution but unfortunately it caused more issues.

2. Emission Probability Matrix:

Emission probabilities for a pixel in a column are defined by a ratio of the pixel's intensity to the sum of intensities of the column as given by the image gradient values. This is a reasonable function since the essential information is captured i.e., higher the pixel gradient, greater the probability that it lies on the horizon. Another challenge faced was numeric underflow, so the multiplicative factor has been used.

3. Transition Probability Matrix:

For a index x in column c, the transtitions to column c+1 have been considered only for indices in x-t to x+t, where t was varied between 1, 2, and 3, and was ultimately fixed on 3. Rest of the transitions for x are asisgned to 0. This was done to counter extreme jump-arounds caused by the fluctuating image gradient values.

4. Countering numeric underflow and/or overflow with logarithmic operations:

Upon visualizing the probability outputs from the Viterbi algorithm (as low as -360th power of 10), we tried to use logarithm of the probabilities to scale up the results but the log function caused more issues than it solved. For example, one run-time warning was "Division by zero encountered in logarithm". We replaced zero values in the transition and emission probability matrices by small values (order of -10th power of 10, so as to not let these affect the actual probability distributions) but still these warnings continued.

#### Results:

1. For the image mountain1.jpg results are as follows:

![](part2/results/output__37_114simple.jpg?raw=true)
![](part2/results/output__37_114_map.jpg?raw=true)
![](part2/results/output__37_114_human.jpg?raw=true)

2. For the image mountain2.jpg results are as follows:

![](part2/results/output_2_62_209simple.jpg?raw=true)
![](part2/results/output_2_62_209_map.jpg?raw=true)
![](part2/results/output_2_62_209_human.jpg?raw=true)

3. For the image mountain4.jpg results are as follows:

![](part2/results/output_4_67_200simple.jpg?raw=true)
![](part2/results/output_4_67_200_map.jpg?raw=true)
![](part2/results/output_4_67_200_human.jpg?raw=true)

4. For the image mountain5.jpg results are as follows:

![](part2/results/output_5_55_43simple.jpg?raw=true)
![](part2/results/output_5_55_43_map.jpg?raw=true)
![](part2/results/output_5_55_43_human.jpg?raw=true)

5. For the image mountain7.jpg results are as follows:

![](part2/results/output_7_38_155simple.jpg?raw=true)
![](part2/results/output_7_38_155_map.jpg?raw=true)
![](part2/results/output_7_38_155_human.jpg?raw=true)

6. For the image mountain8.jpg results are as follows:

![](part2/results/output_8_63_114simple.jpg?raw=true)
![](part2/results/output_8_63_114_map.jpg?raw=true)
![](part2/results/output_8_63_114_human.jpg?raw=true)

7. For the image mountain9.jpg results are as follows:

![](part2/results/output_9_76_77simple.jpg?raw=true)
![](part2/results/output_9_76_77_map.jpg?raw=true)
![](part2/results/output_9_76_77_human.jpg?raw=true)

8. A further experiment was also carried out with the intuition that incorporating more human feedback would improve the results from Part 3. So after carefully choosing 3 distinct points for the given image mountain2.jpg, the following result was obtained:

![](part2/results/multiple_points.jpg?raw=true)

The code for the further experiment can be found in the file experiment.py.

#### References:

1. https://stackoverflow.com/questions/9729968/python-implementation-of-viterbi-algorithm
2. http://www.adeveloperdiary.com/data-science/machine-learning/implement-viterbi-algorithm-in-hidden-markov-model-using-python-and-r/
3. https://github.com/aimacode/aima-python/blob/master/viterbi_algorithm.ipynb
4. https://stackoverflow.com/questions/6771428/most-efficient-way-to-reverse-a-numpy-array

