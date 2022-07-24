# Defining a class in JavaScript

They are three ways to define a class in JavaScript. A sample `Clazz` class
implementation is shown below.

- [The class keyword](#using-the-class-keyword)
- [Prototype-based inheritance](#using-prototype-based-inheritance)
- [Object litterals](#using-object-litterals)

## Using the class keyword

This approach is modern (ECMAScript 6, since 2015). It provides a syntactical
sugar over the existing prototype-based inheritance.

```javascript
class Clazz {
    constructor(...) { ... }
    ...
}
```

## Using prototype-based inheritance

Using constructor functions and the prototype chain was the only way to create
class in JavaScript. This is still the only way to create classes in JavaScript,
but a better syntax has been provided by ECMAScript 6 (2015).

```javascript
// constructor
function Clazz(...) {
    ...

    // set a public property for this instance of the class
    this.x = 0;

    // start private properties with _ by convention (basically, private data
    // are poorly encapsulated, but it is a contract between the developer and
    // the API)
    this._x = 0;

    // property y will be looked up in the prototype chain; if found, its value
    // will be printed, otherwise undefined is printed
    console.log(this.y);

    ...
}

// instance function, callable as in this.instanceFuncName()
Clazz.prototype.instanceFuncName = function(...) {
    ...
};

// static function, callable as in Clazz.staticFuncName()
Clazz.staticFuncName = function(...) {
    ...
};

// creating and using an instance of the class
const obj = new Clazz(...);
obj.instanceFuncName();
Clazz.staticFuncName();
```

## Using object litterals

No class-like syntax here, and private data are better encapsulated.

```javascript
function createClazzObject(...) {
    ...

    // private variables unless exposed
    var x1 = ...;
    var _x2 = ...;
    ...

    // private functions unless exposed
    function func1() { ... }
    function func2() { ... }
    ...

    // return an object litteral exposing only public attributes
    return {
        'x1': x1,
        'func1': func1,
        'anotherProperty': ...,
        ...
    };
}

const obj = createClazzObject(...);
obj.func1();
```
