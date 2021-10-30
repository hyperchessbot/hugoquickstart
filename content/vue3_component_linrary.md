---
title: "Vue 3 component library with typescript, rollup and composition API"
date: "2021-10-29T10:19:58+01:00"
draft: true
---

## Vue 3 component library with typescript, rollup and composition API

https://github.com/hyperbotauthor/vue3complib

## Motivation

Equip usual widgets ( inputs, tabpane ) with persistence, that is storing widget state in lcoal storage and provide these widgets as an npm package for use in Vue 3 apps.

## Acknowledgments

This project was inspired by https://www.npmjs.com/package/@adamdehaven/vue-custom-tooltip . This project creates a custom tooltip Vue 3 component using typescript, rollup and the composition API. ( I actually used this component in https://openingtrainer.netlify.app . ) The project is fully featured and academic. By fully featured I mean that every detail of implementation is worked out, like checking of properties, documentation, possibility of global component export, or even adding ARIA ( Accessible Rich Internet Application ) label to the component ( https://www.w3.org/TR/wai-aria/ ) which implementations can use to provide acces for disabled people. By academic I mean that it is very disciplined and rigorous, for example in `package.json` where `repository` field is usually just a string of the url, in this project it is formed the most detailed way, as an object with with type and url fields. So ne cheap shortcuts, everything is according to 'book'.

## Project setup

We start from creating a Vue 3 cli-service app. This is useful for testing the components, and also Vue CLI sets up this porject with useful defaults for `tsconfig.json`, linting etc.

We need a `.ts` file for each component, these will be stored in `src/compontents` and an `index.ts`, the entry point of our bundle, which will import all these components, then export them.

For rollup we need typescript support, this is provided by `rollup-plugin-typescript2`, which is a better version of the original rollup typescript plugin. Useful because unlike its predecessor, it generates type declarations files, once you set `declaration: true` in `tsconfig.json`'s `compilerOptions`.

For output format we use `umd` ( Universal Module Definition ) which is AMD, CommonJS and IIFE all in one.

To bundle `.vue` files we need `rollup-plugin-vue`, and for css `rollup-plugin-postcss` ( this will run into problems if you not install `postcss` as well ). Of course typescript does not know about `.vue` files, so we declare them in `shims-vue.d.ts` like

```typescript
/* eslint-disable */
declare module "*.vue" {
  import type { DefineComponent } from "vue";
  const component: DefineComponent<{}, {}, any>;
  export default component;
}
```

`shims-vue` is just an arbitrary name, but conventional as long as it refers to being a shim for `.vue` files ( shim is a small piece of software that fits between two layers that communicate with each other, typically used to adapt one interface to another ).

Having bundling in place we can get down to creating the components.

## Creating components

## Options API

This is what you are used to, you have a `<template>` and then add functionality to this in the `<script>`. It is perfectly possible to do this, and I used this technique in `Perstext` component, just to demonstrate that it is possible.

## Composition API

This is the case when you create your component's DOM tree programatically in a render function without having any HTML template. This article https://vuejsdevelopers.com/2020/02/17/vue-composition-api-when-to-use/ argues that composition API can be useful for even simple components for several reasons. However I would say that even if it does not offer any advantage, it is a cleaner thing than messing with an HTML template. If you want something quickly going and reusability, maintainability etc. is not an issue, then templates are your friend. But for a library, you should better take the pains of using the composition API, for the functionality of your code is more transparent this way. Just to give you an example, in composition you can write within the component definition something like:

```typescript
onClick: () => {
    setLocal(props.id, {
        selectedTab: i,
    });
    setSelected();
    emit("tabpanechanged", {
        event: "tabpanechanged",
        id: props.id,
        selectedTab: i,
        selectedTabCaption: tabs[i],
    });
},
```

while with templates you create a method, can call it from the template, so in the template it is not readily evident what you are aiming for. Having the component and the handler in once place helps readability and removes the need of named handlers, they can be inlined.

### Mindset of composition API

You have to change your mindset for the composition API, because `this` is no longer available. Upon invoking the render function you don't get a refernce to `this`, so things like `this.$refs` or `this.$emit` won't work. You have to find the equivalent of all options API features without using `this`.

