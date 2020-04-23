# Picture-Cross
A solver for the game ['Picture Cross'](https://apps.apple.com/us/app/picture-cross/id977150768)

enjoyed the game for a couple of rounds and wondered if I could make a solver of it.
## thoughts
the game looks like a different version of 'Sudoku'. 
'Sudoku' board is 9X9, each grid can be 1-9. the worst dfs permutation would be O(9^81) And I know it is doable on a decent labtop given effective PRUNING. 
'Picture Cross', the board can go up to 15x15, each grid is binary. the worst dfs permutation would be O(2^225) or O(8^75) So their scales are similar.
### solver set up
input: the list of horizontal line restrictions, lsh, and the list of vertical line restrictions, lsv

output: the filled board

1. check exceptions: 
    1. the lengths of lsh and lsv differ (according to the game, we are only having square boards) 
    2. the sums of lsh and lsv differ (the total marks on board)
  
2. build the board: 
    1. n = len(lsh); board = nxn grids
    2. each grid = [h, v]; 
        1. h = 0               -> undetermined;
        2. h = -1              -> cannot mark;
        3. h = n               -> need to mark, value undetermined;
        4. h = i, i in 1~(n-1) -> marked with (i-1)th element in the line restriction
    
3. prefill the board: each line has n grids, so that each element in a line restriction has a highest-start and a lowest-end points, if there are grids between those boundaries, they should be marked.

4. fill the board

### prefill/fill the board
approach 1: fill line by line (horizontally). 

For each line, with the line restriction and the line on the current board, there should be a few possible ways to fill the line. let's call them 'possible lines'. if there is no possible lines, the current attempt fail. reset the board to before this trial. return False. For each 'possible line' in the 'possible lines', fill the line according to 'possible line', check if the board is still valid according to vertical restrctions. if invalid, reset the board, return False; else, dfs to next line horizontally. filling will succeed when finish the last row.

this approach works with some 8x8 board, fairly quickly. however, when it comes to 15x15, it is too slow. too many branches.

approach 2: keep 'prefilling' (need different function, may call it 'superPrefill') the board till no progress is made in an iteration; scan each line (both horizontally and vertically) for 'possible lines'; start dfs with the line that has minimum possible lines.
