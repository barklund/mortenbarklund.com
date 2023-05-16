---
title: "Spreading props correctly with React + TypeScript"
date: 2023-01-04T21:00:00+01:00
weight: 50
covercaption: "Spreading props is like spreading jam - if you do it wrong, you get sticky fingers!"
coveralt: "Jam on toast"
author: barklund
categories: ["how-to"]
tags: ["react", "typescript"]
---

In this blog post, we're going to explore how to create a React component that can be passed any number of properties which will be passed through to some other component. This seems like a really narrow use-case, but it is very common especially when you're creating UI libraries or similar generic components.

> This article is part of my ["Advanced React and TypeScript series"]({{< ref "/blog/advanced-react-typescript" >}} "Advanced React and TypeScript series") - go check out the other articles in the series too!

We will in particular be discussing how to do this in TypeScript and how to correctly type the properties of the component to allow the correct set of properties to be passed in. There actually happens to be 6 different ways to do this in TypeScript and React, so which should you use? And which should you definitely not use?

We'll be going over these steps:

1. How and why we spread props in React
2. The problems that TypeScript introduce
3. The different possible solutions in TypeScript with pros and cons
4. Key takeaways

---

## 1.  How and why we spread props in React

Imagine that you're tasked with creating a generic *input-and-label* component in React named `<InputGroup />`. You want the component to accept a label property with the label text, but besides that, the component will pass any other property passed to it straight on the the `<input />` element. This will allow people to use it as a text input with a placeholder with `<InputGroup label="name" placeholder="E.g. John Doe" />` but also as a required password input with `<InputGroup label="password" type="password" required />`.

How would you create this component? The naive way is of course to list every possible property of the input and passing it on, but that quickly becomes unreasonable:

```jsx
function InputGroup({ label, type, required, placeholder, id }) {
  return (
    <label>
      {label}:
      <input type={type} required={required} placeholder={placeholder} id={id} />
    </label>
  );
}
```

Here we've added just 4 of the attributes that we would need to pass on. And with a total of no less than 250-ish(!) attributes, that's a terrible approach. Instead, we use the spreading operator to just allow any other property to be passed along:

```jsx
function InputGroup({ label, ...rest }) {
  return (
    <label>
      {label}:
      <input {...rest} />
    </label>
  );
}
```

This is a lot smarter!

