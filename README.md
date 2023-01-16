## About
This repo demonstrates that microbundle generates typings matching the directory structure of `src`.
For a single-file package, this introduce relative dependencies in the root `.d.ts`. This creates a
problem when a dependent package tries to resolve those types, as TS does not resolve these relative
declarations.

This is specifically targeting the "modern" module build where:

| Package   | Config File   | Configuration                            |
| --        | --            | --                                       | 
| library   | package.json  | type: module                             | 
|           |               | exports.types: ./dist/index.d.ts         | 
|           | tsconfig.json | compilerOptions.module: node16           | 
|           |               |                                          |
| dependent | package.json  | type: module                             | 
|           | tsconfig.json | compilerOptions.moduleResolution: node16 | 

## The library package

Clone repo and run:

```
npm run build
```

The the resulting build directory looks like:

```
dist/
├── foo.d.ts <-------.
├── index.cjs        | 
├── index.cjs.map    | Relative dependency
├── index.d.ts <-----'
├── index.modern.mjs
├── index.modern.mjs.map
├── index.module.mjs
├── index.module.mjs.map
├── index.umd.js
└── index.umd.js.map
```

## The dependent package

Navigate to `test-package/`. Run `npm run build` to build the dependent package with `tsc` version `4.9.4`.

Run `./trace-build` to build the package and log where the module resolution goes wrong. Output:

```
======== Module name 'microbundle-typings' was successfully resolved to '~/workspace/microbundle-typings/dist/index.d.ts' with Package ID 'microbundle-typings/dist/index.d.ts@1.0.0'. ========
File '~/workspace/microbundle-typings/dist/package.json' does not exist.
Found 'package.json' at '~/workspace/microbundle-typings/package.json'.
'package.json' does not have a 'typesVersions' field.
======== Resolving module './foo' from '~/workspace/microbundle-typings/dist/index.d.ts'. ========
Explicitly specified module resolution kind: 'Node16'.
Resolving in ESM mode with conditions 'node', 'import', 'types'.
Loading module as file / folder, candidate module location '~/workspace/microbundle-typings/dist/foo', target file type 'TypeScript'.
Directory '~/workspace/microbundle-typings/dist/foo' does not exist, skipping all lookups in it.
Loading module as file / folder, candidate module location '~/workspace/microbundle-typings/dist/foo', target file type 'JavaScript'.
Directory '~/workspace/microbundle-typings/dist/foo' does not exist, skipping all lookups in it.
======== Module name './foo' was not resolved. ========
```

Note that rejiggering the library's `exports` doesn't matter, because TS doesn't consider the
original module's `package.json` when looking for relative dependents.
Namely, this doesn't change anything:

```json
"exports": {
  "./foo": {
    "types": "./dist/foo.d.ts",
  },
  ".": {
    "types": "./dist/index.d.ts",
    "import": "./dist/index.modern.mjs"
  }
},
```