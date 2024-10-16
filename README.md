# @mapbox/fusspot

![build](https://github.com/mapbox/fusspot/actions/workflows/ci.yml/badge.svg?branch=master)

Fusspot is a tiny runtime type-assertion library.

It can run in the browser as well as Node, and it's lightweight, flexible, and extensible.

## Table Of Contents

- [Why does this exist?](#why-does-this-exist)
- [Installation](#installation)
- [Usage](#usage)
  - [Basic](#basic)
  - [Required](#required)
  - [Composition](#composition)
- [Assertions](#assertions)
  - [v.assert(rootValidator, options)](#vassertrootvalidator-options)
- [Primitive Validators](#primitive-validators)
  - [v.any](#vany)
  - [v.boolean](#vboolean)
  - [v.number](#vnumber)
  - [v.finite](#vfinite)
  - [v.plainArray](#vplainarray)
  - [v.plainObject](#vplainobject)
  - [v.func](#vfunc)
  - [v.date](#vdate)
  - [v.string](#vstring)
  - [v.nonEmptyString](#vnonemptystring)
- [Higher-Order Validators](#higher-order-validators)
  - [v.shape(validatorObj)](#vshapevalidatorobj)
  - [v.strictShape(validatorObj)](#vstrictshapevalidatorobj)
  - [v.objectOf(validator)](#vobjectofvalidator)
  - [v.arrayOf(validator)](#varrayofvalidator)
  - [v.tuple(...validators)](#vtuplevalidators)
  - [v.required(validator)](#vrequiredvalidator)
  - [v.oneOfType(...validators)](#voneoftypevalidators)
  - [v.equal(value)](#vequalvalue)
  - [v.oneOf(...values)](#voneofvalues)
  - [v.range(\[valueA, valueB\])](#vrangevaluea-valueb)
  - [v.instanceOf(Class)](#vinstanceofclass)
- [Custom validators](#custom-validators)
  - [Simple](#simple)
  - [Customizing the entire error message](#customizing-the-entire-error-message)

## Why does this exist?

Many existing runtime type-assertion libraries solve a special problem, like form validation or component props, or aren't great for usage in the browser, because of size or syntax. We wanted something similar to React's [prop-types](https://github.com/facebook/prop-types) but not attached to a specific use case. So we ended up creating Fusspot.

## Installation

```bash
npm install @mapbox/fusspot
```

## Usage

### Basic

In the example below we have a simple validation of an object and its properties. The outermost validator [`v.shape`](#vshapevalidatorobj) checks the shape of the object and then runs the inner validator [`v.arrayOf(v.string)`](#varrayofvalidator) to validate the value of `names` property.

**`@mapbox/fusspot` exports a single object for its API. In the examples below we name it `v` (for "validator").**

```javascript
const v = require("@mapbox/fusspot");

assertObj = v.assert(
  v.shape({
    names: v.arrayOf(v.string)
  })
);

assertObj({ names: ["ram", "harry"] }); // pass
assertObj({ names: ["john", 987] }); // fail
assertObj({ names: "john" }); // fail
```

### Required

By default `null` and `undefined` are acceptable values for all validators. To not allow `null`/`undefined` as acceptable values, you can pass your validator to [`v.required`](#vrequiredvalidator) to create a new validator which rejects `undefined`/`null`.

```javascript
// without v.required
assertObj = v.assert(
  v.shape({
    name: v.string
  })
);
assertObj({}); // pass
assertObj({ name: 'ram' }); // pass
assertObj({ name: undefined }); // pass
assertObj({ name: null }); // pass
assertObj({ name: 9 }); // fail

// with v.required
strictAssertObj = v.assert(
  v.shape({
    name: v.required(v.string)
  })
);

strictAssertObj({}); // fail
strictAssertObj({ name: 'ram' }); // pass
strictAssertObj({ name: undefined }); // fail
strictAssertObj({ name: null }); // fail
strictAssertObj({ name: 9 }); // fail
```

### Composition

You can compose any of the [Higher-Order Validators](#higher-order-validators) to make complex validators.

```javascript
const personAssert = v.assert(
  v.shape({
    name: v.required(v.string),
    pets: v.arrayOf(
      v.shape({
        name: v.required(v.string),
        type: v.oneOf("dog", "pig", "cow", "bird")
      })
    )
  })
);

// assertion passes
personAssert({
  name: "john",
  pets: [
    {
      name: "susy",
      type: "bird"
    }
  ]
});

// assertion fails
personAssert({
  name: "harry",
  pets: [
    {
      name: "john",
      type: "mechanic"
    }
  ]
});
// Throws an error
//   pets.0.type must be "dog", "pig", "cow", or "bird".
```

```javascript
const personAssert = v.assert(
  v.shape({
    prop: v.shape({
      person: v.shape({
        name: v.oneOfType(v.arrayOf(v.string), v.string)
      })
    })
  })
);

// assertion passes
personAssert({ prop: { person: { name: ["j", "d"] } } });
personAssert({ prop: { person: { name: ["jd"] } } });

// assertion fails
personAssert({ prop: { person: { name: 9 } } });
// Throws an error
//   prop.person.name must be an array or string.
```

## Assertions

### v.assert(rootValidator, options)

Returns a function which accepts an input value to be validated. This function throws an error if validation fails else returns void.

**Parameters**

- `rootValidator`: The root validator to assert values with.
- `options`: An options object or a string. If it is a string, it will be interpreted as `options.description`.
- `options.description`: A string to prefix every error message with. For example, if `description` is `myFunc` and a string is invalid, the error message with be `myFunc: value must be a string`. (Formerly `options.apiName`, which works the same but is deprecated.)

```javascript
v.assert(v.equal(5))(5); // undefined
v.assert(v.equal(5), { description: "myFunction" })(10); // Error: myFunction: value must be 5.
v.assert(v.equal(5), 'Price')(10); // Error: Price: value must be 5.
```

## Primitive Validators

### v.any

This is mostly useful in combination with a higher-order validator like [`v.arrayOf`](#varrayofvalidator);

```javascript
const assert = v.assert(v.any);
assert(false); // pass
assert("str"); // pass
assert(8); // pass
assert([1, 2, 3]); // pass
```

### v.boolean

```javascript
const assert = v.assert(v.boolean);
assert(false); // pass
assert("true"); // fail
```

### v.number

```javascript
const assert = v.assert(v.number);
assert(9); // pass
assert("str"); // fail
```

### v.finite

```javascript
const assert = v.assert(v.finite);
assert(9); // pass
assert("str"); // fail
assert(NaN); // fail
```

### v.plainArray

```javascript
const assert = v.assert(v.plainArray);
assert([]); // pass
assert({}); // fail
```

### v.plainObject

```javascript
const assert = v.assert(v.plainObject);
assert({}); // pass
assert(new Map()); // fail
```

### v.func

```javascript
const assert = v.assert(v.func);
assert('foo'); // fail
assert({}); // fail
assert(() => {}); // pass
```

### v.date

Passes if input is a `Date` that is valid (`input.toString() !== 'Invalid Date'`).

```javascript
const assert = v.assert(v.date);
assert('foo'); // fail
assert(new Date('2019-99-99')); // fail
assert(new Date()); // pass
assert(new Date('2019-10-04')); // pass
```

### v.string

```javascript
const assert = v.assert(v.string);
assert("str"); // pass
assert(0x0); // fail
```

### v.nonEmptyString

```javascript
const assert = v.assert(v.nonEmptyString);
assert("str"); // pass
assert(""); // fail
assert(7); // fail
```

## Higher-Order Validators

Higher-Order Validators are functions that accept another validator or a value as their parameter and return a new validator.

### v.shape(validatorObj)

Takes an object of validators and returns a validator that passes if the input shape matches.

```javascript
const assert = v.assert(
  v.shape({
    name: v.required(v.string),
    contact: v.number
  })
);

// pass
assert({
  name: "john",
  contact: 8130325777
});

// fail
assert({
  name: "john",
  contact: "8130325777"
});
```

### v.strictShape(validatorObj)

The same as [`v.shape`](#vshapevalidatorobj), but rejects the object if it contains any properties that are not defined in the schema.

```javascript
const assert = v.assert(
  v.strictShape({
    name: v.required(v.string),
    contact: v.number
  })
);

// passes, just like v.shape
assert({
  name: "john",
  contact: 8130325777
});

// fails, just like v.shape
assert({
  name: "john",
  contact: "8130325777"
});

// fails where v.shape would pass, because birthday is not defined in the schema
assert({
  name: "john",
  birthday: '06/06/66'
});
```

### v.objectOf(validator)

Takes a validator as an argument and returns a validator that passes if and only if every value in the input object passess the validator.

```javascript
const assert = v.assert(
  v.objectOf({ name: v.required(v.string) })
);

// pass
assert({
  a: { name: 'foo' },
  b: { name: 'bar' }
});

// fail
assert({
  a: { name: 'foo' },
  b: 77
});
```

### v.arrayOf(validator)

Takes a validator as an argument and returns a validator that passes if and only if every item of the input array passes the validator.

```javascript
const assert = v.assert(v.arrayOf(v.number));
assert([90, 10]); // pass
assert([90, "10"]); // fail
assert(90); // fail
```

### v.tuple(...validators)

Takes multiple validators that correspond to items in the input array and returns a validator that passes if and only if every item of the input array passes the corresponding validator.

A "tuple" is an array with a fixed number of items, each item with its own specific type. One common example of a tuple is coordinates described by a two-item array, `[longitude, latitude]`.

```javascript
const assert = v.assert(v.tuple(v.range(-180, 180), v.range(-90, 90)));
assert([90, 10]); // pass
assert([90, "10"]); // fail
assert([90, 200]); // fail
assert(90); // fail
```

### v.required(validator)

Returns a strict validator which rejects `null`/`undefined` along with the validator.

```javascript
const assert = v.assert(v.arrayOf(v.required(v.number)));
assert([90, 10]); // pass
assert([90, 10, null]); // fail
assert([90, 10, undefined]); // fail
```

### v.oneOfType(...validators)

Takes multiple validators and returns a validator that passes if one or more of them pass.

```javascript
const assert = v.assert(v.oneOfType(v.string, v.number));
assert(90); // pass
assert("90"); // pass
```

### v.equal(value)

Returns a validator that does a `===` comparison between `value` and `input`.

```javascript
const assert = v.assert(v.equal(985));
assert(985); // pass
assert(986); // fail
```

### v.oneOf(...values)

Returns a validator that passes if input matches (`===`) with any one of the `values`.

```javascript
const assert = v.assert(v.oneOf(3.14, "3.1415", 3.1415));
assert(3.14); // pass
assert(986); // fail
```

### v.range(\[valueA, valueB])

Returns a validator that passes if input inclusively lies between `valueA` & `valueB`.

```javascript
const assert = v.assert(v.range([-10, 10]));
assert(4); // pass
assert(-100); // fail
```

### v.instanceOf(Class)

Returns a validator that passes if input is an instance of the provided `Class`, as determined by JavaScript's `instanceof` operator.

```javascript
class Foo {}
class Bar {}
class Baz extends Bar {}

const assert = v.assert(v.instanceOf(Bar))
assert(new Bar()); // pass
assert(new Baz()); // pass
assert(new Foo()); // fail
```

## Custom validators

One of the primary goals of Fusspot is to be customizable out of the box. There are multiple ways to which one can create a custom validator. After creating a custom validator you can simply use it just like a regular validator i.e. pass it to `v.assert()` or use it with [Higher-Order Validators](#higher-order-validators).

### Simple

A simple custom validator is a function which accepts the input `value` and returns a `string` if and only if the input `value` doesn't pass the test. This `string` should be a noun phrase describing the expected value type,  which would be inserted into the error message like this `value must be a(n) <returned_string>`. Below is an example of a path validator for node environment.

```javascript
const path = require('path');

function validateAbsolutePaths(value) {
  if (typeof value !== 'string' || !path.isAbsolute(value)) {
    return 'absolute path';
  }
}

const assert = v.assert(validateAbsolutePaths);
assert('../Users'); // fail
// Error: value must be an absolute path.
assert('/Users'); // pass

**For more examples look at the [src code](https://github.com/mapbox/fusspot/blob/master/lib/index.js#L238).**
```

### Customizing the entire error message

 If you need more control over the error message, your validator can return a function `({path}) => '<my_custom_error_message>'` for custom messages, where `path` is an array containing the path **(property name for objects and index for arrays)** needed to traverse the input object to reach the value. The example below help illustrate this feature.

```javascript
function validateHexColour(value) {
  if (typeof value !== "string" || !/^#[0-9A-F]{6}$/i.test(value)) {
    return ({ path }) =>
      `The input value '${value}' at ${path.join(".")} is not a valid hex colour.`;
  }
}

const assert = v.assert(
  v.shape({
    colours: v.arrayOf(validateHexColour)
  })
);

assert({ colours: ["#dedede", "#eoz"] }); // fail
// Error: The input value '#eoz' at colours.1 is not a valid hex colour.
assert({ colours: ["#abcdef"] }); // pass
```
