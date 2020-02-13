---
title: "Using evolution to crack passwords"
date: 2020-02-12T12:26:48+01:00
draft: false
---

# Motivation
After getting to know the basic functioning of evolutionary algorithms through self-study and one of my university courses (Interactive Artificial Intelligence), I wanted to get some hands-on experience and try to implement some of the theoretical principles I had learned, as well as compare some of the different variants of tuning the algorithm.
Besides learning about AI, I also wanted to improve on my coding skills, with special focus on writing clean code. Having just learned about important principles of software quality at university, trying to make use of these in practice was also one of my objectives.






# How it works

## Basic structure (evolution)
At first, a (random) population of individuals is spawned. Next, natural selection works its magic to decimate the population. The remaining individuals procreate until their number reaches the original amount, before some individuals mutate. This new population is then the basis for the next evolutionary cycle.


## What population?
While this basic underlying principle is applicable to many aspects (possible populations) in both biology and computer science, for this implementation I focused on a rather simple one: cracking (alphabetical) passwords.
What this means in practice is that the population consists of Strings, all of which are made up of random letters (their genes, so to say) and are just as long as the pre-defined solution (the correct password).


## Methods of Selection, Crossover and Mutation
In implementing the evolutionary algorithm, one has to choose which methods in particular should be used for natural selection, procreation and mutation.

For selecting which individuals will die and which will live on, I chose Truncation, meaning a set proportion of the _fittest_ individuals will survive (in this case: the "best" 50% survive). 

To determine how many individuals will change their "genes" (and in what way), I chose Gaussian Mutation. This means that each individual will have its genes altered with a certain probability (I chose a mutation rate of 5%). In practice, this means that each letter of every password-candidate (individual) has a 5% chance of being switched with another random letter.

To be able to make a comparison, I chose two methods of procreation: Uniform Crossover and K-Point Crossover.
The first variant means that, going through all genes sequentially, each "baby" is born by choosing either the mum's or the dad's gene. So every letter will be either the one the mum or the dad had in that position.
K-Point Crossover means that the parents will be cut up into _k_ points, and the baby will always get either the dad's or the mum's portion in alternating fashion.


## Which passwords are fitter than others?
One remaining question is how to distinguish which individuals are fitter than others. My fitness function used for the natural selection process describes the proportion of already correct letters when compared to the solution-password. I understand how this is not exactly the most realistic scenario (passwords are really either correct or incorrect, there is nothing in between), but it should suffice for proof of concept.






# Code
You can find the code [here](https://github.com/manuelsinn/genetic-pw-algorithm "Code on Github").







# Empirical Comparison of Crossover Strategies
Using a simple self-coded Export class, I gathered some csv data and imported it to Excel to compare the performance of the different crossover functions. The measured variable is the population's generation at which the fitness is 1.0, that is when the generation's fittest individual is equal to the password.

Whether the solution-password was 28 or only 10 characters, all crossover implementations fared significantly better than brute force. Here is an overview of the data gathered:

|                       | password-length = 28 | password-length = 10 |
|-----------------------|----------------------|----------------------|
| Brute Force           | 304                  | 115                  |
| Uniform Crossover     | 31.37                | 9.03                 |
| k-Point with _k_ = 2  | 193.73               | 17.30                |
| k-Point with _k_ = 4  | 202.10               | 17.57                |







# Lessons learned

## AI related
Gaining an insight into an aspect of AI that is not related to Machine Learning has helped me to appreciate the field's vastness and complexity. It is still astonishing to me how this combination of randomness and survival of the fittest manages to continuously deliver such good results, no matter the discipline.


## Code Improvements
Looking at this project again after a while through the perspective of software quality, there have been quite a few aspects of improvement.
I was able to organize the code's structure to be much cleaner and more efficient by using certain code patterns, as well as structuring the classes more effectively in packages.
To disconnect the underlying algorithm from concrete implementation details, I used abstraction and dependency inversion; Specifically the Strategy pattern (for mutation and selection, minimizing coupling through the use of interfaces), as well as the Template Method pattern (for crossover, since the common skeleton justified a stronger coupling).

Although I did not make use of test driven development in the first place, I did now that I was going to refactor a lot of the code. It showed me the value of getting an immediate confirmation when my code was broken, and in general sped up the process a lot. Besides refactoring the production code with the _SOLID_ principles in mind, I also made sure to use the _arrange-act-assert_ and _given-when-then_ conventions in my test code. I also evened out the functions' abstraction levels and improved the overall (Java) documentation.






# Conclusion
As the project was intended in a proof-of-concept kind of fashion, I am happy about the way it turned out, having accomplished my objectives of practicing clean code and learning about AI in a very practical manner. 
This is despite there of course being room for improvement, whether it's improving on the unrealistic fitness function or gathering more meaningful data for a more thorough statistical analysis (e.g. using a independent two-way analysis of variance or playing around with different passwords).
