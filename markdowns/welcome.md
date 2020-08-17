** it's a draft for now **

# Prerequisites

I am assuming you are familiar with minimax (negamax) algorithm, preferably with alpha-beta pruning and optionally with iterative deepening, transposition tables etc. Also, you should know how Monte Carlo Tree Seach works, because the algorithm I will present will mostly resemble MCTS with [early playout termination](https://link.springer.com/chapter/10.1007/978-3-319-27992-3_2).


# Introduction

Recently I've been playing with neural networks to improve the evaluation function of the minimax. Other top bots have shown that Oware is quite NN-friendly. I've been succesful training good evaluation function (at least much better than my handcrafted one). Having a good evaluation function, one can use it in:
- minimax, to replace the handcrafted eval
- mcts, to guide the selection phase, or to guide the simulation phase
- mcts, for early playout termination and return the score

For Oware, mcts with early playout termination was used by at least 2 good bots that I'm aware of. And the termination was at 0 depth, so they effectively used eval as playout's score. I too had some success with it. But by experimentation and partly by accident, I created algorithm that was better. Then of course I discovered that it was nothing new. In literature it was called [Best-First Minimax Search](https://www.chessprogramming.org/Best-First_Minimax_Search), though my approach is a bit different.


# Monte Carlo Tree Search

Let's recall 4 steps of MCTS:
- **selection**: select child according to UCT: wins/visits + C * sqrt(log(parent visits) / visits), until you reach node leaf
- **expansion**: expand the node leaf (either by 1 child or every possible children) and choose one of its child
- **simulation**: from that child simulate the game randomly until the end of game; get the result [win, draw, lose] (1, 0, -1) or some implementations (1, 0.5, 0)
- **backpropagation**: propagate the result accordingly, i.e. if win was for black player, then +1 for black player's nodes and -1 for white player's nodes, up until root

And **final selection**: select move with highest visits; sometimes best score or lower confidence bound is used

In early playout termination, we play (semi)randomly for N moves, then use eval to propagate score. Some implementations use eval directly, other impementations may use eval > 0 = win, eval < 0 = lose.

When I used 0 depth playouts, in other words used the eval, I found that using eval directly is better than scaling it to win/lose. I've been experimenting with it and I discovered that overwriting scores instead of adding got me better results. I came up with ["new"](https://www.microsoft.com/en-us/research/publication/best-first-minimax-search/) algorithm, but somewhat incorporated UCT in it.


# Best-First Minimax Search with UCT 

It's practically the same as MCTS, with few differences:
- **selection**: select child according to eval value + C * sqrt(log(parent visits) / visits), or if child is not visited: eval value + FPU, until you reach node leaf
- **expansion**: expand the node leaf by every possible children (every possible moves), eval every children
- **"simulation"**: no simulation, we added eval to every children
- **backpropagation**: *overwrite* each parent's eval in negamax style: parent's eval value = - max of children's value

**Final selection**: select move according to formula: move's eval value + log(move's visits)

*C* is exploration constant to be tuned, *FPU* is First Play Urgency also to be tuned. I.e. for oware i have C = 1.5, FPU = 0.5.

The *final selection* will selection move mostly with highest visits, unless the eval score of less visited move is much higher.

I should remark I use NN for eval, which is in the range [-1,1] and the win is 10000 - 10 * game's rounds, and lose is -10000 + 10 * game's rounds.

# Results

With this approach, I have great success in Oware, Breakthrough, Onitama and Othello (I use N-tuple instead of NN in the last two). Generally the winrate against my minimax bot is at least 70% using the same eval. Though I should remark, the minimax bot is rather generic alpha-beta with move ordering using transposition tables and killer moves. I didn't use any fancy pruning, quiescence search or search extensions. Nevertheless, with low effort I can beat the minimax approach.




