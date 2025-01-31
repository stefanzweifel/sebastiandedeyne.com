---
title: "Gary Bernhardt on tests and TypeScript"
slug: gary-bernhardt-on-tests-and-typescript
date: 2020-04-14
link: https://www.executeprogram.com/blog/are-tests-necessary-in-typescript
tags:
  - TypeScript
---

Confession: I don't write a lot of tests for my frontend code. This isn't something I talk about a lot because I'm scared of the pitchforks on the horizon.

I write React frontends with TypeScript. I don't don't write tests because I'm lazy, or because I don't have time, I simply don't see a lot of value in the amount of maintenance they require.

Gary Bernhardt wrote an article on how his apps are covered, and it hits the nail on the head.

> We don't want tests covering most of our React components, though. That wouldn't help with the main difficulties in writing components:
>
> 1. We might pass props around incorrectly. TypeScript already solves this problem for us almost completely.
> 2. The components might use the API incorrectly. Again, we've solved this problem with TypeScript.
> 3. The components might look wrong when rendered. Tests are very bad at this. We don't want to go down the esoteric and labor-intensive path of automated image capture and image diffing.

One thing I *do* write tests for is complex code that isn't coupled to components, like reducers. And the nail gets hit again:

> Although we don't test components directly, there are some other parts of the frontend code that are tested directly. For example, we have some high-value tests around some React reducers because they're tricky and full of conditionals.

Read on *Are Tests Necessary in TypeScript?* [executeprogram.com](https://www.executeprogram.com/blog/are-tests-necessary-in-typescript).
