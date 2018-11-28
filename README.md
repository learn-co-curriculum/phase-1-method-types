# Building Cells: Method Types

## Learning Goals

- Recognize the syntactic differences between regular, static, getter and setter
  methods
- Recognize the different uses of each method type

## Introduction

So far, we've seen some examples of `class`es that have their own custom
methods:

```js
class Square {
	constructor(sideLength) {
		this.sideLength = sideLength;
	}

	area() {
		return this.sideLength * this.sideLength;
	}
}
```

It turns out, however, there are four different types of methods we can write in
a `class`. Each of these behaves differently, and this variety provides us with
flexibility in how we want our `class`es to behave.

In this lesson, we're going to briefly look at each type of method and consider
some use cases for each.

## Standard Methods

Most `class` methods you will see use the following, standard syntax:

```js
area() {
  return this.sideLength * this.sideLength;
}
```

These methods are available to any instance of the `class` they belong to,
as we've seen:

```js
let square = new Square(5);
square.area(); // => 25
```

Methods can be called from inside other methods just like properties:

```js
class Square {
	constructor(sideLength) {
		this.sideLength = sideLength;
	}

	area() {
		return this.sideLength * this.sideLength;
	}

	areaMessage() {
		return `The area of this square is ${this.area()}`;
	}
}
square.area(); // => 25
square.areaMessage(); // => LOG: The area of this square is 25
```

In the `class` above, we can access `area()` directly, or use it to provide
dynamic content for other methods. These methods are the most common - they act
as the 'behaviors' of a `class` instance.

## Static Methods

Static methods are `class` level methods - they are not callable on instances of
a `class`, only the `class` itself. These are often used in 'utility' `class`es -
`class`es that encapsulate a set of related methods but don't need to be
made into instances. For example, we could write a `CommonMath` `class` that
stores a series of math related methods:

```js
class CommonMath {
	static triple(number) {
		return number * number * number;
	}

	static findHypotenuse(a, b) {
		return Math.sqrt(a * a + b * b);
	}
}
```

To access, these static methods:

```js
let num = CommonMath.triple(3);
num; // => 27
let c = CommonMath.findHypotenuse(3, 4);
c; // => 5
```

This sort of `class` might be useful in many different situations, but we don't
ever need an _instance_ of it.

## Define `get` Keyword in JavaScript Class Context

Often, when writing methods for a `class`, we want to return information
derived from that instance's properties. In the Square `class` example, `area()`
returns a calculation based on `this.sideLength`, and `areaMessage()` returns
a `String`.

In modern JavaScript, new syntax, `get`, has been introduced to `class`es
to distinguish methods which serve a specific purpose to 'get' data from an
instance.

The `get` keyword turns a method into a 'pseudo-property', that is - it allows
us to write a method that interacts like a property. To use `get`, write a
`class` method like normal, preceded by `get`:

```js
class Square {
	constructor(sideLength) {
		this.sideLength = sideLength;
	}

	get area() {
		return this.sideLength * this.sideLength;
	}
}
```

As a result of this, `area` will now be available as though it is a
property just like `sideLength`:

```js
let square = new Square(5);
square.sideLength; // => 5
square.area; // => 25
```

If you try to use `this.area()`, you'll receive a TypeError - `area` is no
longer considered a function!

This may seem strange - you could also just write the following and achieve the
same result:

```js
class Square {
	constructor(sideLength) {
		this.sideLength = sideLength;
		this.area = sideLength * sideLength;
	}
}
```

This is valid code, but what we've done is load our `constructor` with more
calculations. The main benefit to using `get` is that your `area` calculation
isn't actually run until it is accessed. The 'cost' of calculating is offset,
and may not be needed at all if `this.area` is never called. For calculations or
processes that are costly, this can be a handy tool.

## Define `set` Keyword in JavaScript Class Context

In the previous section we saw we could write the following after employing `get`:

```js
let square = new Square(5);
square.sideLength; // => 5
square.area; // => 25
```

In the `Square` `class`, we only have one real property,
`this.sideLength`. If we want to change that directly, we would write
`this.sideLength =` whatever value we assign it.

```js
let square = new Square(5);
square.sideLength; // => 5
square.area; // => 25
square.sideLength = 10;
```

Which will change the value returned by `area`:

```js
square.area; // => 100
```

At the moment, although `area` looks and behaves like a property, we can't
assign it a new value like a real property. Using `get` to create a
pseudo-prototype is only half the story, since it is only used for retrieving
data from an instance.

To change data, we have `set`.

To make `area` fully act like a real property, we can create both `get` and
`set` methods for it:

```js
class Square {
	constructor(sideLength) {
		this.sideLength = sideLength;
	}

	get area() {
		return this.sideLength * this.sideLength;
	}

	set area(newArea) {
		this.sideLength = Math.sqrt(newArea);
	}
}
```

The result is that we can now 'set' the pseudo-property, `area`, and modify
`this.sideLength` based on a reverse of the calculation we used in `get`:

```js
let square = new Square(5);
square.sideLength; // => 5
square.area; // => 25
square.area = 64;
square.sideLength; // => 8
square.area; // => 64
```

We can now interact with `area` as though it is a modifiable property, even
though `area` is derived.

Using `set`, we can set up _indirect_ control to real properties. From the outside,
it looks like a property is being set, but behind the scenes, we can apply
conditional statements:

```js
set area(newArea) {
  if (newArea > 0) {
    this.sideLength = Math.sqrt(newArea)
  } else {
    console.warn("Area cannot be less than 0");
  }
}
```

Using `set` works well when we want to treat properties as private:

```js
class Student {
	constructor(firstName, lastName) {
		this._firstName = this.sanitize(firstName);
		this._lastName = this.sanitize(lastName);
	}

	get firstName() {
		return this.capitalize(this._firstName);
	}

	set firstName(firstName) {
		this._firstName = this.sanitize(firstName);
	}

	capitalize(string) {
		// capitalizes first letter
		return string.charAt(0).toUpperCase() + string.slice(1);
	}

	sanitize(string) {
		// removes any non alpha-numeric characters except dash and single quotes (apostrophes)
		return string.replace(/[^A-Za-z0-9-']+/g, '');
	}
}

let student = new Student('Carr@ol-Ann', ')Freel*ing');
student; // => Student { _firstName: 'Carrol-Ann', _lastName: 'Freeling' }

student.firstName = 'Hea@)@(!$)ther';
student.firstName; // => 'Heather'
```

In this `Student` class, we've set up a pseudo property, `firstName`, which
refers to a 'private' property `_firstName`. We've also included a `sanitize()`
method that removes any non alpha-numeric characters except those that commonly
appear in names.

Because we are using `set` and a 'private' property, we can call `sanitize()`
when a `Student` instance is constructed, _or_ when we try to modify
`_firstName`. This would not be possible if we were modifying a real
property directly.

With `get` and `set` we can fine tune exactly how we want our data to be
accessed and changed.

## Conclusion

In the Object Oriented JavaScript world, we have a variety of ways to build our
`class`es. As we continue to learn about OO JS, we will see that this
flexibility is important - it allows us to design many `class`es that work
together, each serving their own specific purpose, entirely customized.

## Resources

- [`get`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get
- [`set`]:
- [static methods]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static
