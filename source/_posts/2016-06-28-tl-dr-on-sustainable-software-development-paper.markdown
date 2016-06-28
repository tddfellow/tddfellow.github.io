---
layout: post
title: "TL;DR on Sustainable Software Development Paper"
date: 2016-06-28 02:22:07 +0200
comments: true
categories:
- extreme-programming
- grounded-theory
- white-paper
- tldr
- code-ownership
- sustainable-software-development
---

This is a TL;DR of a white paper [Sustainable Software Development through Overlapping Pair Rotation](https://www.researchgate.net/publication/304014117_Sustainable_Software_Development_through_Overlapping_Pair_Rotation) that is created by Todd Sedano, Paul Ralph, and Cécile Péraire.

*NOTE: this TL;DR includes a lot of compressed awesome information.*

## Theory of Sustainable Software Development

The paper introduces us to principles, policies, and practices, that are the results of applying of [Constructivist Grounded Theory](https://en.wikipedia.org/wiki/Grounded_theory#Constructivist_grounded_theory) research method to the five projects at Pivotal Labs.

### Principles

1. By **Engendering Positive Attitudes Toward Disruption** teams can transform a challenge of team disruption into a business opportunity:
   - team members rolling off from the project are replaced with new members,
   - new members are viewed as a source of fresh perspective and as an opportunity,
   - new members often questioned team's assumptions (challenging "cargo culting").
2. Policies & practices that **Encourage Knowledge Sharing and Continuity** mitigate the risk of significant knowledge loss for the project when the team is disrupted. Policies are:
   - **Team Code Ownership**, and **Shared Team Schedule**. While practices are:
   - **Continuous Pair Programming**, **Overlapping Pair Rotation** and **Knowledge Pollination**.
3. Sustainable development is not possible if the team is incurring the technical debt. **Caring about Code Quality** is a way to mitigate this risk. Policy here is:
   - **Avoid Technical Debt**, and practices are:
   - **Test-Driven Development** / **Behavior-Driven Development** and **Continuous Refactoring**.

### Policies

1. **Team Code Ownership**: every developer is empowered to work on and to refactor any part of the team's code.
2. **Shared Schedule**: every team member has agreed to a fixed schedule to achieve the benefits of Sustainable Software Development. Teams are, preferably, co-located.
3. **Avoid Technical Debt**: a pair creates well-crafted code without shortcuts and short-term fixes. Solutions are best and simplest for **current present** without over-engineering. When inherited a legacy code base, a team is actively paying down technical debt while delivering new features.

### Removing Knowledge Silos Practices

1. **Continuous Pair Programming**: two developers are collaborating to write software together as their normal mode of software development. It enables knowledge transfer, reduces knowledge silos and improves code quality. Developers always work in pairs, unless exceptional circumstances arise.
2. **Overlapping Pair Rotation**: one developer rolls off the track of work and another developer rolls on, keeping the continuity of one developer at each rotation. It results in knowledge continuity. There are three strategies for spreading the knowledge via pair rotation when fighting emerging knowledge silos:
   - Optimizing for people rotation: developers try to pair with the person they "least recently paired with";
   - Optimizing for personal preferences: developer pick with whom they will pair at each rotation, based on personal preferences;
   - Optimizing for context sharing: a developer who has not been working on the track for the longest track is rotating onto the track. The goal each day is for the developer leaving the track next to empower the developer who will remain on the track.
3. **Knowledge Pollination**:
   - Daily stand-ups create awareness of who is working on what.
   - Using backlog to communicate information and status of stories.
   - Osmotic communication: a developer overhears a discussion in other pair and offers their help.
   - The pair can interrupt another pair, product manager or designer to gain the needed information.
   - Calling out updates to the entire team.
   - Interruptions are encouraged - it makes the team more efficient as knowledge pollinates across the team.

### Caretaking the Code Practices

1. **Test-Driven Development/Behavior-Driven Development**: developers use BDD (to describe interactions between the user and the system) and TDD (at unit test level). Design emerges from the creation and exploration of the test cases. Teams use a variety of TDD strategies, including:
   - testing the responsibilities and interactions, or
   - contract testing using mocks.
2. **Continuous Refactoring**: developers do refactoring as part of implementing feature stories. Developers are encouraged to improve the code's design, make the code easier to read, understand, and increase the discoverability of a component based on its responsibility. The team prefers **pre-refactoring**, where the developer makes a complicated work to make the current story as simple and as easy as possible.
3. **Live on Master**: every pair merges their code to master many times a day. Developers may use branches to save "spikes" and to move work-in-progress between pairing stations when rotating.

## Thanks!

This is where this TL;DR ends. If you are interested in the topic, go ahead and read the paper, it has the following:

- Detailed description of the research method, data collection, and analysis;
- Research context and one project case study;
- Detailed description of Principles, Policies, and Practices of Sustainable Software Development; also anti-patterns for each policy and practice;
- Theory evaluation and conclusions.

Great thanks to the original authors of the paper, researchers, and all participants!
