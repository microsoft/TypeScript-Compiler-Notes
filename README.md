# TypeScript Compiler Notes

This repo is a corpus of notes from many engineers over time on different systems inside the TypeScript codebase. It is not meant as a "one true" source of authoritative documentation for the TypeScript compiler API, but as a way to ease contributions to the [`microsoft/TypeScript`](https://github.com/microsoft/TypeScript) repo.

If you're already familiar with the TypeScript codebase and want to help out, we're open to external folks sending PRs improving or adding areas!

## Get Started

If you are completely new to the TypeScript codebase, this YouTube video covers all of the major systems involved in converting files to JavaScript, and type-checking types which will provide a high level guide to what you may be interested in focusing on: 

<a href='https://www.youtube.com/watch?v=X8k_4tZ16qU&list=PLYUbsZda9oHu-EiIdekbAzNO0-pUM5Iqj&index=4'><img src="https://user-images.githubusercontent.com/49038/140491214-720ce354-e526-4599-94ec-72cdbecc2b01.png" /></a>


<details>
          <summary>If you are really short on time, here is a quick overview of the compilation process.</summary>

The process starts with preprocessing. 
The preprocessor figures out what files should be included in the compilation by following references (`/// <reference path=... />` tags, `require` and `import` statements).

The parser then generates AST `Node`s. 
These are just an abstract representation of the user input in a tree format. 
A `SourceFile` object represents an AST for a given file with some additional information like the file name and source text. 

The binder then passes over the AST nodes and generates and binds `Symbol`s.
One `Symbol` is created for each named entity.
There is a subtle distinction but several declaration nodes can name the same entity.
That means that sometimes different `Node`s will have the same `Symbol`, and each `Symbol` keeps track of its declaration `Node`s.
For example, a `class` and a `namespace` with the same name can *merge* and will have the same `Symbol`.
The binder also handles scopes and makes sure that each `Symbol` is created in the correct enclosing scope.

Generating a `SourceFile` (along with its `Symbol`s) is done through calling the `createSourceFile` API.

So far, `Symbol`s represent named entities as seen within a single file, but several declarations can merge multiple files, so the next step is to build a global view of all files in the compilation  by building a `Program`.

A `Program` is a collection of `SourceFile`s and a set of `CompilerOptions`.
A `Program` is created by calling the `createProgram` API. 

From a `Program` instance a `TypeChecker` can be created. 
`TypeChecker` is the core of the TypeScript type system. 
It is the part responsible for figuring out relationships between `Symbols` from different files, assigning `Type`s to `Symbol`s, and generating any semantic `Diagnostic`s (i.e. errors).

The first thing a `TypeChecker` will do is to consolidate all the `Symbol`s from different `SourceFile`s into a single view, and build a single Symbol Table by "merging" any common `Symbol`s (e.g. `namespace`s spanning multiple files).  

After initializing the original state, the `TypeChecker` is ready to answer any questions about the program.
Such "questions" might be: 
* What is the `Symbol` for this `Node`?
* What is the `Type` of this `Symbol`?
* What `Symbol`s are visible in this portion of the AST?
* What are the available `Signature`s for a function declaration?
* What errors should be reported for a file?

The `TypeChecker` computes everything lazily; it only "resolves" the necessary information to answer a question.
The checker will only examine `Node`s/`Symbol`s/`Type`s that contribute to the question at hand and will not attempt to examine additional entities.

An `Emitter` can also be created from a given `Program`.
The `Emitter` is responsible for generating the desired output for a given `SourceFile`; this includes `.js`, `.jsx`, `.d.ts`, and `.js.map` outputs. 
 
</details>

From there, you can start in the [First Steps to Contributing to the TypeScript Repo](https://github.com/microsoft/TypeScript-Compiler-Notes/tree/main/intro#the-typescript-compiler) consult the [Glossary](./GLOSSARY.md) or dive directly into the [`./codebase/`](./codebase) or [`./systems/`](./systems) folders.

## Asking Questions

One of the best places to ask questions is in the 'compiler-internals-and-api' channel of the [TypeScript Community Discord](https://discord.gg/typescript).

## Related TypeScript Info

- Learn how TypeScript works by reading the [mini-TypeScript implementation](https://github.com/sandersn/mini-typescript#mini-typescript)
- [Basarat's guide to the Compiler Internals](https://basarat.gitbook.io/typescript/overview)

## Compilers in General

- Recommended link for learning how compilers work: https://c9x.me/compile/bib/

## Interesting PRs

If you learn better by seeing how big features are added to TypeScript, here are a few big well-scoped Pull Requests:

- Unions - [microsoft/TypeScript#824](https://github.com/microsoft/TypeScript/pull/824)
- Type Aliases - [microsoft/TypeScript#957](https://github.com/microsoft/TypeScript/pull/957)
- Async Functions - [microsoft/TypeScript#3078](https://github.com/microsoft/TypeScript/pull/3078)
- TSX - [microsoft/TypeScript#3564](https://github.com/microsoft/TypeScript/pull/3564)
- Intersection Types - [microsoft/TypeScript#3622](https://github.com/microsoft/TypeScript/pull/3622)
- String Literal Types - [microsoft/TypeScript#5185](https://github.com/microsoft/TypeScript/pull/5185)
- JS in TS - [microsoft/TypeScript#5266](https://github.com/microsoft/TypeScript/pull/5266)
- Using JSDoc to extract types - [microsoft/TypeScript#6024](https://github.com/microsoft/TypeScript/pull/6024)
- Nullable types - [microsoft/TypeScript#7140](https://github.com/microsoft/TypeScript/pull/7140)
- Control Flow Analysis - [microsoft/TypeScript#8010](https://github.com/microsoft/TypeScript/pull/8010)
- Mapped Types - [microsoft/TypeScript#12114](https://github.com/microsoft/TypeScript/pull/12114)
- Rest Types - [microsoft/TypeScript#13470](https://github.com/microsoft/TypeScript/pull/13470)
- Strict Functions - [microsoft/TypeScript#18654](https://github.com/microsoft/TypeScript/pull/18654)
- Unknown - [microsoft/TypeScript#24439](https://github.com/microsoft/TypeScript/pull/24439)
- Optional Chaining - [microsoft/TypeScript#33294](https://github.com/microsoft/TypeScript/pull/33294)
- Node ESM Support - [microsoft/TypeScript#45884](https://github.com/microsoft/TypeScript/pull/45884)

# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

# Legal Notices

Microsoft and any contributors grant you a license to the Microsoft documentation and other content
in this repository under the [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode),
see the [LICENSE](LICENSE) file, and grant you a license to any code in the repository under the [MIT License](https://opensource.org/licenses/MIT), see the
[LICENSE-CODE](LICENSE-CODE) file.

Microsoft, Windows, Microsoft Azure and/or other Microsoft products and services referenced in the documentation
may be either trademarks or registered trademarks of Microsoft in the United States and/or other countries.
The licenses for this project do not grant you rights to use any Microsoft names, logos, or trademarks.
Microsoft's general trademark guidelines can be found at http://go.microsoft.com/fwlink/?LinkID=254653.

Privacy information can be found at https://privacy.microsoft.com/en-us/

Microsoft and any contributors reserve all other rights, whether under their respective copyrights, patents,
or trademarks, whether by implication, estoppel or otherwise.