We can now pass any valid property to the input component, but actually we can also pass any invalid property to it. We can do anything here! So while there's nothing stopping us from doing `<InputGroup label="message" rows="5" />` thinking it's a text area (it's not, inputs cannot have multiple rows), it's allowed and the rows property is added to the input for no purpose.

---

## 2.  The problems that React in TypeScript introduce

When we add TypeScript to the mix, we specify the type of the properties object, so TypeScript can validate, that we only ever pass valid properties to our component.

So for our naive approach before, this would work:

```ts
interface InputGroupProps {
  label: string;
  type?: string;
  required?: boolean;
  placeholder?: string;
  id?: string;
}
function InputGroup({ label, type, required, placeholder, id }: InputGroupProps) {
  return (
    <label>
      {label}:
      <input type={type} required={required} placeholder={placeholder} id={id} />
    </label>
  );
}
```

But this is not a good approach for several reasons. First of all, our types above aren't as precise as the built-in ones. For example, the type property should not just be any string, but only one of a few select strings, that are actually valid input types.

But more importantly, we don't want to enumerate all the properties in neither the component definition, nor the type definition. So we want to only specify the label property in our type, but somehow have the type also include all the normal properties used by an intrinsic input element.

What we want is something like this:

```ts
interface InputGroupProps extends AllPossiblePropsForAnInput {
  label: string;
}
function InputGroup({ label, ...rest }: InputGroupProps) {
  return (
    <label>
      {label}:
      <input {...rest} />
    </label>
  );
}
```

Unfortunately, `AllPossiblePropsForAnInput` doesn't exist because we just made it up. But there are other interfaces!

---

## 3.  Six different solutions with pros and cons
Here are six other interfaces, that we can actually extend from – starting with the best one:

#### a. `ComponentPropsWithoutRef`

React has a built-in type to extract the type of props for a given component (type) excluding any potential ref that the component accepts. It is called `ComponentPropsWithoutRef` and exists on the `React` namespace.

You would normally use it to extract the type of props from some other component, but you can use that same interface to extract the props of the intrinsic HTML types by passing in the string identifier of the HTML element:

```ts
interface InputGroupProps extends React.ComponentPropsWithoutRef<"input"> {
  label: string;
}
```

It is a bit verbose, but you can of course import it from React which makes it slightly shorter:

```ts
import type { ComponentPropsWithoutRef } from 'react';
interface InputGroupProps extends ComponentPropsWithoutRef<"input"> {
  label: string;
}
```

However, there are no other downsides. This interface is pretty straight-forward to use and very easy to change to a different element, e.g. a `<span />` or a `<textarea />`. And it allows all the relevant properties with the narrowest possible attributes.

#### b. `ComponentProps`

Similarly to `ComponentPropsWithoutRef`, there's also the `ComponentProps` type. It works in the same way, except it also includes the `ref` property, which you would never be able to accept as a props parameter. So using `ComponentProps` would allow you to write this:

```ts
import type { ComponentProps } from 'react';
interface InputGroupProps extends ComponentProps<"input"> {
  label: string;
}
function InputGroup({ label, ref, ...rest }: InputGroupProps) {
  return (
    <label>
      {label}:
      <input ref={ref} {...rest} />
    </label>
  );
}
```

This would seem to be valid - at least in TypeScript's eyes - but you would never actually be able to accept a ref like that. You would have to use `forwardRef` to do that.

#### c. `InputHTMLAttributes`

React has another interface, or actually a whole set of interfaces, that you can use. However, they are generic and the generic type indicates the types for the event properties (such as the type of the `onClick` event handler). 

If we were to use this interface, we would have to type:

```ts
import type { InputHTMLAttributes } from 'react';
interface InputGroupProps extends InputHTMLAttributes<HTMLInputElement> {
  label: string;
}
...
```

But what if we had a text area element instead? Well, then the interface to extend is `TextAreaHTMLAttributes<HTMLTextAreaElement>`. Notice how we need to put the name of the HTML element in two different places and the order of the prefixes aren't event identical with one being `ElementHTML*` and the other `HTMLElement*`.

This approach works though and is a perfectly fine solution. But it is quite verbose and requires you to remember a ton of interface names so it's a bit shaky.

#### d. `JSX.IntrinsicElements`

All the intrinsic JSX elements are actually defined in a global namespace, named `JSX`. From here, we can extract the type for any by indexing the `JSX.IntrinsicElements` with the HTML node name, e.g. `JSX.IntrinsicElements['input']`.

But due to a quirk of how TypeScript syntax works, we can't actually extend this. You can't use square brackets with the `extends` keyword for some reason. So we cannot do `interface InputGroupProps extends JSX.IntrinsicElements['input']`. We would have to save it to a temporary type, which we would then use like so:

```ts
type InputProps = JSX.IntrinsicElements['input'];
interface InputGroupProps extends InputProps {
  label: string;
}
```

Furthermore, this also allows the ref property, as with the `ComponentProps` option. So this is not a great option because it allows an invalid prop and it requires an extra type.

#### e. `HTMLAttributes`

Rather than using the element-specific interfaces such as `InputHTMLAttributes` can we instead use the general `React.HTMLAttributes` interface.

But this has two problems. First, it still requires a generic, which is the HTML element interface such as `HTMLAttributes<HTMLInputElement>`. But secondly, and a lot more important, this interface only includes HTML attributes that are applicable to all elements such as `className`, `id`, `style`, etc and not the element-specific ones such as `type` in case of the input element. Remember, the HTML element passed as a generic parameter only defines the types for event handlers, it does not describe any properties directly.

So that's a complete no-go!

#### f. `HTMLProps`

Finally, there's the all-encompassing interface `React.HTMLProps`. It still requires the HTML element interface as a generic argument (`HTMLProps<HTMLInputElement>`), but even worse, this interface allows **all** properties, that are allowed on any single element in HTML. So you can put an `open` property on your input, because `open` is a valid  property on the `<details />` element – and a 100 other properties, that aren't actually applicable to input elements.

And we can't actually use it, because when we try to spread the rest argument to the `<input />`, TypeScript will complain and point out, that it could contain invalid properties not allowed on an input. So this is the worst option by far.

---

## 4.  Key takeaways and final solution

TypeScript is a tremendously useful tool in React, and once you've started using it, you probably won't look back. But some component patterns, that are simple and everyday in pure JavaScript become a lot more complex in TypeScript, but they do also come with complete type safety, so that's a huge bonus.

In this post, we've discussed how to correctly accept spreadable props in a React component using TypeScript. While there are many ways, extending your props from the `ComponentPropsWithoutRef<"component">` is the simplest, safest, and easiest solution, but there are other solutions just as safe, just a bit more verbose and requiring remembering the names of a lot more interfaces.

| Interface | Easy to<br>write | Excludes<br>`ref` | Includes<br>specific<br>properties | Excludes<br>invalid<br>properties |
|-|-|-|-|-|
| `ComponentPropsWithoutRef<"*">` | ✅ | ✅ | ✅ | ✅ | 
| `ComponentProps<"*">` | ✅ | 🚫 | ✅ | ✅ | 
| `*HTMLAttributes<HTML*Element>` | 🤔 | 🚫 | ✅ | ✅ | 
| `JSX.IntrinsicElements["*"]` | 🤬 | 🚫 | ✅ | ✅ | 
| `HTMLAttributes<HTML*Element>` | ✅ | 🚫 | 🚫 | ✅ | 
| `HTMLProps<HTML*Element>` | ✅ | 🚫 | ✅ | 🤯 |

Using this technique, here's how we would implement an `<InputGroup />` component in React and TypeScript with spreadable props served by CodeSandbox:

{{% sandbox id="react-typescript-and-props-spreading-lypxqb" view="editor" height="700" module="%2Fsrc%2FApp.tsx" highlights="3,4,5,7" %}}

And this is the resulting application - looks great, no?

{{% sandbox id="react-typescript-and-props-spreading-lypxqb" view="preview" height="600" %}}

 You can [play around with the sandbox for yourself](https://codesandbox.io/s/react-typescript-and-props-spreading-lypxqb).