<img src="./logo.png" width="366" height="114" alt="scriptum"><br><br><br>

# Status

This functional library is still work in progress but already in production use.

## What

A type-directed functional library that adapts well-known functional patterns to untyped Javascript.

Type-directed programming in an untyped language means to handle code as if it were typed and make these hypothetically types explicit with type signature annotations through comments. With this approach you obtain some of the benefits of a typed language like easier reasoning about your algorithms and guidance during the development process.

### Runtime

scriptum is meant for node.js but can also be run in the browser. For this reason the library avoids global node.js dependencies, but defines them at the function level as formal parameters, that is to say you have to pass dependencies explicitly when using such functions:

```Javascript
const fileRead_  = fs => enc => path =>
  Task((res, rej) =>
    fs.readFile(path, enc, (e, s) =>
      e ? rej(e) : res(s)));
      
// and import as

const fs = require("fs"),
  {..., fileRead_, ...} = require("scriptum"),
  fileRead = fileRead_(fs);
```
This is a tradeoff to keep up browser support.

# Mission Statement

### Pragmatism over Dogmatism

Javascript lacks a non-trivial type system and all guarantees that comes along with it and it has no functional data types. As a consequence mutations and reassignments are allowed and sometimes even necessary, as long as they remain local, i.e. are not observable in the parent scope.

### Convention over Coercion

scriptum relies very much on conventions and policies and that developers adhere to them. There is no mechanism to enforce purity or mathematical laws.

### Expressions over Statements

Expressions are superior to statements, because you can compose them and pass them around like data. scriptum provides means to express almost everything as an expression. However, sometimes algorithms are more comprehensible if you assign intermediate values to variables or arrange conditional branches with `if`/`elese` and `switch`. So whenever you feel the need to decompose your complex function compositions you don't need to be ashamed of it.

### Type Signatures over Function Descriptions

When you create a function the first thing to do is to define its type signature and its name. The implementation of its body will follow this signature.

### Semantic Typing over Anonymous `Object` Trees

Use types as simple wrappers whose main reason is to add a semantic layer to your code. This approach guides you during development, renders your code more readable and minimizes the distance between thrown errors and their origin in many cases.

### Curried over Multi-Argument Functions

scriptum prefers curried to multi-argument functions. This simplifies both function and partial application. However, we can isomorphically transform curried to uncurried functions by applying the `curry`/`uncurry` combinators. So there is no risk in using both forms.

### Unions of Records over just Records

You should consider modelling your business domain in the form of alternatives rather than hierarchies. The latter only allow to add information when you move from top to bottom. But the real world isn't assambled in such a schematic way. Alternatives on the other hand are way more flexible to represent a chaotic reality as a data structure. In scriptum alternatives are expressed with tagged unions, which can be nested and may contain records and other product types.

### Directory Passing over Prototypes

scriptum doesn't rely on Javascript's prototype system but enables principled ad-hoc polymorphism through directory passing, i.e. typeclasses are passed as common arguments to functions. As a convetion, typeclass arguments are always placed leftmost in the parameter list and if the function expects several typeclasses they are bundled by an `Object`.

### Effects as Values over Side Effects

scriptum promotes effect handling through monads and effect composition through monad transformer stacks. This way we can defer and encapsulate effects to run them in a principled manner and separate the pure from the impure part of our program. 

### Structural Recursion over Direct Recursion over Loops

Recursion is a big win compared to imperative loops. However, in Javascript we have neither tail call optimization nor more advanced optimization strategies. So we are stuck with tail recursion implemented through trampolines, which are structurally just loops.

What we seek for is a mechanism to abstract from direct recursion altogether. scriptum uses recursion schemes (catamorphism et al.) to separate the recursion from the algorithms and domain logic. These schemes have to be implemented as trampolines for each data type though, to avoid stack overflows and improve performance.

### Function Composition and Thunks over Generators/Iterators

scriptum achieves loop fusion through function composition and lazyness through explicit thunks. Hence there is often no need to use stateful generators/iterators unless you need these effects inside imperative loop or control structures.

# Custom Types

There are a couple of constructors both for union (`union`) and product types (`struct`). Both merely wrap a value into a plain old Javascript `Object`, which is augmented with some properties useful for reasoning and debugging.

## Product Types (Records)

### `struct`

The default constructor for product types with a single field:

```Javascript
const SomeType = struct("SomeType");
SomeType("foo"); // constructs a value of this type
```
### `structn`

Constructor for product types with several fields:

