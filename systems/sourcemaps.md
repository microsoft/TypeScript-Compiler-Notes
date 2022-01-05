### Nodes

Every node in the TS AST has an optional `emitNode`, this node provides a sort of wrapping context to a node when
it is emitted. For example it keeps track of trailing/leading comments and importantly for sourcemaps:
`sourceMapRange` and `tokenSourceMapRange`.

These are applied via code transformers in TypeScript

### Creating the Files

Files are created inside `emitter.ts` - the opening function being `emitWorker`.

- Each source file then gets `writeFile` called on it which triggers the AST traversal on the source file
- A `sourceMapGenerator` is created
- Each node gets triggered via `pipelineEmit`
  - `pipelineEmit` runs all transformers against the node
  - This is where the pipeline-ing for TS -> ES2020 -> ES2018 -> ES2017 -> ES2015 -> ES6 happens to that node
  - Each node calls `emitSourcePos` which tells the `sourceMapGenerator` that a mapping exists for this node

This means when looking into how TS handles sourcemap, you need to find the part of the above pipeline which
transforms the syntax you're looking for.
