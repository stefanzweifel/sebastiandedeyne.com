---
title: "Forget about component lifecycles and start thinking in effects"
subtitle: "About React's useEffect hook"
date: 2019-06-20T17:47:02+02:00
categories: ["articles"]
tags:
    - React
    - JavaScript
---

React components have always relied on lifecycle methods for side effects. While lifecycle methods get the job done, they're often overly verbose and have large margins for error.

It's easy to forget to "clean up" a side effect when a component unmounts, or update the side effect when props change. As Dan Abramov preaches: [Don't stop the data flow](https://overreacted.io/writing-resilient-components/#principle-1-dont-stop-the-data-flow).

React recently introduced a new way to deal with side effects: the `useEffect` hook. Translating lifecycle methods to `useEffect` calls can be confusing at first. It's confusing because we shouldn't be translating imperative lifecycle methods to declarative `useEffect` calls in the first place.

<!--more-->

## Listening to websockets

Consider a `ChatChannel` component that listens to websockets.

```js
import React, { Component } from 'react';
import websockets from 'websockets';

class ChatChannel extends Component {
  state = {
    messages: [];
  }

  componentDidMount() {
    websockets.listen(
      `channels.${this.props.channelId}`,
      message => {
        this.setState(state => {
          return { messages: [...state.messages, message] };
        });
      }
    );
  }

  render() {
    // ...
  }
}
```

If we run this in our browser, it will work! However, there are a two silent issues.

First, we never clean up our side effect. When the `ChatChannel` component unmounts, the websocket listener is still registered. When a new event comes in, the callback will run and will try to update the state of a component that doesn't exist anymore. We need to implement a `componentWillUnmount` method to clean things up.

```js
import React, { Component } from 'react';
import websockets from 'websockets';

class ChatChannel extends Component {
  state = {
    messages: [];
  }

  componentDidMount() {
    websockets.listen(
      `channels.${this.props.channelId}`,
      message => {
        this.setState(state => {
          return { messages: [...state.messages, message] };
        });
      }
    );
  }

  componentWillUnmount() {
    websockets.unlisten(`channels.${this.props.channelId}`);
  }

  render() {
    // ...
  }
}
```

We'll also be seeing some very weird chat threads when the `channelId` prop changes over time. The component would render a different channel, but the initial `channelId`'s events will still be pushed through.

To solve this we add a `componentDidUpdate` implementation and shuffle things around to keep things DRY.

```js
import React, { Component } from 'react';
import websockets from 'websockets';

class ChatChannel extends Component {
  state = {
    messages: [];
  }

  componentDidMount() {
    this.startListeningToChannel(this.props.channelId);
  }

  componentDidUpdate(prevProps) {
    if (this.props.channelId !== prevProps.channelId) {
      this.stopListeningToChannel(prevProps.channelId);
      this.startListeningToChannel(this.props.channelId);
    }
  }

  componentWillUnmount() {
    this.stopListeningToChannel(this.props.channelId);
  }

  startListeningToChannel(channelId) {
    websockets.listen(
      `channels.${channelId}`,
      message => {
        this.setState(state => {
          return { messages: [...state.messages, message] };
        });
      }
    );
  }

  stopListeningToChannel(channelId) {
    websockets.unlisten(`channels.${channelId}`);
  }

  render() {
    // ...
  }
}
```

The problem with lifecycle methods for side effects is that they run based on implementation details of the way React renders components. In practice, side effects are based on what truly matters: *our* data that can change over time. In this case, props.

## Refactoring to `useEffect`

Let's rewrite our `ChatChannel` component with `useEffect`.

```jsx
import React, { useEffect, useState } from 'react';
import websockets from 'websockets';

function ChatChannel({ channelId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    websockets.listen(
      `channels.${channelId}`,
      message => setMessages(messages => [...messages, message])
    );

    return () => websockets.unlisten(`channels.${channelId}`);
  }, [channelId]);

  // ...
}
```

Instead of hooking into the component lifecycle's nitty gritty details, we simply declare that there's some `listen` side effect based on `channelId`'s current value. We also return a `unlisten` call in a closure. React will run that cleanup callback whenever the effect is invalidated: either when `channelId` changes, or when the component unmounts.

Instead of thinking about when *we* should apply the side effect, we declare the side effect's dependencies. This way *React* knows when it needs to run, update, or clean up.

That's where the power of `useEffect` lies. The websocket listener doesn't care about mounting and unmounting components, it cares about the value of `channelId` over time.

> The question is not "when does this effect run" the question is "with which state does this effect synchronize with"
>
> useEffect(fn) // all state<br> useEffect(fn, []) // no state<br> useEffect(fn, [these, states])
>
> <cite><a href="https://twitter.com/ryanflorence/status/1125041041063665666">@ryanflorence on Twitter</a></cite>
</blockquote>
