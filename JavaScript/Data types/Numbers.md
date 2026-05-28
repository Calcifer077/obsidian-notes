In modern JavaScript, there are two types of numbers:
1. Regular numbers stored in 64-bit format, also known as "double precision floating point numbers".
2. BigInt numbers represent integers of arbitrary length. They are sometimes need because regular integer can't safely exceed `(25^3-1)` or be less than `-(25^3-1)`.

In this note we will learn about regular numbers.

## More ways to write a number

Imagine we need to write 1 billion. The obvious way is:
```javascript
let billion = 1000000000;
```

We also can use underscore `_` as the separator:
```javascript
let billion = 1_000_000_000;
```

Here the underscore `_` acts like "syntatic sugar", just to make the number more readable. The JavaScript engine simply ignores `_` between digits.

We can shorten a number by appending the letter `"e"` to it and specifying the zeroes count:
```js
let billion = 1e9;  // 1 billion, literally: 1 and 9 zeroes

alert( 7.3e9 );  // 7.3 billions (same as 7300000000 or 7_300_000_000)
```

In other words, `e` multiplies the number by `1` with the given zeroes count.
```javascript
1e3 === 1 * 1000; // e3 means *1000
1.23e6 === 1.23 * 1000000; // e6 means *1000000
```

If you use negative number after `e`, these zeroes will be placed on the left side.
```js
let mcs = 1e-6; // five zeroes to the left from 1
```

If we count the zeroes in `0.000001`, there are 6 of them. So naturally it’s `1e-6`. In other words, a negative number after `"e"` means a division by 1 with the given number of zeroes:
```javascript
// -3 divides by 1 with 3 zeroes
1e-3 === 1 / 1000; // 0.001

// -6 divides by 1 with 6 zeroes
1.23e-6 === 1.23 / 1000000; // 0.00000123

// an example with a bigger number
1234e-2 === 1234 / 100; // 12.34, decimal point moves 2 times
```

### Hex, binary and octal numbers
Hexadecimal numbers are widely used in JavaScript to represent colours, encode characters, and for many other things. There exists a shorter way to write them: `0x` and then the number.
```js
alert( 0xff ); // 255
alert( 0xFF ); // 255 (the same, case doesn't matter)
```

Binary and octal numeral systems are rarely used, but also supported using the `0b` and `0o` prefixes:
```javascript
let a = 0b11111111; // binary form of 255
let b = 0o377; // octal form of 255

alert( a == b ); // true, the same number 255 at both sides
```

There are only 3 numeral systems with such support.

## toString(base)

The method `num.toString(base)` returns a string representation of `num` in the numeral system with the given `base`.
```js
let num = 255;

alert( num.toString(16) );  // ff
alert( num.toString(2) );   // 11111111
```

The `base` can vary from `2` to `36` . By default, it's `10`.

#### Two dots to call a method
Note that two dots in `123456..toString(36)` is not a typo. If we want to call a method directly on a number, like `toString` in the example above, then we need to place two dots `..` after it.

If we placed a single dot: `123456.toString(36)`, then there would be an error, because JavaScript syntax implies the decimal part after the first dot. And if we place one more dot, then JavaScript knows that the decimal part is empty and now uses the method.

Also could write `(123456).toString(36)`.

## Rounding

There are several built-in functions for rounding:

`Math.floor`
Rounds down: `3.1` becomes `3`, and `-1.1` becomes `-2`.

`Math.ceil`
Rounds up: `3.1` becomes `4`, and `-1.1` becomes `-1`.

`Math.round`
Rounds to the nearest integer: `3.1` becomes `3`, `3.6` becomes `4`. In the middle cases `3.5` rounds up to `4`, and `-3.5` rounds up to `-3`.

`Math.trunc` (not supported by Internet Explorer)
Removes anything after the decimal point without rounding: `3.1` becomes `3`, `-1.1` becomes `-1`.

Here’s the table to summarize the differences between them:

||`Math.floor`|`Math.ceil`|`Math.round`|`Math.trunc`|
|---|---|---|---|---|
|`3.1`|`3`|`4`|`3`|`3`|
|`3.5`|`3`|`4`|`4`|`3`|
|`3.6`|`3`|`4`|`4`|`3`|
|`-1.1`|`-2`|`-1`|`-1`|`-1`|
|`-1.5`|`-2`|`-1`|`-1`|`-1`|
|`-1.6`|`-2`|`-1`|`-2`|`-1`|

