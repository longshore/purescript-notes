# purescript-notes
Notes as I'm learning the Purescript language

### Importing

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
