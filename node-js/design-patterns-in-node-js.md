
## 1. Module Design Pattern/ Immediately Invoked Function Expressions (IIFE)


That allows you to define and call a function at the same time. Due to the way JavaScript scopes works, using IIFEs can be great to simulate things like private properties in classes. In fact, this particular pattern is sometimes used as part of the requirements of other, more complex ones.

### 1.1 What does an IIFE look like?

```js
(function() {
   var x = 20;
   var y = 20;
   var answer = x + y;
   console.log(answer);
})();
```

The template for an IIFE consists of an anonymous function declaration, inside a set of parenthesis (which turn the definition into a function expression, a.k.a an assignment) and then a set of calling parenthesis at the end tail of it. Like so:

```js
(function(/*received parameters*/) {
//your code here
})(/*parameters*/)
```

### 1.2 Use cases

#### Simulating static variables
- Remember static variables
```js
function autoIncrement() {
    static let number = 0
    number++
    return number
}
```
- The above function would return a new number every time we call it
```js
let autoIncrement = (function() {
    let number = 0

    return function () {
     number++
     return number
    }
})()
```

#### Simulating private variables

```js
const autoIncrementer = (function() {
  let value = 0;

  return {
    incr() {
        value++
    },

    get value() {
        return value
    }
  };
})();
```

## 2. Factory pattern

> A factory can also be used as an encapsulation mechanism, thanks to closures.

> Encapsulation refers to the technique of controlling the access to some internal details of an object by preventing the external code from manipulating them directly. The interaction with the object happens only through its public interface, isolating the external code from the changes in the implementation details of the object. This practice is also referred to as information hiding. Encapsulation is also a fundamental principle of object-oriented design, together with inheritance, polymorphism, and abstraction.



In essence, the factory method allows you to centralize the logic of creating objects (meaning, which object to create and why) in a single place. This allows you to forget about that part and focus on simply requesting the object you need and then using it.

### 2.1 What does the factory method pattern look like?

```js
( _ => {

    let factory = new MyEmployeeFactory()

    let types = ["fulltime", "parttime", "contractor"]
    let employees = [];
    for(let i = 0; i < 100; i++) {
     employees.push(factory.createEmployee({type: types[Math.floor( (Math.random(2) * 2) )]})    )}
    
    //....
    employees.forEach( e => {
     console.log(e.speak())
    })

})()
```

You can now look at the actual implementation, as you can see, there is a lot to look at, but it’s quite straightforward:

```js
class Employee {
    speak() {
     return "Hi, I'm a " + this.type + " employee"
    }

}

class FullTimeEmployee extends Employee{
    constructor(data) {
     super()
     this.type = "full time"
     //....
    }
}

class PartTimeEmployee extends Employee{
    constructor(data) {
     super()
     this.type = "part time"
     //....
    }
}

class ContractorEmployee extends Employee{
    constructor(data) {
     super()
     this.type = "contractor"
     //....
    }
}

class MyEmployeeFactory {

    createEmployee(data) {
     if(data.type == 'fulltime') return new FullTimeEmployee(data)
     if(data.type == 'parttime') return new PartTimeEmployee(data)
     if(data.type == 'contractor') return new ContractorEmployee(data)
    }
}
```

### 2.2 Use case

the previous code already shows a generic use case, but if we wanted to be more specific, one particular use case I like to use this pattern for is handling error object creation.

Imagine having an Express application with about 10 endpoints, wherein every endpoint you need to return between two to three errors based on the user input. We’re talking about 30 sentences like the following:

```js
if(err) {
  res.json({error: true, message: “Error message here”})
}
```

Now, that wouldn’t be a problem, unless of course, until the next time you had to suddenly add a new attribute to the error object. Now you have to go over your entire project, modifying all 30 places. And that would be solved by moving the definition of the error object into a class. That would be great unless of course, you had more than one error object, and again, you’re having to decide which object to instantiate based on some logic only you know. See where I’m trying to get to?

If you were to centralize the logic for creating the error object then all you’d have to do throughout your code would be something like:

```js
if(err) {
  res.json(ErrorFactory.getError(err))
}
```

## 3. Singleton pattern

