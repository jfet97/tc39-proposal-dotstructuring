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
Why we need a destructuring-like construct instead of a a destructuring construct (_ObjectAssignmentPattern_)?\
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
1. pick chosen properties from the source
2. create a fresh new object 
3. copy chosen props and their values into the just created object (shallow copy)
4. return the just created object

```js
const source = {prop1: 42, prop2: 'foo', [Symbol.iterator]: .. };

const newObject = source.{prop1, [Symbol.iterator]};
newObject; // {prop1: 42,  [Symbol.iterator]: .. };

source; // {prop1: 42, prop2: 'foo', [Symbol.iterator]: .. };
```
The source object remains unaffected.\
The [[Prototype]] property is not copied.\
&nbsp;
### undefined properties
If a not existing property is picked the corrisponding value will be undefined:
```js
const source = {prop1: 42};

const newObject = source.{prop2};
newObject; // {prop2: undefined};

source; // {prop1: 42};
```
&nbsp;
### default values for properties
Dotstructuring syntax let us to set default value for properties like destructuring:
```js
const source = {prop1: 42};

const newObject = source.{prop2 = 'foo'};
newObject; // {prop2: 'foo'};

source; // {prop1: 42};
```
&nbsp;
### renaming properties
Dotstructuring syntax let us to rename properties like destructuring:
```js
const source = {prop1: 42};

const newObject = source.{prop1:p1};
newObject; // {p1: 42};

source; // {prop1: 42};
```
&nbsp;
### extracting nested properties
Dotstructuring syntax let us to extract nested properties like destructuring:
```js
const source = {obj1: {prop1: 42}};

const newObject = source.{ obj1:{prop1} };
newObject; // {prop1: 42};

source; // {obj1: {prop1: 42}};
```
## Possible RHS access transpilation
All the RHS object dotstructuring (symbols included) cases could be transpiled:
```js
const source = {prop: .., [Symbol.iterator]: ..};
const newObject = source.{prop, [Symbol.iterator]};
```
into:
```js
const source = {prop: .., [Symbol.iterator]: ..};
const newObj = (function(obj){
    var tmp = {};
    return ({ prop: tmp.prop, [Symbol.iterator]: tmp[Symbol.iterator] } = obj, tmp);
})(source);
```
## LHS access (left-hand side)
In this situation the object dotstructuring will:
1. pick chosen properties from the source
2. select corresponding properties in the __already existing__ target object
3. copy chosen props and their values into the target object (shallow copy), overwriting those already present in case of conflicts
4. return the just updated object

```js
const source = {prop1: 42, prop2: 'foo'};
const target = {prop1: 'bar', prop3: 'baz'};

target.{prop1, prop2} = source;

source; // {prop1: 42, prop2: 'foo'};
target; // {prop1: 42, prop2: 'foo', prop3: 'baz'};
```
The source object remains unaffected.\
The [[Prototype]] property is never copied.\
The _target_ must be an already existing object because if a fresh new one is needed with one or more properties picked by another object, the RHS access has to be used:
```js
const source = {prop1: 42, prop2: 'foo'};
const target.{prop1, prop2} = source; // ERROR: LHS dotstructuring does not provide an implicit object creation
```
on the contrary, use this:
```js
const source = {prop1: 42, prop2: 'foo'};
const target = source.{prop1, prop2}; //  RHS dotstructuring provides an implicit object creation
```
&nbsp;
### undefined properties
If a not existing property is picked the corrisponding value in the target will be undefined:
```js
const source = {};
const target = {};

target.{prop1} = source;

source; // {}
target; // {prop1:undefined}
```
&nbsp;
If the property exists in the target but not in the source, that property inthe target will become undefined:
```js
const source = {};
const target = {prop1:42};

target.{prop1} = source;

source; // {}
target; // {prop1:undefined}
```
&nbsp;
### default values for properties
Dotstructuring syntax let us to set default value for properties like destructuring:
```js
const source = {};
const target = {};

target.{prop1 = 42} = source;

source; // {}
target; // {prop1:42}
```
&nbsp;
### renaming properties
Dotstructuring syntax let us to rename properties like destructuring:
```js
const source = {prop1:42};
const target = {};

target.{prop1:p1} = source;

source; // {prop1:42}
target; // {p1:42}
```
&nbsp;
### extracting nested properties
Dotstructuring syntax let us to extract nested properties like destructuring:
```js
const source = {obj1: {prop1: 42}};
const target = {};

target.{obj1:{prop1}} = source;

target; // {prop1: 42};
source; // {obj1: {prop1: 42}};
```
### handling properties conflicts
Target properties will be overwritten in case of conflicts:
```js
const source = {prop1:42};
const target = {prop1:'foo'};

target.{prop1} = source;

target; // {prop1: 42};
source; // {prop1: 42};
```
## Possible LHS access transpilation
All the LHS object dotstructuring (symbols included) cases could be transpiled:
```js
const source = {prop1: .., [Symbol.iterator]: ..};
const target = {prop2: ..};

target.{prop1, [Symbol.iterator]} = target;
```
into:
```js
const source = {prop1: .., [Symbol.iterator]: ..};
const target = {prop2: ..};

Object.assign(target, (function(obj){
    var tmp = {};
    return ({ prop: tmp.prop, [Symbol.iterator]: tmp[Symbol.iterator] } = obj, tmp);
})(source));
```
## Reasons
Currently JavaScript syntax does not let us to pick one or more properties from an object to create a new one nor let us to merge only some chosen properties from an object to another. Object dotstructuring could be useful to easily create objects subsets and to merge/clone/replace chosen properties from an object to another.

Bob Myers' [proposal](https://github.com/rtm/js-pick-notation) supports the following:

Expressiveness and brevity for the common use cases of picking properties from objects into existing or new objects are reached by the proposal.\
Currently we have destructuring assignment, which provides a useful way to extract properties from objects. However, it is limited to assigning the values to variables. This proposal can be thought of as a natural extension to destructuring assignment, leveraging its syntax, to allow destructuring into properties of objects. It brings parity to the concept of destructuring.\
Picking properties from an object is a common use case in today's JavaScript, but currently requires approaches such as

```js
const newObject = {prop1: object.prop1, prop2: object.prop2};

or

```js
const {prop1, prop2} = object;
const newObject = {prop1, prop2};
```

The use case for this feature can also be considered to be validated by the existence of the _\_.pick_ utilities available in several popular libraries. People in the real world also regularly wonder about the absence of native facilities for picking/destructuring from objects into objects, instead of just variables.

It makes good sense to re-use the current dot notation, which programmers have used and loved for two decades. since what we are trying to do is in fact setting and retrieving object properties, the difference being that we are setting or retrieving more than one at a time.

In that sense, this proposal can be viewed as bringing parity to both object destructuring, and dot notation, by allowing the existing destrucuturing concept to be applied uniformly in conjunction with dot notation.