```Javascript
const SomeType = structn("SomeType")
  (SomeType => x => y => SomeType([x, y]));
  
SomeType("foo") (123); // constructs a value of this type
```
### `structGetter`

Constructor for product types with one or several fields where you can define the getter for the `runXYZ` property explicitly. The following example defines a `Defer` type with a lazy getter, so that the `runDefer` property can be accessed without parenthesis:

```Javascript
const Defer = structGetter("Defer")
  (Defer => thunk => Defer({get runDefer() {return thunk()}}));
```
In this example a lazy getter replaces itself with a normal property after the initial lookup:

```Javascript
const Task = structGetter("Task")
  (Task => k => Task({
    get runTask() {
      const r = k(id, id);
      delete this.runTask;
      return this.runTask = l => l(r);
    }}));
```
## Sum Types (Tagged Unions)

### `union`

The default constructor for tagged unions:

```Javascript
const Either = union("Either");

const Left = x =>
  Either("Left", x);

const Right = x =>
  Either("Right", x);
  
// creates values of this type

Right(123);
Left("error");
```
### `unionGetter`

Constructor for unions with explicit getters:

```Javascript
const Option = unionGetter("Option");

const None = Option("None", {get runOption() {return None}});

const Some = x => Option("Some", {runOption: x});
```
In this example the `None` value constructor returns another `None` when the `runOption` property is accessed.

## Predefined Types

scriptum implements a couple of common functional types.

# Typeclasses

scriptum realizes typeclasses through typeclass functions, which have to be passed to ad-hoc polymorphic functions explicitly. An function with a single typeclass constraint defines it as its leftmost formal parameter. Functions that require several typeclass functions receive a dictionary, which is merely an `Object` with the typeclasses as properties. This way there is only a single function call necessary and the typeclass function order doesn't matter.

```Javascript
const kleisliComp = chain => fm => gm => x => // single typeclass constraint
  chain(fm) (gm(x));

const foldMap = ({fold, append, empty}) => f => // several typeclass constraints
  fold(comp2nd(append) (f)) (empty);
```
Here is a list of typeclasses scriptum does or will provide the necessary functions for:

* Alt
* Applicative
* Bifunctor
* Bounded
* Category
* Choice
* Clonable
* Comonad
* Contravariant
* Enum
* Filterable
* Foldable
* Functor
* Ix
* Monad
* MonadRec
* Monoid
* Ord
* Plus
* Profunctor
* Read
* Semigroup
* Serializable
* Deserializable
* Setoid
* Show
* Strong
* Traversable
* Unfoldable

# Pattern Matching

Pattern matching is a low level concept that requires native support of the language. Javascript doesn't have pattern matching and neither does scriptum. However, there are two functions that simplify the handling with unions.

## `match`

Takes an `Object` amd a union type and tries to access the property corresponding to the union type:

```Javascript
const tx = Some("foo"),
  ty = None;
  
match(tx, {
  type: "Option",
  get Some() {return tx.runOption.toUpperCase()},
  None: ""
}); // "FOO"

match(ty, {
  type: "Option",
  get Some() {return tx.runOption.toUpperCase()},
  None: ""
}); // ""
```
## `matchExp`

Takes an `Object` and an expression that evaluates to a union type and assigns this result to an argument visible in the same scope as the provided `Object`.

# `Promise` Compatibility

There are two types in scriptum `Task` and `Parallel` that handle asynchronous computations sequential and parallel respectively. A `Promise` returning function (or rather action) can be easily wired with these types:

```Javascript
const foo = x =>
  new Promise((res, rej) => x > 0
    ? res(x)
    : rej("must be greater than zero"));

const fooTask = x => foo(x)
  .then(y => Task((res, rej) => res(x)))
  .catch(e => Task((res, rej) => rej(e)));
```
# Debugging

scriptum ships with two simple combinators that facilitate debugging of functional code a great deal: `trace` and `debug`. The former lets you inject a `console.log` into any compostion precisely at the desired position and the latter the `debugger` statement. This even works within deeply nested compositional function expressions:

```Javascript
pipe3(
  arrMap)
    (arrMap)
      (arrMap)
        (debug(sqr))
          ([[[1,2,3]]]);
```
If you are just interested in the evaluation result of an expression, pass the identity function:

```Javascript
debug(id) (pipe3(
  arrMap)
    (arrMap)
      (arrMap)
        (sqr)
          ([[[1,2,3]]]));
```

# Type Signatures