This one is another oldie but a goodie. It’s quite a simple pattern, mind you, but it helps you keep track of how many instances of a class you’re instantiating. Actually, it helps you keep that number to just one, all of the time. Mainly, the singleton pattern, allows you to instantiate an object once, and then use that one every time you need it, instead of creating a new one without having to keep track of a reference to it, either globally or just passing it as a dependency everywhere.

### 3.1. What does the singleton pattern look like?

Normally, other languages implement this pattern using a single static property where they store the instance once it exists. The problem here is that, as I mentioned before, we don’t have access to static variables in JS. So we could implement this in two ways, one would be by using IIFEs instead of classes.

The other would be by using ES6 modules and having our singleton class using a locally global variable, in which to store our instance. By doing this, the class itself gets exported out of the module, but the global variable remains local to the module.

I know, but trust me, it sounds a lot more complicated than it looks:

```js
let instance = null

class SingletonClass {

    constructor() {
     this.value = Math.random(100)
    }

    printValue() {
     console.log(this.value)
    }

    static getInstance() {
     if(!instance) {
         instance = new SingletonClass()
     }

     return instance
    }
}

module.exports = SingletonClass
```

And you could use it like this:

```js
const Singleton = require(“./singleton”)
const obj = Singleton.getInstance()
const obj2 = Singleton.getInstance()

obj.printValue()
obj2.printValue()

console.log("Equals:: ", obj === obj2)

```
It does not matter how many times you will require this module in your application; it will only exist as a single instance.



### 3.2. Use cases

When trying to decide if you need a singleton-like implementation or not, you need to consider something: how many instances of your classes will you really need? If the answer is 2 or more, then this is not your pattern.

But there might be times when having to deal with database connections that you might want to consider it.

Think about it, once you’ve connected to your database, it might be a good idea to keep that connection alive and accessible throughout your code. Mind you, this can be solved in a lot of different ways, yes, but this pattern is indeed, one of them.

Using the above example, we can extrapolate it into something like this:

```js
const driver = require("...")

let instance = null


class DBClass {

    constructor(props) {
     this.properties = props
     this._conn = null
    }

    connect() {
     this._conn = driver.connect(this.props)
    }

    get conn() {
     return this._conn
    }

    static getInstance() {
     if(!instance) {
         instance = new DBClass()
     }

     return instance
    }
}

module.exports = DBClass
```
And now, you’re sure that no matter where you are if you’re using the getInstance method, you’ll be returning the only active connection (if any).



## 4. Observer pattern

This one is a very interesting pattern, in the sense that it allows you to respond to certain input by being reactive to it, instead of proactively checking if the input is provided. In other words, with this pattern, you can specify what kind of input you’re waiting for and passively wait until that input is provided in order to execute your code. It’s a set and forget kind of deal, if you will.

In here, the observers are your objects, which know the type of input they want to receive and the action to respond with, these are meant to “observe” another object and wait for it to communicate with them.

The observable, on the other hand, will let the observers know when a new input is available, so they can react to it, if applicable. If this sounds familiar, it’s because it is, anything that deals with events in Node is implementing this pattern.

### 4.1. What does the observer pattern look like?

Have you ever written your own HTTP server? Something like this:

```js
const http = require('http');


const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Your own server here');
});

server.on('error', err => {
    console.log(“Error:: “, err)
})

server.listen(3000, '127.0.0.1', () => {
  console.log('Server up and running');
});
```

There, hidden in the above code, you’re looking at the observer pattern in the wild. An implementation of it, at least. Your server object would act as the observable, whilst your callback function is the actual observer. The event-like interface here (see the bolded code), with the on method, and the event name there might obfuscate the view a bit, but consider the following implementation:


```js
class Observable {

    constructor() {
     this.observers = {}
    }

    on(input, observer) {
     if(!this.observers[input]) this.observers[input] = []
     this.observers[input].push(observer)
    }

    triggerInput(input, params) {
     this.observers[input].forEach( o => {
         o.apply(null, params)    
     })
    }
}

class Server extends Observable {

    constructor() {
     super()
    }


    triggerError() {
     let errorObj = {
         errorCode: 500,
         message: 'Port already in use'
     }
     this.triggerInput('error', [errorObj])
    }
}
```

