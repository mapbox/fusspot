# Validator

Validator lets you validate javascript objects while being lightweight and flexible.

## Table Of Contents

- [Why does this exist?](#why-does-this-exist)
- [Installation](#installation)
- [Usage](#usage)
  * [Basic](#basic)
  * [Required](#required)
  * [Composition](#composition)
- [Assertions](#assertions)
  * [v.assert(rootValidator, options)](#vassertrootvalidator-options)
- [Primitive Validators](#primitive-validators)
  * [v.boolean](#vboolean)
  * [v.number](#vnumber)
  * [v.plainArray](#vplainarray)
  * [v.plainObject](#vplainobject)
  * [v.string](#vstring)
  * [v.date](#vdate)
  * [v.coordinates](#vcoordinates)
- [Higher-Order Validators](#higher-order-validators)
  * [v.shape(validatorObj)](#vshapevalidatorobj)
  * [v.arrayOf(validator)](#varrayofvalidator)
  * [v.required(validator)](#vrequiredvalidator)
  * [v.oneOfType(...validators)](#voneoftypevalidators)
  * [v.equal(value)](#vequalvalue)
  * [v.oneOf(...values)](#voneofvalues)
  * [v.range([valueA, valueB])](#vrangevaluea-valueb)


## Why does this exist?

Many of the existing libraries solve a special problem like form validation or are too big to use (see [hapijs/joi](https://github.com/hapijs/joi)). We wanted something which works similar to react's [prop-types](https://github.com/facebook/prop-types), but not attached to a specific platform. So we ended up creating Validator.

## Installation

```bash
npm install @mapbox/validator
```

## Usage

### Basic

In the example below we have a simple validation of an object and its properties. The outermost validator [`v.shape`](#vshapevalidatorobj) checks the shape of the object and then runs the inner validator [`v.arrayOf(v.string)`](#varrayofvalidator) to validate the value of `names` property. 

```javascript
const v = require("@mapbox/validator");

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

<details><summary>Example 1</summary>

```javascript
const personAssert = v.assert(
  v.shape({
    name: v.required(v.string),
    family: v.arrayOf(
      v.shape({
        name: v.required(v.string),
        relation: v.oneOf("wife", "husband", "son", "daughter")
      })
    )
  })
);

// assertion passes
personAssert({
  name: "john",
  family: [
    {
      name: "susy",
      relation: "wife"
    }
  ]
});

// assertion fails
personAssert({
  name: "harry",
  family: [
    {
      name: "john",
      relation: "father"
    }
  ]
});
// Throws an error
//   family.0.relation must be a "wife", "husband", "son" or "daughter".
```

</details>

<details><summary>Example 2</summary>

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

</details>

## Assertions

### v.assert(rootValidator, options)

Returns a function which accepts an input value to be validated. This function throws an error if validation fails else returns void.

**Parameters**

- `rootValidator`: The root validator to assert values with.
- `options`: An options object.
- `options.apiName`: String to prefix every error message with.

```javascript
v.assert(v.equal(5))(5); // undefined
v.assert(v.equal(5), { apiName: "Validator" })(10); // Error: Validator: value must be a 5.
```

## Primitive Validators

### v.boolean

```javascript
const assert = v.assert(v.boolean);
assert(false); // pass
assert("true"); // fail
```

</details>

### v.number

```javascript
const assert = v.assert(v.number);
assert(9); // pass
assert("str"); // fail
```

</details>

### v.plainArray

```javascript
const assert = v.assert(v.plainArray);
assert([]); // pass
assert({}); // fail
```

</details>

### v.plainObject

```javascript
const assert = v.assert(v.plainObject);
assert({}); // pass
assert(new Map()); // fail
```

</details>

### v.string

```javascript
const assert = v.assert(v.string);
assert("str"); // pass
assert(0x0); // fail
```

</details>

### v.date

```javascript
const assert = v.assert(v.date);
assert(98765); // pass
assert("1969-12-31T23:59:59.997Z"); // pass
assert(new Date()); // pass
assert(false); // fail
assert({}); // fail
```

</details>

### v.coordinates
Passes when input is an `[longitude, latitude]`, where longitude lies inclusively between `[-180, 180]` degrees and inclusively between `[-90, 90]` degrees. 

```javascript
const assert = v.assert(v.coordinates);
assert([150, 60]); // pass
assert([60, 150]); // fail
```

</details>

## Higher-Order Validators

Higher Order Validators are functions that accept another validator or a value as their parameter and return a new validator.

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

### v.arrayOf(validator)

Takes a validator as an argument and returns a validator that passes if and only if every item of the input array passes the validator.

```javascript
const assert = v.assert(v.arrayOf(v.number));
assert([90, 10]); // pass
assert([90, "10"]); // fail
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

### v.range([valueA, valueB])

Returns a validator that passes if input inclusively lies between `valueA` & `valueB`.

```javascript
const assert = v.assert(v.range([-10, 10]));
assert(4); // pass
assert(-100); // fail
```