# purescript-notes
Notes as I'm learning the Purescript language

## The `import` Keyword

psci (the Purescript repl) imports __no__ libraries by default. For that matter, neither `npm install -g purescript` nor `npm install -g pulp` will even download the core libraries for you. You'll need to use bower (`npm install -g bower`) to pull them down into your development folder. __TODO:__ remember how to download all of the core libraries without getting each one individually.

#### Importing Errors
##### Unknown module Prelude
```
Error found:
in module $PSCI.Support
at  line 3, column 1 - line 4, column 1

  Unknown module Prelude


See https://github.com/purescript/purescript/wiki/Error-Code-UnknownModule for more information,
or to contribute content related to this error.
```
__Diagnosis:__ If you get this when you try to do _anything_ in psci, you probably don't have Prelude installed where psci can find it. Where does psci look? I have no idea, but if you run psci with `pulp psci` it will look in the `bower_components` folder at the root of your project.

__Fix:__ `pulp init` to get a project, `npm install purescript` in that project to get psci's runtime dependencies (like Prelude).

##### Unkown value (+) [or whatever]
```
> 3 + 3
Error found:
in module $PSCI
at  line 1, column 1 - line 1, column 5

  Unknown value (+)


See https://github.com/purescript/purescript/wiki/Error-Code-UnknownValue for more information,
or to contribute content related to this error.
```
__Diagnosis:__ You might think something as basic as 2 + 2 shouldn't need any libraries, but + is actually defined in Prelude. The compiler has special handling for addition, and the repl needs Prelude to display this error, but they won't share their copies. You gotta get your own.

__Fix:__ In the repl, run `import Prelude` to get everything defined in Prelude dumped into your namespace.

## Algebraic Data Types: The `data` keyword

__Simplest form:__ `data ThingType = Thing`

The first word after `data` is the name of the type. The first word after `=` and each `|` are the name(s) of your type's constructor(s). (Constructors are values.) Note that the type and all constructors must be Capitalized.† Here, `Thing` is a value of type `ThingType`, and `ThingType` is a type of kind `*`. Values and types have their own namespaces, so `data Atom = Atom` is legal and idiomatic. There's no case were both a type and a value are both syntactically legal, so there's never any ambiguity. (because types are not first class values?)

†except for the `Boolean` type's values, `true` and `false`. Presumably they're lowercase because JavaScript.

__Multiple constructors:__ `data TriBool = Yes | No | Maybe`

`TriBool` here has three values/constructors: `Yes`, `No`, and `Maybe`. There's also a `Maybe` type defined in `Data.Maybe`; we can have both `TriBool` and the `Maybe` type in the same scope no problem, because our `Maybe` is a value and values and types have different namespaces. Don't actually do this though, it's confusing.

__Constructors with parameters:__ `data RationalNumberType = RationalNumber Int Int`

Our new type `RationalNumberType` is exactly the same kind of type as `ThingType` or `Atom` or `TriBool`. It's the constructor and shape of its values that are different. If you enter `:t RationalNumber` to check its type, you'll get `Int -> Int -> RationalNumberType` - the type of `RationalNumber` isn't `RationalNumberType`! It's a function that, given two `Ints`, returns a value of type `RationalNumberType`. We can use it like any other function - curry it, `map` it, bind it to other names, etc. Constructors will always be values - they'll either be a value of the type directly (like `Thing` or `Yes`) or a function (functions are values) that ultimately returns a value of that type. Note that we declare the types of `RationalNumber`'s parameters differently than with regular function syntax. If we were defining division as a non-constructor function, we would say:

```
let otherDivision numerator denominator = (Data.Int.toNumber numerator) / (Data.Int.toNumber denominator)
```

and the compiler would infer that the arguments `numerator` and `denominator` must be `Int`s (because `Data.Int.toNumber` only works on `Int`s) and that the return type must be `Number` because that's what division by a `Number` returns. With RationalNumber, we had to specify the argument types (`Int`) and return type (`RationalNumberType`) explicitly. As a side effect of this, we have no way to name its parameters! We have to hope our user knows the convention that the first `Int` is the numerator and the second is the denominator.

