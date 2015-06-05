# MOBservable

[![Build Status](https://travis-ci.org/mweststrate/MOBservable.svg?branch=master)](https://travis-ci.org/mweststrate/MOBservable)
[![Coverage Status](https://coveralls.io/repos/mweststrate/MOBservable/badge.svg?branch=master)](https://coveralls.io/r/mweststrate/MOBservable)

[![NPM](https://nodei.co/npm/mobservable.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/mobservable/)

Installation: `npm install mobservable --save`

MOBservable is light-weight stand-alone observable implementation, that helps you to create reactive data structures, based on the ideas of observables in bigger frameworks like `knockout`, `ember`, but this time without 'strings attached'. 
MOBservables allows you to observe primitive values, references, functions and arrays and makes sure that all changes in your data are propagated automatically, atomically and synchronously.

TODO: blog post

# Examples

[Fiddle demo: MOBservable + JQuery](http://jsfiddle.net/mweststrate/vxn7qgdw)
https://jsfiddle.net/mweststrate/vxn7qgdw/1/embedded/result/

TODO: react fiddle

## Example: Observable values and functions

```javascript
var mobservable = require('mobservable');

var nrOfCatz = mobservable(3);
var nrOfDogs = mobservable(8);

// Create a function that automatically observes values:
var nrOfAnimals = mobservable(function() {
    // calling an mobservable without arguments acts as getter
    return nrOfCatz() * nrOfDogs();
});

// Print a message whenever the observable changes:
nrOfAnimals.observe(function(amount) {
    console.log("Total: " + amount);
}, true);
// -> Prints: "Total: 11"

// calling an mobservable with a value acts as setter, 
// ...and automatically updates all computations in which it was used
nrOfCatz(34);
// -> Prints: "Total: 42"
```

## Example: Observable objects & properties

```javascript
var mobservable = require('mobservable');

var Person = function(firstName, lastName) {
    // define the observable properties firstName, lastName and fullName on 'this'.
    mobservable.props(this, {
        firstName: firstName,
        lastName: lastName,
        fullName: function() {
            return this.firsName + " " + this.lastName;
        }
    });
}

var jane = new Person("Jane","Dôh");

// (computed) properties can be accessed like any other property:
console.log(jan.fullName);
// prints: "Jan Dôh"

// properties can be observed as well:
mobsevable.observeProperty(jane, "fullName", console.log);

// values can be assigned directly to observable properties
jane.lastName = "Do";
// prints: "Jane Do"
```

## Example: Arrays

```javascript
import mobservable = require('mobservable');

// create an array, that works by all means as a normal array, except that they are observable!
var someNumbers = mobservable.value([1,2,3]);
var sum = mobservable.value(function() {
    for(var s = 0, i = 0; i < someNumbers.length; i++)
        s += someNumbers[i];
});
sum.observe(console.log);

someNumbers.push(4);
// Prints: 10
someNumbers[2] = 0;
// Prints: 7
someNumbers[someNumbers.length] = 5;
// Prints: 12
```

## Example: TypeScript classes and annotations

```typescript
/// <reference path="./node_modules/mobservable/mobservable.d.ts"/>
import mobservable = require('mobservable');
var observable = mobservable.observable;

class Order {
    @observable orderLines: OrderLine[] = [];
    @observable total() {
        return this.orderLines.reduce((sum, orderLine) => sum + orderLine.total, 0)
    }
}

class OrderLine {
    @observable price:number = 0;
    @observable amount:number = 1;

    constructor(price) {
        this.price = price;
    }

    @observable total() {
        return "Total: " + this.price * this.amount;
    }
}

var order1 = new Order();
order1.total.observe(console.log);

order1.orderLines.push(new OrderLine(7));
// Prints: Total: 7
order1.orderLines.push(new OrderLine(12));
// Prints: Total: 12
order1.orderLines[0].amount = 3;
// Prints: Total: 33
```

# API 
[Typescript typings](https://github.com/mweststrate/MOBservable/blob/master/mobservable.d.ts)


# Observable values

The `mobservable.value(valueToObserve)` method (or just its shorthand: `mobservable(valueToObserve)`) takes a value or function and creates an observable value from it. A quick example:

```typescript
/// <reference path='./node_modules/mobservable/mobservable.d.ts'/>
import mobservable = require('mobservable');

var vat = mobservable.value(0.20);

var order = {};
order.price = mobservable.value(10),
order.priceWithVat = mobservable.value(() => order.price() * (1 + vat()));

order.priceWithVat.observe((price) => console.log("New price: " + price));

order.price(20);
// Prints: New price: 24
vat(0.10);
// Prints: New price: 22
```

# Processing observables

Observable values, arrays and functions created by `mobservable` possess the following characteristics:

* _synchronous_. All updates are processed synchronously, that is, the pseudo expressions `a = 3; b -> a * 2; a = 4; print(b); ` will always print `4`; `b` will never yield a stale value (unless `batch` is used).
* _atomic_. All computed values will postpone updates until all inputs are settled, to make sure no intermediate values are visible. That is, the expression `a = 3; b -> a * 2; c -> a * b; a = 4; print(c)` will always print `36` and no intermediate values like `24`.
* _real time dependency detection_. Computed values only depend on values actually used in the last computation, for example in this `a -> b > 5 ? c : b` the variable `c` will only cause a re-evaluation of a if `b` > 5. 
* _lazy_. Computed values will only be evaluated if they are actually being observed. So make sure computed functions are pure and side effect free; the library might not evaluate the expression as often as you thought it would.   
* _cycle detection_. Cycles in computes, like in `a -> 2 * b; b -> 2 * a;` will be deteced automatically.  
* _error handling_. Exceptions that are raised during computations are propagated to consumers.

#### .props or .value?

MOVE

| .value | .props |
| ---- | ---|
| ES3 complient | requires ES 5 |
| explicit getter/setter functions: `obj.amount(2)`  | object properties with implicit getter/setter: `obj.amount = 2 ` |
| easy to make mistakes; e.g. `obj.amount = 3` instead of `obj.amount(3)`, or `7 * obj.amount` instead of `7 * obj.amount()` wilt both not achieve the intended behavior | Use property reads / assignments |
| easy to observe: `obj.amount.observe(listener)` | `mobservable.observeProperty(obj,'amount',listener)`  |

# API

## mobservable

Shorthand for `mobservable.value`

## mobservable.value

`mobservable.value<T>(value? : T[], scope? : Object) : IObservableArray<T>`

`mobservable.value<T>(value? : T|()=>T, scope? : Object) : IObservableValue<T>`

Function that creates an observable given a `value`. 
Depending on the type of the function, this function invokes `mobservable.array`, `mobservable.computed` or `mobservable.primitive`.
See the examples above for usage patterns. The `scope` is only meaningful if a function is passed into this method.

## mobservable.primitive

`mobservable.primitive<T>(value? : T) : IObservableValue<T>`

Creates a new observable, initialzed with the given `value` that can change over time. 
The returned observable is a function, that without arguments acts as getter, and with arguments as setter.
Furthermore its value can be observed using the `.observe` method, see `IObservableValue.observe`.  

Example:
```
var vat = mobservable.primitive(3);
console.log(vat()); // prints '3'
vat.observe(console.log); // register an observer
vat(4); // updates value, also notifies all observers, thus prints '4'
```


## mobservable.reference

`mobservable.reference<T>(value? : T) : IObservableValue<T>`
Synonym for `mobservable.primitive`, since the equality of primitives is determined in the same way as references, namely by strict equality.
(TODO: future work) See `mobservable.struct` if values need to be compared structuraly (by using deep equality).

## mobservable.computed

`mobservable.computed<T>(expr : () => T, scope?) : IObservableValue<T>`

`computed` turns a function into an observable value. 
The provided `expr` should not have any arguments, but instead really on other observables that are in scope to determine its value.
The latest value returned by `expr` determines the value of the observable. When one of the observables used in `expr` changes, `computed` will make sure that the function gets re-evaluated, and all updates are propogated to the children.

```javascript
    var amount = mobservable(3);
    var price = mobservable(2);
    var total = mobservable.computed(function() {
        return amount() * price();
    });
    console.log(total()); // gets the current value, prints '6'
    total.observe(console.log); // attach listener
    amount(4); // update amount, total gets re-evaluated automatically and will print '8'
    amount(4); // update amount, but total will not be re-evaluated since the value didn't change
```

The optional `scope` parameter defines `this` context during the evaluation of `expr`. 

`computed` will try to reduce the amount of re-evaluates of `expr` as much as possible. For that reason the function *should* be pure, that is:
* The result of `expr` should only be defined in terms of other observables, and not depend on any other state.
* Your code shouldn't rely on any side-effects, triggered by `expr`; `expr` should be side-effect free.
* The result of `expr` should always be the same if none of the observed observables did change.

It is not allowed for `expr` to have an (implicit) dependency on its own value.

It is allowed to throw exceptions in an observed function. The thrown exceptions might only be detected late. 
The exception will be rethrown if somebody inspects the current value, and will be passed as first callback argument
to all the listeners. 

## mobservable.array

`mobservable.array<T>(values? : T[]) : IObservableArray<T>`
**Note: ES5 environments only**

Constructs an array like, observable structure. An observable array is a thin abstraction over native arrays that adds observable properties. 
The most notable difference between built-in arrays is that these arrays cannot be sparse, that is, values assigned to an index larger than `length` are considered out-of-bounds and not oberved (nor any other property that is assigned to a non-numeric index). 

Furthermore, `Array.isArray(observableArray)` and `typeof observableArray === "array"` will yield `false` for observable arrays, but `observableArray instanceof Array` will return `true`. 

```javascript
var numbers = mobservable.array([1,2,3]);
var sum = mobservable.value(function() {
    return numbers.reduce(function(a, b) { return a + b }, 0);
});
sum.observe(function(s) { console.log(s); });

numbers[3] = 4;
// prints 10
numbers.push(5,6);
// prints 21
numbers.unshift(10)
// prints 31
```

## mobservable.props

```typescript
props(target:Object, name:string, initialValue: any):Object;
props(target:Object, props:Object):Object;
props(target:Object):Object;
``` 
**Note: ES5 environments only**

Creates observable properties on the given `target` object. This function uses `mobservable.value` internally to create observables.
Creating properties has as advantage that they are more convenient to use. See also [props or variables](TODO).
The original `target`, with the added properties, is returned by this function. Functions used to created computed observables will automatically
be bound to the correct `this`.

```javascript
var order = {};
mobservable.props(order, {
    amount: 3,
    price: 5,
    total: function() {
        return this.amount * this.price; // note that no setters are needed
    }
});
order.amount = 4;
console.log(order.total); // Prints '20'
```

Note that observables created by `mobservable.props` don't have an `.observe` method, to observe properties, see [`mobservable.observeProperty`](TODO)

Other forms in which this function can be called:
```javascript
mobservable.props(order, "price", 3); // equivalent to mobservable.props(order, { price: 3 });
var order = mobservable.props({ price: 3}); // uses the original object as target, that is, all values in it are replaced by their observable counterparts
```


observable(target:Object, key:string); // annotation

// observables to not observables
toPlainValue<T>(any:T):T;

// observe observables
observeProperty(object:Object, key:string, listener:Function, invokeImmediately?:boolean):Lambda;
watch<T>(func:()=>T, onInvalidate:Lambda):[T,Lambda];

// change a lot of observables at once
batch<T>(action:()=>T):T;

// Utils
debugLevel: number;
SimpleEventEmitter: new()=> ISimpleEventEmitter;
