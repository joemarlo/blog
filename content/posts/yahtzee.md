+++
draft = false
title = "Optimizing Yahtzee"
weight = 9
image = "img/posts/yahtzee/Expected_roll_outcomes_thumbnail.png"
date = 2019-10-13T16:26:44-04:00
showonlyimage = false
+++

*Simulating Yahtzee and determining your next move*
<!--more-->
***

One of the core challenges when playing [Yahtzee](https://en.wikipedia.org/wiki/Yahtzee) is determining which die to keep before throwing your second and third rolls. Plenty has been [written](http://mathworld.wolfram.com/Yahtzee.html) on the [probabilities](https://www.thoughtco.com/probability-of-rolling-a-yahtzee-3126593) of Yahtzee rolls and [simulating](http://galsterhome.com/stats/Tutorial/SAS19.htm) [Yahtzee](https://www.reddit.com/r/dataisbeautiful/comments/8vgxwl/simulating_10000_yahtzee_dice_throws_how_many/) outcomes. The goal of this post is to go one step further and optimize future rolls by determining likely outcomes after your first roll.

The goal of Yahtzee is to get the highest score after 13 rounds of play. Each round consists of one turn by each player, and each turn is conducted by rolling five standard dice and recording the outcome's points based on the [Yahtzee rulebook](https://www.hasbro.com/common/documents/dad2af551c4311ddbd0b0800200c9a66/8302F43150569047F57EB8D746BA9D86.pdf). The player can roll up to three times per turn, withholding any of the dice they wish from their previous throw. This is where the probabilities become important: which dice, if any, should the player withhold from their next roll in order to maximize their points?

Simulating only a given turn is much simpler than playing an entire 13-round game so let's start with that. There's two main parts. First, we need to be able to calculate the score of any given roll. This is relatively simple. However, the second part is much more complicated: we need to figure out the expected probabilities of all the combinations of dice that a player could withhold given that they have already rolled once. E.g. if the player's first roll is a `2, 2, 4, 5, 2`, should they hang on to the three `2`'s or maybe go for a straight by hanging onto `2, 4, 5` and hope to roll a `3`. Or is there another, better combination? 

#### Combinations and permutations
The goal of the second part is determining the right combinations and permutations to draw from the current roll. We want to eventually arrive at the sample space conditioned on which die we withhold. In other words, we want a big list where each observation is a possible outcome of the next die roll. We can then calculate the resulting scores using a custom function `calculate.score()`, and the resulting list of scores will allow us to calculate the probability of any given score conditioned on which die we withheld.

Let's break this down further into two parts. We first need a list of all the `base.rolls` or the combination of dice we are going to keep. E.g. in our example of `2, 2, 4, 5, 2`, if we keep two dice then we will have 5 choose 2 = 10 outcomes. But this still over counts as we don't care which `2` we pick. This results in a list of four possible `base.rolls`: (2, 2), (2, 4), (2, 5), (4, 5). This can automated in R using `combn()` (from the [combinat package](https://www.rdocumentation.org/packages/combinat/versions/0.0-8)) then sorting the results and finally removing the duplicates using `.[!duplicated(.)]`.

The second part is determining the resulting three dice rolls that complete each possible `base.rolls` pair. In our example, these three can take 216 different permutations (there's three slots to fill where each slot can take six values so: 6 * 6 * 6 = 216). In R we can build a vector `1:6` three times using `replicate(3, 1:6)` then call `expand.grid()` on itself to produce the actual 216 permutations.

These 216 permutations now need to combined with our base rolls. This results in 864 possible outcomes (4 * 216), and can be produced using `expand.grid()`. We now have the four conditional sample spaces — one for each `base.rolls` — and can pass them to `calculate.score()`.

Rinse-and-repeat this process for withholding one, three, four, or five different dice. Group the results based on the `base.rolls` and then we can see the likely outcomes for each combination of original dice to withhold. We can also plot the densities to get a more intuitive view of the possible scores:

```
> calculate.die.to.keep(seed.roll = c(2, 2, 4, 5, 2), verbose = TRUE)
# A tibble: 16 x 4
   Base_roll     Mean Median    SD
   <chr>        <dbl>  <dbl> <dbl>
 1 2-4-5         22.9   20    8.01
 2 4-5           22.8   21    6.48
 3 4             21.3   20    6.59
 4 5             21.1   20    6.11
 5 2-4           20.9   18    7.46
 6 Keep no dice  20.1   19    6.55
 7 2-5           20     18    6.70
 8 2             18.9   17    6.94
 9 2-2-4-5       18.8   17.5  5.78
10 2-2-4         17.6   16    5.70
11 2-2-5         17.4   17    4.66
12 2-2           16.6   15    5.66
13 2-2-2-5       16     14.5  4.73
14 2-2-2         15.7   13.5  7.50
15 2-2-2-4       15.3   14    5.09
16 2-2-2-4-5     15     15    0   
[1] 2 4 5
```

<p align="center">
<img src="/img/posts/yahtzee/Expected_roll_outcomes.svg" width=100%>
</p>


That looks great, and it seems like we should hold onto `2, 4, 5` (bottom right plot). But how can we be sure this is really working? Let's run this 500 more times and automatically choose our next roll based on the highest mean expected outcome. Then we can compare the results to just randomly rolling die.

<p align="center">
<img src="/img/posts/yahtzee/Smart_vs_Dumb_boxplot.svg" width=100%>
</p>

Fantastic, it works. Now that we have the two ends of the information spectrum — random rolls acting on no additional information and "smart" rolls acting on perfect probability information — this framework would be a great way to test heuristics that players could actually use during a game. E.g how well does the strategy of always withholding a three-of-a-kind perform? Or withholding a small straight? Sounds like a good v2.0 of this project.

Besides testing heuristics, the next step is expanding this to all 13 turns of a full game of Yahtzee. That will be a little more involved as the box score will need to be kept track of over the course of the game, and scoring options will need to be consequently excluded from the `calculate.score()` function.
<br>

---
*2019 October*  
*Find the code here: [github.com/joemarlo/yahtzee](https://github.com/joemarlo/yahtzee)*