Using scriptum you should get familiar with Hindley-Milner-like type signatures of mono- and polymorphic types. Type signatures are a crucial property of functional programming especially for untyped languages, because they keep the code readable and comprehensible when its base grows.

## A Little Type Theory

### Monomorphism

A value is monomorphic if it has exactly one type.

```Javascript
// String
const foo = "bar";
```

```Javascript
// String -> Number
const strLen = s => s.length;
```
### Polymorphism

A value (and by the way, a function is just another kind of value in this regard) is polymorphic if it can have more than one monomorphic type.

#### Parametric Polymorphism

A parametric polymorphic value is a polymorphic value without any constraints on the polymorphic portion of its type, i.e. nothing is known about this portion and the value can adopt any type.

```Javascript
// [a]
const foo = [];

// a -> a
const id = x => x;
```
The type variable `a` represents the polymorphic portion of both types. Neither the `Array` nor the `Function` do know anything about `a`.

#### Ad-hoc Polymorphism

An ad-hoc polymorphic value is a polymorphic value with additional constraints on the polymorphic portion of its type, i.e. there is a certain knowledge about this portion and the value can only adopt types that implement this knowledge.

```Javascript
// Functor<f> => (a -> b) -> f<a> -> f<b>
const genericMap = map => g => ts => map(g) (ts);
```
In the example above `f` is ad-hoc polymorphic, because it can adapt any type that implements a binary `map` operator, i.e. it is known how to map over `f` and its content `a` respectively. Please note that `f`'s field `a` is parametric polymorphic, because it can adopt any type.

#### Row Polymorphism

A row polymorphic value is a polymorphic `Object` for which only a subset of its properties is known. If these properties coincide with a type it can adopt this type and all properties the `Object` includes beyond that type are assigned to the type's row varibale.

```Javascript
// {foo: String, ...a}
const foo = {foo: "abc", bar: 123},
  bar = {foo: "abc", baz: true};
```
In the example above `foo` and `bar` are inhabitants of the same row polymorphic type. `foo` assigns `bar: Number` to the row variable `a` and `bar` assigns `baz: Boolean`.

## Notation

### `Array`

*  `[?]` - denotes an unknown and probably heterogeneous `Array`
*  `[String]` - denotes a homogenious `Array` of `String`s
*  `[String, Number]` - denotes a pair tuple
*  `[String|Number]` - denotes a heterogeneous `Array` of `String`s and `Number`s
*  `[a]` - denotes a polymorphic `Array`

### `Boolean`

Denotes a `Boolean` value.

### `Date`

Denotes a date-`Object` value.

### `Function`

* `String -> Integer` - denotes a function from `String` to `Integer`
* `(Number, Number) -> String -> Boolean` - denotes a multi-argument function
* `Number...Number -> Number` - denotes a function with rest parameter
*  `(a -> b) -> [a] -> [b]` - denotes a higher order function that takes a polymorphic function and a polymorphic `Array` and returns a polymorphic `Array` that may be of another type
*  `Functor<f> => (a -> b) -> f<a> -> f<b>` - denotes a higher order function that takes a polymorphic function and a ad-hoc polymorphic type and returns a ad-hoc polymorphic type that may contain a different type. There is a Functor contraint on `f`. 

### `Map`

*  `Map(?)` - denotes an unknown map-`Object`
*  `Map(Number: String)` - denotes a homogenious `Map` of `Number`/`String` key/value-pairs
*  `Map(Number: String, Number: Boolean)` - denotes a heterogeneous `Map` of `Number`/`String` and `Number`/`Boolean` key/value-pairs
*  `Map(k: v)` - denotes a parametric polymorphic `Map`

### `NaN`

Denotes the unit type `NaN` (not a number) value.

### `Null`

Denotes the unit type `null` value.

### `Number`

*  `Number` - denotes either a  signed/unsigned `Float` or a signed/unsigned `Integer` value
*  `Integer` - denotes a signed/unsigned `Integer` value
*  `Float` - denotes a signed/unsigned `Float` value

### `Object`

*  `{?}` - denotes an unknown `Object` created by the default constructor
*  `{foo: String, ?}` - denotes a partially known `Object` created by the default constructor
*  `Name{?}` - denotes an unknown `Object` created by the constructor `Name`
*  `Name{foo: String, ?}` - denotes a partially known `Object` created by the constructor `Name`
*  `{foo: String, bar: Number}` - denotes a known `Object` created by the default constructor
*  `Name{foo: String, bar: Number}` - denotes a known `Object` created by the constructor `Name`
*  `{foo: String, bar: Number, ...a}` - denotes a row polymorphic* `Object` with two properties created with the default constructor
*  `Name{foo: String, ...a}` - denotes a row polymorphic `Object` with one property created with the constructor `Name`