These functions cover all of the possible ways to deal with the decimal part of a number. But what if we'd like to round the number to `n-th` digit after decimal?

For instance, we have `1.2345` and want to round it to 2 digits, getting only `1.23`.

There are two ways to do so:
#### 1. Multiply-and-divide.   
For example, to round the number to the 2nd digit after the decimal, we can multiply the number by `100`, call the rounding function and then divide it back.    
```javascript
let num = 1.23456;

alert( Math.round(num * 100) / 100 ); // 1.23456 -> 123.456 -> 123 -> 1.23
```

#### 2. The method `toFixed(n)` rounds the number to `n` digits after the point and returns a string representation of the result
```js
let num = 12.34;
alert( num.toFixed(1) ); // "12.3"
```

This rounds up or down to the nearest value, similar to `Math.round`. Note that the result of `toFixed` is a string. If the decimal part is shorter than required, zeroes are appended to the end:
```js
let num = 12.34;
alert( num.toFixed(5) ); // "12.34000", added zeroes to make exactly 5 digits
```

We can convert it to a number using the unary plus or a `Number()` call, e.g. write `+num.toFixed(5)`.

## Imprecise calculations

Internally, a number is represented in 64-bit format. 52 of those bits are used to store the digit, 11 of them store the position of the decimal point, and 1 bit is for the sign.

If a number is really huge, it may overflow and becomes a special numeric value `Infinity`.
```js
alert( 1e500 ); // Infinity
```

### Loss of precision
Consider this (falsy!) equality test:
```js
alert( 0.1 + 0.2 == 0.3 ); // false
alert( 0.1 + 0.2 ); // 0.30000000000000004
```

But why does the above happen?
A number is stored in memory in its binary form, a sequence of bits – ones and zeroes. But fractions like `0.1`, `0.2` that look simple in the decimal numeric system are actually unending fractions in their binary form.
```js
alert(0.1.toString(2)); // 0.0001100110011001100110011001100110011001100110011001101
alert(0.2.toString(2)); // 0.001100110011001100110011001100110011001100110011001101
alert((0.1 + 0.2).toString(2)); // 0.0100110011001100110011001100110011001100110011001101
```

The numeric format `IEEE-754` solves this by rounding to the nearest possible number. These rounding rules normally don't allow us to see that "tiny precision loss", but it exists. We can see this in action:
```js
alert( 0.1.toFixed(20) ); // 0.10000000000000000555
```

The most reliable method to solve this problem is to use `toFixed(n)`.
```js
let sum = 0.1 + 0.2;
alert( +sum.toFixed(2) ); // "0.30"
```

#### some random things
```js
// Hello! I'm a self-increasing number!
alert( 9999999999999999 ); // shows 10000000000000000
```

This suffers from the same issue: a loss of precision. There are 64 bits for the number, 52 of them can be used to store digits, but that’s not enough. So the least significant digits disappear.

JavaScript doesn’t trigger an error in such events. It does its best to fit the number into the desired format, but unfortunately, this format is not big enough.

Another consequence of the internal representation of numbers is the existence of two zeroes: `0` and `-0`.

That’s because a sign is represented by a single bit, so it can be set or not set for any number including a zero.

## Tests: isFinite and isNaN

- `Infinity` (and `-Infinity`) is a special numeric value that is greater (less) than anything.
- `NaN` represents an error.

They belong to the type `number`, but are not "normal" numbers, so there are special functions to check for them:
### `isNan(value)` converts its arguments to a number and then tests it for being `NaN`:
```js
alert( isNaN(NaN) ); // true
alert( isNaN("str") ); // true
```

But can't we just use the comparison `=== NaN`? Unfortunately not. The value `Nan` is unique in that it does not equal anything, including itself:
```js
alert( NaN === NaN ); // false
```

### `isFinite(value)` converts its argument to a number and returns `true` if it’s a regular number, not `NaN/Infinity/-Infinity`:
```js
alert( isFinite("15") ); // true
alert( isFinite("str") ); // false, because a special value: NaN
alert( isFinite(Infinity) ); // false, because a special value: Infinity
```

