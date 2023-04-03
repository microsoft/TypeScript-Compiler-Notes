# Relations

There are 5 different type relations we have in the checker (in order of strictness, i.e. each is a subset of the next):
- Identity
- Strict subtyping
- Subtyping
- Assignability
- Comparability

We use those relations for different things in the checker, and in general we have all those different relations because sometimes we wanna be more strict, and sometimes we don't.

We use **assignability** to check if an argument can be passed to a function, or if an expression can be assigned to a variable:

```ts
declare let x: number;
declare let y: string;
y = x; // Error: Type 'number' is not assignable to type 'string'.
```

When computing the type of an array literal from the types of its elements, we use a different relation: **strict subtyping**. To compute the type of an array literal, we need to compare the type of each array element to one another, removing the types that are a subtype of another element type. To do this comparison between array element types, we use the strict subtyping relation, as seen in the following example:

```ts
interface SomeObj {
    a: string;
}

interface SomeOtherObj {
    a: string;
    b?: number;
}

function yadda(x: SomeObj, y: SomeOtherObj) {
    x = y; // Ok, uses assignability relation
    y = x; // Ok, uses assignability relation

    return [x, y]; // Type inferred after subtype reduction is 'SomeObj[]',
                   // because 'SomeOtherObj' is a strict subtype of 'SomeObj`.
}
```

The strict subtyping relation relates type `X` to type `Y` if `X` is a subtype of `Y`, but it does so in a way that avoids the problem that the regular (non-strict) subtyping relation has (and the assignability relation has as well) where two *different* types can be mutually related to one another[^1]. So we use the strict subtyping when we want to be sure that if `X` is a strict subtype of `Y`, then `Y` isn't a strict subtype of `X`, because otherwise we might end up having weird behavior, e.g. where the declaration order of a program changes the types the checker produces (see [this issue](https://github.com/microsoft/TypeScript/issues/52100), and [this PR](https://github.com/microsoft/TypeScript/pull/52282) for other possible weird consequences).

[^1]: In mathematical terms, we want the strict subtyping relation to be antisymmetric.

The **subtyping** relation also relates type `X` to type `Y` if `X` is a subtype of `Y`, but it has the issue mentioned above. For instance, `any` and `unknown` are both subtypes of each other. However, we use this relation despite the mentioned issue because we had the subtyping relation in the checker *before* we had the strict subtyping relation, so the subtyping relation is kept for backwards compatibility. An example of where we use the subtyping relation is when resolving overloaded signatures. To see why we use subtyping for that purpose, consider the example of type `any`: something of type `any` is assignable to every other type, but `any` is not a subtype of every other type. So in the example:

```ts
declare function myFunc(x: string): string;
declare function myFunc(x: number): number;
declare function myFunc(x: any): any;

declare const a: any;
myFunc(a); // After overload resolution, we infer `any`.
```

When doing overload resolution, we try to match the argument type (`any`) to the parameter type of each overload signature using the subtype relation. Because `any` is not a subtype of `string` or `number`, we pick the last signature. If we used the assignability relation, we'd go match an `any` argument with the parameter type `string` on the first `myFunc` signature, and it would match, because `any` is assignable to `string`.

> **_Note:_** something that can be confusing is how `any` has a different behavior depending on the relation being used to compare it, and how it differs from `unknown`. Basically, `any` is special because it is assignable to every type, but `any` is not a subtype of every type. And every type is assignable to `any`, and every type is a subtype of `any`. So `any` has this special behavior during assignability where it is assignable to **any**thing.
`unknown`, like `any`, is also a supertype of every other type. However, `unknown` doesn't have `any`'s special assignability behavior: everything is assignable to `unknown`, but `unknown` is not assignable to everything.

Another relation we have is **comparability**. We use comparability to check if two types possibly overlap and values of those types can possibly be compared to one another. For example:

```ts
type Color = "red" | "blue" | "green" | "orange";
type Fruit = "apple" | "banana" | "orange";
type Shape = "square" | "rectangle";

function myFoo(color: Color, fruit: Fruit, shape: Shape) {
    color = fruit; // Error: Type 'Fruit' is not assignable to type 'Color'.
    fruit = color; // Error: Type 'Color' is not assignable to type 'Fruit'.
    if (color === fruit) { // Ok, because 'Fruit' and 'Color' are comparable.
        ...
    }
    if (color === shape) { // Error: This comparison appears to be unintentional because the types 'Color' and 'Shape' have no overlap.
        ...
    }
}
```
The types `Color` and `Fruit`, despite not being assignable to each other, are comparable, because if we think of types as sets, they have values in common (in this case, value `"orange"`). However, types `Color` and `Shape` are not comparable, so the checker will complain if you try to compare values of those two types.

As for the **identity** relation, it means to be used as its name implies: to check if two types are **identical**. We use that one more rarely in the checker. One example is when you declare the same `var` multiple times (which is allowed), the checker checks that the declared types are identical among all of the declarations.

Another usage of identity is in inference. When inferring from one type into another, we get rid of the parts of both types that are identical. For example, when inferring from one union to another union, e.g. from `boolean | string | number` to `T | string | number`, we get rid of the identical types on both sides (`string` and `number`) to end up inferring from `boolean` into `T`.
Yet another example is when checking assignability of two instantiations of the same invariant generic type: two instantiations are still related if their type arguments is identical.

> **_Implementation note:_** Most of the times, identity is used for object types, and we try to make it quick to check that by interning that information.
For example, instantiations of the same type with the same arguments produce the same type, so it's fast to check that the instantiated types are identical.
However, we don't intern all identical object types: anonymous object literal types are not interned. So they have distinct types, but they are identical, so we end up having to do a more expensive check to see that they're identical.

If you want to see more about the implementation and use of those relations in the checker, the entry point to using all those relations is the function `isTypeRelatedTo`.