### `RegExp`

Denotes a regular expression `Object` value.

### `Set`

*  `Set(?)` - denotes an unknown set-`Object`
*  `Set(String)` - denotes a homogeneous `Set` of `String`s
*  `Set(String, Number)` - denotes a heterogeneous `Set` of `String`s/`Number`s
*  `Set(v)` - denotes a parametric polymorphic `Set`

### `String`

`Char` - denotes a `String` of length one
`String` - denotes a `String`

### `Undefined`

Denotes the unit type `undefined` value.

### `union`/`struct` Types

*  `Name<String> - denotes a type with a field of type `String` created by the constructor `Name`
*  `Name<String, Number> - denotes a type with two fields of type `String`/`Number` created by the constructor `Name`
*  `Name<a, b>` - denotes a type with two parametric polymorphic fields
*  m<a>` - denotes a type that consists of a context type with a monad constraint and a wrapped parametric polymorphic value

### Placeholders

You can shorten long type signatures by replacing sensible sub signatures with a placeholder:

```Javascript
// _1_:
// {foo: String, bar: Number, ...a lot more properties}

// _1_ -> SomeType<_1_>
```
### Semantic Typing

This is a made-up term. It means that sometimes you should use types for the single reason to enable more telling type signatures. This justifies the additional boilerplate to wrap and unwrap the raw data respectively:

```Javascript
// Task<[[ParserResult<...>]], Error> ->
// Task<[[[SqlQuery<String>]]], Error>
const buildSqlQueries =
  tMap(
    arrFold(acc => tx =>
      arrPush(acc)
        (buildSqlQueries_(tx))) ([]));
```
is transformed into

```Javascript
// Task<[BankAdvice<[ParserResult<...>]>], Error> ->
// Task<[BankAdvice<[AdviceRecord<[SqlQuery<String>]>]>], Error>
```
# Naming Convetions

## Function Names

The following rules apply to function names:

* with an f-postfix (e.g. `foof`) denotes a function with its arguments flipped: `const subf = y => x => x - y`
* with an x-postfix (e.g. `foox`) denotes a destructive function, whose mutation is visible in the parent scope
* with an var-prefix (e.g. `varFoo`) denotes most likely a variadic one
* with an _-prefix (e.g. `_foo`) is used to avoid naming claches with reserved names.
* with an _-postfix (e.g. `foo_`) denotes a slightly modified variant of an already existing one

## Variable Names

* `f`/`g` denotes pure functions
* `x`/`y` denotes a value of arbitrary type
* `tx`/`ty`/... denotes a value wrapped in a type without specifying any constraints like monadic, applicative, functorial etc.
* `tf`/`tg` denotes a function wrapped in a type
* `ts` denotes an array of values wrapped in a type
* `mx`/`my` denotes a value wrapped in a monadic type
* `ax`/`ay` denotes a value wrapped in a applicative type
* `fm`/`gm` denotes a monadic action, i.e. a function that returns a monad
* `ix`/`iy` denotes a stream of values generated by an iterator

# Advanced Topics

## Category Composition

Function composition is generalized in scriptum by the `Category` typeclass. This typeclass comprises the following functions:

* comp (right-to-left compostion)
* pipe (left-to-right compostion)
* comp3 (composing three functions - for convenience)
* pipe3 (piping three functions - for convenience)
* varComp (variadic composition)
* varPipe (variadic pipe)

That means each function type that implements `Category` has automatically access to a variadic interface. This is desirable because it allows for flat compositions/pipes and avoids explicitly nested function call trees:

```Javascript
const inc = x => x + 1,
  sqr = x => x * x;
  
funVarComp(inc) (sqr) (inc) (inc) (inc).runVarComp(1); // 25
funVarPipe(inc) (inc) (inc) (inc) (sqr).runVarComp(1); // 25
```
`runVarComp` is the indicator to start evaluating the composition tree.

Here is another example for applicative lifting through the variadic interface:

```Javascript
const sum3 = x => y => z => x + y + z;

optVarLiftA(sum3) (Some(1)) (Some(2)) (Some(3)); // Some(6)
optVarLiftA(sum3) (Some(1)) (None) (Some(3)); // None
```
Other suitable function types are

* applicative chaining/sequencing
* monadic lifting
* monadic chaining/sequencing
* kleisli composition

All variadic combinators are based on the `varArgs` function. You can easily create your own variadic combinators with it:

```Javascript
const add = x => y => x + y;

