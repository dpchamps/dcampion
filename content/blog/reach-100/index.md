---
title: Fun With Reach 100
date: 2023-01-28T19:30:33.645Z
categories: [programming, math, graphs]
comments: true
draft: false
---

A couple of days ago I found this fun game posted on Hacker News: [Reach 100](https://reach-100.com/). I poked around a couple of 
times, with varying degrees of success: 65, 75. Then I started an attempt to assemble a strategy for maximizing the
number of hops. I don't really remember what I tried now. The result was a high score of 90. But I couldn't break it, 
after spending about 30 minutes or so getting significantly lower than 90 with different heuristics, I decided to cheat
and write some software to solve it for me. 

The first solution I came up with might be pretty un-remarkable to anyone who's been in this space before, or perhaps
for those who have had the displeasure to grind-out leetcode problems. Backtracking seemed like a good place to start
, so I cobbled together a first iteration:

```typescript
// setup code omitted for brevity, to inspect the entire script see: 

type SolveState = {
  x: number,
  y: number,
  path: [number, number][];
  board: number[];
}

const solve = ({x, y, path, board}: SolveState): SolveState['path']|undefined => {
  if(path.length === GAME_BOARD_WIDTH*GAME_BOARD_WIDTH) return path;

  const nextValidMoves = getNextMoves(x, y).filter(([nextX, nextY]) => isValidMove(nextX, nextY, board));

  for(const [nextX, nextY] of nextValidMoves) {
    path.push([nextX, nextY]);
    board[xyToOneDim(nextX, nextY)] = 1;

    const result = solve({x: nextX, y: nextY, path, board});

    if(result) return result;

    path.pop();
    board[xyToOneDim(nextX, nextY)]
  }


  return undefined;
}
```

While I knew that this would work, I also knew that the complexity seemed pretty bleak. Without putting pencil 
to paper to do a proper time-complexity analysis, just thinking about the problem space: a 10X10 grid, and the backtracking
solution, I winced as I ran the program. And I waited.

And I waited...

I got up to make some tea. And sometime between the kettle boiling and getting back to the desk, the result was there in 
my terminal. Ok, I thought. But let's see it in action. Theoretically this must be a valid answer, but I'm also the King Of Bugs:

# (todo: add a reach 100 gameboard here)

So, satisfied to see some non-deserved "Incredible" praise from the game, the program solution was quite unsatisfying.
I decided to go back and actually put pencil to paper.

### Time Complexity Analysis

So what is the time complexity of the above algorithm. This should be pretty straightforward