You can now, again, set the same observer, in exactly the same way:

```js
server.on('error', err => {
    console.log(“Error:: “, err)
})
```

And if you were to call the triggerError method (which is there to show you how you would let your observers know that there is new input for them), you’d get the exact same output:

```
Error:: { errorCode: 500, message: 'Port already in use' }

```

> If you were to be considering using this pattern in Node.js, please look at the [EventEmitter][01] object first, since it’s Node.js’ own implementation of this pattern, and might save you some time.

### 4.2. Use cases

This pattern is, as you might have already guessed, great for dealing with asynchronous calls, since getting the response from an external request can be considered a new input. And what do we have in Node.js, if not a constant influx of asynchronous code into our projects? So next time you’re having to deal with an async scenario consider looking into this pattern.

Another widely spread use case for this pattern, as you’ve seen, is that of triggering particular events. This pattern can be found on any module that is prone to having events triggered asynchronously (such as errors or status updates). Some examples are the HTTP module, any database driver, and even socket.io, which allows you to set observers on particular events triggered from outside your own code.

## 5. Chain of responsibility

The chain of responsibility pattern is one that many of use in the world of Node.js have used, without even realizing it.

### 5.1 What does the chain of responsibility look like?

```js
function processRequest(r, chain) {

    let lastResult = null
    let i = 0
    do {
     lastResult = chain[i](r)
     i++
    } while(lastResult != null && i < chain.length)
    
    if(lastResult != null) {
     console.log("Error: request could not be fulfilled")
    }
}

let chain = [
    function (r) {
     if(typeof r == 'number') {
         console.log("It's a number: ", r)
         return null
     }
     return r
    },
    function (r) {
     if(typeof r == 'string') {
         console.log("It's a string: ", r)
         return null
     }
     return r
    },
    function (r) {
     if(Array.isArray(r)) {
         console.log("It's an array of length: ", r.length)
         return null
     }
     return r
    }
]

processRequest(1, chain)
processRequest([1,2,3], chain)
processRequest('[1,2,3]', chain)
processRequest({}, chain)
```

```console
It's a number:  1
It's an array of length:  3
It's a string:  [1,2,3]
Error: request could not be fulfilled
```

### 5.1. Use cases

The most obvious case of this pattern in our ecosystem is the middlewares for ExpressJS. With that pattern, you’re essentially setting up a chain of functions (middlewares) that evaluate the request object and decide to act on it or ignore it. You can think of that pattern as the asynchronous version of the above example, where instead of checking if the function returns a value or not, you’re checking what values are passed to the next callback they call.

```js
var app = express();

app.use(function (req, res, next) {
  console.log('Time:', Date.now());
  next(); //call the next function on the chain
});
```

Middlewares are a particular implementation of this pattern since instead of only one member of the chain fulfilling the request, one could argue that all of them could do it. Nevertheless, the rationale behind it is the same.


## 6. Proxy pattern

A proxy is an object that controls access to another object, called subject. Both have identical interface and this allows us to transparently swap one for the other. A proxy intercepts all or some of the operations that are meant to be executed on the subject, modifying their behavior.

A proxy is useful to several circumstances like data validation, security, caching, lazy loading, logging or making remote object appear like local.

There are some techniques for implementing proxies like object augmentation, object composition or just use the Proxy object introduced in ES6. Composition can be considered the safest way of creating a proxy as it doesn’t modify the subject so we are going to use it in the code example.

```js
const createProxy = subject => ({
    // proxied method
    hello:() => `${subject.hello()} world!`,

    // delegated method
    goodbye:() => subject.goodbye.apply(subject, arguments)
});

const foo = {
    hello : () => "Hello foo",
    goodbye: () => "Goodbye foo"
};

const proxy = createProxy(foo);
console.log(proxy.hello)); // Hello foo world!
console.log(proxy.goodbye)); // Goodbye foo
```

## 7. Decorator pattern

Decorator is a structural pattern that consists of dynamically augmenting the behavior of an existing object. It’s different from classical inheritance, because the behavior is not added to all the objects of the same class but only to the instances that are explicitly decorated.

