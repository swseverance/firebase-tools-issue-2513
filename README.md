# Reproduction for https://github.com/firebase/firebase-tools/issues/2513

## Success path

Follow the following steps to successfully compile the firebase functions in the `/functions` folder. 

```
git clone git@github.com:swseverance/firebase-tools-issue-2513.git

cd firebase-tools-issue-2513

npm install
npm --prefix functions install

npm --prefix functions run build
```

This works because [in TypeScript 6+ the default value is now](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-6-0.html#types-now-defaults-to-):
```json
"types": []
```
This prevents @types in any parent directory from interfering with the compilation of the functions in `/functions`

## Failure path

To observe how @types in a parent node_modules folder can interfere with the compilation of the functions in `/functions` run the following command:

```
npm --prefix functions run build:broken
```

You will observe errors similar to the following:
```
../node_modules/@types/aria-query/index.d.ts:9:25 - error TS1011: An element access expression should take an argument.

9     values: () => Value[];
                          

../node_modules/@types/aria-query/index.d.ts:9:26 - error TS1005: ',' expected.

9     values: () => Value[];
                           ~


Found 8 errors in the same file, starting at: ../node_modules/@types/aria-query/index.d.ts:4
```

This failure occurs because `functions/tsconfig.broken.json` has `"types": ["*"]` which replicates the behavior of TypeScript prior to version 6.

## Additional notes

1. The only difference between `functions/tsconfig.json` and `functions/tsconfig.broken.json` is the inclusion of `"types": ["*"]` in `functions/tsconfig.broken.json`

2. There is a `post-install` script in the parent directory that intentionally breaks one of the files in `node_modules/@types/aria-query` to induce compilation errors. It was the best way to exhibit that @types in parent directories can affect the compilation of firebase functions under certain conditions.