+++
title = 'AI-Powered Movie Tier List'
date = 2024-11-18T11:50:05+02:00
draft = true
tags = ["Python", "React", "AI", "Movies"]
+++
Over the weekend, I built a movie recommendation app with a twist: instead of the usual 5-star ratings, users organize movies into tier lists (S/A/B/C/D) through drag-and-drop. As you rank movies, the app learns your preferences and suggests new ones. I wanted to explore what it's like to build a full-stack application with AI assistance, particularly using ChatGPT.

## The Good Parts
I prompted Claude with the idea and it generated a basic boilerplate for the React frontend. Then I asked it to add the tier interface, this pretty much worked out of the box and I only had to make a few small modification to change the design more to my liking.

With the basic UI working, I asked it to add drag-and-drop functionality for the tier list. The first version it generated relied on a deprecated React library, so I asked it to use a different one, and after retrying a couple of times it was able to get it working.

## The Challenges
However, the experience wasn't without its hurdles. While AI excels at generating isolated components and basic functionality, it struggles with more complex integration tasks. Connecting the frontend to the backend required significant manual intervention, especially when dealing with real-time updates and state management.
The recommendation system also needed careful attention. While ChatGPT could help structure the API calls to OpenAI's GPT-4, fine-tuning the prompts and handling edge cases required human oversight. The AI sometimes generated overly complex solutions or missed important error handling scenarios.

## Key Learnings
Building this project taught me that AI is best used as a coding co-pilot rather than a complete solution. It's incredibly valuable for:

Quick prototyping and boilerplate generation
Solving well-defined, isolated problems
Providing templates for common patterns

However, you still need solid engineering skills for:

- System architecture and integration
- Error handling and edge cases
- Performance optimization
- Security considerations

In the end, the project was a success - I built a functional movie recommendation system that users can interact with through an intuitive tier list interface. More importantly, I gained valuable insights into how to effectively leverage AI in the development process.
The code is available on GitHub, and I'm continuing to improve it based on user feedback. If you're interested in building something similar, I'd recommend starting with small, well-defined components and gradually integrating them into a complete system.