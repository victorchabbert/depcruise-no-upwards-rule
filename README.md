# no-upward-rule

This is a challenge I'm trying to solve but I'm bad at regex...

With `dependency-cruiser`, I would like to enforce this rule :

> All source dependencies should point inwards

Visually it would look something like this (simplified):

![](./rule-simplified.png)

## Rules

- [ ] 1. Each file can import any other file that is :
  - [ ] at the same level
  - [ ] nested in a folder and subs from the same level

Example :

`index0` can import `index1` and `index2`.

`index1` can import `index2`.

```
+-- ...
    +-- index0
    +-- sub1
        +-- index1
        +-- sub2
            +-- index2
```

- [ ] 2. Each file cannot import any file that is above it's current level

Example :

`index2` can **not** import `index1` and `index0`.

`index1` can **not** import `index0`.

```
+-- ...
    +-- index0
    +-- sub1
        +-- index1
        +-- sub2
            +-- index2
```

- [ ] 3. Each file can import a file or sub file from an exempted folder list
  (Exempted folders can be hardcoded 😌)
- [ ] 4. Exempted folders follow the #1 rule internally (no upward imports)

## Test case

I've set up a sample case.
Run `npm run gc:check` with your rule and verify if the rules have been fufilled.

## Expected

This is the graph expected :

![](./rule-expected.svg)

To check if it's similar, run `gc:osage` package.json command.

As text :
```
Not valid :
src/module11/module2/not-valid.js → src/module11/index.js
src/module11/module2/not-valid.js → src/index.js
src/module12/not-valid.js → src/index.js

Should be valid :
src/module11/module2/not-valid.js → src/exempted1/index.js
src/module11/module2/not-valid.js → src/exempted2/index.js
src/module12/not-valid.js → src/exempted1/index.js
src/module12/not-valid.js → src/exempted2/index.js
```


## Exploration
The following are what I've tried. They may help you get started if you want to take a crack at it !

1. Try to extract the leading folder path
```
These parts :

    src/module11/module2/index.js
$1  ^^^^^^^^^^^^^|      |
$2               ^^^^^^^^
```

Regex: 
```
^src\/(([^/]+\/)*)([^/]+\/){1}[^/].[^/.]+
```

Problems:
  - This does not scale well because of `{1}`.
  - How to enforce the rules after that ??

2. Reverse check

Capture everything before the last `/`.
Using that group, forbid everything except all sub files and folders.

[Playground](https://regexr.com/4f2js)

```js
...
{
  name: "no-upward-imports",
  comment: "Don't allow dependencies to import parents",
  from: {
    path: "^src/(([^/]+/)*)[^/]+",
  },
  to: {
    pathNot: "^src/$1[^/]+",
  },
},
...
```
This command fails with an `unsafe regular expression. Bailing out.`.

I'm not sure how to go forward now.
Any help is greatly appreciated :pray:

Thanks and good luck !
