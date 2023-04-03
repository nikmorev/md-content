

## Trying to grasp the difference between \_\_proto\_\_, prototype, constructor and a bit more.

### About \_\_proto\_\_

All object in JS have a hidden property `[[Prototype]]`. This property can be `null` or another object that is kind of inherited by the current one and adds extra properties and methods.  
This property can be accessed via property `__proto__`. `__proto__` is defined under the hood as setter and getter.  

For example:
```
const user = { name: 'Nik' }

const admin = { isAdmin: true }

Object.setPrototypeOf(admin, user)
// or using old way
// admin.__proto__ = user

console.log(admin.name) // 'Nik'
```
If current object has no property/method - the property/method are searched in prototype, and so on till the first prototype.  

`__proto__` can be object or null, and it should not have circular dependencies.

Only reading operations can influenced by prototype. Writing/changing property is always related to the current object (not to the prototype).  

For example:
```
const user = { name: 'Nik' }

const admin = { isAdmin: true }

admin.__proto__ = user

admin.name = 'Vik'

console.log(admin.name) // 'Vik'
console.log(user.name, admin.__proto__.name) // 'Nik', 'Nik'
```

And example with accessors:
```
const user = { 
    set name(newName) { this._name = newName },
    get name() { return this._name },
    _name: 'Nik'
}

const admin = { isAdmin: true }

admin.__proto__ = user

admin.name = 'Vik'

console.log(admin.name) // 'Vik'
console.log(user.name, admin.__proto__.name) // 'Nik', 'Nik'
```

`for ... in` considers all properties (including iherited). To check if prop is own - use `obj.hasOwnProperty(key)` or `Object.getOwnPropertyNames`.  

### About 'prototype' and constructor

Long type ago, when objects were mostly created using constructor function (`new Func`), the only way to reach (set, read, change) the prototype was using property `prototype`.  

For example:
```
function User(name) {
    this.name = name
}

const customer = { customer: true }

User.prototype = customer

const client = new User('Nik')

console.log(client.customer) // true
```

`Func.prototype` is a usual property and it's applied as as prototype (`[[Prototype]]`) only when `new Func` is called. If prototype is changed after creating the object, created object will keep the previous version of prototype (the one that was in time the object was created).  

By default every function has a property `prototype` with one and only property `constructor`, that relates to that function constructor.
```
function Func() {}

console.log(Func.prototype, Func.prototype.constructor.name) // { constructor: f }, 'Func'
```

### Constructor in class

Along with functions used for object creating, javascript has `class` that nowadays is more appropriate for such need. Class has a `constructor` method, that is used to initialize the object on its creation and is called on `new User()`.  
Class is a type of function in javascript, but it is different from function and due to such differences it is not just a syntax sugar.

1. `class` has internal property `[[IsClassConstructor]]: true` that prohibits to call a class constructor without new.
```
class User {}

User() // Error: Class constructor User cannot be invoked without 'new'
```
2. All methods of class are enumerable (`enumerable:  false` for all methods on descriptions level).
3. Classes always use `use strict` internally.

### Off top
All methods of classes and objects have internal property `[[HomeObject]]` that allows correct work of `super` to reach correct methods from prototype (from derived classes/objects).


## References
* [https://learn.javascript.ru/prototypes](https://learn.javascript.ru/prototypes)
* [https://learn.javascript.ru/class-inheritance](https://learn.javascript.ru/class-inheritance)
* [https://learn.javascript.ru/class](https://learn.javascript.ru/class)

