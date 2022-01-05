### Terminology from inside the codebase

* **Core TypeScript Compiler**

 * **Parser:** Starting from a set of sources, and following the productions of the language grammar, to generate an Abstract Syntax Tree (AST). Also: [see Parser](https://basarat.gitbooks.io/typescript/docs/compiler/parser.html),

 * **Binder:** Linking declarations contributing to the same structure using a Symbol (e.g. different declarations of the same interface or module, or a function and a module with the same name). This allows the type system to reason about these named declarations. [See Binder](https://basarat.gitbooks.io/typescript/docs/compiler/binder.html)
 * **Type resolver/ Checker:** Resolving types of each construct, checking semantic operations and generate diagnostics as appropriate. [See Checker](https://basarat.gitbooks.io/typescript/docs/compiler/checker.html)

 * **Emitter:** Output generated from a set of inputs (.ts and .d.ts) files can be one of: JavaScript (.js), definitions (.d.ts), or source maps (.js.map)

 * **Pre-processor:** The "Compilation Context" refers to all files involved in a "program". The context is created by inspecting all files passed in to the compiler on the command line, in order, and then adding any files they may reference directly or indirectly through `import` statements and `/// <reference path=... />` tags.
The result of walking the reference graph is an ordered list of source files, that constitute the program.
When resolving imports, preference is given to ".ts" files over ".d.ts" files to ensure the most up-to-date files are processed.
The compiler does a node-like process to resolve imports by walking up the directory chain to find a source file with a .ts or .d.ts extension matching the requested import.
Failed import resolution does not result in an error, as an ambient module could be already declared.

* **Standalone compiler (tsc):** The batch compilation CLI. Mainly handle reading and writing files for different supported engines (e.g. Node.js)

* **Language Service:** The "Language Service" exposes an additional layer around the core compiler pipeline that are best suiting editor-like applications.
The language service supports the common set of a typical editor operations like statement completions, signature help, code formatting and outlining, colorization, etc... Basic re-factoring like rename, Debugging interface helpers like validating breakpoints as well as TypeScript-specific features like support of incremental compilation (--watch equivalent on the command-line). The language service is designed to efficiently handle scenarios with files changing over time within a long-lived compilation context; in that sense, the language service provides a slightly different perspective about working with programs and source files from that of the other compiler interfaces.
> Please refer to the [[Using the Language Service API]] page for more details.

* **Standalone Server (tsserver):** The `tsserver` wraps the compiler and services layer, and exposes them through a JSON protocol.
> Please refer to the  [[Standalone Server (tsserver)]] for more details.

### Type stuff which can be see outside the compilers

* Node: The basic building block of the Abstract Syntax Tree (AST). In general node represent non-terminals in the language grammar; some terminals are kept in the tree such as identifiers and literals.

* SourceFile: The AST of a given source file. A SourceFile is itself a Node; it provides an additional set of interfaces to access the raw text of the file, references in the file, the list of identifiers in the file, and mapping from a position in the file to a line and character numbers.

* Program: A collection of SourceFiles and a set of compilation options that represent a compilation unit. The program is the main entry point to the type system and code generation. 

* Symbol: A named declaration. Symbols are created as a result of binding. Symbols connect declaration nodes in the tree to other declarations contributing to the same entity. Symbols are the basic building block of the semantic system. 

* `Type`: Types are the other part of the semantic system. Types can be named (e.g. classes and interfaces), or anonymous (e.g. object types). 

* Signature``: There are three types of signatures in the language: call, construct and index signatures.

- `Transient Symbol` - A symbol created in the checker, as opposed to in the binder
- `Freshness` - When a literal type is first created and not expanded by hitting a mutable location, see [Widening
  and Narrowing in TypeScript][wnn].

- `Expando` - This is [the term](https://developer.mozilla.org/en-US/docs/Glossary/Expando) used to describe taking a JS object and adding new things to it which expands the type's shape

  ```js
  function doSomething() {}
  doSomething.doSomethingElse = () => {}
  ```
  
  In TS, this is only allowed for adding properties to functions. In JS, this is a normal pattern in old school code for all kinds of objects. TypeScript will augment the types for `doSomething` to add `doSomethingElse` in the type system in both.


- `Structural Type System` - A school of types system where the way types are compared is via the structure of
  their properties.

  For example:

  ```ts
  interface Duck {
    hasBeak: boolean;
    flap: () => void;
  }

  interface Bird {
    hasBeak: boolean;
    flap: () => void;
  }
  ```

  These two are the exact same inside TypeScript. The basic rule for TypeScript’s structural type system is that
  `x` is compatible with `y` if `y` has at least the same members as `x`.

* `Literal` - A literal type is a type that only has a single value, e.g. `true`, `1`, `"abc"`, `undefined`.

  For immutable objects, TypeScript creates a literal type which is the value. For mutable objects TypeScript
  uses the general type that the literal matches. See [#10676](https://github.com/Microsoft/TypeScript/pull/10676)
  for a longer explanation.

  ```ts
  // The types are the literal:
  const c1 = 1; // Type 1
  const c2 = c1; // Type 1
  const c3 = "abc"; // Type "abc"
  const c4 = true; // Type true
  const c5 = c4 ? 1 : "abc"; // Type 1 | "abc"

  // The types are the class of the literal, because let allows it to change
  let v1 = 1; // Type number
  let v2 = c2; // Type number
  let v3 = c3; // Type string
  let v4 = c4; // Type boolean
  let v5 = c5; // Type number | string
  ```

  Literal types are sometimes called unit types, because they have only one ("unit") value.

- `Control Flow Analysis` - using the natural branching and execution path of code to change the types at
  different locations in your source code by static analysis.

  ```ts
  type Bird = { color: string, flaps: true };
  type Tiger = { color: string, stripes: true };
  declare animal: Bird | Tiger

  if ("stripes" in animal) {
    // Inside here animal is only a tiger, because TS could figure out that
    // the only way you could get here is when animal is a tiger and not a bird
  }
  ```

- `Generics` - A way to have variables inside a type system.

  ```ts
  function first(array: any[]): any {
    return array[0];
  }
  ```

  You want to be able to pass a variable type into this function, so you annotate the function with angle brackets
  and a _type parameter_:

  ```ts
  function first<T>(array: T[]): T {
    return array[0];
  }
  ```

  This means the return type of `first` is the same as the type of the array elements passed in. (These can start
  looking very complicated over time, but the principle is the same; it just looks more complicated because of the
  single letter.) Generic functions should always use their type parameters in more than one position (e.g. above,
  `T` is used both in the type of the `array` parameter and in the function’s return type). This is the heart of
  what makes generics useful—they can specify a _relationship_ between two types (e.g., a function’s output is the
  same as input, or a function’s two inputs are the same type). If a generic only uses its type parameter once, it
  doesn’t actually need to be generic at all, and indeed some linters will warn that it’s a _useless generic_.

  Type parameters can usually be inferred from function arguments when calling generics:

  ```ts
  first([1, 2, 3]); // 'T' is inferred as 'number'
  ```

  It’s also possible to specify them explicitly, but it’s preferable to let inference work when possible:

  ```ts
  first<string>(["a", "b", "c"]);
  ```

* `Outer type parameter` - A type parameter declared in a parent generic construct:

  ```ts
  class Parent<T> {
    method<U>(x: T, y: U): U {
      // 'T' is an *outer* type parameter of 'method'
      // 'U' is a *local* type parameter of 'method'
    }
  }
  ```

* `Narrowing` - Taking a union of types and reducing it to fewer options.

  A great case is when using `--strictNullCheck` when using control flow analysis

  ```ts
  // I have a dog here, or I don't
  declare const myDog: Dog | undefined;

  // Outside the if, myDog = Dog | undefined
  if (dog) {
    // Inside the if, myDog = Dog
    // because the type union was narrowed via the if statement
    dog.bark();
  }
  ```

- `Expanding` - The opposite of narrowing, taking a type and converting it to have more potential values.

```ts
const helloWorld = "Hello World"; // Type; "Hello World"

let onboardingMessage = helloWorld; // Type: string
```

When the `helloWorld` constant was re-used in a mutable variable `onboardingMessage` the type which was set is an
expanded version of `"Hello World"` which went from one value ever, to any known string.

- `Transient` - a symbol created in the checker.

  The checker creates its own symbols for unusual declarations:

  1. Cross-file declarations

  ```ts
  // @filename: one.ts
  interface I { a }
  // @filename: two.ts
  interface I { b }
  ```

  The binder creates two symbols for I, one in each file. Then the checker creates a merged symbol that has both declarations.

  2. Synthetic properties

  ```ts
  type Nats = Record<'one' | 'two', number>
  ```

  The binder doesn't create any symbols for `one` or `two`, because those properties don't exist until the Record mapped type creates them.

  3. Complex JS patterns

  ```js
  var f = function g() {
    g.expando1 = {}

  }
  f.expando2 = {}
  ```

  People can put expando properties on a function expression inside or outside the function, using different names.
- `Partial Type` -
- `Synthetic` - a property that doesn't have a declaration in source.
- `Union Types`
- `Enum`
- `Discriminant`
- `Intersection`
- `Indexed Type` - A way to access subsets of your existing types.

```ts
interface User {
  profile: {
    name: string;
    email: string;
    bio: string;
  };
  account: {
    id: string;
    signedUpForMailingList: boolean;
  };
}

type UserProfile = User["profile"]; // { name: string, email: string, bio: string }
type UserAccount = User["account"]; // { id: string, signedUpForMailingList: string }
```

This makes it easier to keep a single source of truth in your types.

- `Index Signatures` - A way to tell TypeScript that you might not know the keys, but you know the type of values
  of an object.

  ```ts
  interface MySettings {
    [index: string]: boolean;
  }

  declare function getSettings(): MySettings;
  const settings = getSettings();
  const shouldAutoRotate = settings.allowRotation; // boolean
  ```

- `IndexedAccess` - ( https://github.com/Microsoft/TypeScript/pull/30769 )
- `Conditional Types`
- `Contextual Types`
- `Substitution`
- `NonPrimitive`
- `Instantiable`
- `Tuple` - A mathematical term for a finite ordered list. Like an array but with a known length.

TypeScript lets you use these as convenient containers with known types.

```ts
// Any item has to say what it is, and whether it is done
type TodoListItem = [string, boolean];

const chores: TodoListItem[] = [["read a book", true], ["done dishes", true], ["take the dog out", false]];
```

Yes, you could use an object for each item in this example, but tuples are there when it fits your needs.

- `Mapped Type` - A type which works by taking an existing type and creating a new version with modifications.

```ts
type Readonly<T> = { readonly [P in keyof T]: T[P] };

// Map for every key in T to be a readonly version of it
// e.g.
interface Dog {
  furColor: string;
  hasCollar: boolean;
}

// Using this
type ReadOnlyDog = Readonly<Dog>;

// Would be
interface ReadonlyDog {
  readonly furColor: string;
  readonly hasCollar: boolean;
}
```

This can work where you

- `Type Assertion` - override its inferred and analyzed view of a type

```ts
interface Foo {
  bar: number;
  bas: string;
}
var foo = {} as Foo;
foo.bar = 123;
foo.bas = "hello";
```

- `Incremental Parsing` - Having an editor pass a range of edits, and using that range to invalidate a cache of
  the AST. Re-running the type checker will keep the out of range nodes and only parse the new section.
- `Incremental Compiling` - The compiler keeps track of all compilation hashes, and timestamps when a file has
  been transpiled. Then when a new module is changed, it will use the import/export dependency graph to invalidate
  and re-compile only the affected code.

### Rarely heard

- `Deferred`
- `Homomorphic`

### JS Internals Specifics

[`Statement`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements) - "JavaScript
applications consist of statements with an appropriate syntax. A single statement may span multiple lines.
Multiple statements may occur on a single line if each statement is separated by a semicolon. This isn't a
keyword, but a group of keywords."

[wnn]: https://github.com/sandersn/manual/blob/master/Widening-and-Narrowing-in-Typescript.md
