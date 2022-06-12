---
title: "Learning Consensus"
excerpt: "Training Deep RL agents to attack consensus protocols"
collection: projects
date: 2021-03-01
---
### Beginnings 
From late 2019 to 2021, I worked with Professor Kartik Nayak and two classmates at the intersection of Deep RL and Consensus Protocols. We started out with a simple question: Can we use Deep Reinforcement Learning to train agents to learn a basic consensus algorithm? 

### Phase 1: Teaching Agents to Learn Consensus
We were initially inspired by OpenAI Five, which used multi-agent RL algorithms to train agents to play (and win at) Dota 2. Agents in this game had to understand the impacts of certain actions on their team, communicate with one another, and come to agreement on optimal strategies. Similarly, agents in a consensus environment must be able to all come to agreement (consistency) depending on what they are initialized to (validity) within some bounded time (termination) in the present of crash or byzantine faults. We started out by creating a simple environment with 3 honest agents that were initialized to either 0 or 1 and had to figure out a value to commit to at the end of 2 rounds. We gave the agents penalties based on violating consistency, violating validity, violating time constraints, violating safety, and rewards based on committing correctly. Within 400 epochs of training, agents in this setting were able to come to agreement with a basic 2 round protocol (send, commit). 


### Phase 2: Teaching Agents to Attack Consensus Protocols
We later increased the complexity of this environment by freezing the action space of the honest agents replacing one with a byzantine agent that could observe the state of the entire environment. Here, the byzantine agent eventually learned to equivocate, or send conflicting values to the honest agents, by optimizing for rewards that were the inverse of the honest agents'. Interestingly enough, when training both honest and byzantine agents together, the honest agents maximized their rewards by always committing to the same value, ignoring the byzantine messages and giving them a 50% win rate. At the time, this randomization phenomenon reminded me of the coin-flip in Ben-Or's protocol that restarted the initial input if there was not agreement... perhaps we had to add another round to this protocol. 

To build upon our system, we decided to try see if agents could learn Avalanche's Slush algorithm to come to consensus despite the presence of a byzantine agent. An environment with 4 honest agents and 1 byzantine agent was able to reliably reach consensus; however, 3 honest agents were unable to reliably reach consensus (50% win-rate) in the presence of 2 byzantine agents. After seeing how agents got stuck in a local minima when trying to learn slush, we decided to update our infrastructure to use multi-agent PPO, instead of simple DNNs. In addition, we pivoted the project to focus on training byzantine agents to focus on finding vulnerabilities in consensus protocols. 

Eventually, we were able to build a multi-agent PPO environment that integrated an abstracted version of PKI and a synchronous byzantine agreement consensus [protocol](https://eprint.iacr.org/2018/1028.pdf). Byzantine gents were able to successfully attack the protocol if we removed a round of communication or removed equivocation checks by honest agents!

Feel free to reach out for more information on this project, including relevant graphs, research reports, etc. 

[Repository](https://github.com/TrentBrick/LearningConsensus)

[Results Summary](https://docs.google.com/presentation/d/1n4jvtWxaJaTn8I4tY7RwuNYydPY-bhVheQpvUw85a_Y/edit?usp=sharing)

[Report 1](http://127.0.0.1:4000/files/LearningConsensus.pdf)

[Report 2](https://docs.google.com/presentation/d/1Po8cs_iZW7M-6bx6R5ANghMItMP-s1Eb/edit?usp=sharing&ouid=106978351856045405483&rtpof=true&sd=true)

<!-- [Miscellaneous Graphs](https://www.overleaf.com/read/cdckxrjksgdz)  -->

