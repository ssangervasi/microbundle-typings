This repo demonstrates that microbundle generates typings matching the directory structure of `src`, when they should be concatenated.

# To use

Clone repo and run:

```
yarn dev
```

In `/dist`, you'll see a `foo/index.d.ts` file. Typings files should be all included in the single `index.d.ts` file.
