---
title: "Laravel Typescript Transformer"
slug: laravel-typescript-transformer
date: 2020-09-29T08:00:00+02:00
tags:
  - Laravel
  - TypeScript
  - Inertia
---

My colleague Ruben released a new Spatie package to generate type declarations in TypeScript from a Laravel application.

<!--more-->

> We're writing out types two times: one time in PHP and one time in TypeScript.
>
> What will happen if we remove the guitar entry from the enum in PHP? It would still be in the TypeScript definition. Wich can break our code or give us incorrect type errors. Or what if we add a violin to our PHP Enum? This can go bad very quickly.
>
> In small projects where one or two people are working on, this isn't such a big problem. You can communicate with your colleagues about these changes if the amount of types is small.
>
> In this project, we worked with seven people, and these seven people would sometimes not work on the project for weeks. There were backend developers who wouldn't write TypeScript and frontend developers who wouldn't write PHP. Even worse, we knew the number of types grow dramatically in the timespan of the project.

The package was born in a large project we're working on with an [Inertia](https://inertiajs.com), React, and TypeScript stack.

I pushed back against the idea at first. Parsing PHP files and transforming them to another language isn't a trivial task. Especially for what I considered a marginal gain. However, the rest of the team was excited about the idea, so we decided to experiment and see where it would lead us.

After working with the automated system for a few days I quickly changed my mind. For backend developers, it saves time and mental overhead since they don't need to dive into TypeScript. Frontend developers are more confident using the types since they're sure nothing got lost in translation.

Read Ruben's in-depth introduction [on his blog](https://rubenvanassche.com/typing-your-frontend-from-the-backend/), or check out the package [on GitHub](https://github.com/spatie/laravel-typescript-transformer).