const sum = ns =>
  ns.reduce((acc, n) => acc + n, 0);

const varSum = varArgs(sum);

varSum(1) (2) (3) (4).runVarArgs // 10
```
## Structural Folding

### Catamorphism et al.

For some datatypes a catamorphism is a generalization of a fold/reduction (e.g. trees). For others both coincide (e.g. `Array`). And yet others don't have a fold at all (e.g. `Option`). 

For any non-primitive type the associated catamorphism is the dual of the constructor. The constructor defines the introduction rule, whereas the catamorphism defines the elimination rule. Hence catamorphisms represent the notion of destructuring data types.

scriptum implements catamorphisms as trampolines to obtain stack safety. Here is an example for the `Array` type, where catamorphism and fold coincide:

```Javascript
const add = x => y => x + y,
  mul = x => y => x * y,
  sum = arrFold(add) (0),
  prod = arrFold(mul) (1);

sum([1,2,3,4,5]); // 15
prod([1,2,3,4,5]); // 120
```
There is also a fold with short circuit semantics:

```Javascript
const lte = y => x => x <= y;

const addWhile = p => x => y =>
  p(x + y)
    ? Done(x + y)
    : Loop(x);

arrFoldWhile(addWhile(lte(9))) (0) ([1,2,3,4,5]); // 6
```
`arrFoldWhile` takes an algebra that determines the short circuit behavior of the fold. It uses the `Step` union type to indicate either another iteration (`Loop`) or short circuiting (`Done`).

Maybe you've noticed that the given examples are based on a left associative fold. Even though left and right folds are isomorphic by `flip`/`Array.prototype.reverse`, scriptum provides a distinct implementation of a right associative fold mainly for performance reasons. As opposed to Haskell's `foldr` it is strictly evaluated though.

More folds will follow:

* Paramorphism (fold with current state of the context)
* Hylomorphism (unfold composed with fold)
* Zygomorphism (one fold depending on another - semi-mutual recursive)
* Mutumorphism (mutual recursive fold - mutual recursive)
* Histomorphism (fold with access to all previous intermediate results)

### Anamorphism et al.

Anamorphisms are the dual of catamorphisms. You start with a seed value and apply the coalgebra to the seed and then iteratively to the result of the previous application, while all intermediate results are stored in a structure:

```Javascript
const nextLetter = c =>
  String.fromCharCode (c.charCodeAt (0) + 1)

const main = arrUnfold(c =>
  c > "z"
    ? None
    : Some([c, nextLetter(c)]));

main("a"); // ["a", "b", "c", "d", "e", ...]
```
More unfolds will follow:

* Apomorphism (unfold with early termination)
* Futumorphism (unfold with access to values still to be computed)

## Stack-Safe Recursion with Trampolines

### Direct Recursion in Tail Position

scriptum uses a Javascript port of clojure's `loop`/`recur` combinators as a trampoline:

```Javascript
const loop = f => {
  let step = f();

  while (step && step.type === recur)
    step = f(...step.args);

  return step;
};

const recur = (...args) =>
  ({type: recur, args});
```
Now we don't have to bother with stack overflows any longer but can utilize recursion when we see fit:

```Javascript
const stackSafeFold = f => acc_ => xs =>
  loop((acc = acc_, i = 0) =>
    i === xs.length
      ? acc
      : f(acc) (xs[i]) (acc_ => recur(acc_, i + 1)));

const xs = Array(1e6)
  .fill(0)
  .map((x, i) => i);
  
const stackSafeSum = stackSafeFold(x => y => k => x < Infinity ? k(x + y) : x) (0);

stackSafeSum(xs); // 499999500000
```
This works, because `stackSafeFold` implements direct recursion in tail position.

### Indirect Recursion in Tail Position

For mutual recursion we need a more complex trampoline:

```Javascript
const tramp = f => (...args) => {
  let step = f(...args);

  while (step && step.type === recur) {
    let [f, ...args_] = step.args;
    step = f(...args_);
  }

  return step;
};

const recur = (...args) =>
  ({type: recur, args});
```
Now we can express mutual recursion in a stack-safe manner too:

```Javascript
const even = n =>
  n === 0
    ? true
    : recur(odd, n - 1);

const odd = n =>
  n === 0
    ? false
    : recur(even, n - 1);

