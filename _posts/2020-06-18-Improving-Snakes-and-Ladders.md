---
layout: post
title: Improving Snakes and Ladders
subtitle: A simple change to add agency to an otherwise autonomous game
#cover-img: /assets/img/path.jpg
tags: [gaming, homebrew, tabletop]
---

I just moved to the USA, and as a leaving gift, a cousin gave a New Zealand themed version of the classic game Snakes and Ladders (the NZ theme being that this was *Taniwha* and Ladders, of course).

Of course, we all know the rules of Snakes and Ladders (well - there's some debate around a few house rules), but in general, you roll a six sided die (also known as a d6), you move across the board, if you exactly land on the start of a ladder you go to its top, and if you exactly land on the mouth of a snake you go to the end of its tail. To win the game, you need to be the first person to reach the end tile. A typical house rule is that you must land on the end tile exactly - if you overshoot your roll (e.g. its 4 spaces away and you rolled a 5) you *bounce-back*, i.e. move 4 spaces forward and then 1 space back to make a total of 5.

# The problem

When we are young playing this game we don't notice the major issue with its rules - you, as a player, have *no bearing* on the outcome of the game. 

You have no agency. 

The only action you can make is to roll a d6, and then you move based on the output of the die. As the die is random, your moves are random, and that's really all there is to it.

# Our solution

We quickly brainstormed a number of solutions, actually, from the simple (e.g. allowing players to choose to move forwards or backwards based on their die roll), to the complex (allowing players to move the snakes and ladders on the actual board). 
However, we wanted to constrain the game's modification so that the setup remains simple. The beauty of this game is that there's just tokens, a die, and a fixed board. So, while some options could be neat (adding powerups in the form of cards to be played on later turns) they might make the game too complex.

Instead, we settled upon one simple addition - add more kinds of die. Specifically, we added a four sided die (a d4) and an eight sided die (d8). We also add a new concept to the game - the *active die*.
At the start of the game, before the first player's first turn, the game's active die is set to the default d6.

Now, on a given turn, a player can choose between one of two options. Either they roll the game's currently active die, or they can *change the active die* and *not roll*.
To prevent deadlocks and add further strategic options, we also prevent the player that rolls after a die change from taking this option (i.e. if the player before you changed the active die, you must roll with the new die).

## Why would you want to change the active die?

Well, the different die have different probabilities for each number. If you have the base of a ladder just 1 step away, then having the d4 active means you have a 25% chance of landing on it, as opposed to the d6 where it is a 16.7% change, or the d8 where it is a 12.5% chance. So, instead of rolling the less ideal die you might choose to spend your turn changing it.
However, there is of course a tradeoff. If you change the active die, you don't move, allowing the other players a further roll to catch up.

## The end-game and changing the active die

With the integrated *bounce-back* end-of-game rule, having the ideal active die when approaching the end of my Taniwha and Ladders board can grant a major advantage.
Consider you are in the following situation:






