---
title: "Roll for Optimisation"
date: 2024-09-09T19:40:40+02:00
tags:
  - Optimisation
draft: false
---

Okay, I know this is my second post, and this post will reveal a lot of things about myself.

So, my friend group and I go on vacation together at least once a year, to a nice little village somewhere in the mountains.
And since we all love board games and TTRPGs, some of us do us the honour of preparing a one evening session of their current favourite TTRPG, think [Dungeons & Dragons](https://dnd.wizards.com/de), but also other, less well-known games like [Lancer](https://massifpress.com/lancer) or [Call of Cthulhu](https://www.chaosium.com/call-of-cthulhu-rpg/).
As assigning people to sessions is a yearly challenge for us organisers, I decided last year to put my brain cells and maths degree to good use and come up with a fair and rational way to design a player schedule.
To make planning and session attendance easier, we send out a survey in advance.

Here's the issue we're facing. We have got three different options, two D&D sessions, called _D&D1_ and _D&D2_, which run over two evenings each, and a fun combination of one evening each of [Hexxen 1733](https://ulisses-spiele.de/game-system/hexxen-1733/) and [The Witch is Dead](https://gshowitt.itch.io/the-witch-is-dead), with two different game leaders, called _Witch_. In total there are ten players. Each option can have either three or four players with the exception of _Witch_, since the leader of the Hexxen 1733 game has to be a player in The Witch is Dead, and vice-versa.
We gave the attendees a quick overview of each game and asked them to rank their preferences.

To get a feasible solution to this problem, I used Google OR-Tools' [CP-SAT-Solver](https://developers.google.com/optimization/cp/cp_solver?hl=de). The first choice of a player gets a happiness-coefficient of $2$, the second choice a coefficient of $1$, and $0$ for the non-choice. So, for each player, we have a three-tuple of coefficients.

To summarize, the constraints now are:

- each player can only participate in one of the three games,
- D&D1 and D&D2 can have three or four players,
- Witch must have three players,
- and there is a special constraint that player 2 and player 3 must play the same game.

The goal is to maximise total happiness. This is modelled by introducing a 0-1-variable for each player and each game, where 1 indicated the player gets to participate in this game. For simplicity reasons I used an index for each player which is then mapped to the name.

The implementation then is pretty straight forward, where `player_count` equals ten:

```python
# introduce variables for each game and player
witch = [model.NewIntVar(0, 1, f'hex{i}') for i in range(player_count)]
dd1 = [model.NewIntVar(0, 1, f'dd1{i}') for i in range(player_count)]
dd2 = [model.NewIntVar(0, 1, f'dd2{i}') for i in range(player_count)]

# make sure each player only plays in one game
for i in range(10):
    model.Add(witch[i] + dd1[i] + dd2[i] <= 1)
```

The additional constraints as detailed above are:

```python
model.Add(sum(witch) == 3)
model.Add(sum(dd1) <= 4)
model.Add(sum(dd2) <= 4)
model.Add(sum(dd1) >= 3)
model.Add(sum(dd2) >= 3)

# additional coupling of player 2 and 3
model.Add(witch[2] == witch[3])
model.Add(dd1[2] == dd1[3])
model.Add(dd2[2] == dd2[3])
```

and finally, the objective function, where the `happiness_coefficient` is a list of lists, with the happiness coefficients of each player:

```python
model.Maximize(sum([a * coeff[0] + b * coeff[1] + c * coeff[2] for a,b,c,coeff in zip(witch, dd1, dd2, happiness_coefficient)]))
```

Lastly, there is some code to print the result:

```python
if status == cp_model.OPTIMAL or status == cp_model.FEASIBLE:
    print(f'Maximum of objective function: {solver.ObjectiveValue()}\n')
    for i in range(10):
        print(f'{i}: witch {solver.Value(witch[i])}, dd1: {solver.Value(dd1[i])}, dd2: {solver.Value(dd2[i])}')
```

Great, right? We just use the coefficients from the survey and get a perfect, feasible optimisation plan. I thought I had it all figured out. The numbers lined up, the variables defined, the objective maximised. I was proud, even a bit smug.

Yeah, no.

I presented the schedule to my fellow organisers: A mathematically perfect solution, but emotionally... not so much.

As it turns out, personal preferences, and the intricate dynamic of social groups are hard to capture neatly in a few lines of code and mathematical formulas.
Player 3 did not want to play with player 7; others would feel overlooked, since they always make space for new comers in other sessions, and so on.
We sat there, with this optimal schedule in hand, but still not everybody accepted it.
Ultimately, we re-worked, discussed, and re-re-worked, with the the preferred schedule then consisted of a split character between the leader of Hexxen and someone else, so that player got to play The Witch is Dead - an option not even considered in the algorithm.

Moral of the story: no matter how much logic, mathematics, and careful planning we apply to social situations, people are not algorithms.
In the end, even though we could not maximise the happiness coefficient, we still had fun and memorable evenings - because ultimately, that’s what matters most.

Needless to say, this year we did not employ happiness-optimisation anymore.
