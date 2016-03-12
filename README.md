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

Simplest form: `data ThingType = Thing`

The first word after `data` is the name of the type. The first word after `=` and each `|` are the name(s) of your type's constructor(s). (Constructors are values.) Note that the type and all constructors must be Capitalized.* Here, `Thing` is a value of type `ThingType`, and `ThingType` is a type of kind `*`. Values and types have their own namespaces, so `data Atom = Atom` is legal and idiomatic. There's no case were both a type and a value are both syntactically legal, so there's never any ambiguity. (because types are not first class values?)

Multiple constructors: `data TriBool = Yes | No | Maybe`

`TriBool` here has three values/constructors: `Yes`, `No`, and `Maybe`. There's also a `Maybe` type defined in `Data.Maybe`; we can have both `TriBool` and the `Maybe` type in the same scope no problem, because our `Maybe` is a value and values and types have different namespaces. Don't actually do this though, it's confusing. If you're ever

Constructors with parameters: `data RationalNumberType = RationalNumber Int Int`

Our new type `RationalNumberType` is exactly the same kind of type as `ThingType` or `Atom` or `TriBool`. It's the constructor and shape of its values that are different. If you enter `:t RationalNumber` to check its type, you'll get `Int -> Int -> RationalNumberType` - the type of `RationalNumber` isn't `RationalNumberType`! It's a function that, given two `Ints`, returns a value of type `RationalNumberType`. We can use it like any other function - curry it, `map` it, bind it to other names, etc. Constructors will always be values - they'll either be a value of the type directly (like `Thing` or `Yes`) or a function (functions are values) that ultimately returns a value of that type. Note that we declare the types of `RationalNumber`'s parameters differently than with regular function syntax. If we were defining division as a non-constructor function, we would say:

```
let integerDivisionWithFloatingPointRemainder numerator denominator = (Data.Int.toNumber numerator) / (Data.Int.toNumber denominator)
```

and the compiler would infer that the arguments `numerator` and `denominator` must be `Int`s (because `Data.Int.toNumber` only works on `Int`s) and that the return type must be `Number` because that's what division by a `Number` returns. With RationalNumber, we had to specify the argument types (`Int`) and return type (`RationalNumberType`) explicitly. As a side effect of this, we have no way to name its parameters! We have to hope our user knows the convention that the first `Int` is the numerator and the second is the denominator.

We can sort of name our parameters by using a row type:
```
data RationalNumberType2 = RationalNumber2 { numerator :: Int, denominator :: Int }
let fourSevenths = RationalNumber2 { numerator: 4, denominator: 7 }
```

Remember, two colons to declare type of object field, one colon to declare value of field! If you do this:
```
let wrongBadDoNotDoThis = RationalNumber2 { numerator = 5, denominator = 9 }
```

you'll get some kind of error about not being able to match types. That implies that { foo = 5 } is valid syntax for _something_ but I have no idea what.

*except for the `Boolean` type's values, `true` and `false`. Presumably they're lowercase because JavaScript.
