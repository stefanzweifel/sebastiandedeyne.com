---
title: "Masonry layouts with CSS grid"
slug: masonry-layout
date: 2020-11-04
tags:
  - CSS
  - CSS Grid
---

Masonry layout support has been added to the CSS grid specification! 🎉

<!--more-->

> A Level 3 of the CSS Grid specification has been published as an Editor’s Draft, this level describes a way to do Masonry layout in CSS.
>
> A masonry layout is one where items are laid out one after the other in the inline direction. When they move onto the next line, items will move up into any gaps left by shorter items in the first line. It’s similar to a grid layout with auto-placement, but without sticking to a strict grid for the rows.

To create a masonry layout in CSS, use the new `grid-template-rows: masonry` property.

```css
.container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: masonry;
}
```

The masonry layout was popularized by Pinterest (they call it a "Collage" in [their design system](https://gestalt.netlify.app/Collage)). It's a useful tool to display text and images without truncating or expanding blocks to a fixed size.

This is CSS a feature I've been hoping for for a while. It's one of the few things left that's (mostly) not possible without JavaScript. CSS-Tricks has a [compilation](https://css-tricks.com/piecing-together-approaches-for-a-css-masonry-layout/) of ways to create Masonry grids, but all non-JS implementations have at least on tradeoff to keep in mind.

I use a masonry layout or [my recipes site](https://recipes.sebastiandedeyne.com). It's a CSS-only implementation with CSS columns. The problem is that the content is ordered from top to bottom, then left to right. Since I don't often browse the recipes (I use the search input), this was a tradeoff I could live with.

```txt
             Columns        |       Masonry
             A C E G        |       A B C D
             B D F H        |       E F G H
```

Masonry with CSS grid currently only works in Firefox, but I hope other browsers will follow soon. I'm not a CSS grid power user. Old habits die hard and I often use flexbox unless I need a grid-specific feature. Time to head back to [Grid Garden](https://cssgridgarden.com) to flex those muscles!

You can read all about the new masonry features in Rachel Andrew's article on [Smashing Magazine](https://www.smashingmagazine.com/native-css-masonry-layout-css-grid/).
