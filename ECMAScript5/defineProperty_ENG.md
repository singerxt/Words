## Something you may not know about ES5 - The Object.defineProperty () method

Currently everyone is looking forward to JavaScript ES6. However for me and many other JavaScript developers, ES5 has several interesting built-in methods which are still not commonly known or used. Itâs definitely worthwhile to devote some time learning those less popular methods. In my opinion they are the perfect solution when JavaScript is not only used to modify the DOM elements, but is also responsible for the web application's logic.

## Problem 
Our task is to create a __Person__ constructor which takes two parameters:  __firstName__ and __lastName__. Our object will expose four attributes: __firstName__, __lastName__, __fullName__ and __species__. The first three will react to changes, and the last one  __species__ will have a constant value of `'human'`. Example below:

```JavaScript
var woman = new Person('Tom', 'Khowalski');

woman.firstName; // 'Tom'
woman.lastName; // 'Khowalski'
woman.fullName; //'Tom Khowalski
woman.species; // human

/*
 * Change firstName
 */
 
woman.firstName = 'Mat';
woman.firstName; // 'Mat'
woman.lastName; // 'Khowalski'
woman.fullName; // 'Mat Khowalski' - How to do that !?
woman.species = 'fish';
woman.species; // human - No you can't change this propriety.
 
 /*
  * Change fullName
  */
  
woman.fullName = 'John Stevens';
woman.firstName; //John
woman.lastName; //Stevens
```

## How to do it ?

Hereâs the part where [Object.defineProperty](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) comes to the rescue. It is a method of ECMAScript 5 and itâs used to define or override the attributes of an object. 
It accepts as a parameter the object to be modified. The second parameter is the name of the key. The third is the 'description' attribute. The 'description' field is an object that provides us with the following fields:

- __configurable__ - true if and only if the type of this property descriptor may be changed and if the property may be deleted from the corresponding object.
Defaults to false.

- __enumerable__ - true if and only if this property shows up during enumeration of the properties on the corresponding object.
Defaults to false.

A data descriptor also has the following optional keys:

- __value__ - The value associated with the property. Can be any valid JavaScript value (number, object, function, etc).
Defaults to undefined.

- __writable__ - true if and only if the value associated with the property may be changed with an assignment operator.
Defaults to false.

- __get__ - A function which serves as a getter for the property. Whatever the function returns will be used as the value of the property. When it is `undefined` the getter behaviour is ignored.
Defaults to undefined.

- __set__ - A function which serves as a setter for the property, or undefined if there is no setter. The function will receive one argument with the new value being assigned to the property.
Defaults to undefined.

--[source](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

Letâs check now how to solve this task by building the `Person` constructor - which for now will have 3 fields: __firstName__, __lastName__ and __species__. The __fullName__ field will be implemented later. As we read in the documentation of `Object.defineProperty` we can add a field to the object and in descriptor set the `writable` attribute to false, which will allow us to protect the value from being overwritten. Let's see how this works in practice:

```JavaScript
var Person = function (first, last) {
  this.firstName = first;
  this.lastName = last;
}

Object.defineProperty(Person, 'species', {
  writable: false,
  value: 'human'
});

var woman = new Person('Tom', 'Khowalski');
woman; // Person {firstName: 'Tom', lastName: 'Khowalski', species: 'human'}

/*
 * Try to overwrite species
 */
 
woman.species = 'fish';
woman.species; // 'human' - We can't change this value.
```

We have now created an attribute which cannot be overwritten. Congratulation on our first success âş

__*Pro tip: never use the defineProperty in constructor method *__

Letâs move on to the second part of our task. We need to define a __fullName__ attribute which always will return the current name.  Also we want to be able to set the value of __fullName__ and have the __lastName__ and __firstName__ attributes update accordingly. By using __get__ and __set__ we can define what will happen with the object and what will be returned. Let's solve this now and see how it works in practice:

```JavaScript
var Person = function (first, last) {
  this.firstName = first;
  this.lastName = last;
};

Object.defineProperty(Person, 'species', {
  writable: false,
  value: 'human'
});

Object.defineProperty(Person, 'fullName', {
  get: function () {
    return this.firstName + ' ' + this.lastName;
  },
  set: function (value) {
    var splitString = value.trim().split(' ');

    if(splitString.length === 2) {
      this.firstName = splitString[0];
      this.lastName = splitString[1];
    }
  }
});

var woman = new Person('Tom', 'Khowalski');

woman.firstName; // 'Tom'
woman.lastName; // 'Khowalski'
woman.fullName; //'Tom Khowalski
woman.species; // human

/*
 * Change name
 */
 
woman.firstName = 'Mat';
woman.firstName; // 'Mat'
woman.lastName; // 'Khowalski'
woman.fullName; // 'Mat Khowalski'
woman.species = 'fish';
woman.species; // human - No, you can't change this properity.
 
 /*
  * Change fullName
  */
  
woman.fullName = 'John Stevens';
woman.firstName; //John
woman.lastName; //Stevens
```

## Support

`Object.defineProperty` is supported by all [modern browsers](http://kangax.github.io/compat-table/es5/#Object.defineProperty) - Internet Explorer 8 supports `Object.defineProperty` only for DOM objects. It is worth mentioning that Firefox has [performance problems](https://bugzilla.mozilla.org/show_bug.cgi?id=626021) but this is only noticeable when we have more than 100k objects.

## Good to know

This is just one of the many hidden gems of ES5. I encourage you to read the documentation of many other interesting methods like `Object.defineProperties()` , `Object.freeze()` , `Object.keys()`.
