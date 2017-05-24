# "Operator"(-ish) Overloading Techniques in PHP
I'll preface this by defining *operator overloading* for our purposes to be achieving arbitrary behaviour given some language symbol, and one or more variables. I won't distingish whether or not you have control over what value is returned by these "operators": in some cases you will have control over the return value, in some cases you may only control some side effect. Discard items to your hearts content.

## Binary/Ternary Operators

### `• -> • = •` Accessor-Assignment
Under some circumstances you may overload the assignment operator. Namely, if your class implements the `__set()` method which should have the following signature. 

```php
public function __set(string $name, $value): void
```

This is a ternary operator if you manipulate all three values available to you `• -> • = •`, or binary if you consider the accessed property to be a single value `• = •`.

If an inaccessible property on your class is attempted to be set (whether that be because of visibility, or non-existance) then PHP will defer to this method if implemented.

This method will only be called when attempting to access via the use of `->`, and will not be called if using the scope resolution operator `::`.

Note that return values from this function are ignored, so code like

```php
if ($object->var = 1)
```

will make sense in the written context (will be truthy because `1` is truthy) but you can not change the result of this `if` statement by returning a falsey value.

### `• -> •` Accessor
Just because its worth mentioning it alongside `__set()`, you can similarly use `__get()` with method signature
```php
public function __get(string $name)
```
when attempting to access inaccesible properties.
Unlike before, the return value here is taken into account.


### `• []= •` Array Append
An object that implements the `ArrayAccess` interface may use
```php
abstract public function offsetSet ($offset, $value): void
```
to utilise the array append operator. In this case, `$offset` will have a value of `null`. It is up to you whether you wish to consider this a binary operator, and the similar version with `$offset` being non null a ternary one.

###  `•[•]= •` Array-Key Assignment
As above, an object that implements the `ArrayAccess` interface may use
```php
abstract public function offsetSet ($offset, $value): void
```
There is no restriction on `$offset` nor `$value`, unlike when using arrays.

### `•[•]` Array-Key Accessor
An object that implements the `ArrayAccess` interface may use
```php
abstract public function offsetGet($offset)
```
There is no restriction on `$offset`, unlike when using arrays.

Personally I find this to be the most interesting operator so far, because it allows arbitrary values in both slots (though obviously the value in slot one must implement the required interface).

This operator could be designed to be commutative if both values implement the required interface. And could also/instead be polymorphic based on type detection.


## `n`-ary Operators
### `•(•, •, ...)` Call Operator
An object may implement the magic method `__invoke()`, which will be called if the object itself is called, passing all arguments through.

This is the freest operator you have control over (though probably also the most ambiguous).

## Unary Pseudo-Operators
### Stringyness
Okay, so this is not an operator (even by the very loose definition of "some language symbol" above) – but its behaviour can be invoked by one (among many things).
A class may implement the `__toString()` method to determine how it should behave when treated like a string. It should have method signature.

```php
public function __toString(): string
```

You can invoke `__toString()` by treating an object like a string in one of various ways, e.g.

```php
echo $object;

$object.'';
```