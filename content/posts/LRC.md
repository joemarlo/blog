+++
draft = false
image = "img/posts/LRC/LRC.gif"
date = "2019-10-13T14:20:22+05:30"
showonlyimage = false
title = "Is it possible to beat Left Right Center?"
weight = 9
+++

*Choosing the right seat at the table*
<!--more-->
***

Around the holidays my family enjoys a game of [Left Right Center](https://en.wikipedia.org/wiki/LCR_(dice_game)), a ~~completely~~ largely chance-based dice game where everyone puts a few dollars on the line. It's a nice reprieve from the typical skill-based card games much of family tends to play, and the only decision involved is: what position around the table do you sit? 
  
<p align="center">
<img src="/img/posts/LRC/LRC.gif" width=80%>
</p>

#### A quick primer on the game
###### (...at least how my family plays it)
The goal of the game is to be the last person with at least one dollar. Participants sit in a circle, and each turn consists of a single participant rolling dice. The die is six-sided but contains four markings: one side has an "L," one side a "R," one side a "center", and three sides have single dots. If the die lands on the "L" then the roller hands a dollar to the participant to their left and similarly right for a "R." If it lands on "center" then the roller places a dollar in the center of the table, and if it lands on a dot then the roller keeps their dollar. Each participant starts with three dollars, and will roll an equalvalent amount of dice — up to a maximum of three dice per turn. For example, if I have two dollars then I toss two dice but if I have four dollars then I only toss three dice. If I have zero dollars than I don't roll at all, however I can receive dollars from rollers to my left or right. The game ends once only one player has at least one dollar left. The pile of dollars accumulated in the center of the table are their winnings.

#### Simulating it
If the game is completely up to chance then what is the point of simulating it? Is there actually an edge to be had? It comes down to this: the expected average outcome of any given turn is negative — i.e. an average turn will result in less dollars for the rolling player as only half the die faces (the dots) result in keeping dollars, the other half result in giving away dollars, and none result in the rolling player receiving dollars. Does this mean that its best to go last?

In order to simulate the game we need to first break it down into its core parts:

- a function to calculate a single roll of the dice: `rollDice()`
- a function to calculate the impact of `rollDice()` on the roller's score and the participants adjacent to them. We'll call this function `takeTurn()`
- a function that pulls it all together and simulates a full game. It should return a matrix of the full game with turn-by-turn scores. This is `playLRC()`

<font size="2">*Note you can find the code here: [github.com/joemarlo/left-right-center](https://github.com/joemarlo/left-right-center)*</font>

If we run `playLRC()` specifying three players we get a nice, tidy matrix showing the dollars left per player per turn. Here the participant in position 2 won after going head-to-head with player 3 for final three rolls.

```
> playLRC(n.player = 3)
   1 2 3
1  3 3 3
2  0 4 3
3  0 3 4
4  2 3 2
5  0 4 3
6  1 3 3
7  3 3 1
8  2 4 1
9  2 4 1
10 2 4 1
11 1 5 1
12 2 3 2
13 2 3 2
14 1 4 2
15 1 3 3
16 1 4 2
17 0 4 2
18 0 3 3
19 0 5 0
```

We can now simulate many games using `playLRC()` with any number of participants we choose. Running this 50,000 times (10 different game sizes * 5,000 runs each) gives us simulated data of who typically wins the game for a given number of participants.

<p align="center">
<img src="/img/posts/LRC/LRC_winners.svg" width=100%>
</p>

There's clearly a gain for going last -- at least when the number of players is low. If there's only two participants then the second participant wins ~1,200 more times over 5,000 games or about a **66% increase** over the first participant. Once the group gets bigger than ~five, position doesn't appear to make a substantial difference. Let's quantify this advantage for all game sizes.

<p align="center">
<img src="/img/posts/LRC/Player_advantage.svg" width=100%>
</p>

What does this tell us? I'm not sure. The advantage of sitting further from the first roller is clear until you reach a game size of six. Then it could be a disadvantage, but as you continue to increase game size the results are up and down. Perhaps the advantage is completely erased once the game size reaches six and the up/down is randomness.

[comment]: # (run a t-test here on the randomness?)

You might not be able to get an edge during the larger games but at least you'll be in for something a little more exciting. The larger games tend to produce many more winners who come back after having zero dollars.

<p align="center">
<img src="/img/posts/LRC/Comeback_winners.svg" width=100%>
</p>

<br>

---
*2019 October*  
*Find the code here: [github.com/joemarlo/left-right-center](https://github.com/joemarlo/left-right-center)*
