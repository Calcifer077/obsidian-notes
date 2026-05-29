In JavaScript, the textual data is stored as strings. There is no separate type for a single character.

## Quotes

Strings can be enclosed within either single quotes `''`, double quotes `""` or backticks. Single and double quotes are the same. Backticks allows us to embed any expression into the string, by wrapping it in `${...}`: 
```js
function sum(a, b) {
  return a + b;
}

alert(`1 + 2 = ${sum(1, 2)}.`); // 1 + 2 = 3.
```

Another advantage of using backticks is that they allow a string to span multiple lines:
```js
let guestList = `Guests:
 * John
 * Pete
 * Mary
`;

alert(guestList); // a list of guests, multiple lines
```

Single or double quotes do not work this way. If we use them and try to use multiple lines, there'll be an error:
```js
let guestList = "Guests: // Error: Unexpected token ILLEGAL
  * John";
```

## Special characters

It is still possible to create multiline strings with single and double quotes by using "newline character", `\n`, which denotes a line break:
```js
let guestList = "Guests:\n * John\n * Pete\n * Mary";

alert(guestList); // a multiline list of guests, same as above
```

Below two lines are equal, just written differently:
```js
let str1 = "Hello\nWorld"; // two lines using a "newline symbol"

// two lines using a normal newline and backticks
let str2 = `Hello
World`;

alert(str1 == str2); // true
```

Other less common special characters:

| Character            | Description                                                                                                                                                                                              |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `\n`                 | New line                                                                                                                                                                                                 |
| `\r`                 | In Windows text files a combination of two characters `\r\n` represents a new break, while on non-Windows OS it’s just `\n`. That’s for historical reasons, most Windows software also understands `\n`. |
| `\'`, `\"`, `` \` `` | Quotes                                                                                                                                                                                                   |
| `\\`                 | Backslash                                                                                                                                                                                                |
| `\t`                 | Tab                                                                                                                                                                                                      |
| `\b`, `\f`, `\v`     | Backspace, Form Feed, Vertical Tab – mentioned for completeness, coming from old times, not used nowadays (you can forget them right now).                                                               |

If we need to show an actual backlash `\` within the string, we need to double it:
```js
alert( `The backslash: \\` ); // The backslash: \
```

So-called "escaped" quotes `\'`, `\"`, `` \` `` are used to insert a quote into the same-quoted string.
```js
alert( 'I\'m the Walrus!' ); // I'm the Walrus!
```

As you can see, we have to prepend the inner quote by the backlash `\'`, because otherwise it would indicate the string end.

## String length

`.length` gives string length:
```js
alert( `My\n`.length ); // 3
```

Note that `\n` is a single "special" character, so the length is indeed `3`.

## Accessing characters

To get a character at position `pos`, use square brackets `[pos]` or call the method `str.at(post)`.
```js
let str = `Hello`;

// the first character
alert( str[0] ); // H
alert( str.at(0) ); // H

// the last character
alert( str[str.length - 1] ); // o
alert( str.at(-1) );
```

If `pos` is negative, then it's counted from the end of the string. The square brackets always return `undefined` for negative indexes. 

We can also iterate over characters using `for...of`:
```js
for (let char of "Hello") {
  alert(char); // H,e,l,l,o (char becomes "H", then "e", then "l" etc)
}
```

## Strings are immutable

Strings can't be changed in javascript. It is impossible to change a character.
```js
let str = 'Hi';

str[0] = 'h'; // error
alert( str[0] ); // doesn't work
```

The usual workaround is to create a whole new string and reassign it to `str` instead of the old one.
```js
let str = 'Hi';

str = 'h' + str[1]; // replace the string

alert( str ); // hi
```

## Searching for a substring

### `str.indexOf`
The first method is [str.indexOf(substr, pos)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/indexOf).

It looks for the `substr` in `str`, starting from the given position `pos`, and returns the position where the match was found or `-1` if nothing can be found.
```js
let str = 'Widget with id';

alert( str.indexOf('Widget') ); // 0, because 'Widget' is found at the beginning
alert( str.indexOf('widget') ); // -1, not found, the search is case-sensitive

alert( str.indexOf("id") ); // 1, "id" is found at the position 1 (..idget with id)
```

The optional second parameter allows us to start searching from a given position.

#### `str.lastIndexOf(substr, position)`
There is also a similar method [str.lastIndexOf(substr, position)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/lastIndexOf) that searches from the end of a string to its beginning. It would list the occurrences in the reverse order.

### `includes`, `startsWith`, `endsWith`

All three returns `true/false` depending on whether `str` contains `substr`, starts with `substr` or ends with `substr`.
```js
alert( "Widget with id".includes("Widget") ); // true
alert( "Hello".includes("Bye") ); // false

alert( "Widget".includes("id") ); // true
alert( "Widget".includes("id", 3) ); // false, from position 3 there is no "id"

alert( "Widget".startsWith("Wid") ); // true, "Widget" starts with "Wid"
alert( "Widget".endsWith("get") ); // true, "Widget" ends with "get"
```