trampoline(even) (1e6 + 1)); // false
```
As you can see the trampoline API leaks on the calling site and there is nothing we can do about it. Stac-safe mutual recursion is a big win though, especially when you have to deal with data types that are defined in terms of each other.

### Non-Tail Recursion

Handling non-tail recursive algorithms with trampolines and explicit stacks remains a matter of research for the time being.

## Transducer

Transducers entail the following properties:

* they are composable and thus allow loop fusion
* they are data type agnostic as long as these data types are foldable

The following example illustrates loop fusion:

```Javascript
const mapper = f => reduce => acc => x =>
  reduce(acc) (f(x));

const filterer = p => reduce => acc => x =>
  p(x) ? reduce(acc) (x) : acc;
  
const arrFold = alg => zero => xs => {
  let acc = zero;

  for (let i = 0; i < xs.length; i++)
    acc = alg(acc) (xs[i], i);

  return acc;
};

const comp = f => g => x => f(g(x));

const add = x => y => x + y; // reducer
const sqr = x => x * x; // transformer
const isOdd = x => (x & 1) === 1; // predicate

// MAIN

const main = 
  arrFold(comp(
    filterer(isOdd)) 
      (mapper(sqr))
        (add)) (0);

main([1,2,3]); // 10 - array is only traversed once
```
## Effect Handling

Effects are handled with monads and their superclasses in scriptum. A monad is essentially a type that you cannot freely leave, i.e. an effect in a monad is effectively encapsulated.

### Single Effects

TODO

### Effect Composition

You can compose functorial and applicative effects in a principled manner either by hand or using the `Comp` type. Monadic effects, however, where the next monadic computation can depend on the result of the previous one, can only be composed by hand, because every composition of two monads is specific to the involved monads.

#### Monad Transformers

Monad transformers facilitate the composition of monads.

TODO

## Asynchronous Computations

As already mentioned scriptum distinguishes between sequential and parallel evaluation of asynchronous computations. In order to make this distinction explicit each veriant has its own type.

### `Task`

TODO

### `Parallel`

TODO

### Sharing

Sharing just means that intermediate results of asynchronous computations are evaluated only once but can be used any number of times.

TODO

## Functional Optics

TODO

### Lenses

TODO

### Prisms

TODO

### Folds

TODO

### Traversals

TODO

## Lazy Evaluation

### Functions

Functions generalize expressions by substituting subexpressions with arguments. As an effect such generalized expressions are only evaluated when all arguments are provided. While this is a simple form of lazy evaluation, the passed arguments are strictly evaluated.

### ETA Abstraction

In a strictly evaluated language like Javascript ETA abstraction is a way to add some extra lazyness to expressions:

```Javascript
const fold = f => acc => xs =>
  xs.reduce((acc_, x) => f(acc_) (x), acc);
  
const sum = fold(xs => y =>
  (xs.push(y + xs[xs.length - 1]), xs)) ([0]);

const sumEta = xs =>
  fold(ys => y => (ys.push(y + ys[ys.length - 1]), ys)) ([0]) (xs);

sum([1,2,3]); // [1,3,6]
sum([1,2,3]); // [1,3,6,7,9,12]

sumEta([1,2,3]); // [1,3,6]
sumEta([1,2,3]); // [1,3,6]
```
### Function Composition

From another perspective you can think of a function composition as a function `f` that takes an argument `g`, which is stuck in another function `x => f(g(x))` and thus only evaluated if the final argument is passed:

```Javascript
const comp = f => g => (x => f(g(x))); // redundant parenthesis to illustrate the idea
```
In lazily evaluated languages with call by need or call by name evaluation strategy such lazily evaluated arguments are the default.

### Continuation Passing Style

We can go beyond lazyness through function composition by encoding functions in continuation passing style:

```Javascript
const inck = x => k => k(x + 1),
  id = x => x;

const mapk = f => xs => k => {
  const go = (acc, i) =>
    i === xs.length
    ? acc
    : f(xs[i]) (x => go(acc.concat(x), i + 1));
  
  return k(go([], 0));
};

const main = mapk(inck) ([1,2,3]); // still lazy

