---
date: 2018-05-25
title: Vue templates in JSX
categories: ["articles"]
tags:
    - Vue.js
    - jsx
    - JavaScript
---

In my most recent project at work, I'm experimenting with JSX templates in Vue. Vue offers [first-party support](https://vuejs.org/v2/guide/render-function.html#JSX) for JSX with near-zero configuration, but it doesn't seem to be commonly used in the ecosystem.

<!--more-->

_Here's the tl;dr. Every one of these is discussed in detail below._

**PRO**

- The full power of JavaScript to your disposal
- No need to register components
- Spread operator
- Casing is easy
- No clashes with existing HTML elements
- You're not tied to the one-component-per-file requirement

**CON**

- It looks foreign to designers
- No control structure directives
- No more style tags in your single file components
- JSX is unpopular in the Vue ecosystem

I'm going to share my initial thoughts on using JSX with Vue. I'll be posting side-by-side examples of Vue templates and their JSX counterparts.

To get the ball rolling, here's a straightforward example of what JSX looks like in a simple Vue component:

```html
<template>
  <h1>{{ message }}</h1>
</template>

<script>
  export default {
    data: () => ({
      message: "Hello, JSX!"
    })
  };
</script>
```

```js
export default {
  data: () => ({
    message: "Hello, JSX!"
  }),

  render() {
    return <h1>{this.message}</h1>;
  }
};
```

## Pro: The full power of JavaScript to your disposal

Vue templates are limited to what's registered in the component's options. With JSX, you can do _anything_ inside the `render` function.

No need to assign functions to `methods`, which means a little less boilerplate.

```html
<template>
  <span>{{ formatPrice(this.price) }}</span>
</template>

<script>
  import { formatPrice } from "./util";

  export default {
    props: ["price"],

    methods: {
      formatPrice
    }
  };
</script>
```

```js
import { formatPrice } from "./util";

export default {
  props: ["price"],

  render() {
    return <span>{formatPrice(this.price)}</span>;
  }
};
```

## Pro: No need to register components

Another small quality of life change that reduces boilerplate. You can directly use your components in the `render` function instead of aliasing them to a string in the `components` option.

```html
<template>
  <span class="price-tag">
    <formatted-price :price="price"></formatted-price>
  </span>
</template>

<script>
  import FormattedPrice from "./FormattedPrice";

  export default {
    data: () => ({
      price: 100
    }),

    components: {
      FormattedPrice
    }
  };
</script>
```

```js
import FormattedPrice from "./FormattedPrice";

export default {
  data: () => ({
    price: 100
  }),

  render() {
    return (
      <span class="price-tag">
        <FormattedPrice price={this.price} />
      </span>
    );
  }
};
```

## Pro: Spread operator

In Vue templates, we can use `v-bind` to pass an object as component props. An example from the Vue docs:

```html
<blog-post v-bind="post"></blog-post>

<!-- Will be equivalent to: -->

<blog-post v-bind:id="post.id" v-bind:title="post.title"></blog-post>
```

This is very similar to JavaScript's [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax), which _is_ available to us in JSX.

An added benefit of the spread syntax, is that it can be used multiple times per component. Since `v-bind` is an attribute, it's limited to a single declaration.

```html
<template>
  <!-- This doesn't work! -->
  <blog-post v-bind="post" v-bind="metaData"></blog-post>
</template>

<script>
  // ...
</script>
```

```js
export default {
  // ...
  render() {
    return <BlogPost {...this.post} {...this.metaData} />;
  }
};
```

## Pro: Casing is easy

Casing in Vue is rough.

Templates want everything to be kebab-case, while everything in your script is probably camelCase.

From the Vue docs:

> HTML attribute names are case-insensitive, so browsers will interpret any uppercase characters as lowercase. That means when you’re using in-DOM templates, camelCased prop names need to use their kebab-cased (hyphen-delimited) equivalents

You _can_ use PascalCase for components and camelCase for props your `.vue` files, but then they won't work in in-DOM templates. Oh, and that only applies to component names and props. Events need to be written exactly as-is, no behind-the-scene case changes there.

