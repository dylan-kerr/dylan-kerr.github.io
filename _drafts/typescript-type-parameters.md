---
title: Function Parameter - Type Parameter Analogies in TypeScript
---

```ts
// Base
type Example<AllowNumber extends boolean> =
    | string
    | (AllowNumber extends true ? number : never);

// Optional
type Example<AllowNumber extends boolean = false> =
    | string
    | (AllowNumber extends true ? number : never);
// Like JS, optional parameters must come last.

// Named
type Example<T extends { AllowNumber: boolean }> =
    | string
    | (T['AllowNumber'] extends true ? number : never);

// Named and Optional
type Example<T extends { AllowNumber?: boolean }> =
    | string
    | (T['AllowNumber'] extends true ? number : never);

// Named and extra Optional
type Example<T extends { AllowNumber?: boolean } = {}> =
    | string
    | (T['AllowNumber'] extends true ? number : never);

// Pass flags/enum values?
```
