---
title: React fundamentals
date: 2021-11-06 21:21:49
author: Sinner Joe
tags: React
---

One of the most widely used front-end libraries nowadays is React. Many people have this library on their CV's but rarely
do they have an understanding of how React works under the covers. It indeed has a lot of features and implementation features (I'm looking at you, React Fiber) that
leave you with more questions than answers. Oftentime we settle on the fact that it is just magic, as Arthur C. Clarke said _"Any sufficiently advanced technology is indistinguishable from magic."_.

I cannot boast of exact knowledge of the implementation. I'm only armed with my practical experience of **React** and some implementation of **Preact** which tries to mimic **React** interfaces and implementation, albeit at the same time being much more compact.

## React as a library

Before discussing how **React** works at lower level we have to understand why it is called a library.
React developers decided to split the module that is responsible for the virtual DOM, other functions that
we routinely use from (ex: hooks) and diffing from the module that applies the changes to the **DOM** that determines what the browser will render on the page.

As a result 2 npm packages were created:

- `react` - the former module, which we use directly by calling it's code throughout our components.
- `react-dom` - the later module, responsible for changing the **DOM** by using the browser's **API**. This module is told what to do from the diffs provided from the
  `react` module. There are other libraries at the level of rendering like
  `react-native`, `pdf-react`. Let's call this part of the functionality the _Rendering Layer_.

## Virtual DOM

I feel like every article on **React** should include a discussion about virtual DOM(shortly **VDOM**). **VDOM** is a tree of plain **JS** objects
which is used for the purpose of diffing the changes between successive version of the **VDOM** tree instances.

### Here are some examples of how VDOM is represented in different ways

**VDOM object tree example**

```js
{
  type: "div",
  key: null,
  ref: null,
  props: {
    className: "disclaimer",
    children: [
      {
        type: "span",
        key: null,
        ref: null,
        props: {
          children: "Some label"
        }
      },
      {
        type: "a",
        key: null,
        ref: null,
        props: {
          href: "https://www.google.com",
          children: "Check out this URL"
        }
      },
      {
        type: {
          render: function Button(props, ref) {/* component implementation */}
        },
        key: null,
        ref: null,
        props: {
          ghost: true,
          children: "Click on me"
        }

      }
    ]
  }
}
```

**VDOM written using JSX/TSX language extension**

```xml
  <div className="disclaimer">
      <span>
        Some label
      </span>
      <a href="https://www.google.com">
        Check out this URL
      </a>
      <Button ghost>Click on me</Button>
  </div>
```

**The JS code that JSX is compiled into**

```js
React.createElement(
  "div",
  {
    className: "disclaimer",
  },
  React.createElement("span", {}, "Do you want an unmaintainable mess?"),
  React.createElement(
    "a",
    {
      href: "https://www.google.com",
    },
    "Check out this URL"
  ),
  React.createElement(
    Button, // the function that creates the button
    {
      ghost: true,
    },
    "Click on me"
  )
);
```

### Why do we need this extra step at all?

Originally react was created to be used in browsers. Browsers need to redraw the picture of the page every time some changes are made to the page.
Modern browsers are pretty fast, they now use **GPU** acceleration to optimize the drawing of elements modified by the **CSS** property _transform_
and when scrolling (since it's the same "texture" always loaded in the graphic memory). Unfortunately the majority of changes that involve **DOM** node manipulations need to be processed by the **CPU** every time. The more of those changes and the bigger they are, the more the **CPU** will be stuck reflowing the page. **React** can tell exactly what needs to be updated in the UI so that the **DOM** nodes previously created can be reused with some patches, when possible and also it can batch the changes together to avoid multiple reflows of the page.

### Isn't maintaining a virtual DOM expensive in memory and execution time?

It has an effect on performance, but thankfully JS engines evolved to be very fast. Creating object trees and then finding the differences between them is faster than wastefully creating new **DOM** nodes on every change or reflowing the page multiple times. There are some people that don't
aggree with the tradeoffs **VDOM** makes and they have implemented UI frameworks that function without them.

## React Diffing algorithm
