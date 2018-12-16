# tc39-proposal-dotstructuring
## Proposal: Object dotstructuring - dot property(ies) access + object destructuring

This proposal is also known as: Pick Notation, Extended Object Destructuring, Extended Dot Notation.
Inspired by Bob Myers' [proposal](https://github.com/rtm/js-pick-notation) released under MIT License. 

This is a proposal to allow read and write operations on multiple objects' properties (symbols included), leveraging object destructuring syntax to support renaming and defaults. The suggested syntax would enable quick objects creation from existing objects, to create subsets, and quick properties merging/replacing.

## Proposal details
To access multiple properties from an object (sometimes referred to as "picking"), we simply extend dot notation by writing the object name (or an expression evaluating to an object), then a dot, then a destructuring-like construct:
```js
obj.{prop1, prop2};
```
Why we need a destructuring-like construct instead of a a destructuring construct (_ObjectAssignmentPattern_)?
Dotstructuring symbols is a particular case because in normal destructuring we cannot extract directly a Symbol, since it would not make any sense:
```js
const {[Symbol.iterator]} = [];
```
We have to rename it:
```js
const {[Symbol.iterator]:foo} = [];
```
Considering that dotstructuring always involves two object, it has to be permitted __only for dotstructuring purposes__ to allow a complete selection of properties to copy.

## RHS access (right-hand side)
In this situation the object dotstructuring will:
1. pick chosen properties 
2. create a fresh new object 
3. copying chosen props and values into the just created object (shallow copy)
4. return the just created object

```js
const obj = {prop1: 42, prop2: 'foo', [Symbol.iterator]: .. };

const newObject = obj.{prop1, [Symbol.iterator]};
newObject; // {prop1: 42,  [Symbol.iterator]: .. };

obj; // {prop1: 42, prop2: 'foo', [Symbol.iterator]: .. };
```
The source object remains unaffected.

### undefined properties
If a not existing property is picked the corrisponding value willbe undefined:
```js
const obj = {prop1: 42};

const newObject = obj.{prop2};
newObject; // {prop2: undefined};

obj; // {prop1: 42};;
```
