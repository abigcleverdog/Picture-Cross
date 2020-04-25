# Picture-Cross
A solver for the game ['Picture Cross'](https://apps.apple.com/us/app/picture-cross/id977150768)

enjoyed the game for a couple of rounds and wondered if I could make a solver of it.
## thoughts
the game looks like a different version of 'Sudoku'. 
'Sudoku' board is 9X9, each grid can be 1-9. the worst dfs permutation would be O(9^81) And I know it is doable on a decent labtop given effective PRUNING. 
'Picture Cross', the board can go up to 15x15, each grid is binary. the worst dfs permutation would be O(2^225) or O(8^75) So their scales are similar.
### [Approach 1](https://github.com/abigcleverdog/Picture-Cross/blob/master/Picture%20Cross%201.ipynb)
* solver()

input: the list of horizontal line restrictions, lsh, and the list of vertical line restrictions, lsv

output: the filled board, or boards if there are multiple

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

4. fill the board: fill line by line (horizontally). 

    For each line, with the line restriction and the line on the current board, there should be a few possible ways to fill the line. let's call them 'possible lines'. if there is no possible lines, the current attempt fail. reset the board to before this trial. return False. For each 'possible line' in the 'possible lines', fill the line according to 'possible line', check if the board is still valid according to vertical restrctions. if invalid, reset the board, return False; else, dfs to next line horizontally. filling will succeed when finish the last row.

this approach works with some 8x8 board, fairly quickly. however, when it comes to 15x15, it is too slow. too many branches.

*3x3 board -> 97 us; 4x4 board -> 206 us; 8x8 -> 12 ms;* **15x15 -> too long**
```
%timeit solve([[1],[2],[1,1]], [[2],[1],[1,1]])
97.3 µs ± 2.8 µs per loop (mean ± std. dev. of 7 runs, 10000 loops each)
In [6]:
%timeit solve([[1],[3],[1],[2]], [[1,1],[1,1],[2],[1]])
206 µs ± 3.7 µs per loop (mean ± std. dev. of 7 runs, 10000 loops each)
In [7]:
%timeit solve([[1,2],[1,2],[1,2],[2],[3],[2,1],[2,1],[1,1]],[[3,2],[2],[2],[2],[6],[2],[2],[1]])
12 ms ± 868 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
In [8]:
# %timeit solve(ls_h,ls_v) # took too long the run
```
**it was a little surprising for 15x15** the slowest step must be the dfs, so if there are 10 possible ways to fill each line, the dfs need to go through 10^15 which should be handlable. I must made the validation function quite inefficient.

### [Approach 2](https://github.com/abigcleverdog/Picture-Cross/blob/master/Picture%20Cross%202.ipynb)

* solver()

keep 'prefilling' (need different functions, may call it 'superPrefill') the board till no progress is made in an iteration; scan each line (both horizontally and vertically) for 'possible lines'; start dfs only when has to.

input: the list of horizontal line restrictions, lsh, and the list of vertical line restrictions, lsv

output: the filled board, or boards if there are multiple

1. check exceptions:
    1. the lengths of lsh and lsv differ (according to the game, we are only having square boards) 
    2. the sums of lsh and lsv differ (the total marks on board)
  
2. build the board:
    1. n = len(lsh); board = nxn grids
    2. each grid = v; 
        1. v = 0               -> undetermined;
        2. v = -1              -> cannot mark;
        3. v = 1               -> marked;

3. prefill the board: each line has n grids, so that each element in a line restriction has a highest-start and a lowest-end points, if there are grids between those boundaries, they should be marked.

4. scan the board line by line (horizontally and vertically), for each line, find all possible ways to fill the line according to the line restriction and the current board. 

5. scan line by line, for each line, screen all the possible ways of filling according to current board. if the number of options decrease, meaning some change can be made, confirm the grids in the line whose value is constant through all the trimmed possible fillings. loop till the no progress can be made.

6. start dfs if necessary.

*approch 1 results for comparison*

*3x3 board -> 97 us; 4x4 board -> 206 us; 8x8 -> 12 ms;* **15x15 -> too long**

3x3 board -> 93 us; 4x4 board -> 391 us; 8x8 -> 6 ms; 15x15 -> 997ms
```
%timeit solve([[1],[2],[1,1]], [[2],[1],[1,1]])
92.8 µs ± 2.56 µs per loop (mean ± std. dev. of 7 runs, 10000 loops each)
In [8]:
%timeit solve([[1],[3],[1],[2]], [[1,1],[1,1],[2],[1]])
391 µs ± 46 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
In [9]:
%timeit solve([[1,2],[1,2],[1,2],[2],[3],[2,1],[2,1],[1,1]],[[3,2],[2],[2],[2],[6],[2],[2],[1]])
5.82 ms ± 1.16 ms per loop (mean ± std. dev. of 7 runs, 100 loops each)
In [10]:
%timeit solve(ls_h,ls_v)
997 ms ± 236 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```


Now I am a little curious about the 4x4 board where there are multiple possible answers. To be fair, there are more than just one strategic difference between the 2 approaches. I should make a board generator so I can have many boards to test.
