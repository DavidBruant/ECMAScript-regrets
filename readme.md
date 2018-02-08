ECMAScript Regrets
==================

Learning from mistakes of the past.

# Foreword

## Web technologies are ugly and there is no way back...

Web technologies follow this path: they are implemented, some content use some technology. Whether the technology (language, API...) is good or not, consistent or not does not matter, people use them as they are. And consequently rely on the technologies as they are. This makes impossible to change the technology afterward without breaking the content that relies on the technology (a.k.a. "breaking the web"), so all technologies stay mostly as they are. It doesn't mean they are good. It just means it prevents some content to break.
        

## ...but there is a way forward

Existing web technologies cannot be changed, but new technologies can be created.

The ECMA TC39 committee is also making an excellent work at improving the language by adding things (since nothing can be removed). Progress can be followed on the [wiki](http://wiki.ecmascript.org/) or on [es-discuss](https://mail.mozilla.org/listinfo/es-discuss).

Recently, to "work around" JavaScript, we've seen the rise of languages that compile down to JavaScript like [CoffeeScript](http://coffeescript.org/) or [Roy](http://roy.brianmckenna.org/).
(Add a list like http://altjs.org/ or https://github.com/jashkenas/coffee-script/wiki/List-of-languages-that-compile-to-JS )


## Documenting history and educating those who arrived late to the party

This page aims at listing things that experienced JavaScript developers wish were removed or different in the language. The goal here is to explain to newcomers why things are as they are and what are the traps to avoid. For those who'd be tempted by creating a language that compiles down to JavaScript, this page aims also at listing errors that should not be reproduced.


# Regrets

## ````typeof null```` shouldn't be ````"object"````

The first implementation of JavaScript, name-coded "Mocha" represented value with a type tag and the value itself. The type tag was in 3 bits to discriminate the 6 types. The type tag value for objects was 0. Meanwhile, the ````null```` JavaScript value was represented in C++ as the ````NULL```` pointer, which in most architectures is represented as the 0 value (all bytes at 0x00), mistakenly making the ````null```` value having a type tag of 0.
[Mocha was never open sourced](https://twitter.com/BrendanEich/status/226310723691741185), so there is no trace of this explanation besides people sharing this story.

### In your code

To discriminate whether a value is ````null```` or an object, ````typeof```` is unreliable.

````javascript
// test whether a value is equal to null:
value === null

// test whether a value is of type object
value === Object(value) // this test doesn't differentiate functions and non-callable objects.

// to test whether a value is a function:
typeof value === 'function'

````


## ````==```` as a comparison operator

This operator does type coercion before comparing. Conversions are implicit and rules are not always obvious or consistent. Using the ````==```` is consequently considered as a bad practice. Always use ````===````. The only case where ````==```` is tolerated is in the following case:

````javascript
val == null // Test whether value is undefined or null
````


## Wrong 'this' in callbacks

Consider the following case:
````javascript
function Person(){
    this.age = 0;

    setInterval(function growUp(){
        this.age++;
    }, 1000);
}

var p = new Person();
````

In the ````growUp```` function above, we'd expect ````this.age```` to refer to the person's age. It is not the case; in JavaScript, every function defines its own ````this```` parameter (or, in other words, every function is also a method). Thus, the ````growUp```` function's ````this```` name shadows the ````Person```` function's ````this```` name in the lexical scope.

In non-strict mode, by default, ````this```` refers to the global object (with implicit conversion which makes the bug hard to track down in this case). In strict mode, the default value is ````undefined````, so the above code will throw a TypeError when trying to access the ````age```` property of ````undefined````.


### In your code

To work around this problem, two solutions are usually used:
````javascript
// 1) An alias to |this| with some name that isn't shadowed

function Person(){
    var self = this; // some choose 'that' instead of 'self'. Choose one and be consistent
    self.age = 0;

    setInterval(function growUp(){
         // the callback refers to the self variable which value is the expected object
        self.age++;
    }, 1000);
}


// 2) bind the callback function

function Person(){
    this.age = 0;

    setInterval(function growUp(){
        this.age++;
    }.bind(this), 1000);
}

// The setInterval function is bound and its |this| value is the |this| value
// of the enclosing context.
// Function.prototype.bind has been introduced in ECMAScript 5 and is available
// in Internet Explorer 9+. 
// Documentation and polyfill on MDN: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function/bind
````


### ES.next to the rescue

(Talk about arrow functions)


# See also

Axel fluent talk http://www.youtube.com/watch?v=ifFCZQlndGw