## Getting a substring

There are 3 methods in JavaScript to get a substring: `substring`, `substr` and `slice`.
### `str.slice(start [, end])`
Returns the part of the string from `start` to (but not including) `end`.
```js
let str = "stringify";
alert( str.slice(0, 5) ); // 'strin', the substring from 0 to 5 (not including 5)
alert( str.slice(0, 1) ); // 's', from 0 to 1, but not including 1, so only character at 0
```

If there is no second argument, then `slice` goes till the end of the string. Negative values for `start/end` are also possible. They mean the position is counted from the string end.

### `str.substring(start, [, end])`
Returns the part of the string _between_ `start` and `end` (not including `end`). This is almost the same as `slice`, but it allows `start` to be greater than `end` (in this case it simply swaps `start` and `end` values). Negative arguments are (unlike slice) not supported, they are treated as `0`.

### `str.substr(start, [, length])`
Returns the part of the string from `start`, with the given `length`. In contrast with the previous methods, this one allows us to specify the `length` instead of the ending position:
```js
let str = "stringify";
alert( str.substr(2, 4) ); // 'ring', from the 2nd position get 4 characters
```

The first argument may be negative, to count from the end.

| method                  | selects…                                        | negatives                |
| ----------------------- | ----------------------------------------------- | ------------------------ |
| `slice(start, end)`     | from `start` to `end` (not including `end`)     | allows negatives         |
| `substring(start, end)` | between `start` and `end` (not including `end`) | negative values mean `0` |
| `substr(start, length)` | from `start` get `length` characters            | allows negative `start`  |

#### Which one to choose?
All of them can do the job. Formally, `substr` has a minor drawback: it is described not in the core JavaScript specification, but in Annex B, which covers browser-only features that exist mainly for historical reasons. So, non-browser environments may fail to support it. But in practice it works everywhere.

Of the other two variants, `slice` is a little bit more flexible, it allows negative arguments and shorter to write.

So, for practical use it’s enough to remember only `slice`.

## Comparing strings

Strings are compared character-by-character in alphabetical order. Strings in JavaScript are encoded using [UTF-16](https://en.wikipedia.org/wiki/UTF-16). That is: each character has a corresponding numeric code. There are special methods that allow to get the character for the code and back.

### `str.codePointAt(pos)`
Returns a decimal number representing the code for the character at position `pos`:
```js
// different case letters have different codes
alert( "Z".codePointAt(0) ); // 90
alert( "z".codePointAt(0) ); // 122
alert( "z".codePointAt(0).toString(16) ); // 7a (if we need a hexadecimal value)
```

### `String.fromCodePoint(code)`
Creates a character by its numeric `code`
```js
alert( String.fromCodePoint(90) ); // Z
alert( String.fromCodePoint(0x5a) ); // Z (we can also use a hex value as an argument)
```

## Correct comparisons

The “right” algorithm to do string comparisons is more complex than it may seem, because alphabets are different for different languages. So, the browser needs to know the language to compare.

Luckily, modern browsers support the internationalization standard [ECMA-402](https://www.ecma-international.org/publications-and-standards/standards/ecma-402/).

It provides a special method to compare strings in different languages, following their rules.

The call [str.localeCompare(str2)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare) returns an integer indicating whether `str` is less, equal or greater than `str2` according to the language rules:
- Returns a negative number if `str` is less than `str2`.
- Returns a positive number if `str` is greater than `str2`.
- Returns `0` if they are equivalent.

```javascript
alert( 'Österreich'.localeCompare('Zealand') ); // -1
```

This method actually has two additional arguments specified in [the documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/localeCompare), which allows it to specify the language (by default taken from the environment, letter order depends on the language) and setup additional rules like case sensitivity or should `"a"` and `"á"` be treated as the same etc.

## Summary

- There are 3 types of quotes. Backticks allow a string to span multiple lines and embed expressions `${…}`.
- We can use special characters, such as a line break `\n`.
- To get a character, use: `[]` or `at` method.
- To get a substring, use: `slice` or `substring`.
- To lowercase/uppercase a string, use: `toLowerCase/toUpperCase`.
- To look for a substring, use: `indexOf`, or `includes/startsWith/endsWith` for simple checks.
- To compare strings according to the language, use: `localeCompare`, otherwise they are compared by character codes.

There are several other helpful methods in strings:
- `str.trim()` – removes (“trims”) spaces from the beginning and end of the string.
- `str.repeat(n)` – repeats the string `n` times.
- …and more to be found in the [manual](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String).

Strings also have methods for doing search/replace with regular expressions. But that’s big topic, so it’s explained in a separate tutorial section [Regular expressions](https://javascript.info/regular-expressions).

Also, as of now it’s important to know that strings are based on Unicode encoding, and hence there’re issues with comparisons. There’s more about Unicode in the chapter [Unicode, String internals](https://javascript.info/unicode).

## tags

#javascript 