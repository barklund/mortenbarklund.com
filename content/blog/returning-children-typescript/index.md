---
title: "Returning children in React + Typescript"
date: 2023-01-05T20:00:00+01:00
weight: 100
covercaption: "Sometimes you need to return children without modifying them at all. Sounds familiar?"
coveralt: "A child in a cardboard box"
author: barklund
categories: ["how-to"]
tags: ["react", "typescript"]
---

In this blog post, we'll go over the topic of returning the `children` property directly from a React component discussing why you would want to do that, what issues you can run into when introducing TypeScript, and how we can get around it.

---

Let's say we're tasked with creating a React component to display a numbered list of pages for a pagination result. You know, those underlined numbers below search results with 1, 2, 3, etc. However, we don't want the current page to be a link, only the other pages in the list.

That seems like an easy exercise. We are going to need two components, one for the list, `PageLinkList`, and one for each number in it, `PageLink`.

Let's do so in plain JavaScript and React to start with.

```jsx
function PageLinkList({ numPages, currentPage }) {
  // Create an array like [1,2,3,...,numPages].
  const pageLinks = Array.from({ length: numPages }).map((v, k) => k);
  return (
    <nav>
      {pageLinks.map((page) =>
        <PageLink key={page} link={`?page=${page}`} isCurrent={page === currentPage}>{page+1}</PageLink>
      )}
    </nav>
  );
}
function PageLink({ isLink, link, children }) {
  return isLink ? <a href={link}>{children}</a> : children;
}
```

This all seems reasonable, and we can use it to create a nice list of links like this:

```jsx
function App() {
  return (
    <main>
      <PageLinkList numPages={7} currentPage={2} />
    </main>
  );
}
```

---

But we want to use TypeScript of course. React, like any other JavaScript project, becomes so much more pleasant to work with using a nicely typed language such as TypeScript.

So, let's add some types. The most obvious solution is to do this:

```ts
import { PropsWithChildren } from "react";
function PageLink({
  isLink,
  link,
  children
}: PropsWithChildren<{ isLink: boolean; link: string }>) {
  return isLink ? <a href={link}>{children}</a> : children;
}
function PageLinkList({
  numPages,
  currentPage
}: {
  numPages: number;
  currentPage: number;
}) {
  // Create an array like [0,1,2,...,numPages-1].
  const pageLinks = Array.from({ length: numPages }).map((v, k) => k);
  return (
    <nav>
      {pageLinks.map((page) => (
        <PageLink
          key={page}
          link={`?page=${page}`}
          isLink={page !== currentPage}
        >
          {page + 1}
        </PageLink>
      ))}
    </nav>
  );
}
```

That all looks pretty straight-forward. We type the `<PageLink />` component using the `React.PropsWithChildren` so we can allow any type of child or list of children to be passed to it, and we type all other properties using their primitives.

But this doesn't work. If you do this, you will see this error where we use the `<PageLink />` component:

```
'PageLink' cannot be used as a JSX component.
  Its return type 'ReactNode' is not a valid JSX element.
    Type 'undefined' is not assignable to type 'Element | null'.ts(27
```

Why is that? Well, that is because we're returning the `children` prop directly under some circumstances. And that prop does not qualify as a proper return value.

That is because the `PropsWithChildren` interface types the `children` property as a ReactNode, which in turn is defined as:

```ts
type ReactNode = ReactChild | ReactFragment | ReactPortal | boolean | null | undefined
```

Note that this includes `undefined`. Components aren't allowed to return `undefined`, only `null` in case they shouldn't render anything.

So what can we do about that?

---

There are three posible fixes.

1. We can define the `children` property in a way, where it doesn't allow `undefined`.
2. We can return `null` in case the `children` property is  `undefined` and we want to return it as-is.
3. We can make sure to wrap the `children` property in a valid JSX element.

Option 1 is a bit tricky to do. It would require manually defining the type for `children` which isn't very forwards-compatible. When we use the built-in `PropsWithChildren` interface, we're guaranteed it follows the internal type system of React going forwards, but if we circumvent that, we can't be sure it still does that in the future.

Options 2 seems a lot more approachable. We can just make sure to return `null` in case `children` is `undefined`, quite easily like so:

```ts
  return isLink ? <a href={link}>{children}</a> : (children ?? null);
```

Here we use _nullish coalescing_  to return `null` in case `children` are `null` or `undefined`.

