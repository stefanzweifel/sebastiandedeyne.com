---
date: 2019-05-20
title: "React for Vue developers"
categories: ["articles"]
tags:
    - React
    - Vue.js
    - JavaScript
---

For the past three years, I've been using both React and Vue in different projects, ranging from smaller websites to large scale apps.

Last month I wrote a post about [why I prefer React over Vue](https://sebastiandedeyne.com/why-i-prefer-react-over-vue/). Shortly after I joined Adam Wathan on [Full Stack Radio](http://www.fullstackradio.com/114) to talk about React from a Vue developer's perspective.

We covered a lot of ground on the podcast, but most things we talked about could benefit from some code snippets to illustrate their similaraties and differences.

This post is a succinct rundown of most Vue features, and how I would write them with React in 2019 with hooks.

<!--more-->

*Did I miss anything? Are there other comparisons you'd like to see? Or do you just want to share your thoughts about Vue or React? Talk to me on [Twitter](https://twitter.com/sebdedeyne)!*

## Table of contents

- [Templates](#templates)
- [Props](#props)
- [Data](#data)
- [Computed properties](#computed-properties)
- [Methods](#methods)
- [Events](#events)
- [Lifecycle methods](#lifecycle-methods)
- [Watchers](#watchers)
- [Slots & scoped slots](#slots--scoped-slots)
- [Provide / inject](#provide--inject)
- [Custom directives](#custom-directives)
- [Transitions](#transitions)

## Templates

*React alternative: JSX*

Vue uses HTML strings with some custom directives for templating. They recommend using `.vue` files to seperate the templates and script (and optionally styles).

```html
<!-- Greeter.vue -->

<template>
  <p>Hello, {{ name }}!</p>
</template>

<script>
export default {
  props: ['name']
};
</script>
```

React uses [JSX](https://facebook.github.io/jsx/), which is an extension of ECMAScript.

```jsx
export default function Greeter({ name }) {
  return <p>Hello, {name}!</p>;
}
```

### Conditional rendering

*React alternative: Logical `&&` operator, ternary statements, or early returns*

Vue uses `v-if`, `v-else` and `v-else-if` directives to conditionally render parts of a template.

```html
<!-- Awesome.vue -->

<template>
  <article>
    <h1 v-if="awesome">Vue is awesome!</h1>
  </article>
</template>

<script>
export default {
  props: ['awesome']
};
</script>
```

React doesn't support directives, so you need to use the language to conditionally return parts of a template.

The `&&` operator provides a succinct way to write an `if` statement.

```jsx
export default function Awesome({ awesome }) {
  return (
    <article>
      {awesome && <h1>React is awesome!</h1>};
    </article>
  );
}
```

If you need an `else` clause, use a ternary statement instead.

```jsx
export default function Awesome({ awesome }) {
  return (
    <article>
      {awesome ? (
        <h1>React is awesome!</h1>
      ) : (
        <h1>Oh no 😢</h1>
      )};
    </article>
}
```

You could also opt to keep the two branches completely separated, and use an early return instead.

```jsx
export default function Awesome({ awesome }) {
  if (!awesome) {
    return (
      <article>
        <h1>Oh no 😢</h1>
      </article>
    );
  }

  return (
    <article>
      <h1>React is awesome!</h1>
    </article>
  );
}
```

### List rendering

*React alternative: `Array.map`*

Vue uses the `v-for` directive to loop over arrays and objects.

```html
<!-- Recipe.vue -->

<template>
  <ul>
    <li v-for="(ingredient, index) in ingredients" :key="index">
      {{ ingredient }}
    </li>
  </ul>
</template>

<script>
export default {
  props: ['ingredients']
};
</script>
```

With React, you can "map" the array to a set of elements using the built in `Array.map` function.

```jsx
export default function Recipe({ ingredients }) {
  return (
    <ul>
      {ingredients.map((ingredient, index) => (
        <li key={index}>{ingredient}</li>
      ))}
    </ul>
  );
}
```

Iterating objects is a bit trickier. Vue allows you to use the same `v-for` directive for keys & values.

```html
<!-- KeyValueList.vue -->

<template>
  <ul>
    <li v-for="(value, key) in object" :key="key">
      {{ key }}: {{ value }}
    </li>
  </ul>
</template>

<script>
export default {
  props: ['object'] // E.g. { a: 'Foo', b: 'Bar' }
};
</script>
```

I like to use the built in `Object.entries` function with React to iterate over objects.

```jsx
export default function KeyValueList({ object }) {
  return (
    <ul>
      {Object.entries(object).map(([key, value]) => (
        <li key={key}>{value}</li>
      ))}
    </ul>
  );
}
```

### Class and style bindings

*React alternative: Manually pass props*

Vue automatically binds `class` and `style` props to the outer HTML element of a component.

```html
<!-- Post.vue -->

<template>
  <article>
    <h1>{{ title }}</h1>
  </article>
</template>

<script>
export default {
  props: ['title'],
};
</script>

<!--
<post
  :title="About CSS"
  class="margin-bottom"
  style="color: red"
/>
-->
```

With React, you need to manually pass `className` and `style` props. Note that `style` must be an object with React, strings are not supported.

```jsx
export default function Post({ title, className, style }) {
  return (
    <article className={className} style={style}>
      {title}
    </article>
  );
}

{/* <Post
  title="About CSS"
  className="margin-bottom"
  style={{ color: 'red' }}
/> */}
```

If you want to pass down all remaining props, the object rest spread operator comes in handy.

```jsx
export default function Post({ title, ...props }) {
  return (
    <article {...props}>
      {title}
    </article>
  );
}
```

If you miss Vue's excellent `class` API, look into Jed Watson's [classnames](https://github.com/JedWatson/classnames) library.

## Props

*React alternative: Props*

Props behave pretty much the same way in React as Vue. One minor difference: React components won't [inherit](https://vuejs.org/v2/api/#inheritAttrs) unknown attributes.

```html
<!-- Post.vue -->

<template>
  <h1>{{ title }}</h1>
</template>

<script>
export default {
  props: ['title'],
};
</script>
```

```jsx
export default function Post({ title }) {
  return <h3>{title}</h3>;
}
```

Using expressions as props in Vue is possible with a `:` prefix, which is an alias for the `v-bind` directive. React uses curly braces for dynamic values.

```html
<!-- Post.vue -->

<template>
  <post-title :title="title" />
</template>

<script>
export default {
  props: ['title'],
};
</script>
```

```jsx
export default function Post({ title }) {
  return <PostTitle title={title} />;
}
```

## Data

*React alternative: The `useState` hook*

In Vue the `data` option is used to store local component state.

```html
<!-- ButtonCounter.vue -->

<template>
  <button @click="count++">
    You clicked me {{ count }} times.
  </button>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    }
  }
};
</script>
```

React exposes a `useState` hook which returns a two-element array containing the current state value and a setter function.

```jsx
import { useState } from 'react';

export default function ButtonCounter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  );
}
```

You can choose whether you prefer to distribute state between multiple `useState` calls, or keep it in a single object.

```jsx
import { useState } from 'react';

export default function ProfileForm() {
  const [name, setName] = useState('Sebastian');
  const [email, setEmail] = useState('sebastian@spatie.be');

  // ...
}
```

```jsx
import { useState } from 'react';

export default function ProfileForm() {
  const [values, setValues] = useState({
    name: 'Sebastian',
    email: 'sebastian@spatie.be'
  });

  // ...
}
```

### `v-model`

`v-model` is a convenient Vue directive that combines passing down a `value` prop with listening to an `input` event. This makes it *look* like Vue does two-way binding, while it's still just *"props down, events up"* under the hood.

```html
<!-- Profile.vue -->

<template>
  <input type="text" v-model="name" />
</template>

<script>
export default {
  data() {
    return {
      name: 'Sebastian'
    }
  }
};
</script>
```

Vue expands the `v-model` directive to the following:

```html
<template>
  <input
    type="text"
    :value="name"
    @input="name = $event.target.value"
  />
</template>
```

React doesn't have a `v-model` equivalent. You always need to be explicit:

```jsx
import { useState } from 'react';

export default function Profile() {
  const [name, setName] = useState('Sebastian');

  return (
    <input
      type="text"
      value={name}
      onChange={event => setName(event.target.name)}
    />
  );
}
```

## Computed properties

*React alternative: Variables, optionally wrapped in `useMemo`*

Vue has computed properties for two reasons: to avoid mixing logic and markup in templates, and to cache complex computations in a component instance.

Without computed properties:

```html
<!-- ReversedMessage.vue -->

<template>
  <p>{{ message.split('').reverse().join('') }}</p>
</template>

<script>
export default {
  props: ['message']
};
</script>
```

```jsx
export default function ReversedMessage({ message }) {
  return <p>{message.split('').reverse().join('')}</p>;
}
```

With React, you can extract the computation from the template by assigning the result to a variable.

```html
<!-- ReversedMessage.vue -->

<template>
  <p>{{ reversedMessage }}</p>
</template>

<script>
export default {
  props: ['message'],

  computed: {
    reversedMessage() {
      return this.message.split('').reverse().join('');
    }
  }
};
</script>
```

```jsx
export default function ReversedMessage({ message }) {
  const reversedMessage = message.split('').reverse().join('');

  return <p>{reversedMessage}</p>;
}
```

If performance is a concern, the computation can be wrapped in a `useMemo` hook. `useMemo` requires a callback that returns a computed result, and an array of dependencies.

In the following example, `reversedMessage` will only be recomputed if the `message` dependency changes.

```jsx
import { useMemo } from 'react';

export default function ReversedMessage({ message }) {
  const reversedMessage = useMemo(() => {
    return message.split('').reverse().join('');
  }, [message]);

  return <p>{reversedMessage}</p>;
}
```

## Methods

*React alternative: Functions*

Vue has a `methods` option to declare functions that can be used throughout the component.

```html
<!-- ImportantButton.vue -->

<template>
  <button onClick="doSomething">
    Do something!
  </button>
</template>

<script>
export default {
  methods: {
    doSomething() {
      // ...
    }
  }
};
</script>
```

In React you can declare plain functions inside our component.

```jsx
export default function ImportantButton() {
  function doSomething() {
    // ...
  }

  return (
    <button onClick={doSomething}>
      Do something!
    </button>
  );
}
```

## Events

*React alternative: Callback props*

Events are essentially callbacks that are called when something happened in the child component. Vue sees events as a first-class citizen, so you can "listen" to them with `@`, which is shorthand for the `v-on` directive.

```html
<!-- PostForm.vue -->

<template>
  <form>
    <button type="button" @click="$emit('save')">
      Save
    </button>
    <button type="button" @click="$emit('publish')">
      Publish
    </button>
  </form>
</template>
```

Events don't have any special meaning in React, they're just callback props will be called by the child component.

```jsx
export default function PostForm({ onSave, onPublish }) {
  return (
    <form>
      <button type="button" onClick={onSave}>
        Save
      </button>
      <button type="button" onClick={onPublish}>
        Publish
      </button>
    </form>
  );
}
```

### Event modifiers

*React alternative: Higher order functions if you really want*

Vue has a few modifiers like `prevent` and `stop` to change the way an event is handled without touching it's handler.

```html
<!-- AjaxForm.vue -->

<template>
  <form @submit.prevent="submitWithAjax">
    <!-- ... -->
  </form>
</template>

<script>
export default {
  methods: {
    submitWithAjax() {
      // ...
    }
  }
};
</script>
```

There's no modifier syntax in React. Preventing defaults and stopping propagation is mostly handled in the callback.

```jsx
export default function AjaxForm() {
  function submitWithAjax(event) {
    event.preventDefault();
    // ...
  }

  return (
    <form onSubmit={submitWithAjax}>
      {/* ... */}
    </form>
  );
}
```

If you really want to have something modifier-like, you could use a higher order function.

```jsx
function prevent(callback) {
  return (event) => {
      event.preventDefault();
      callback(event);
  };
}

export default function AjaxForm() {
  function submitWithAjax(event) {
    // ...
  }

  return (
    <form onSubmit={prevent(submitWithAjax)}>
      {/* ... */}
    </form>
  );
}
```

## Lifecycle methods

*React alternative: The `useEffect` hook*

<aside>
<p><strong>DISCLAIMER</strong></p>
<p>With class components, React has a very <a href="https://reactjs.org/docs/react-component.html#the-component-lifecycle">similar API</a> to Vue when it comes to the component lifecycle. With hooks, most lifecycle-related problems can be solved with <code>useEffect</code>. Effects and lifecycle methods are completely different paradigms, so they're hard to compare. In turn, this section is limited to a few practical examples, as effects deserve their own article.</p>
</aside>

A common case for lifecycle methods is to set up and tear down third party libraries.

```html
<template>
  <input type="text" ref="input" />
</template>

<script>
import DateTimePicker from 'awesome-date-time-picker';

export default {
  mounted() {
   this.dateTimePickerInstance =
     new DateTimePicker(this.$refs.input);
  },

  beforeDestroy() {
    this.dateTimePickerInstance.destroy();
  }
};
</script>
```

With `useEffect`, you can declare a "side effect" that needs to run after a render. When you return a callback from `useEffect`, it will be invoked when the effect gets cleaned up. In this case, when the component is destroyed.

```jsx
import { useEffect, useRef } from 'react';
import DateTimePicker from 'awesome-date-time-picker';

export default function Component() {
  const dateTimePickerRef = useRef();

  useEffect(() => {
    const dateTimePickerInstance =
      new DateTimePicker(dateTimePickerRef.current);

    return () => {
      dateTimePickerInstance.destroy();
    };
  }, []);

  return <input type="text" ref={dateTimePickerRef} />;
}
```

This looks similar to registering a `beforeDestroy` listener in `mounted` in a Vue component.

```html
<script>
export default {
  mounted() {
    const dateTimePicker =
      new DateTimePicker(this.$refs.input);

    this.$once('hook:beforeDestroy', () => {
      dateTimePicker.destroy();
    });
  }
};
</script>
```

Similar to `useMemo`, `useEffect` accepts an array of dependencies as a second parameter.

Without any specified dependencies, the effect will run after every render, and will clean up before every next render. This functionality is similar to a combination of `mounted`, `updated`, `beforeUpdate` and `beforeDestroy`.

```jsx
useEffect(() => {
    // Happens after every render

    return () => {
        // Optional; clean up before next render
    };
});
```

If you specify that the effect has no dependencies, the effect will only run when the component renders the first time, because it has no reason to update. This functionality is similar to a combination of `mounted`, and `beforeDestroyed`.

```jsx
useEffect(() => {
    // Happens on mount

    return () => {
        // Optional; clean up before unmount
    };
}, []);
```

If you specify a dependency, the effect will only run when the dependency changes. We'll get back to this in the [watchers](#watchers) section.

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
    // Happens when `count` changes

    return () => {
        // Optional; clean up when `count` changed
    };
}, [count]);
```

Trying to directly translating lifecycle hooks to `useEffect` calls is generally a bad idea. It's better to rethink things as a set of declarative side effects. *When* the effect is called is an implementation detail.

As Ryan Florence sums it up:

> The question is not "when does this effect run" the question is "with which state does this effect synchronize with"
>
> useEffect(fn) // all state<br> useEffect(fn, []) // no state<br> useEffect(fn, [these, states])
>
> <cite><a href="https://twitter.com/ryanflorence/status/1125041041063665666">@ryanflorence on Twitter</a></cite>
</blockquote>

## Watchers

*React alternative: The `useEffect` hook*

Watchers are conceptually similar to lifecycle hooks: *"When X happens, do Y"*. Watchers don't exist in React, but you can achieve the same with `useEffect`.

```html
<!-- AjaxToggle.vue -->

<template>
  <input type="checkbox" v-model="checked" />
</template>

<script>
export default {
  data() {
    return {
      checked: false
    }
  },

  watch: {
    checked(checked) {
      syncWithServer(checked);
    }
  },

  methods: {
    syncWithServer(checked) {
      // ...
    }
  }
};
</script>
```

```jsx
import { useEffect, useState } from 'react';

export default function AjaxToggle() {
  const [checked, setChecked] = useState(false);

  function syncWithServer(checked) {
    // ...
  }

  useEffect(() => {
    syncWithServer(checked);
  }, [checked]);

  return (
    <input
      type="checkbox"
      checked={checked}
      onChange={() => setChecked(!checked)}
    />
  );
}
```

Note that `useEffect` will also run in the first render. This is the same as using the `immediate` parameter in a Vue watcher.

If you don't want the effect to run on the first render, you'll need to create a `ref` to store whether or not the first render has happened yet or not.

```jsx
import { useEffect, useRef, useState } from 'react';

export default function AjaxToggle() {
  const [checked, setChecked] = useState(false);
  const firstRender = useRef(true);

  function syncWithServer(checked) {
    // ...
  }

  useEffect(() => {
    if (firstRender.current) {
      firstRender.current = false;
      return;
    }
    syncWithServer(checked);
  }, [checked]);

  return (
    <input
      type="checkbox"
      checked={checked}
      onChange={() => setChecked(!checked)}
    />
  );
}
```

## Slots & scoped slots

*React alternative: JSX props or render props*

If you render a template inside between a component's opening and closing tags, React passes it as a `children` prop.

With Vue, you need to declare a `<slot />` tag where inner contents belong. With React, you render the `children` prop.

```html
<!-- RedParagraph.vue -->

<template>
  <p style="color: red">
    <slot />
  </p>
</template>
```

```jsx
export default function RedParagraph({ children }) {
  return (
    <p style={{ color: 'red' }}>
      {children}
    </p>
  );
}
```

Since "slots" are just props in React, we don't need to declare anything in our templates. We can just accept props with JSX, and render them where and when we want.

```html
<!-- Layout.vue -->

<template>
  <div class="flex">
    <section class="w-1/3">
        <slot name="sidebar" />
    </section>
    <main class="flex-1">
        <slot />
    </main>
  </div>
</template>

<!-- In use: -->

<layout>
  <template #sidebar>
    <nav>...</nav>
  </template>
  <template #default>
    <post>...</post>
  </template>
</layout>
```

```jsx
export default function RedParagraph({ sidebar, children }) {
  return (
    <div className="flex">
      <section className="w-1/3">
        {sidebar}
      </section>
      <main className="flex-1">
        {children}
      </main>
    </div>
  );
}

// In use:

return (
  <Layout sidebar={<nav>...</nav>}>
    <Post>...</Post>
  </Layout>
);
```

Vue has scoped slots to pass data to the slot that will be rendered. The key part of scoped slots is *will be rendered*.

Regular slots are rendered before they get passed to the parent component. The parent component then decides what to do with the rendered fragment.

Scoped slots can't be rendered before the parent component, because they rely on data they'll receive from the parent component. In a way, scoped slots are lazily evaluated slots.

Lazily evaluating something in JavaScript is rather straightforward: wrap it in a function and call it when needed. If you need a scoped slot with React, pass a function that will render a template when called.

For a scoped slots, we can once again use `children`, or any other prop for named scoped slots. However, we'll pass down a function instead of declaring a template.

```html
<!-- CurrentUser.vue -->

<template>
  <span>
    <slot :user="user" />
  </span>
</template>

<script>
export default {
  inject: ['user']
};
</script>

<!-- In use: -->

<template>
  <current-user>
    <template #default="{ user }">
      {{ user.firstName }}
    </template>
  </current-user>
</template>
```

```jsx
import { useContext } from 'react';
import UserContext from './UserContext';

export default function CurrentUser({ children }) {
  const { user } = useContext(UserContext);

  return (
    <span>
      {children(user)}
    </span>
  );
}

// In use:

return (
  <CurrentUser>
    {user => user.firstName}
  </CurrentUser>
);
```

## Provide / inject

*React alternative: `createContext` and the `useContext` hook*

Provide / inject allows a component to share state with its subtree. React has a similar feature called context.

```html
<!-- MyProvider.vue -->

<template>
  <div><slot /></div>
</template>

<script>
export default {
  provide: {
    foo: 'bar'
  },
};
</script>

<!-- Must be rendered inside a MyProvider instance: -->

<template>
  <p>{{ foo }}</p>
</template>

<script>
export default {
  inject: ['foo']
};
</script>
```

```jsx
import { createContext, useContext } from 'react';

const fooContext = createContext('foo');

function MyProvider({ children }) {
  return (
    <FooContext.Provider value="foo">
      {children}
    </FooContext.Provider>
  );
}

// Must be rendered inside a MyProvider instance:

function MyConsumer() {
  const foo = useContext(FooContext);

  return <p>{foo}</p>;
}
```

## Custom directives

*React alternative: Components*

Directives don't exist in React. However, most problems that directives solve can be solved with components instead.

```html
<div v-tooltip="Hello!">
  <p>...</p>
</div>
```

```jsx
return (
  <Tooltip text="Hello">
    <div>
      <p>...</p>
    </div>
  </Tooltip>
);
```

## Transitions

*React alternative: Third party libraries*

React doesn't have any built in transition utilities. If you're looking for something similar to Vue, a library that doesn't actually animates anything but orchestrates animations with classes, look into [react-transition-group](https://github.com/reactjs/react-transition-group).

If you prefer a library that does more heavy lifting for you, look into [Pose](https://popmotion.io/pose/).
