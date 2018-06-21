# Validator

A tiny library for run time type assertions.

## Installation

```bash
npm install @mapbox/validator
```

## Usage

### Basic
The basic usage 

```javascript
const v = require('@mapbox/validator');

assertObj = v.assert(
  v.shape({
      names: v.arrayOf(v.string)
  })
);

assertObj({names: ['ram', 'harry']}); // pass
assertObj({names: ['john', 987]})// fail
assertObj({names: 'john'})// fail
```

### Required

Validation of each value is optional by default, which means passing `null` or `undefined` would not fail the assertion. You should wrap your validator with `v.required` in case you want strict assertion.

```javascript
assertObj = v.assert(
  v.shape({
      name: v.string
  })
);
assertObj({}); // pass
assertObj({name: undefined}); // pass
assertObj({name: null}); // pass
assertObj({name: 9}); // fail

strictAssertObj = v.assert(
     v.shape({
      name: v.required(v.string)
  })
)
strictAssertObj({}); // fail
strictAssertObj({name: undefined}); // fail
strictAssertObj({name: null}); // fail
```

### Composition
You can compose any of the [Higher Order Validators](#higher-order-validators) to make complex validators.
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
      relation: "father" // family.0.relation must be a "wife", "husband", "son" or "daughter".
    }
  ]
});
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
personAssert({ prop: { person: { name: ['j', 'd'] } } });
personAssert({ prop: { person: { name: ['jd'] } } });

// assertion fails
personAssert({ prop: { person: { name: 9 } } }); // prop.person.name must be an array or string.
```
</details>


## Assertions

### v.assert(rootValidator, options)

Returns a function which accepts a value and runs the assertion on it.

**Parameters**
- `rootValidator`: The root validator to assert values with.
- `options`: An options object.
- `options.apiName`: String to prefix every error message with.

```javascript
v.assert(v.equal(5), { apiName: "Validator" })(10); // Error: Validator: value must be a 5.
```

### Primitive Validators

<details><summary>v.boolean</summary>

```javascript
const assert = v.assert(
    v.boolean
);
assert(false); // pass
assert('true'); // fail
```
</details>

<details><summary>v.number</summary>

```javascript
const assert = v.assert(
    v.number
);
assert(9); // pass
assert('str'); // fail
```
</details>

<details><summary>v.plainArray</summary>

```javascript
const assert = v.assert(
    v.plainArray
);
assert([]); // pass
assert({}); // fail
```
</details>

<details><summary>v.plainObject</summary>

```javascript
const assert = v.assert(
    v.plainObject
);
assert([]); // pass
assert(new Map()); // fail
```
</details>

<details><summary>v.string</summary>

```javascript
const assert = v.assert(
    v.string
);
assert('str'); // pass
assert(0x0); // fail
```
</details>

<details><summary>v.date</summary>

```javascript
const assert = v.assert(
    v.date
);
assert(98765); // pass
assert('1969-12-31T23:59:59.997Z'); // pass
assert(new Date()); // pass
assert(false); // fail
assert({}); // fail
```
</details>

<details><summary>v.coordinates</summary>
Passes when input is [longitude, latitude]

```javascript
const assert = v.assert(
    v.coordinates
);
assert([150, 60]); // pass
assert([60, 150]); // fail
```
</details>

## Higher Order Validators
Higher Order Validators are functions that accept another validator or a value as their parameter and return a new validator.

### v.shape(validatorObj)
Takes an object of validators and passes if the input shape matches 

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
Takes a validator as an argument and returns a validator that passes when each nad every item of input array passes the validator.

```javascript
const assert = v.assert(
  v.arrayOf(v.number)
);
assert([90, 10]); // pass
assert([90, '10']); // fail
assert(90); // fail
```

### v.required(validator)
Returns a strict validator which on also does `null` and `undefined` check.
```javascript
const assert = v.assert(
  v.arrayOf(v.required(v.number))
);
assert([90, 10]); // pass
assert([90, 10, null]); // fail
assert([90, 10, undefined]); // fail
```


### v.oneOfType(...validators)
Takes multiple validators and returns a validator that passes if any one of them passes.

```javascript
const assert = v.assert(
  v.oneOfType(v.string, v.number)
);
assert(90); // pass
assert('90'); // pass
```

### v.equal(value)
It does a `===` comparison between `value` and `input`.

```javascript
const assert = v.assert(
  v.equal(985)
);
assert(985); // pass
assert(986); // fail
```

### v.oneOf(...values)
Passes if input matches (`===`) with any one of the `values`.

```javascript
const assert = v.assert(
  v.oneOf(3.14, '3.1415', 3.1415)
);
assert(3.14); // pass
assert(986); // fail
```

### v.range([valueA, valueB])
Passes if input inclusively lies between valueA & valueB.

```javascript
const assert = v.assert(
  v.range([-10, 10])
);
assert(4); // pass
assert(-100); // fail
```
