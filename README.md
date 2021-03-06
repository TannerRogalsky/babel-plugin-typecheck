# Babel Typecheck

This is a [Babel](https://babeljs.io/) plugin for static and runtime type checking using [flow type](http://flowtype.org/) annotations.

[![Build Status](https://travis-ci.org/codemix/babel-plugin-typecheck.svg)](https://travis-ci.org/codemix/babel-plugin-typecheck)

> Note: Now requires babel 6.1, babel 5 users [see the 2.0 tag](https://github.com/codemix/babel-plugin-typecheck/tree/2.0.0).

# What?

Turns code like this:
```js
function sendMessage (to: User, message: string): boolean {
  return socket.send(to, message);
}
```
into code like this:
```js
function sendMessage(to, message) {
  var _socket$send;

  if (!(to instanceof User)) throw new TypeError("Value of argument 'to' violates contract.");
  if (typeof message !== "string") throw new TypeError("Value of argument 'message' violates contract.");
  _socket$send = socket.send(to, message);
  if (typeof _socket$send !== "boolean") throw new TypeError("Function 'sendMessage' return value violates contract.");
  return _socket$send;
}
```

And guards against some silly mistakes, for example the following code will fail to compile with a `SyntaxError`, because the function can return the wrong type.

```js
function foo (): boolean {
  if (Math.random() > 0.5) {
    return "yes"; // <-- SyntaxError - string is not boolean
  }
  else {
    return false;
  }
}

function bar (input: string = 123): string { // <-- SyntaxError: default value is not string
  return input + "456";
}
```

# Examples

The basic format is similar to [Flow Type Annotations](http://flowtype.org/docs/type-annotations.html).

Here are a few examples of annotations this plugin supports:

```js
function foo(
    aNum: number,
    anOptionalString: ?string, // will allow null/undefined
    anObject: Object,
    aDate: Date,
    anError: Error,
    aUnionType: Object|string,
    aClass: User,
    aShape: {foo: number, bar: ?string},
    anArray: Array,
    arrayOf: string[] | Array<string>,
    {x, y}: {x: string, y: number}, // destructuring works
    es6Defaults: number = 42
) : number {
  return aNum;
}
```

# Optimization

In cases where typecheck can statically verify that the return value is of the correct type, no type checks will be inserted, for instance:
```js
function bar (): string|Object {
  if (Math.random() > 0.5) {
    return "yes";
  }
  else {
    return {
      message: "no"
    };
  }
}
```
will produce no type checks at all, because we can trivially tell that the function can only return one of the two permitted types.
This is also true for simple cases like:
```js
function createUser (): User {
  return new User(); // <-- no typecheck required
}
```
This is currently quite limited though, as the plugin can only statically infer the types of literals and very simple expressions, it can't (yet) statically verify e.g. the result of a function call. In those cases a runtime type check is required:
```js
function createUser (): User {
  return User.create(); // <-- produces runtime typecheck
}
```


## Changes in 3.0.0

### Supports type aliases:
```js
type Foo = string|number;

function demo (input: Foo): string {
  return input + '  world';
}

demo('hello'); // ok
demo(123); // ok
demo(["not", "a", "Foo"]); // fails
```

### Better static type inference
```js
function demo (input: string): string[] {
  return makeArray(input); // no return type check required, knows that makeArray is compatible
}

function makeArray (input: string): string[] {
  return [input];
}
```

### Type propagation
```js
function demo (input: string): User {
  const user = new User({name: input});
  return user; // No check required, knows that user is the correct type
}
```

### Assignment tracking
```js
let name: string = "bob";

name = "Bob"; // ok
name = makeString(); // ok
name = 123; // SyntaxError, expected string not number

function makeString (): string {
  return "Sally";
}
```

### Type casting
```js
let name: string = "bob";

name = "Bob";
((name: number) = 123);
name = 456;
name = "fish"; // SyntaxError, expected number;
```

### Array type parameters
```js
function demo (input: string[]): number {
  return input.length;
}

demo(["a", "b", "c"]); // ok
demo([1, 2, 3]); // TypeError
```

### Shape tracking
```js
type User = {
  name: string;
  email: string;
};

function demo (input: User): string {
  return input.name;
}

demo({}); // TypeError
demo({name: 123, email: "test@test.com"}); // TypeError
demo({name: "test", email: "test@test.com"}); // ok
```

# Installation

First, install via [npm](https://npmjs.org/package/babel-plugin-typecheck).
```sh
npm install --save-dev babel-plugin-typecheck
```
Then, in your babel configuration (usually in your `.babelrc` file), add `"typecheck"` to your list of plugins:
```json
{
  "plugins": ["typecheck"]
}
```


# License

Published by [codemix](http://codemix.com/) under a permissive MIT License, see [LICENSE.md](./LICENSE.md).