```html
<!-- App.vue -->
<template>
  <!-- How it should be, PascalCase components, kebab-cased attributes -->
  <PostList
    posts="posts"
    link-color="blue"
    @post-click="handlePostClick"
  ></PostList>

  <!-- This also works, but not in in-DOM templates -->
  <!-- Don't forget you can't change event casing! -->
  <PostList
    posts="posts"
    linkColor="blue"
    @post-click="handlePostClick"
  ></PostList>
</template>

<!-- PostList.vue -->
<script>
export default {
  // Meanwhile, props should be declared with camelCase
  props: ['linkColor'],
}
```

All of these issues disappear when using JSX. Since you're writing JavaScript, you simply use PascalCase and kebabCase _everywhere_.

```js
export default {
  // ...
  render() {
    return (
      <PostList
        posts={this.posts}
        linkColor="blue"
        onPostClick={() => this.handlePostClick()}
      />
    );
  }
};
```

## Pro: No clashes with existing HTML elements

More added benefits because we're breaking away from HTML. From the Vue docs:

> The name you give a component may depend on where you intend to use it. When using a component directly in the DOM (as opposed to in a string template or single-file component), we strongly recommend following the W3C rules for custom tag names (all-lowercase, must contain a hyphen). This helps you avoid conflicts with current and future HTML elements.

Additionally, if you want a custom `form` component, you need to rename it, because it would clash with the existing `form` tag. Naming things is hard enough as it is!

```html
<template>
  <my-form></my-form>
</template>
```

```js
import Form from "./Form";

export default {
  render() {
    return <Form />;
  }
};
```

## Pro: You're not tied to the one-component-per-file requirement

Sometimes you want to write a little component, that's _only_ going to be used in the context of another component. With `.vue` files, you'd need to create two files, even though the second one is trivial and shouldn't be reused anywhere else.

With JSX, you can structure things however you like. I generally stick to one component per file, but it can be useful to extract bits of that to make things more readable.

```html
<template>
  <article>
    <post-title :title="title"></post-title>
    <section>{{ post.contents }}</section>
  </article>
</template>
<script>
  import PostTitle from "./PostTitle";

  export default {
    props: ["post"],

    components: {
      PostTitle
    }
  };
</script>

<!-- PostTitle can now be imported anywhere, while we only created
     it for the Post component -->
<template>
  <h1>{{ title }}</h1>
</template>
<script>
  export default {
    props: ["title"]
  };
</script>
```

```js
// Since the rest of the application doesn't need PostTitle,
// we shouldn't expose it.
const PostTitle = {
  props: ["title"],

  render() {
    return <h1>{this.title}</h1>;
  }
};

export default {
  props: ["post"],

  render() {
    return (
      <article>
        <PostTitle title={this.post.title} />
        <section>{this.post.contents}</section>
      </article>
    );
  }
};
```

## Con: It looks foreign to designers

If someone else is coding your app's design, you're forcing them into scary-JavaScript-territory instead of happy-HTML-land.

It depends on your team and situation if this is a tradeoff worth making.

## Con: No control structure directives

You might miss the control structures Vue templates offer. I personally don't mind fully embracing JavaScript, for example by using `map` instead of `v-for`.

Less custom directives mean less abstraction between the template and the code it compiles too.

```html
<template>
  <ul>
    <li v-for="post in posts" :key="post.id">{{ post.title }}</li>
  </ul>
</template>
```

```js
export default {
  // ...
  render() {
    return (
      <ul>
        {this.posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    );
  }
};
```

## Con: No more style tags in your single file components

If you're using `style` tags, or want scoped styles in your Vue components, you'll need to look for a different solution.

I'm rarely use these myself, so I can't say what the alternative would look like right now.

## Con: JSX is unpopular in the Vue ecosystem

While Vue offers a first-party solution for JSX, it doesn't seem to have much traction in the Vue community.

With JSX, you're doing something different. I generally prefer to stick with the herd when it comes to tooling, unless doing otherwise offers a substantial benefit.

## Giving it a shot

I'm working on a project that has quite an amount of low-level components. They contain lots of scripting with small amounts of templating. JSX feels like a breath of fresh air in this scenario.

On the other hand, when building large views that consist of large chunks of html with some custom components and directives, Vue templates are a better fit.

Luckily, we don't need to pick one, we can use both! I'll be my low-level components with JSX, and the "views", which will be written by other developers, will be writting with familiar Vue templates.

I suppose I'll see how this all goes, only one way to find out! If I encounter a bunch tradeoffs in the coming months, expect a follow-up post about why I reverted back to `.vue` files.
