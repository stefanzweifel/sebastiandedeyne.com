---
title: "Mailcoach's (lack of) JavaScript stack"
slug: mailcoachs-lack-of-javascript-stack
date: 2020-01-30
categories: ["articles"]
tags:
  - JavaScript
  - Spatie
---

Yee-haw, we released [Mailcoach](https://mailcoach.app)! Mailcoach is a self-hosted newsletter solution. My contributions were on the JavaScript side of things: I helped decide which tech stack to use, and implemented it.

After building apps with almost exclusively Vue and React the past few years, we decided to go with vanilla JavaScript for Mailcoach. There's no frontend framework involved, but we're pulling in some npm packages where needed.

I'm not going to dive into implementation details. I'm going to talk about *why* we decided on this stack and go in-depth on the structure of the application code, bundle sizes, and choosing external dependencies.

<!--more-->

## Going vanilla

We're distributing Mailcoach as a Composer package that you can install into any existing Laravel application. The Mailcoach interface doesn't have a vast amount of screens, but a fundamental design principle of the package is that it needs to be easily extensible by package users.

![A screenshot of the Mailcoach interface](/media/mailcoach/mailcoach-ui.jpg)

Extensibility is a key part of why we didn't use Vue or React. If we were to make heavy use of a front-end framework, package users would need knowledge of our tools to extend the interface. There would be a big chance that developers would need to modify our JavaScript code or write their own to tweak the UI. Besides framework knowledge, they'd need enough frontend know-how to make a custom build of the application scripts with Laravel Mix, and we all know how frustrating the Node ecosystem is for the uninitiated. With a frontend framework, developers that want to play with the UI would need to take ownership of a lot of UI code.

Alternatively, we would need to carry the burden of adding all sorts of abstractions to make our interface extensible. Laravel Nova proves this is possible, but it's hard to get right and a lot of work to maintain.

We want it to be dead easy for eager developers to tinker with the Mailcoach interface. By keeping the stack as small as possible, we impose the least amount of required knowledge as possible.

Keeping the stack small isn't about loading fewer kilobytes of JavaScript; it's about keeping the amount of moving parts down. Mailcoach isn't a "Blade and Vue" or "Blade and React" app. It's just a Blade app.

Distributing Mailcoach with Composer also means we need to push our processed assets to the git repository. In a JavaScript-heavy interface, this can be a frustrating source of merge conflicts. By keeping most of the interface in Blade, we're able to sidestep this problem.

## JavaScript setup

According to [CLOC](https://github.com/AlDanial/cloc), Mailcoach's UI counts 500 lines of custom JavaScript code, divided over 20-ish files.

```txt
--------------------------------------------------------
Language      files        blank      comment       code
--------------------------------------------------------
JavaScript       20          126            5        494
```

The file structure is pretty straightforward.

```txt
components/
  ...
util/
  ...
app.js
```

There's one entry point: `app.js`, which imports modules from `components`. Some examples are `modal.js`, `dropdown.js`, and `datepicker.js`. These get mounted using `data-` attributes.

A small dropdown example:

```html
<div class="dropdown" data-dropdown>
  <button class="icon-button" data-dropdown-trigger>
    <i class="fas fa-ellipsis-v | dropdown-trigger-rotate"></i>
  </button>
  <ul
    class="dropdown-list dropdown-list-left | hidden"
    data-dropdown-list
  >
    <!-- Dropdown items -->
  </ul>
</div>
```

`util` contains all sorts of utility functions, which are used by the components. Examples include a `debounce` function, a helper to trap focus (used in modals), and wrappers around DOM APIs like `document.querySelector`.

## External libraries

Altogether, Mailcoach's `app.js` weighs in at about 45 KB (minified and gzipped). About 42 KB of this is spent on external libraries. The remaining 3 KB is the custom application code, as explained above.

| Dependency | Size |
|:---|--:|
| `choices.js` | 18 KB |
| `flatpickr` | 13 KB |
| `turbolinks` | 9 KB |
| `morphdom` | 2 KB |

We're picky about our dependencies. Every piece of third party code that ends up in our application bundle must meet the following criteria:

First, it solves a hard problem OR enables us to simplify parts of our code significantly.

Second, it's worth its cost in KBs. If a small problem requires a large dependency to solve, there's probably a better solution lurking around the corner.

Our heaviest dependencies are `choices.js` and `flatpickr.js`. Choices is used for autocompleted multi-selects to manage tags, and Flatpickr is a date picker to schedule a campaign. We'd have a hard time solving those interface problems better than a respected npm package, so deciding to pull these in was a natural choice.

Turbolinks makes the UI significantly more snappy. It also enables us to build datatables without providing separate AJAX endpoints. Turbolinks improves the user experience and simplifies our backend code.

![A datatable in the Mailcoach UI](/media/mailcoach/mailcoach-datatable.jpg)

We use morphdom to patch bits of HTML with fresh chunks from the server ([server fetched partials](https://laracasts.com/series/javascript-techniques-for-server-side-developers/episodes/1)). We're using morphdom instead of manually setting `innerHTML` with the native DOM API because it enables us to animate transitions with CSS. Since it's a small library with a simple API and we could drop it without significantly degrading the user experience, it's allowed in the bundle.

## Alpine

Even with helper functions in place, using the DOM API for small components like dropdowns and modals can feel repetitive and cumbersome. The new and shiny [Alpine.js](https://github.com/alpinejs/alpine) would probably be an excellent fit for a project like this. It wouldn't replace all of our JavaScript code, but I wouldn't be surprised if it would reduce our code count by half.

Alpine wasn't available yet when we were knee-deep into building Mailcoach, but we might give it a spin in a future refactor or our next big project.

## Fruits of a smaller stack

The last few projects I worked on are built with [Inertia](https://inertiajs.com) and React. If I compare the amount of code between Blade and JavaScript, one Inertia project contained 59% JavaScript code and another a whopping 91%. Mailcoach is just 13% JavaScript, most of the UI is plain Blade.

When building new screens, I often didn't even need to run `yarn watch` unless I needed to write some custom script for the page. Going back to building things without spinning up a build process in the background was a breath of fresh air. It makes you realize how much complexity we've been shoving into our client-side code.

Some interfaces thrive with a JavaScript-heavy solution like Inertia or a single page application, but you'd be surprised how far you can get without (and how much lighter it feels).

## Vanilla JavaScript Diet

Building an app with vanilla JavaScript was a great experience. We're not dropping React and Vue from our tech stack at Spatie, but we're going to think twice before mindlessly grabbing a framework.

Inspired by my experience with Mailcoach, I'm starting a blog series next week: Vanilla JavaScript Diet. I'll be sharing the techniques used in Mailcoach, going into detail on how we're communicating with the DOM, and structuring our application code. See you there!
