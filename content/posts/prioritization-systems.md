---
title: "Prioritization systems"
date: 2019-10-30
categories: ["articles"]
tags:
  - planning
  - product management
---

I work at an agency that started out building websites and small apps, but the past years have shifted towards larger web apps and products.

Small projects are easy to manage. GitHub issues or Basecamp todos are more than enough to keep things going. Adding a feature or refactoring a small project is also easy, because there aren't too many moving parts to take consider.

Large and especially long-living projects are constant moving targets. It's harder to estimate things and technical debt can become a real issue. A linear, chronological task list won't do it anymore, we need to start prioritizing things.

I want to share four systems that have helped me, both on a personal and team level, prioritize work on large projects.

<!--more-->

## Measure impact vs effort

Impact effort matrixes are a commonly used system in product management to prioritize features.

<img src="/media/impact-effort.jpg" alt="An example impact/effort diagram illustration" class="bordered">
*Image by [Dave Gray](https://www.flickr.com/photos/davegray/5172059225/)*

On the X axis we have effort, and on the Y axis impact. If a task requires a high amount of effort, but only yields a small impact, it's probably not worth pursuing. On the other hand, a low effort task that has a big impact on your users is an investment worth making.

As the effort increases, the risk increases. Even though a high effort task yields a high impact, it may not be worth it.

Continuously look for the smallest things you can do that have the biggest impact on users.

You don't necessarily need to build a matrix to get value from measuring impact vs effort. In fact, I've never even created one of these diagrams myself. Use the idea as a mental framework every time you're about to build something. Ask yourself whether the effort required is worth the outcome. If the answer is not a resounding _yes_, maybe you should reconsider.

It doesn't stop here. Once you've decided what you're going to prioritize, keep looking for places where you can reduce the amount of effort you need to put in, while retaining most of the impact.

## Just give it a shot

This is a follow up on impact/effort. Sometimes it's hard to determine how much effort something is going to take, so you need to use something else to aid your decisions.

If you really want a to build a feature, or do a risky refactor, but you're afraid it's going to be too much work, just give it a shot.

One crucial element of this idea: before you get started, decide on a set amount of time you can lose if it doesn't work out. For small tasks this could be an hour, for larger ones this could be a day.

Work on the task for that amount of time, then decide whether it's worth pursuing or not. If you were able to overcome some hard problems within that time, it's probably worth finishing. If you have as many questions as before you started, abort.

## Stick true to design principles

Set up a few design principles for a project, and stick true to them.

For example, "every user interaction should be fast". Even better if you add something measurable, like "every user interaction should be below 300ms". If you can't complete a task without sticking true to your principles, set it aside.

This will not only help you prioritize work, but create a more coherent product.

## Make data-driven decisions

This one is only relevant after your application is in production. Use data to back your decisions. Feedback from users is valuable to help you make decisions, but not as valuable as their behaviour.

You don't necessarily need an analytics tool for this, sometimes a simple database query can tell you how and how often a feature gets used. This can be valuable information when reiterating, or prioiritizing which bugs need to be fixed.