The core is the `setup` function ( https://v3.vuejs.org/guide/composition-api-setup.html#setup ) which takes as paramters the component `props` ( you already know what that is ) and the `context`. This latter has your `slots`, provides an `emit` function etc.

It is not well documented what these actually mean.

For example just looking at `slots` you would expect an array of the child nodes of your component. However you have to invoke a function `default()` on `slots` and this will return an array, but still not of the nodes, but of more complex objects, of which you need the `el` ( element ) property. This will finally have the `props` of your slot.

The same goes for refs. You create a ref by invoking vue's `ref` function. As argument it takes a Javascript value. But for the purposes of referring to elements what value it holds is not important, this is why we just use `ref(0)`. The important part is that if you add a ref created this way to a component's attributes and store it in some variable, then later you can use this variable to access the HTML DOM element, by the ref's `._rawValue` property.

You create an element with vue's `h` function, which takes the HTML tag name, the extended attributes ( which include event handlers ) and an array of its child nodes, which child nodes can also be constructed with this function.

Finally you have to take care of hooks somehow. For this you can import hook functions from vue, the most important of which is `onMounted`. Any function you pass to this funcion will be executed upon mount.

Once you have the right mindset, some things become more straightforward.

For example generating nodes from an array is now a question of using a map, that maps from the array to an h function generated node. This is more clear and straightforward, then to set the array up somewhere else and send it to the template which would iterate it with quite messy and verbose syntax.

Here is a complete example of a component with the composition API:

```typescript
<script>
import { defineComponent, h, ref, onMounted } from "vue";

import { setLocal, getLocal } from "../utils.ts";

export default defineComponent({
  name: "Perscheck",
  props: {
    id: {
      type: String,
      required: true,
    },
    default: {
      type: Boolean,
      default: false,
    },
  },
  setup(props, { emit }) {
    const cref = ref(0);

    onMounted(() => {
      const checked = getLocal(props.id, props.default);
      cref._rawValue.checked = checked;
    });

    return () => {
      return h(
        "input",
        {
          type: "checkbox",
          ref: cref,
          id: props.id,
          onChange: (event) => {
            const checked = event.target.checked;
            setLocal(props.id, checked);
            emit("perscheckchanged", {
              event: "perscheckchanged",
              id: props.id,
              checked,
            });
          },
        },
        []
      );
    };
  },
});
</script>

<style></style>
```

If you wonder about complications with slots, getting props of slots and dynamically setting visibility of them, here is the Tabpane component:

```typescript
<script>
import { defineComponent, h, onMounted, ref } from "vue";

import { setLocal, getLocal } from "../utils.ts";

export default defineComponent({
  name: "Tabpane",
  props: {
    id: {
      type: String,
      required: true,
    },
    width: {
      type: Number,
      default: 600,
    },
    height: {
      type: Number,
      default: 400,
    },
  },
  setup(props, { slots, emit }) {
    const slotElements = slots.default();

    const tabs = slotElements.map((e, i) =>
      e.props && e.props.caption ? e.props.caption : `Tab ${i}`
    );

    const tabRefs = tabs.map((_) => ref(0));

    const setSelected = () => {
      const selectedTab = getLocal(props.id, { selectedTab: 0 }).selectedTab;

      tabRefs.forEach((tabRef, i) => {
        const e = tabRef._rawValue;

        e.classList.remove("selected");

        if (i === selectedTab) e.classList.add("selected");

        const s = slotElements[i].el;

        s.classList.add("slot");

        s.classList.remove("active");

        if (i === selectedTab) s.classList.add("active");
      });
    };

    onMounted(() => {
      setSelected();
    });

    return () => {
      const tabDivs = h(
        "div",
        {
          class: ["tabs"],
        },
        tabs.map((tab, i) =>
          h(
            "div",
            {
              ref: tabRefs[i],
              class: ["tab"],
              onClick: () => {
                setLocal(props.id, {
                  selectedTab: i,
                });
                setSelected();
                emit("tabpanechanged", {
                  event: "tabpanechanged",
                  id: props.id,
                  selectedTab: i,
                  selectedTabCaption: tabs[i],
                });
              },
            },
            [tab]
          )
        )
      );

      const contentDiv = h(
        "div",
        {
          style: `width: ${props.width}px; height: ${props.height}px;`,
          class: ["slotscont"],
        },
        slots
      );

      const vertCont = h(
        "div",
        {
          class: ["vertcont"],
        },
        [tabDivs, contentDiv]
      );

      return h(
        "div",
        {
          class: ["tabpane"],
          id: props.id,
        },
        vertCont
      );
    };
  },
});
</script>

<style>
.tabppane {
  border: solid 1px;
}
.tabpane .tab.selected {
  background-color: #ffa;
  border: solid 3px #0f0 !important;
}
.tabpane .tabs .tab {
  padding: 10px;
  margin: 2px;
  border: solid 1px #777;
  cursor: pointer;
  user-select: none;
}
.tabpane .tabs {
  display: flex;
  align-items: center;
  border: solid 1px #777;
}
.tabpane .vertcont {
  display: flex;
  flex-direction: column;
}
.slotscont {
  position: relative;
  overflow-y: scroll;
  padding: 5px;
  border: solid 1px #777;
}
.slot {
  visibility: hidden;
  position: absolute;
}
.slot.active {
  visibility: visible;
}
</style>
```

Here again using Javascript native `classList` is more clean than having to construct class lists as a string and try to work them into the template. May be setting paramterized style like `width: ${props.width}px; height: ${props.height}px;` is a bit ad hoc, one could use css vars, but I think for simple case like this you would pay too high a prize by resorting to css vars. The inspirational project does in fact use css global vars for this.