main(id); // [2,3,4]
```
With CPS we can define lazily evaluated function call trees. However, CPS encodings get also quickly convoluted. We can probably ease the pain by abstracting from CPS with the continuation monad. I need to do more research on this promissing topic.

### Generators

Generators are the most natural form of expressing lazy evaluation in Javascript and the most harmful as well: They are stateful - not by design but by desicion. scriptum tries to avoid generators as often as possible. However, if we need lazyness inside imperative statements like `if`/`else` conditions or `while` loops we need to fall back to them, as there is no other way to suspend these strictly evaluated control structures.

### Explicit Thunks

Whenever we run synchronous effects (e.g. `Window.localeStorage` or `Date.now()`) it is a good idea to defer this operation and make it explicit by wrapping it in a thunk, which itself is wrapped in an appropriate type. scriptum differs between efffectful computations with and without memoization.

#### `Defer` w/o Sharing

`Defer` wraps an expression in a thunk and evaluates it on each call, i.e. doesn't share intermediate results:

```Javascript
const effectfulExp = Defer(
  () => {
    const r = 5 * 5;
    console.log(r);
    return r;
  });
  
effectfulExp.runDefer(); // logs + returns 25
effectfulExp.runDefer(); // logs + returns 25
```
#### `Lazy` with Sharing

`Lazy` wraps an expression in a thunk and evaluates it only once, i.e. does share intermediate results:

```Javascript
const effectfulExp = Lazy(
  () => {
    const r = 5 * 5;
    console.log(r);
    return r;
  });
  
effectfulExp.runLazy(); // logs + returns 25
effectfulExp.runLazy(); // returns 25
```
### Getters/Setters

Getters and setters are just thunks (and functions) and inherit their lazy traits. Why do they have their own section then? Because they allow us to introduce lazyness into Javascript without altering the calling side, because they are treated as normal properties:

```Javascript
const cons = (head, tail) => ({head, tail});

const list = cons(1, cons(2, cons(3, cons(4, cons(5, null)))));

const take = n =>
  n === 0
    ? xs => null
    : xs => xs && {
      head: xs.head,
      get tail() {
          delete this.tail;
          return this.tail = take(n - 1) (xs.tail);
      }
};

take(3)(list); // stack safe
```
although `take` isn't tail recursive it is stack safe no matter how long the list is. Lazy getters give us tail call optimization modulo cons for free!

scriptum utilizes lazy getters to allow for a simple yet concise form of pattern matching on tagged unions:

```Javascript
const match = ({[TYPE]: type, [TAG]: tag}, o) =>
  o.type !== type ? _throw(new UnionError("invalid type"))
    : !(tag in o) ? _throw(new UnionError("invalid tag"))
    : o[tag];
    
const optCata = none => some => tx =>
  match(tx, {
    type: "Option",
    None: none,
    get Some() {return some(tx.runOption)}
  });
```
## Delimited Continuations with `shift`/`reset`

TODO

## Persistent Data Structures

TODO

### Hash Array Mapped Trie (HAMT)

TODO

## Functional Reactive Programming

TODO

### Behavior

TODO

### Events

TODO

### Rules

TODO

## Library Specific Combinators

### `comp2nd`/`pipe2nd`

Composes/pipes a binary function on its second argument:

```Javascript
arrFold(
    comp2nd(
      objGet(0) (k))
        (add)) (0)
          ([{foo: 1}, {foo: 2}, {foo: 3}]); // 6
```
### `compBin`/`pipeBin`

Composes an unary with a binary function:

```Javascript
const add = x => y => x + y;
const sqr = x => x * x;

compBin(sqr) (add) (2) (3); // 25
```
### `compBoth`/`pipeBoth`

Composes a binary in both its arguments with an unary function:  

```Javascript
const add = x => y => x + y;
const length = s => s.length;

compBoth(add) (length) ("foo") ("bar"); // 6
```
### `infixl`/`infixr`

Just mimics Haskell's left/right associative, binary operators in infix position. This sometimes improves the readability of code:

```Javascript
infixl([1,2], arrAppend, [3,4]); // [1,2,3,4]
infixr([1,2], arrAppend, [3,4]); // [3,4,1,2]
```
### `invoke`

Takes a method name, any number of arguments and the corresponding object type and invokes the right method on the passed object. It makes the implicit context explicit so to speak.

```Javascript
invoke("map") (x => x * x) ([1,2,3]); // [1,4,9]
```
### `_let`

mimics let bindings as expressions and thus leads to concise function bodies:

```Javascript
const orDef = f => def => x => {
  const y = f(x);
    
  if (y === undefined || y === null || Number.isNaN(y))
    return def;

  else return y;
};
```
is simplified to

```Javascript
const orDef = f => def => x =>
  _let((y = f(x)) => 
    y === undefined || y === null || Number.isNaN(y)
      ? def : y);
