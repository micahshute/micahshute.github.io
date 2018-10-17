---
layout: post
title:      "Understanding the Monty Hall Problem"
date:       2018-10-17 05:37:47 +0000
permalink:  understanding_the_monty_hall_problem
---


This isn't exactly about development or coding, but it is an exercise in problem solving, logic and probability, all of which are fundamentals of our trade. As engineers and programmers, logic and mathematics, particularly probability and induction, are areas in which we should pursue mastery.

I was recently listening to a podcast that was recommended by a blogger on this site, and during one episode discussing big-O theory, the Monty Hall problem was mentioned. The commentators talked a little about it, at which point they admitted to not 'getting it' and moved on. This motivated me to share my understanding of the problem because I think that as engineers we should be the ones who are able to grasp this concept and problems like it.    


So here's how I 'got it'. It took some thinking, but in the end, it's actually not too complicated.

In case you are unfamiliar with the problem, the Monty Hall problem is as follows:  

- You are presented with 3 doors. Behind only one is a prize.  


![3 doors](https://thepracticaldev.s3.amazonaws.com/i/txe872kzoah3ynxycj3k.png)  

- You pick a door which you think the prize may be behind.  
  
![initial choice](https://thepracticaldev.s3.amazonaws.com/i/z84dm0puooe6pvyv9oyr.png)  

- After you pick the door, a 3rd party (ie the gameshow host) opens up a door which does NOT have the prize. 
- You are then given the option to either stay with your initial guess or switch your guess to the door which the game show host did NOT open. 

![final choice](https://thepracticaldev.s3.amazonaws.com/i/ruv6l2o6tzz9e5hrijta.png)  
  
- Which should you do? Switch or stay? Does it matter?

The initial intuition for most people is that once you are shown the door with no prize, the possibility is now 50-50 between the two remaining doors, so it doesn't matter if you switch or not. But this is not the case. It is actually probabilistically in your best interest to switch to the other door. But let's be good engineers and follow the scientific process with an initial hypothesis that you have a 50% chance of winning either way.

We have our hypothesis, so let's experiment.

Code:
```ruby
rand_gen = Random.new
prize_locations = []
switch_wins = 0
stay_wins = 0

#generate winning spots
10000.times do
    # rand_gen.rand(3) returns  0, 1, or 2
    prize_locations << (rand_gen.rand(3) + 1)
end


prize_locations.each do |prize_loc|
    # contestant chooses a random location
    initial_choice = rand_gen.rand(3) + 1
    # monty opens a door without a prize -> init. choice is prize ? (random opening of one of the remaining 2 doors) : (since 1, 2, and 3 must all be an option, 6 minus the other two will always give us the leftover one)
    opened_door = initial_choice ==  prize_loc ? (prize_loc + rand_gen.rand(2)) % 3 + 1 : (6 - initial_choice - prize_loc)
    # Universe split 1: I decide to stay with my choice.
    stay_wins += 1 if prize_loc == initial_choice
    # Universe split 2: I switch 
    switch_wins += 1 if prize_loc == (6 - initial_choice - opened_door)
end

puts "Total wins if you stay: #{stay_wins}, which is #{stay_wins / 10000.0}%"
puts "Total wins if you switch: #{switch_wins}, which is #{switch_wins / 10000.0}%"

```

Output:
```
[07:28:18] monty_hall  
//⚓   [⚛ micah] ruby monty_hall.rb  
Total wins if you stay: 3344, which is 0.3344%  
Total wins if you switch: 6656, which is 0.6656%  

[07:28:20] monty_hall  
//⚓   [⚛ micah] ruby monty_hall.rb  
Total wins if you stay: 3316, which is 0.3316%  
Total wins if you switch: 6684, which is 0.6684%  

[07:28:21] monty_hall  
//⚓   [⚛ micah] ruby monty_hall.rb  
Total wins if you stay: 3348, which is 0.3348%  
Total wins if you switch: 6652, which is 0.6652%  
```
Another way you could do it which can give you a litte more info is randomly switch between staying and switching your doors:

Code:
```ruby
rand_gen = Random.new
prize_locations = []
switch_wins = 0
stay_wins = 0

#generate winning spots
10000.times do
    #rand_gen.rand(3) returns  0, 1, or 2
    prize_locations << (rand_gen.rand(3) + 1)
end


prize_locations.each do |prize_loc|
    #contestant chooses a random location
    initial_choice = rand_gen.rand(3) + 1
    #monty opens a door without a prize -> init. choice is prize ? (random opening of one of the remaining 2 doors) : (since 1,2, and 3 must all be an option, 6 minus the other two will always give us the leftover one)
    opened_door = initial_choice ==  prize_loc ? (prize_loc + rand_gen.rand(2)) % 3 + 1 : (6 - initial_choice - prize_loc)
    #This time, let's just keep one universe and randomly choose
    switch_or_stay = rand_gen.rand(2) # 0 means stay, 1 means switch
    stay_wins += 1 if prize_loc == initial_choice && switch_or_stay == 0
    switch_wins += 1 if prize_loc == (6 - initial_choice - opened_door) && switch_or_stay == 1
end

puts "Total wins from when you stay: #{stay_wins}, which is #{stay_wins / 10000.0}%"
puts "Total wins from when you switch: #{switch_wins}, which is #{switch_wins / 10000.0}%"
puts "Total wins: #{wins = switch_wins + stay_wins}, which is #{wins / 10000.0}%"
```
Output:
```
[07:43:07] monty_hall
//⚓   [⚛ micah] ruby monty_hall.rb
Total wins from when you stay: 1685, which is 0.1685%
Total wins from when you switch: 3303, which is 0.3303%
Total wins: 4988, which is 0.4988%

[07:43:08] monty_hall
//⚓   [⚛ micah] ruby monty_hall.rb
Total wins from when you stay: 1689, which is 0.1689%
Total wins from when you switch: 3307, which is 0.3307%
Total wins: 4996, which is 0.4996%

[07:43:09] monty_hall
//⚓   [⚛ micah] ruby monty_hall.rb
Total wins from when you stay: 1662, which is 0.1662%
Total wins from when you switch: 3398, which is 0.3398%
Total wins: 5060, which is 0.506%
```
Ok, our hypothesis is wrong. Looks like we should probably switch doors. Time to refine my hypothesis and this time rely on more than just naive intuition. (Maybe you saw some new intuition above: that the only way you get a stay_win is if you guessed right initially)

There are a few ways to look at the problem. First, let's look at a split universe timeline:

- You have 3 doors and a prize behind one of them. So, for your initial pick, what is the probability that you correctly choose where the prize is? 
    - That's right - 1/3
- Let's freeze time right here and do a little more analysis. There are 2 possibilities that just happened in your choice. So let's split our universe.
    - **Universe 1**: You fell within the 33.3% chance and chose correctly. Now, the game show host can open either of the remain doors, as neither of them has a prize. Now you have ANOTHER CHOICE (another split!)
        - **Universe 1.1** You stay with your initial pick. (Human factor, let's say 50%)
            - You win. (100%)
        - **Universe 1.2** You switch. (50%) 
            - You lose. (100%)
    - **Universe 2** You initially choose wrong, which happens 66.7% of the time. Now, the game show host can ONLY open the door without the prize behind it (leaving the other door with the prize) **SPLIT Universe 2**
        - **Universe 2.1** You stay with your initial pick. (50%)
            - You lose (100%)
        - **Universe 2.2** You switch. (50%)
            - You win. (100%)



**Analyzing mathematically**: So what's our problem statement? We want to know whether we win more if we switch or stay, right? So let's calculate the probability of winning given that you stay with your initial door, and then the probability of winning given that you switch doors. Note that we are giving the person a random 50% chance that they choose to switch or stay with their original door.

    P(win | stay) = [P(win) * P(stay | win)] / P(stay)  

    P(win) = (Universe 1 && Universe 1.1) || (Universe 2 && Universe 2.2) =  (1/3) * (1/2) + (2/3) * (1/2) = 1/2    

    P(stay | win) = Out of wins possibilities only, what percentage of those did you stay for?  
    =  (Universe 1 && Universe 1.1)  / [(Universe 1 && Universre 1.1) || (Universe 2 && Universe 2.2)] = (1/3) * (1/2) / (1/2) = 1/3

    P(stay) = (Universe 1 && Universe 1.1) || (Universe 2 && Universe 2.1) = (1/3) * (1/2) + (2/3) * (1/2) = 1/2

    P(win | stay) = [1/2 * 1/3] / 1/2 = 1/3

You could also say "Out of the times that I stay, how many times did I win?" That would give you the same answer: 

    P(win | stay): all wins which lie within stay universes
    P(win | stay) = (Universe 1 && Universe 1.1) / [(Universe 1 && Universe 1.1) || (Universe 2 && Universe 2.2)] = (1/3) * (1/2) / [(1/3) * (1/2) + (2/3) * (1/2)] = (1/3) * (1/2) / (1/2) = 1/3
    P(win | stay) = 1/3
Now, let's see the probability of winning given you switch. (While Baye's doesn't really buy us anything since we have a short timeline which describes everything for us, I'll do it both ways for the sake of thoroughness).  

    P(win | switch) =  [P(win) * P(switch | win)] / P(switch)  

    P(win) = (Universe 1 && Universe 1.1) || (Universe 2 && Universe 2.2) =  (1/3) * (1/2) + (2/3) * (1/2) = 1/2    

    P(switch | win) = Out of wins possibilities only, what percentage of those did you switch for?  
    =  (Universe 2 && Universe 2.2)  / [(Universe 1 && Universre 1.1) || (Universe 2 && Universe 2.2)] = (2/3) * (1/2) / (1/2) = 2/3

    P(switch) = (Universe 1 && Universe 1.2) || (Universe 2 && Universe 2.2) = (1/3) * (1/2) + (2/3) * (1/2) = 1/2

    P(win | switch) = [1/2 * 2/3] / 1/2 = 2/3

Or, you could say:

```
P(win | switch) : all wins which lie within switch universes
P(win | switch) = (Universe 2 && Universe 2.2) / [(Universe 1 && Universe 1.1) || (Universe 2 && Universe 2.2)]
P(win | switch) = [(2/3) * (1/2)] / [(1/3) * (1/2) + (2/3) * (1/2)] 
P(win | switch) = (2/3) * (1/2) / (1/2) = 2/3

```
From our mathematical analysis, we can see that you are twice as likely to win if you switch than if you stay. You will also win more (2/3 of the time) if you switch every time than if you randomly choose to switch or stay (in which case you win 1/2 of the time)

**Analyzing with logic**: If you choose to **always switch**, you will **always** be right if you **initially** chose the wrong door. So, since you initially choose wrong ( 1- 1/3 = ) 66.67% of the time, you will be guaranteed to choose the **correct** door 66.67% of the time if you **always** switch. If you choose **never** to switch, you are guaranteed to be correct **only if** you initially picked the correct door out of the 3 on your first try, which was a 33.3% chance.   


**Expanding this problem out to get a better intuition**: Sometimes when a problem seems hard to grasp, I like to take the numbers to extremes while maintaining the necessary relationships. In this case, we know that after your choice of 1 door, the host will open **all of the doors** but **one other** (even though in our case with 3 doors, this is just him opening one door) AND these doors must have no prize. So, using this relationship, let's expand our situation to 1 million doors with one prize. Now after you choose a door, the game show host has to open **all the rest but one**. And he can't open the one with the prize. So, unless you think you chose right on your first try with a million to one odds, don't you think this is a pretty good deal, as the host is practically forced to show you where the prize is?! This is the exact same concept, except now our odds for getting the prize when we switch are 999,999/1,000,000 instead of 2/3. It's much easier to have intuition on this concept when it is expanded in this way.
  

Now, our hypothesis matches our experimental data, and we can call it a day.
