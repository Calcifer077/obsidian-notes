When objects are used in operations like `obj1 + obj2`, `obj1 - obj2`, or `alert(obj)`, JavaScript auto-converts them to primitives first, then carries out the operation — and the result is always a primitive, never another object.

So you can't do `vector1 + vector2` and get a new vector — JS doesn't work that way. When object math happens in real code, it's usually a bug.

## Conversion Rules

There's no boolean conversion for objects — all objects are simply `true` in a boolean context. Only numeric and string conversions exist. Numeric conversion happens with math operations (like subtraction), while string conversion typically happens when outputting with `alert(obj)`.

## Hints — How JS Decides What to Convert To

There are three "hints" that tell JS which type of primitive is needed: `"string"`, `"number"`, and `"default"`.

**`"string"`** — when JS clearly needs a string:

```js
alert(obj);           // needs a string
anotherObj[obj] = 1;  // object used as a property key
```

**`"number"`** — for math:

```js
let n = +obj;
let delta = date1 - date2;
let greater = user1 > user2;
```

**`"default"`** — when JS can't tell which type to expect:

```js
let total = obj1 + obj2; // + works for both strings and numbers
if (user == 1) { ... }   // == with a number is ambiguous
```

In practice, all built-in objects except `Date` implement `"default"` the same way as `"number"`, so you can usually treat them identically.

## The Conversion Algorithm

Once JS knows the hint, it looks for methods in this order:
1. `obj[Symbol.toPrimitive](hint)` — if it exists, use it
2. If hint is `"string"` → try `toString()`, then `valueOf()`
3. If hint is `"number"` or `"default"` → try `valueOf()`, then `toString()`

## `Symbol.toPrimitive` — The Modern Way

There's a built-in symbol `Symbol.toPrimitive` used to name the conversion method. If it exists, it handles all hints in one place.

```js
let user = {
  name: "John",
  money: 1000,

  [Symbol.toPrimitive](hint) {
    return hint == "string" ? `{name: "${this.name}"}` : this.money;
  }
};

alert(user);       // hint: "string"  → {name: "John"}
alert(+user);      // hint: "number"  → 1000
alert(user + 500); // hint: "default" → 1500
```

One method, all cases covered. Clean and explicit.

## `toString` / `valueOf` — The Old Way

If there's no `Symbol.toPrimitive`, JS looks for `toString` and `valueOf`. For the `"string"` hint, `toString` has priority; for `"number"` or `"default"`, `valueOf` has priority.

```js
let user = {
  name: "John",
  money: 1000,

  toString() { return `{name: "${this.name}"}`; }, // used for "string"
  valueOf()  { return this.money; }                 // used for "number"/"default"
};
```

By default (without overriding), a plain object's `toString` returns `"[object Object]"` and `valueOf` returns the object itself — which is why you often see that ugly string when alerting objects.

**Catch-all shortcut:** If you only define `toString`, it handles all hints:

```js
let user = {
  name: "John",
  toString() { return this.name; }
};

alert(user);       // "John"
alert(user + 500); // "John500"
```

## Important: You Don't Have to Return the Hinted Type

The conversion methods don't have to return the hinted type — `toString` can return a number, for example. The only hard rule is that they must return a _primitive_, not an object. For `toString`/`valueOf`, returning an object is silently ignored (for historical reasons). But `Symbol.toPrimitive` is stricter — it throws an error if you return an object.

## Further Conversions — Two-Stage Process

When an object is used in an operation, there are two stages: first the object is converted to a primitive, then if needed, that primitive is converted further.

```js
let obj = { toString() { return "2"; } };

alert(obj * 2); // 4  → "2" becomes the primitive, then "2" * 2 = 4 (string → number)
alert(obj + 2); // "22" → "2" + 2 concatenates because + accepts strings
```

The key insight: `*` forces numeric conversion of the string `"2"`, but `+` happily concatenates it.

## Mental Model Summary

```
Object used in expression
        ↓
JS picks a hint: "string" / "number" / "default"
        ↓
Looks for conversion method:
  1. Symbol.toPrimitive(hint)   ← modern, handles all hints
  2. toString() / valueOf()     ← legacy, priority depends on hint
        ↓
Returns a primitive
        ↓
(Optional) Further type coercion if the operator needs it
```

In everyday practice, implementing just `toString()` as a catch-all is often enough — especially for logging or debugging purposes where a human-readable representation is all you need.

## Sources:
[JavaScript Info](https://javascript.info/object-toprimitive)

## Tags:
#javascript 