```
### `memoMethx`

Defines a memoized method that is guaranteed only called once and than replaced by its result without the interface beeing changed:

```Javascript
const p = thisify(o => {
  memoMethx("foo") (x => (console.log("evaluated"), x * x)) (o);
  return o;
});

o.foo(5); // evaluated 25
o.foo(5); // 25
```
The trailing `x` within the function name indicates that it performs a mutation on the given `Object`.

### `objPathOr`

In Javascript it happens sometimes that you don't know your nested `Object` types. Here is a safe lookup function for arbitrarily nested `Object` trees that provides a default value instead of throwing an error:

```Javascript
const o = {foo: {bar: {baz: 123}}};

objPathOr(0) ("foo") ("bar") ("baz"); // 123
objPathOr(0) ("foo") ("bat") ("baz"); // 0
```
### `throwOnUnit`

Often we want to throw an error if a computation yields an unit type, i.e. a type without values. Javascript includes a remarkable number of unit types:

* undefined
* null
* NaN
* Invalid Date
* Infinity

```Javascript
comp(throwOnUnit)
  ("TypeError", "non-empty array expected")
    (head)
      ([1,2,3]); // 1

comp(throwOnUnit)
  ("TypeError", "non-empty array expected")
    (head)
      ([]); // TypeError
```
You can define your own functions with throw on error semantics by using the `onThrow` combinator.

### `partial`/`partialCurry`

The former just applies partial application, that is to say you call a multi argument function with some of its arguments and provide the rest at the subsequent call.

The latter combines partial application with currying:

```Javascript
const sum4 = (w, x, y, z) => w + x + y + z);

partialCurry(sum4), 1, 2) (3) (4); // 10
```
### `thisify`

Creates a `this`-like context for "methods" depending on other properties. Please recall that as decent functional programmers we don't depend on `this`/`new` directly:

```Javascript
const p = thisify(o => ({
  o.foo = 2;
  o.bar = x => x + o.foo;
}));

p.bar(3); // 5
```
# Language Conflicts

## `Array.prototype.concat`

`Array.prototype.concat` is one of the most harmful methods in Javascript for two reasons:

* it treats `Array`s as if they were persistant data structures with structural sharing
* it has an ambigious type that excepts both `x` and `[x]`

The first issue leads to insanely slow performance and the latter to subtle bugs:

```Javascript
const varArgs = f => {
  const go = args =>
    Object.defineProperties(
      arg => go(args.concat(arg)), {
        "runVarArgs": {get: function() {return f(args)}, enumerable: true},
        [TYPE]: {value: "VarArgs", enumerable: true}
      });

  return go([]);
};
```
If you spot the bug and the resulting trouble right away, well, lucky you. Anyway, scriptum replaces `Array.prototype.concat` with the following combinators:

* `arrAppend`/`arrPrepend` (for non-destructive concatenating)
* `arrPush`/`arrUnshift` (for destructive pushing/unshifting)

Please note that `arrAppend` et al. is rigid in its type, i.e. it expects two `Array`s and throws an error otherwise. If you want to append/prepend a single value just wrap it in a singleton `Array`, e.g. `arrAppend(xs) ([3])`.

## `Object.assign`

`Object.assign` calls every getter/setter strictly during copying. This is undesired if you rely on their lazy evaluation semantics. scriptum comes with the `objUnion`/`objUnionx` combinator pair that replace the method adequately without notable performance penalty.

## `Object` Property Insertion Order

Although not part of the spec all majr Javascript engines traverse `Object` properties in insertion order. With scriptum you must not rely on this property, because the lib treats `Object`s unordered `Map`s.

# TODO

- [ ] add type signatures!!!
- [ ] add common transducers
- [ ] add persistent data structure by hash array mapped trie (HAMT)
- [ ] add List/Catenable List/Random Access/Difference List
- [ ] add Multiway Trees
- [ ] add Zipper data type
- [ ] add useful Comonads
- [ ] add Monad Transformers
- [ ] add Reference Type (value objects)?
- [ ] remove contravariant types (`Compare`, `Equiv`, `Predicate`)?
- [ ] replace `Array.prototype.concat` with `arrAppend`/`arrConcat`?
- [ ] replace `switch` with `match` as often as possible
- [ ] rename `traverse` to `mapA`?
- [ ] how to lift a semigroup into Option forming a monoid?
- [ ] how to create a monoid under Applicative?
- [ ] how to create a monoid under Alternative?
- [ ] shouldn't arrSortBy rely on compBoth?
