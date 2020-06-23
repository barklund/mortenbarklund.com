---
title: "React Architecture: The React Provider Pattern"
date: 2020-05-11T18:23:05-04:00
publishdate: 2020-06-20T08:00:00-04:00
weight: 100
covercaption: "An <AcornProvider /> in action"
coveralt: "A hand holding an acorn"
tags: ["context", "hooks", "architecture", "design patterns", "usereducer"]
---

The React Provider Pattern is one of the main emerging React design patterns in many modern React applications and variations of it can be seen touted by React experts across the board.

This article documents the origins of this pattern, explores its uses and gives additional detail to how best to use it.

This pattern is partially informed by and (among other places) heavily used in the [open-source repository `google/web-stories-wp`](https://github.com/google/web-stories-wp) of which I'm a main contributor and co-architect. Many examples will refer to this fairly significant codebase – which happens to have 20+ instances of this pattern spread throughout the application.

#### <abbr title="Too long; didn't read">TL;DR</abbr>
If you don't need the background information on _[Design Patterns](#background-what-are-design-patterns)_ or _[React architecture](#react-architecture)_ principles and you're all caught up with _[React Context](#react-context-provider-and-consumer)_ and _[the useContext hook](#react-usecontext-hook)_, then you can just skip to the meat of the matter in _[The React Provider Pattern](#putting-it-all-together-the-react-provider-pattern)_.

#### Other formats
The React Provider Pattern is also the main topic my talk "Modern React Design Patterns", that I have presented a number of times including at [ReactNYC S03E10](https://beta2.meetup.com/en-AU/ReactNYC/events/267250157/) and [DeveloperWeek Orlando in February 2020](https://sched.co/YF7T).

---

### Background: What are Design Patterns?
The concept of _Design Patterns_ originates in the seminal book, _[Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns)_. This 1994 book published by Addison-Wesley and authored by Gamma, Helm, Johnson and Vlissides, coloqially known as _The Gang of Four_, was a hallmark publication on the state of object-oriented practical software development.

A design pattern is in essence a way of composing units of code to a larger whole that solves a specific purpose in a well-known setup and constuction and with well-established relationship and roles between the different units. The _Factory_ pattern for instance describes how an interface is used to create instances of a single object with different starting configurations - and how implementations of said pattern can then decide on the implementation details irrelevant to the invoker. Design Patterns are also sometimes described as _micro architecture_, as they can be used over and over on many different scales in the same application.

The most important thing to understand about this book is however, that the four authors did not invent anything new. They merely documented the practices they saw present in real-world development teams and gathered the experience of numerous experts across the field. But more importantly, the coined the terms describing these patterns and put them into context. This enabled developers to have a conversation at a more abstract level about how to construct software without instantly going into the nitty-gritty of actual implementation.

The Gang of Four described a total of [23 design patterns](https://en.wikipedia.org/wiki/Design_Patterns#Patterns_by_type), many of whom are still very well-known today. They all came from the world of object-oriented programming, but many are applicable outside of that realm. Concepts such as _Observer_, _Singleton_, _Proxy_ and many others are very well-known in many languages - including JavaScript.

--- 

### React Architecture

One of the main features and main problems of React is the lack of a common structure or architecture. You can organize your app in whatever way makes sense to you. React itself provides a minimal architecture which in fact is based around some of the design patterns described by _The Gang of Four_. React is at its core one big _Observer_, makes heavy use of the _Flyweight_ pattern to minimize memory usage by reusing objects whereever possible, and of course is based on the _State_ pattern (heavily combined with the overall Observer pattern).

But on the outside, React seems very bare-bones. If you `create-react-app` with the default template, you get the tiniest possible react application with only 1 file of business logic and very little guidance as to how you'd add more functionality to this. This in turn leads different people, teams and companies to develop completely different strategies as to how they prefer to use this underlying library to build huge React applications.

That being said, common patterns of course arise, that other developers tend to follow. Both by change, but also by knowledge sharing. Early patterns included idea such as the separation of container vs presentation, higher-order components (HoC's) and for reversing the composition: render props.

But then React Hooks came, and most of those changed. Completely. What used to be common organizational principles are now completed forgotten or even frowned upon. Even the basic minimal interface provided by the core framework in the form of lifecycle methods is now slowly being deprecated in favor of hook-based approaches.

---

### React Context: Provider and Consumer

[React Context](https://reactjs.org/docs/context.html) is a fairly new feature in React, that is surprisingly unknown to many developers, despite its versatility in React architecture. React Context is considered part of "modern React" along with hooks even though React Context actually came out quite a bit before hooks and can be used completely without any hooks[^1].

[^1]: React Context came with [React 16.3](https://reactjs.org/blog/2018/03/29/react-v-16-3.html) in March 2018 whereas hooks came with [React 16.8](https://reactjs.org/blog/2019/02/06/react-v16.8.0.html) in February 2019.

React Context is a method to send information between components within a component tree. Normally information only travels in one direction, from the Context provider to any component consuming said context. It does nothing you cannot do with simple passing of properties, however it allows you to skip all the intermediate components and send information arbitrarily deep in the component tree.

Let's do a very simple example with a user context holding the name of the currently active user:

```jsx
import { createContext  } from 'react';

const UserContext = createContext({});

function Name() {
  return (
    <UserContext.Consumer>
      {({ name }) => (
        <p>Hello, {name}</p>
      )}
    </UserContext.Consumer>
  );
}

function App() {
  const name = 'World';
  const value = { name };

  return (
    <UserContext.Provider value={value}>
      <h1>Welcome</h1>
      <Name />
    </UserContext.Provider>
  )
}

```

Breaking this down, this consists of three main parts:

* `createContext`: Create a new provider than can be both provided and consumed - with a default value if no parent provider exists.
* `SomeContext.Provider`: A React component that provides a new value for any consumer below it in the component tree.
* `SomeContext.Consumer`: A React component expecting a render prop which will be rendered with the current value of the nearest provider in the component tree above it.

This all seems like an incredibly simple construction, but it's immensely powerful and allows for some very complex constructions – especially when combined with hooks, as we'll see in the next section.

Note that a consumer can have no provider – or multiple providers – above it. These are however both fairly uncommon situations, and we'll ignore those for now. In the following sections, we'll always pair a consumer with a single parent provider. But most often many consumers with the same parent provider.

#### Shortcomings of the React Context Consumer

The consumer component above is a little weird to use, as is often the case when using render props. You cannot directly access the values in your component, but have to either move your logic inside the render prop function or pass the values to a child component:

```jsx
// This will not work
function BrokenWelcome() {
  const message = `Hello, ${name}`;
  return (
    <UserContext.Consumer>
      {({ name }) => (
        <p>{message}</p>
      )}
    </UserContext.Consumer>
  );
}

// Moving the message inside the render prop will work:
function NestedWelcome() {
  return (
    <UserContext.Consumer>
      {({ name }) => {
        const message = `Hello, ${name}`;
        return (
          <p>{message}</p>
        );
      }}
    </UserContext.Consumer>
  );
}

// Moving the message inside the render prop will work:
function WelcomeMessage({ name }) {
  const message = `Hello, ${name}`;
  return (
    <p>{message}</p>
  );
}
function ParentWelcome() {
  return (
    <UserContext.Consumer>
      {({ name }) => (
        <WelcomeMessage name={name}>
      )}
    </UserContext.Consumer>
  );
}
```

While the two latter options are of course workable, they're not as clean as you might want them to be. And that's where the `useContext` hook comes to the rescue!

--- 

### React `useContext` hook

In order to more directly access the current value of the nearest provider for a given context, React has supplied us with the very nice hook, `useContext`. It's a hook, that returns the exact same value, as the render prop of a context consumer would be invoked with - and is most often deconstructed as such.

The above example would now simply read:

```jsx
function BestWelcome() {
  const { name } = useContext(UserContext);
  const message = `Hello, ${name}`;
  return (
    <p>{message}</p>
  );
}
```

How's that for a nice component? Easy to read, no unnecessery prop drilling, just clean simple code.

The beauty of hooks then provides us with the built-in logic, that whenever the nearest context value changes, every hook referencing that context is re-rendered – in turn forcing every component using those hooks to be re-rendered.

This has some problems of its own of course - what if you only use the `name` property of the `UserContext` and someone changes the `image` property? Then unfortunately your component will re-render too, but there's of course ways to fix that. We'll get back to those later.

#### Combining React Context and React Hooks

Let's go back to our previous example of a `UserContext`. This time, let's make our provider a bit more dynamic. Now the name can actually change:

```jsx
import { createContext, useState } from 'react';

const UserContext = createContext({});

function App() {
  const [name, setName] = useState('World');
  const value = { name, setName };

  return (
    <UserContext.Provider value={value}>
      <h1>Welcome</h1>
      <ShowName />
      <EditName />
    </UserContext.Provider>
  )
}
```

Now, `ShowName` is as before, but `EditName` will now use the newly provided `setName` property:

```jsx
function EditName() {
  const { setName } = useContext(UserContext);
  const handleClick = () => setName(prompt('What is your name?'));
  return (
    <button onClick={handleClick} />
  );
}
```

And we can even see this [very simple application](https://codepen.io/barklund/pen/XWXRKpN) (with a sprinkling of Material UI) in action here:

{{% pen id="XWXRKpN" %}}

We now have our first fully working dynamic React Context!

--- 

### Putting it all together: The React Provider Pattern

Going from the above to what I consider to be the React Provider Pattern is merely an act of reorganising all the parts into a more organised structure.

This structure always consists of the same 3 parts (4 if you count an index file) put inside its own package:

1. `folder/context.js` – the file defining the context variable used by the structure
1. `folder/provider.js` – the (main) context provider wrapping whatever children its given with a dynamic context
1. `folder/use.js` – a custom hook giving components access to the current context value

The last requirement is to use a structured approach to the context value in order to increase familiarity and reuse.

Many patterns are possible here, but a very simple structure has proven to serve almost all relevant cases so far with very few exceptions:

```js
const value = {
  state: {
    // Here goes values, that are to be considered immutable
    // Immer.js can be used to ensure immutability is obeyed
  },
  actions: {
    // Here goes functions, that manipulate the above values
    // or return values based on the context internals
  },
};
```

Putting all this together, we get this codebase for our very simple `UserProvider`:

```jsx
// File: user/context.js
import { createContext } from 'react';
const UserContext = createContext({state: {}, actions: {}});

// File: user/use.js
import { useContext } from 'react';
import UserContext from './context';
function useUser() {
  return useContext(UserContext);
}

// File: user/provider.js
function UserProvider({ children }) {
  const [name, setName] = useState('World');
  const value = {
    state: { name },
    actions: { setName },
  };
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  )
}
```

And if we want to use it in our app, we will do it as follows (with the same functionality as before):


```jsx
// File: app.js
import UserProvider from './user/provider';
import ShowName from './show';
import EditName from './edit';
function App() {
  return (
    <UserProvider>
      <h1>Welcome</h1>
      <ShowName />
      <EditName />
    </UserProvider>
  )
}

// File: show.js
import useUser from './user/use';
function ShowName() {
  const { state: { name } } = useUser();
  return <p>Hello, {name}</p>;
}

// File: edit.js
import useUser from './user/use';
function EditName() {
  const { actions: { setName } } = useUser();
  const handleClick = () => setName(prompt('What is your name?'));
  return (
    <button onClick={handleClick} />
  );
}
```

And again, we can see this in action, in this live example. On the surface the example is completely identical to before, but now the context is internalised behind the provider and the custom hook resulting in a much cleaner interface between the provider and the rest of the application:

{{% pen id="RwrVRQe" %}}

--- 

### Examples: How to use the React Provider Pattern

#### Application-wide storage

This pattern can be used as application-wide storage in lieu of other libraries such as redux.

Imagine the classic todo application. Our provider has a state, which is the list of todos, and a couple of functions for adding, removing and marking todos as complete:

```jsx
function TodoProvider({children}) {
  // Start with an empty list
  const [todos, setTodos] = useState([]);

  // When adding a todo, simply append to list:
  const add = useCallback((title) => setTodos(
    (old) => [...old, {title, isDone: false}]
  ), []);

  // Remove a todo by offset in list
  const remove = useCallback((offset) => setTodos(
    (old) => [
      ...old.slice(0, offset,
      ...old.slice(offset + 1),
    ]
  ), []);

  // Mark a todo by offset and isDone flag
  const mark = useCallback((offset, isDone) => setTodos(
    (old) => [
      ...old.slice(0, offset,
      {
        ...old[offset],
        isDone,
      },
      ...old.slice(offset + 1),
    ]
  ), []);

  const value = {
    state: { todos},
    actions: { add, remove, mark },
  };

  return (
    <TodoContext.Provider value={value}>
      {children}
    </TodoContext.Provider>
  )
}
```

Note the use of `useCallback` above. It ensures minimal re-renders of components using this context, as the function is guaranteed to be stable (as its memoized without dependencies). If you don't fully grasp this, don't worry. It's a whole separate and deeply complex topic for a future article.

Furthermore note, that the above provider would actually be a good use-case for `useReducer` – but that's also a topic for another time.

This usage can be found in the `google/web-stories-wp` codebase in the [history provider](https://github.com/google/web-stories-wp/blob/master/assets/src/edit-story/app/history/historyProvider.js) and the [story provider] (https://github.com/google/web-stories-wp/blob/master/assets/src/edit-story/app/story/storyProvider.js) - each a provider of different levels of application-wide storage and each with their own associated states and actions.

Note how both these are applied – among many other providers – in [a very specific order](https://github.com/google/web-stories-wp/blob/master/assets/src/edit-story/app/index.js#L52-L83) in the main application file.

#### UI state

This pattern can be used as component-local storage for UI state - a use that redux handles rather poorly in my opinion.

Imagine a tabbed interface - you want to remember which tab is currently active, but there's no need to bring in some big library to fuel this. It can be done very simple and elegantly with the provider pattern:

```jsx
function TabProvider({children}) {
  const [current, setCurrent] = useState('INFO');
  const value = {
    state: { current },
    actions: { setCurrent },
  };

  return (
    <TabContext.Provider value={value}>
      {children}
    </TabContext.Provider>
  )
}
```

This is almost too simply to be true, but it can actually provide value to the codebase even in a such a distilled example. Because it will often be the case, that you'll add more functionality over time, and then you already have a good abstraction to handle this. Maybe you want the old panel to collapse in an animation before switching to the new one. Or maybe you have some data that requires loading before showing the tabs at all.

Such usage can be found in the `google/web-stories-wp` codebase in the [inspector provider](https://github.com/google/web-stories-wp/blob/master/assets/src/edit-story/components/inspector/inspectorProvider.js) (which provides current tab among a few other minor things relevant for the inspector panels) and the [panel](https://github.com/google/web-stories-wp/blob/master/assets/src/edit-story/components/panels/panel/panel.js) – a small provider around each panel that remembers if the panel is collapsed or not, if resized or not and if so, to what size.

---

### FAQ on the React Provider Pattern

#### Why does the context have its own file?

The reason that the context lives in its own file is three-fold:

1. It allows context reuse - you might actually end up creating multiple providers for the same context (though it's rare).
1. It can be used to mock the provider in testing
1. It reduces the codebase in the rare case your don't need the provider in a given bundle

#### Is the context even necessary

Well, no. Note that the actual context (the variable created in the `context.js` file) is an internal implementation detail of the user provider – and it could technically be swapped for any other technology resulting in similar functionality such as `redux`.

#### What about all those re-renders?

To avoid unneccessary re-renders when unrelated context values change, please use something akin to [`use-context-selector`](https://github.com/dai-shi/use-context-selector), which memoizes the actually used variables from the context and thus only re-renders your components, if the used properties change.

Their examples are a bit crude, but it's fairly simple to combine with the above pattern.

#### So is `redux` just garbage now? Should I just drown it like a lost kitten? Should I chuck it on top of the local tire fire?

First off, easy now – calm down!

Secondly: no, most definitely not. Redux (in particular when combined with `redux-toolkit`) has many functionalities outside of this – and redux actually uses React Context under the hood these days. It's basically this pattern on steroids with many years of optimisation. It does what it does (application-wide storage) very well, but it might be overkill in a smaller (or even medium-sized) applications.