Frustratingly, that **still** doesn't work. Now, we get another TypeScript error:

```
'PageLink' cannot be used as a JSX component.
  Its return type 'string | number | boolean | ReactFragment | Element | null' is not a valid JSX element.
    Type 'string' is not assignable to type 'ReactElement<any, any>'.
```

It turns out, `undefined` isn't the only possible value of the `children` property, that we aren't allowed to return directly from a component.

We could dive further into this and handle the other possible values, but let's turn to option 3 instead. Returning a JSX element no matter what. We can wrap the `children` property in a React fragment like so:

```ts
function PageLink({
  isLink,
  link,
  children
}: PropsWithChildren<{ isLink: boolean; link: string }>) {
  return isLink ? <a href={link}>{children}</a> : <>{children}</>;
}
```

And that's it. This just works with no quirks or anything else required. We return a simple React fragment wrapping the property: `<>{children}</>`.

Why _does_ this work? Well, that's quite simple. The `children` property is typed as _anything that is allowed to be put inside a JSX node_, so we can of course put it inside a fragment node, which is a valid JSX node. And we can return a fragment from a component, because it is a valid JSX node.

You can play around with [this example in TypeScript Playground](https://www.typescriptlang.org/play?#code/JYWwDg9gTgLgBAJQKYEMDGMA0cDecAKUEYAzgOrAwAWAwlcADYAmUSAdnAL5wBmRIcAESt0MQQG4AsACgZPAK5sMwCB3woA5kgAywNgGsAFDhlw4wEroOZTcBnv03pZtPWas2MzgC4CRUhTUdIws7AA8eBZW+r4ARhAQDKhs4nYOviQwUHoaXAB8AJS4tqww8lAcUQ5wAPxwYShwVKw8ALw49gaceTiuIR6cYQD0KHlwvmE9fe7sg0N5UtKcMnKKyqoEmjoOupnGtmzyIOpaJE4u5R4wJ0hevibOcIfHWyS+z7FIUIsXUFc37yOn2+XiKDzMQyGcBoIhgSDgKA4KD+KAAnml9PCANoABkwAEZMAAmTAAOnJmGeNxIAFp8QBdUm2NCqTJwMBbaIkOCtOAAQRRqNJfAgIGMdnYGmogJepy4BVJIBQYEMhgAbth9EVWmMtT84KVyhxDLYzGE2Cg1XlTWZcBytFzFcrVfakNqxibHrbbWEbtEbd6zJjUe1XcsvYGzJ19O0AAY1V2tAAkODDsfDke9VQMoa2cAAhK1eWhLuxrlsM5nrRHI6m8wBqOD4yuR4Z+hzVyMFAot+pDC1W2wFRbh1ZKGAqDh8sAqsElJBlCpwT3esJKvSdwO+zk7CzwKmvdoAdm4Jb+ZZu7SJ3HmNuG67Ym+HXhW0iQAA9ILA4EwkDwUPIDDwNOYCLEAA), or in [the CodeSandbox](https://codesandbox.io/s/returning-children-in-react-typescript-pf51o5?file=/src/App.tsx) that you also see below:

{{% sandbox id="returning-children-in-react-typescript-pf51o5" height="400" %}}

***Note***: If you are using ESLint, you might have a rule prohibiting _"useless"_ fragments (`react/jsx-no-useless-fragment`) , which will (somewhat erroneously) mark this use as a match for that rule.

You can get around that by telling ESLint to ignore this instance:

```ts
function PageLink({
  isLink,
  link,
  children
}: PropsWithChildren<{ isLink: boolean; link: string }>) {
  // eslint-disable-next-line react/jsx-no-useless-fragment -- required for valid return type
  return isLink ? <a href={link}>{children}</a> : <>{children}</>;
}
```

And now even you should be good to go.

To an extent, this _is_ a useless fragment, as it has no practical function, but will actually appear in the resulting JavaScript when the TypeScript is compiled. So it's an extra JavaScript wrapper to make TypeScript happy. This does introduce a tiny overhead, but that's probably negligible in almost all applications.

---

Returning the `children` property directly from a React component is sometimes a useful tool, but TypeScript makes it a bit tricky to do. The solution however is surprisingly simple with only the tiniest of downsides.

If you need to return the `children` property directly in a React component when using TypeScript, make sure to wrap it in a fragment like so: `<>{children}</>`.