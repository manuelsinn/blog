---
title: "AI kills evil monsters from the deep"
date: 2020-06-15T20:02:42+02:00
draft: false
---

What this is:
> Interactive object-oriented Java implementation of the widely known "Wumpus" game, playable through a basic CLI. Also features a knowledge-base AI to play the game, relying on propositional logic.

To see the code for this project, head over to [GitHub](https://github.com/manuelsinn/wumpus-world/).

## The basic idea...
...of the game is this:   
Armed only with a bow and (a single!) arrow, you are set to explore a cave. Do not fall into a pit! Your mission, should you choose to accept it, is to kill the wumpus before it has a chance to eat you alive.

Whenever you take a step (or move to a different dungeon in the original), a few options arise: 
1. You have met the wumpus, the two of you got acquainted and a moment later you are dead.
2. You have stepped into a pit. Legend has it that you are still falling. No one knows what's down there.
3. You have reached a position in which you are not in immediate danger. This leaves three options:
    1. You notice nothing but darkness. Instead of damning the boredom, you celebrate the richness of still being alive and continue your journey ecstatically.
    2. Your nose picks up an abominable stench, and you try not to panic, although this means that the wumpus is either directly in front of you, behind you or by your side. You are basically doomed and should probably head back to where you came from.
    3. Your hair gets all frizzy because a breeze just blew through: a pit is nearby. You remember your fear of heights and proceed with extra caution (and an increasing number of sweat drops on your forehead).

## Different agents at play
Two agents can play the game: you, via the command line interface, and the AI.  
In the CLI version you can always choose between moving and shooting. Until you have shot away your arrow that is, in which case you have either won the game (by killing the wumpus) or are doomed, because you don't have a weapon anymore (= game over).  
The AI explores the world and in doing so continually builds up a knowledge base. The knowledge base is fed by the sensory input: after every step, information is received (e.g. stench, breeze, etc). In the next step, propositional logic is used to determine the right action to take. Put into simple terms, at some point the AI has walked around the cave for long enough to deduce the position of hazards, so it can dodge them smoothly and heroically shoot the wumpus to save the princess. Or something like that.

## Context of creation
To get a deeper understanding of the theoretical background I was taught in the module *Interactive Artificial Intelligence*, I designed and implemented this project from scratch.   
Besides that, I also learned about intelligent agents, decision making, various ways and algorithms of searching, constraint satisfaction problems, logic, and various ways of learning (especially supervised and reinforcement). Speaking of learning, check out [this post](https://manuelsinn.netlify.app/posts/evolution-pw/), showcasing a project I created to crack passwords using evolutionary algorithms.

## What I learned
Besides gaining more hands-on coding experience, I found it valuable to see how an AI does not need to be based on deep learning algorithms. Although this one uses only basic propositional logic, it manages to beat the game with ease.   
While hot topics like machine learning and deep neural networks can accomplish amazing feats, it is nice to see how diverse the field of AI really is. And, how a more minimalist, and rather easy to understand, less buzzwordy option can sometimes turn out to be just as viable, without needing dedicated graphic cards or cloud computing to do so.