Implementation-wise, it is very similar to the Proxy pattern, but instead of enhancing or modifying the behavior of the existing interface of an object, it augments it with new functionalities, as described in the following figure:

It’s pretty similar to proxy, but instead of enhancing or modifying the behavior, it augments with new functionalities. The difference with inheritance is that decorator doesn’t add the behavior to all the objects of the same class, only to the instances that are decorated. Like in the proxy patterns, composition and object augmentation are the main techniques.


```js

const User = function( name) {
    this.name = name;
    this.say = function() {
        console. log( `User: ${thts . name}` );
    };
};

const DecoratedUser = function(user, address, city) {
    this.user = user;
    this.name = user.name; // ensures interface stays the same
    this.address = address;
    this.city = city;
    this.say = function() {
        console.log(`Decorated User: ${this.name} , ${this.address}, ${this.city}` );
    };
};

const user = new User( "Eli Manning" );
user.say() ; // User: Eli Manning
const decorated = new DecoratedUser(user, "Met life Stadium" , "New Jersey" ) ;
decorated.say( ) ;
// Decorated User: Elt Manning, MetLife Stadium, New Jersey
```

The DecoratedUser receives two arguments that help to modify the say function. The structure is pretty similar but the method behavior has changed.

## 8. Middleware/pipelines pattern

It represents a processing pipeline similar to streams. In Node, this pattern is used well beyond the boundaries of the Express framework. It can be a set of processing units, handlers, filters, and any function to perform the preprocessing and post processing of any kind of data. It gives flexibility, in fact, allows us to obtain a plugin infrastructure with little effort, providing an unobtrusive way for extending a system with new filters and handlers.

![img-01]

In this representation, we have an incoming request, that, begore getting into the core of aour app, traverses a number of middlewares. This part of the flow is called inbound or downstream. After the flow reaches the core app, it traverses again all the middlewares in inverse order, so they can perform actions after app logic was executed. This part is known as outbound or upstream.

## 9. Command pattern

This is other pattern with huge importance in Node. A Command is any object that encapsulates all the information necessary to perform an action at a later time. So, instead of invoking a function directly, we create an object representing the intention to perform such an invocation. It’s built around four major components that can vary depending on the way we want to implement the pattern:
- Command: The object encapsulating the information necessary to invoke a function.
- Client: This creates the command and provides it to the invoker.
- Invoker: This is responsible for executing the command on the target.
- Target: Subject of the invocation. It can be a lone function or the method of an object.

Using a command pattern has several advantages. Some of them are:
- A command can be scheduled to be run at a later time
- Keeps a history of operations executed in the system
- Can be serialized and sent over the network so can be distributed.
- A command scheduled can be cancelled if not executed yet


Command pattern


## 10. Prototype Design Pattern

## 11. Strategy

> Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

![img-02]

It allows a strategy (algorithm) to be swapped out at runtime by any strategy without the client realizing it.

In Express, the Strategy pattern can be seen in the way Express supports various different template engines such as Pug, Mustache, EJS, swig, etc.. Take a look at [here](02) for the full list of supported engines

```js
import express from 'express';
import exphbs from 'express-handlebars';
...
const app = express();
app.engine('.html', exphbs({...}));
app.engine('jade', require('jade').__express);
//Select the strategy '.html'
app.set('view engine', '.html');
```

## Referenced
--------------------

[02]: https://github.com/expressjs/express/wiki?_ga=1.216495568.777274470.1463719254#template-engines

[img-02]: images/strategy.png
[img-01]: images/1_dWMuOSIJsuU27gZfuxJwRQ.png
[01]: https://nodejs.org/api/events.html
[source]: https://blog.logrocket.com/design-patterns-in-node-js/
[source]: https://itnext.io/design-patterns-in-nodejs-990fed17c49c
[source]: https://blog.risingstack.com/fundamental-node-js-design-patterns/
[source]: https://hub.packtpub.com/introduction-nodejs-design-patterns/
[source]: http://expressjs.com/en/guide/using-middleware.html#using-middleware
[source]: https://dzone.com/articles/design-patterns-in-expressjs
[s]: https://itnext.io/a-new-and-better-mvc-pattern-for-node-express-478a95b09155