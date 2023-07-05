---
title: 'Common JavaScript Mistake: Optional Chaining'
---

Optional chaining is a very convenient JavaScript feature introduced in ES2020. Dr. Axel has you covered if you need an [introduction](https://2ality.com/2019/07/optional-chaining.html), but as a quick reminder (in TypeScript):

```ts
function example(obj: { prop: string } | undefined): string | undefined {
    return obj?.prop;
}
```

As convenient as it is, it seems to be easy to misunderstand. I've seen the same mistake made by several different people:

```ts
function example(obj: { prop?: string }): string | undefined {
    return obj?.prop;
}
```

The difference is obvious because it's a simple example and the function parameter type declaration is explicit, but this can be hard to spot in the wild: __the optional chaining isn't doing anything__. `obj` is never null or undefined, so `obj.prop` would always be safe (though possibly undefined).

This seems like a harmless error, but there's always a hazard in thinking that code is doing something it isn't - perhaps you wrote `obj?.nested.prop` when you really needed `obj.nested?.prop` and now you get a TypeError at runtime.

I propose a mnemonic to help avoid this:

> The question mark is on the left of the period so it applies to the value on the left.

If that doesn't catch on, use TypeScript and enable the [ESLint rule](https://typescript-eslint.io/rules/no-unnecessary-condition/).