We can sort of name our parameters by using a __constructor with a row type:__
```
data RationalNumberType2 = RationalNumber2 { numerator :: Int, denominator :: Int }
let fourSevenths = RationalNumber2 { numerator: 4, denominator: 7 }
```
`RationalNumber2` is now a function that takes _one_ parameter: an object that has `Int` fields named `numerator` and `denominator` (it may also have other fields, but they wouldn't do anything.)

Remember, two colons to declare type of object field, one colon to declare value of field! If you write `RationalNumber2 { numerator = 5, denominator = 9 } `, you'll get some kind of error about not being able to match types. That implies that `{ foo = 5 }` is valid syntax for _something_ but I have no idea what.

Constructors can be mixed and matched freely:
```
data Color = Black | White | Red | RGB Int Int Int | CMYK { cyan :: Int, magenta :: Int, yellow :: Int } Number
```

They can even be recursive: 
```
data Royalty = Monarch String | Spouse Royalty | Child Royalty Royalty
let louisXIV = Monarch "France"
let maria = Spouse louisXIV
let louisJr = Child louisXIV maria
```

although the values themselves cannot recurse. `let nero = Spouse nero` will fail with Error-Code-CycleInDeclaration. Sorry, Nero.

There's no requirement for recursive types to have a 'base case':
```
data UselessRecursion = Recurse UselessRecursion
```
I don't think you could ever actually instantiate a value of type `UselessRecursion` but you can certainly declare the type! `Recurse` is a perfectly ordinary constructor function. `map Recurse []` will even get you an empty array of type `Array UselessRecursion`.

###Metatypes

__Simplest form:__ `data Meta foo = StrangeAtom`

`Meta` is our first metatype.‡ A metatype is a lot like a type; in some contexts types and metatypes can be used interchangeably, other times they act more like a function. However, metatypes are fundamentally a different _kind_ of thing than types. We can see this by checking a metatype's kind with the `:k` PSCI command:

```
> :k ThingType
*
> :k Meta
* -> *
```

‡Purescript does not call these metatypes. I'm calling them that.

Naming things is hard. Purescript's name for "the kind of things that have values" is `*`, which might be more familiar to you as magickal shorthand for the Eye of Horus. `->` is the function pseudoconstructor (an infix - it's parameterized by the `*`s on both sides). So is `Meta` a function? Not in the Purescript sense: functions are first-class values, while types and metatypes can only be used in places like type annotation. `Meta` is definitely in the latter category. In the mathematical sense, `Meta` is very much a function that maps one type (or other thing of kind `*`) to another, like so:

```
> :k Meta ThingType
*
> :k Meta (Meta ThingType) -- note the left associativity!
*
> :k Meta { count :: Int }
*
```

Note that `Meta Meta` will yield an error; `Meta` expects a thing of kind `*`, but we gave it `Meta`, which is of kind `* -> *`. We will see how to declare metatypes of kind `(* -> *) -> *` soon.

`StrangeAtom` is a value, but `Meta` isn't the kind of thing that can have values. So what is `StrangeAtom`'s type? Check in the interpreter:

```
> :t StrangeAtom
forall t4. Meta t4
```

Without annotations, the we can't say anything more specific than that `StrangeAtom` falls in the `Meta` family. Or rather, we can say with specificity that its type is the quantum superposition of `Meta t4`s for all possible types `t4`. We can collapse the superposition with type annotations:

```
> let strangeQ = StrangeAtom
> :t strangeQ
forall t4. Meta t4
> let strangeInt = StrangeAtom :: Meta Int
> :t strangeInt
Meta Int
> let strangeString = StrangeAtom :: Meta String
> :t strangeString
Meta String
```

or by forcing Purescript to unify the type `Meta t4` with a collapsed StrangeAtom:

```
> :t if true then strangeQ else strangeInt
Meta Int
> :t if true then strangeQ else strangeString
Meta String
```

We _cannot_ go the opposite direction and get the typefamily back from two different types:

```
> :t if true then strangeInt else strangeString
Error found:
in module $PSCI
at  line 1, column 1 - line 1, column 31

  Could not match type

    Int

  with type

    String


while trying to match type Meta Int
  with type Meta String
while inferring the type of if true
                              then strangeInt
                              else strangeString
in value declaration it

See https://github.com/purescript/purescript/wiki/Error-Code-TypesDoNotUnify for more information,
or to contribute content related to this error.
```

Purescript does not have the concept of a generic type like "Meta t4 where t4 is either Int or String". It has discriminated unions; they're just their own type, not a language feature:

```
> import Data.Either
> :t Left strangeInt
forall t5. Either (Meta Int) t5
> :t Right strangeString
forall t6. Either t6 (Meta String)
> :t if true then Left strangeInt else Right strangeString
Either (Meta Int) (Meta String)
```