Sometimes `isFinite` is used to validate whether a string value is a regular number:
```js
let num = +prompt("Enter a number", '');

// will be true unless you enter Infinity, -Infinity or not a number
alert( isFinite(num) );
```

Note that an empty or a space-only string is treated as `0` in all numeric functions including `isFinite`.

#### `Number.isNan` and `Number.isFinite`
`Number.isNaN` and `Number.isFinite` methods are the more “strict” versions of `isNaN` and `isFinite` functions. They do not autoconvert their argument into a number, but check if it belongs to the `number` type instead.

#### Comparison with `Object.is`
There is a special built-in method `Object.is` that compares values like `===`, but is more reliable for two edge cases:
1. It works with `NaN`: `Object.is(NaN, NaN) === true`, that’s a good thing.
2. Values `0` and `-0` are different: `Object.is(0, -0) === false`, technically that’s correct because internally the number has a sign bit that may be different even if all other bits are zeroes.

In all other cases, `Object.is(a, b)` is the same as `a === b`. When an internal algorithm needs to compare two values for being exactly the same, it uses `Object.is` (internally called [SameValue](https://tc39.github.io/ecma262/#sec-samevalue).

## parseInt and parseFloat

Numeric conversion using a plus `+` or `Number()` is strict. If a value is not exactly a number, it fails:
```js
alert( +"100px" ); // NaN
```
The sole exception is spaces at the beginning or at the end of the string, as they are ignored.

But in real life, we often have values in units, like `"100px"` or `"12pt"` in CSS. Also in many countries, the currency symbol goes after the amount, so we have `"19€"` and would like to extract a numeric value out of that.

That’s what `parseInt` and `parseFloat` are for.
They “read” a number from a string until they can’t. In case of an error, the gathered number is returned. The function `parseInt` returns an integer, whilst `parseFloat` will return a floating-point number:
```js
alert( parseInt('100px') ); // 100
alert( parseFloat('12.5em') ); // 12.5

alert( parseInt('12.3') ); // 12, only the integer part is returned
alert( parseFloat('12.3.4') ); // 12.3, the second point stops the reading
```

There are situations when `parseInt/parseFloat` will return `Nan`. It happens when no digits could be read:
```js
alert( parseInt('a123') ); // NaN, the first symbol stops the process
```

#### The second argument of `parseInt(str, radix)`
The `parseInt()` function has an optional second parameter. It specifies the base of the numeral system, so `parseInt` can also parse strings of hex numbers, binary numbers and so on:
```javascript
alert( parseInt('0xff', 16) ); // 255
alert( parseInt('ff', 16) ); // 255, without 0x also works

alert( parseInt('2n9c', 36) ); // 123456
```

## Summary

To write numbers with many zeroes:
- Append `"e"` with the zeroes count to the number. Like: `123e6` is the same as `123` with 6 zeroes `123000000`.
- A negative number after `"e"` causes the number to be divided by 1 with given zeroes. E.g. `123e-6` means `0.000123` (`123` millionths).

For different numeral systems:
- Can write numbers directly in hex (`0x`), octal (`0o`) and binary (`0b`) systems.
- `parseInt(str, base)` parses the string `str` into an integer in numeral system with given `base`, `2 ≤ base ≤ 36`.
- `num.toString(base)` converts a number to a string in the numeral system with the given `base`.

For regular number tests:
- `isNaN(value)` converts its argument to a number and then tests it for being `NaN`
- `Number.isNaN(value)` checks whether its argument belongs to the `number` type, and if so, tests it for being `NaN`
- `isFinite(value)` converts its argument to a number and then tests it for not being `NaN/Infinity/-Infinity`
- `Number.isFinite(value)` checks whether its argument belongs to the `number` type, and if so, tests it for not being `NaN/Infinity/-Infinity`

For converting values like `12pt` and `100px` to a number:
- Use `parseInt/parseFloat` for the “soft” conversion, which reads a number from a string and then returns the value they could read before the error.

For fractions:
- Round using `Math.floor`, `Math.ceil`, `Math.trunc`, `Math.round` or `num.toFixed(precision)`.
- Make sure to remember there’s a loss of precision when working with fractions.

More mathematical functions:
- See the [Math](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Math) object when you need them. The library is very small but can cover basic needs.

## Tags
#javascript 