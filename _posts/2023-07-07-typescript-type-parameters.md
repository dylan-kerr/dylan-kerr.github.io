---
title: Function Parameter - Type Parameter Analogies in TypeScript
---

In TypeScript's type language, type parameters for generic types and functions are often introduced as similar to
function parameters in the value language (i.e. the part of TypeScript that gets compiled to JavaScript). I think the
similarities run deeper than they might appear at first glance, with many of the variations of function parameters
having analogous constructions for type parameters. I'd like to explore some of the similarities (and differences) here.
Some of them might even be useful.

For each analogy, I've tried to implement the same idea (to the extent possible) as a function in the value language and
a generic type in the type language, with examples of each being called or used. I've used a boolean parameter as a
simple example, but the techniques apply to any parameter type.

- toc
{:toc}

## Normal parameters

By "normal" I mean regular positional parameters with no default. The value and type language are almost identical here.

```ts
const example = (toggle: boolean) => toggle;
assert.equal(true, example(true));

type Example<Toggle extends boolean> = Toggle;
true satisfies Example<true>;
```

## Optional parameters

All function parameters are in some sense optional in JavaScript, so the term default parameters would be more accurate,
but TypeScript enforces that non-default parameters are supplied so the two are equivalent in this context.

The definition syntax is very similar between the value and type language, including the constraint that optional
parameters must come after all required parameters. The biggest difference is at the point of use: an empty parameter
list must be provided in the value language, but must be omitted in the type language.

```ts
const example = (toggle: boolean = false) => toggle;
assert.equal(true, example(true));
assert.equal(false, example());

type Example<Toggle extends boolean = false> = Toggle;
true satisfies Example<true>;
false satisfies Example;
```

## Named parameters

JavaScript does not have true named parameters, but they can be approximated by passing an object and treating the
property keys as parameter names. This gives much of the same benefit of making the parameters clearer at the point of
use, and the exact same pattern is possible in the type language.

A key difference here is that the type language does not have an equivalent of object destructuring, so reading the
parameters must be done through a named container. I don't see any immediate reason that destructuring couldn't be
implemented in the type language in future.

Note: there is a [stage 0 proposal](https://github.com/samuelgoto/proposal-named-parameters) to add true named
parameters to JavaScript. If that proposal progresses, it would be interesting to see if similar syntax were also added
to TypeScript's type language.

```ts
const example = ({ toggle }: { toggle: boolean }) => toggle;
assert.equal(true, example({ toggle: true }));

type Example<T extends { Toggle: boolean }> = T['Toggle'];
true satisfies Example<{ Toggle: true }>;
```

## Optional named parameters

The named parameters approach can be combined with defaults/optional parameters at two levels: the individual properties
can be optional, and the "container" parameter can be optional as a whole. The latter is equivalent between the value
and type languages as it was for positional optional parameters, but the latter is not well supported in the type
language; I'll demonstrate them separately.

```ts
// The named parameter is required, the container is optional.
const example = ({ toggle }: { toggle: boolean } = { toggle: false }) => toggle;
assert.equal(true, example({ toggle: true }));
assert.equal(false, example());

type Example<T extends { Toggle: boolean } = { Toggle: false }> = T['Toggle'];
true satisfies Example<{ Toggle: true }>;
false satisfies Example;

// The named parameter is optional, the container is required.
const example = ({ toggle = false }: { toggle?: boolean }) => toggle;
assert.equal(true, example({ toggle: true }));
assert.equal(false, example({}));

// No way to express the default directly in the parameter declaration.
type Example<T extends { Toggle?: boolean }> = T['Toggle'] extends boolean ? T['Toggle'] : false;
true satisfies Example<{ Toggle: true }>;
false satisfies Example<{}>;

// Combined - the named parameter and container are both optional.
const example = ({ toggle = false }: { toggle?: boolean } = {}) => toggle;
assert.equal(true, example({ toggle: true }));
assert.equal(false, example({}));
assert.equal(false, example());

type Example<T extends { Toggle?: boolean } = {}> = T['Toggle'] extends boolean ? T['Toggle'] : false;
true satisfies Example<{ Toggle: true }>;
false satisfies Example<{}>;
false satisfies Example;
```

## Conclusion

Type parameters in TypeScript are indeed very similar to function parameters, but they haven't quite reached feature
parity. Object destructuring in the type language would go a long way to bridge the gap and allow more of the familiar
patterns from the value language to be used in the type language.

Did I miss any function parameter variations? Raise an issue or pull request against
[this site's repository](https://github.com/dylan-kerr/dylan-kerr.github.io) to let me know.
