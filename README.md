<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [@hulu/roca](#huluroca)
  - [Getting Started](#getting-started)
    - [Install](#install)
    - [Add NPM hook](#add-npm-hook)
    - [Directory Layout](#directory-layout)
    - [Modify Channel Entry Point (if necessary)](#modify-channel-entry-point-if-necessary)
    - [Adding a Test](#adding-a-test)
    - [Output & CI Support](#output--ci-support)
      - [CLI Options](#cli-options)
        - [`-r`/`--require`](#-r--require)
  - [API](#api)
    - [Global Functions](#global-functions)
      - [`roca(args = {} as object) as object`](#rocaargs---as-object-as-object)
    - [The `roca` Object](#the-roca-object)
      - [`m.describe(description as string, func as object) as object`](#mdescribedescription-as-string-func-as-object-as-object)
      - [`m.fdescribe(description as string, func as object)`](#mfdescribedescription-as-string-func-as-object)
      - [`m.xdescribe(description as string, func as object)`](#mxdescribedescription-as-string-func-as-object)
      - [`m.beforeEach(func as object)`](#mbeforeeachfunc-as-object)
      - [`m.afterEach(func as object)`](#maftereachfunc-as-object)
      - [`m.it(description as string, func as object, args = invalid as dynamic)`](#mitdescription-as-string-func-as-object-args--invalid-as-dynamic)
      - [`m.fit(description as string, func as object)`](#mfitdescription-as-string-func-as-object)
      - [`m.xit(description as string, func as object)`](#mxitdescription-as-string-func-as-object)
      - [`m.it_each(arrayOfArgs as object, descriptionGenerator as object, func as object)`](#mit_eacharrayofargs-as-object-descriptiongenerator-as-object-func-as-object)
      - [`m.fit_each(arrayOfArgs as object, descriptionGenerator as object, func as object)`](#mfit_eacharrayofargs-as-object-descriptiongenerator-as-object-func-as-object)
      - [`m.xit_each(arrayOfArgs as object, descriptionGenerator as object, func as object)`](#mxit_eacharrayofargs-as-object-descriptiongenerator-as-object-func-as-object)
      - [`m.log(value as dynamic)`](#mlogvalue-as-dynamic)
      - [`m.addContext(ctx as object)`](#maddcontextctx-as-object)
    - [Within test cases: passing, failing, asserts](#within-test-cases-passing-failing-asserts)
      - [`m.pass()`](#mpass)
      - [`m.fail()`](#mfail)
      - [`m.assert.equal(actual, expected, errorMessage)`](#massertequalactual-expected-errormessage)
      - [`m.assert.notEqual(actual, expected, errorMessage)`](#massertnotequalactual-expected-errormessage)
      - [`m.assert.isTrue(value, errorMessage)`](#massertistruevalue-errormessage)
      - [`m.assert.isFalse(value, errorMessage)`](#massertisfalsevalue-errormessage)
      - [`m.assert.isInvalid(value, errorMessage)`](#massertisinvalidvalue-errormessage)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# @hulu/roca

BrightScript unit testing.  No device required.

## Getting Started
Note: All examples use `npm`, but these steps should be easily adapted to [yarn](https://yarnpkg.com/), [pnpm](https://pnpm.js.org/), and many other JavaScript package managers.

### Install
Roca requires the [brs](https://github.com/sjbarag/brs/) runtime, which runs on [node.js](https://nodejs.org/en/).  If needed, initialize an NPM project with:

```bash
npm init
```

Then install `@hulu/roca` and `brs` as development dependencies with:

```bash
npm install --save-dev @hulu/roca brs
```

### Add NPM hook
To allow `npm test` to run your unit tests, simply call `roca` in your `package.json`'s "test" script:

```jsonc
{
    // ...
    scripts: {
        "test": "roca"
    }
}
```

### Directory Layout
Roca tests must be in a *sibling* of your existing `source/` and `components/` tree, like this:


```
.
├── components
│  ├── FooComponent.brs
│  └── FooComponent.xml
├── manifest
├── source
│  ├── main.brs
│  └── util.brs
└── tests
   ├── FooComponent.test.brs
   └── util.test.brs
```

Additionally, `roca` should be called from the same directory as your `manifest` file.  If you keep your entire source tree in a subdirectory, you'll want to `cd` into that before executing `roca`.

### Modify Channel Entry Point (if necessary)
Roca only supports a main channel entry point of `runUserInterface`, which helpfully avoids naming conflicts with the `main` functions used in `roca` and the individual test files.

Luckily `runUserInterface` is supported for all the same use-cases as `main`: https://developer.roku.com/docs/developer-program/getting-started/architecture/dev-environment.md#sub-runuserinterface

### Adding a Test
Just like [Mocha](https://mochajs.org/) tests are written in JavaScript, roca tests are written in BrightScript.  Roca will recursively find and execute all files that match `*.test.brs` in any of the following directories: `source/`, `components/`, `tests/`, and `test/`. The smallest possible unit test must meet the following requirements:

1. A function called `main` that accepts one argument of type `object` and returns a value of type `object`
2. That `main` function initializes a Roca instance by passing the arguments from (1) into `roca()`.
3. That `main` function returns the return value of the `someRocaInstance.describe()` function
4. A test case declared with `m.it`
5. The test case passes or fails with `m.pass()`, `m.fail()`, or an assertion that calls one of those.

Put simply:

```brightscript
function main(args as object) as object
    return roca(args).describe("test suite", sub()
        m.it("has a test case", sub()
            m.pass()
        end sub)
    end sub)
end function
```


### Output & CI Support
Roca exclusively reports its state via the [Test Anything Protocol](http://testanything.org/), and defaults to a Mocha-like "spec" output.  Failed tests cause the `roca` CLI to return a non-zero exit code, which allows most continuous integration systems to automatically detect pass/fail states.

Other output formats are available!  See `roca --help` for more details.

#### CLI Options

| Option                  | Behavior       |
| ------------------------|----------------|
| `-h`/`--help`           | The help menu. |
| `-s`/`--source`         | Path to brs files (if different from source/) |
| `-R`/`--reporter`       | The mocha reporter to use, via [`tap-mocha-reporter`](https://github.com/tapjs/tap-mocha-reporter). See `--help` for options. |
| [`-r`/`--require`](#-r--require) |  Path to a required setup file (will be run before unit tests) |
| `-f`/`--forbid-focused` | Fail if focused test or suite is detected. |
| `-c`/`--coverage-reporters` | The `istanbul` coverage reporters to use. Passing in reporter(s) will enable coverage collection and reporting. Otherwise, it is disabled. See `--help` for options, and [`istanbul`'s docs](https://istanbul.js.org/docs/advanced/alternative-reporters/) for descriptions of the reporters. |

##### `-r`/`--require`
In order to enable custom unit test helper functions and/or unit test setup code, you can create a BrightScript file that will be run before any of your unit tests. Pass it to `roca` via the `-r`/`--require` flag.

For example, say you wanted to define a helper function that you could call in any of your unit tests:
```brs
' inside myProject/tests/MySetupFile.brs
function callMeAnywhere()
  return 123
end function
```

Then in your code, you'd have:
```brs
' inside myProject/tests/MyTestFile.test.brs
...
m.it("adds local and remote login buttons to page", sub()
    print callMeAnywhere() ' => 123
end sub)
```

And then you'd tell to `roca` to use the setup file:
```bash
roca -r "tests/MySetupFile.brs"
```

## API
### Global Functions
#### `roca(args = {} as object) as object`
Roca only has one global function &mdash; `roca` &mdash; used to initialize a new Roca test instance.  You probably want
to call it only once.

Parameters:
* `args = {} as object` - (optional) the set of arguments provided by the Roca test framework

Return value:
* the newly-created test framework as an associative array

Example:
```brightscript
function main(args as object) as object
    r = roca(args)
    print r    ' => an instance of roca
    return r
end sub
```

### The `roca` Object
Defining a test case matches the semantics of [Jasmine](https://jasmine.github.io/), using `it` (or `fit` and `xit`) and `describe` functions accessible on the `m` scope within a suite.

#### `m.describe(description as string, func as object) as object`
Creates a new test suite that will contain test cases.  Note that there must always be one root suite, created by calling `describe()` directly on the result of `roca().

Parameters:
* `description as string` - a string describing the test case executed by `func`
* `func as object` - the function to execute as part of this test suite

Return value:
* the newly-created test suite as an associative-array, including test-pass state and test cases

Example:
```brightscript
function main(args as object) as object
    roca(args).describe("a test suite", sub()
        ' this function is executed when the suite is executed

        m.describe("a sub-suite", sub()
            m.describe("a sub-sub-suite", sub()
                ' sub-suites can be nested arbitrarily
            end sub)
        end sub)
    end sub)
end function
```

#### `m.fdescribe(description as string, func as object)`
Creates a focused test suite &mdash; a test suite that causes all non-focused tests *in all files* to be ignored.  Useful when debugging specific sets of unit tests.

Parameters:
* `description as string` - a string describing the test case executed by `func`
* `func as object` - the function to execute as part of this test case

Example:
```brightscript
m.describe("an unfocused test suite", sub()
    m.it("a test case", sub()
        ' this test won't run, because another suite is focused
    end sub)
end sub)

m.fdescribe("a focused test suite", sub()
    m.it("a test case inside of a focused suite", sub()
        ' this test *will* be executed, because it's inside a focused suite
    end sub)

    ' This sub-suite will run, because it's inside of a focused suite
    m.describe("a sub-suite", sub()
        m.it("a test case inside of the sub-suite", sub()
            ' this test *will* be executed, because it has focused ancestors
        end sub)
    end sub)
end sub)
```

#### `m.xdescribe(description as string, func as object)`
Creates a skipped unit test suite.  Any tests and sub-suites inside it will never execute.

Parameters:
* `description as string` - a string describing the test case executed by `func`
* `func as object` - the function to execute as part of this test case

Example:
```brightscript
m.xdescribe("an skipped test suite", sub()
    m.it("a test case", sub()
        ' this test will be skipped, because it's inside of a skipped suite
    end sub)

    ' This sub-suite will be skipped, because it's inside of a skipped suite
    m.describe("a sub-suite", sub()
        m.it("a test case inside of the sub-suite", sub()
            ' this test will be skipped, because it has skipped ancestors
        end sub)
    end sub)
end sub)
```

#### `m.beforeEach(func as object)`
Runs the given `func` before each child test suite and case. Note: it will run in the `m` context of the following test or suite, which will allow you to set fields on `m` within `beforeEach` and access them in the test or suite. However, **the `m` contex is different for each test/suite, so if you need state to be shared across multiple tests/suites, you should be using [`m.addContext`](#maddcontextctx-as-object)**.

Parameters:
* `func as object` - the function to execute prior to each child test/suite

Example:
```brightscript
m.describe("a test suite", sub()
    m.beforeEach(sub()
        doSomeCoolMocksOrSetup()
        m.foo = "bar"
    end sub)

    m.it("a test case", sub()
        print m.foo ' => "bar"
        m.iWannaShareThis = true

        m.pass()
    end sub)

    m.it("a second test case", sub()
        print m.foo ' => "bar"
        print m.iWannaShareThis ' => invalid

        m.pass()
    end sub)
end sub)
```

#### `m.afterEach(func as object)`
Runs the given `func` after each child test suite and case. Note: it will run in the `m` context of the previous test or suite, which will allow you to set fields on `m` in the test/suite and then access them in `afterEach`. However, **the `m` contex is different for each test/suite, so if you need state to be shared across multiple tests/suites, you should be using [`m.addContext`](#maddcontextctx-as-object)**.

Parameters:
* `func as object` - the function to execute after each child test/suite

Example:
```brightscript
m.describe("a test suite", sub()
    m.afterEach(sub()
        doSomeCoolTeardown()
        print m.foo ' => "bar" (after first test)
                    ' => invalid (after second test)
    end sub)

    m.it("a test case", sub()
        m.foo = "bar"
        m.pass()
    end sub)

    m.it("a second test case", sub()
        m.pass()
    end sub)
end sub)
```

#### `m.it(description as string, func as object, args = invalid as dynamic)`
Creates a test case with a given description.

Parameters:
* `description as string` - a string describing the test case executed by `func`
* `func as object` - the function to execute as part of this test case
* `args = invalid as dynamic` - optional parameter that will be passed in `func` as an argument

Example:
```brightscript
m.it("a test case", sub()
    if 4 / 2 = 2 then
        m.pass()
    else
        m.fail()
    end if
end sub)

m.it("a test case with args", sub(count)
    if count > 0 then m.pass() else m.fail()
end sub, 10)
```

#### `m.fit(description as string, func as object)`
Creates a focused test case &mdash; a test case that causes all non-focused tests *in all files* to be skipped.  Useful when debugging specific unit tests.

Parameters:
* `description as string` - a string describing the test case executed by `func`
* `func as object` - the function to execute as part of this test case
* `args = invalid as dynamic` - optional parameter that will be passed in `func` as an argument

Example:
```brightscript
m.it("a test case", sub()
    ' this test won't run, because another is focused
end sub)

m.fit("a focused test case", sub()
    ' this test *will* be executed, because it's focused
    if 1 + 1 = 3 then
        m.pass()
    else
        m.fail()
    end if
end sub)
```

#### `m.xit(description as string, func as object)`
Creates a skipped unit test case.  It will never execute.

Parameters:
* `description as string` - a string describing the test case executed by `func`
* `func as object` - the (skipped) function to execute as part of this test case
* `args = invalid as dynamic` - optional parameter that will be passed in `func` as an argument

Example:
```brightscript
m.xit("a omitted test case", sub()
    m.fail() ' this test will never be executed, so it doesn't matter if it's explicitly failed
end sub)
```

#### `m.it_each(arrayOfArgs as object, descriptionGenerator as object, func as object)`
Creates a parameterized test cases with generated description.
Will be expanded as `m.it` in for cycle with the passing argument to `func` from arguments array

Parameters:
* `arrayOfArgs as object` - the array of args that will be passed in `func`
* `descriptionGenerator as object` - a function that generate description string
* `func as object` - the array of args, these args will be passed in `func`

Example:
```
m.it_each([
    [1, 1],
    [2, 1],
],
function (args)
    return substitute("compare {0} with {1}", args[0].tostr(), args[1].tostr())
end function,
sub(args)
    if args[0] = args[1] then m.pass() else m.fail()
end sub)
```

#### `m.fit_each(arrayOfArgs as object, descriptionGenerator as object, func as object)`
Creates a parameterized test cases with generated description.
Will be expanded as `m.fit` in for cycle with the passing argument to `func` from arguments array

Parameters:
* `arrayOfArgs as object` - the array of args that will be passed in `func`
* `descriptionGenerator as object` - a function that generate description string
* `func as object` - the array of args, these args will be passed in `func`

Example:
```
m.fit_each([
    [1, 1],
    [2, 1],
],
function (args)
    return substitute("focused test, compare {0} with {1}", args[0].tostr(), args[1].tostr())
end function,
sub(args)
    if args[0] = args[1] then m.pass() else m.fail()
end sub)
```

#### `m.xit_each(arrayOfArgs as object, descriptionGenerator as object, func as object)`
Creates a parameterized test cases with generated description.
Will be expanded as `m.xit` in for cycle with the passing argument to `func` from arguments array

Parameters:
* `arrayOfArgs as object` - the array of args that will be passed in `func`
* `descriptionGenerator as object` - a function that generate description string
* `func as object` - the array of args, these args will be passed in `func`

Example:
```
m.xit_each([1, 2, 3],
function (args)
    return "omitted parameterized test case"
end function,
sub(args)
    m.fail()' this test will never be executed, so it doesn't matter if it's explicitly failed
end sub)
```

#### `m.log(value as dynamic)`
Prints a value to the console in a TAP-safe way.

Parameters:
* `value as dynamic` - the value to print to the console

Example:
```brightscript
m.it("prints diagnostic info", sub()
    m.log(someFunctionCall())
end sub)
```

#### `m.addContext(ctx as object)`
Provides context to cases in the form of extra fields.

- `ctx` should be a `roAssociativeArray`,
- `m.addContext` can be called multiple times,
- Context cascades into sub-suites.

Example:
```brightscript
m.addContext({
    aField: "value 1"
})

m.describe("a test suite with context", sub()
    m.addContext({
        aFn: function()
            return "value 2"
        end function
    })

    m.it("a test case", sub()
        m.assert.equal("value 1", m.aField, "Get context field")
        m.assert.equal("value 2", m.aFn(), "Use context function")
    end sub)
end sub)
```

### Within test cases: passing, failing, asserts

You can directly call `m.pass()` or `m.fail()` within a test case to mark it as passed or failed. Alternatively, `roca` offers an `assert` library (similar to [Chai](https://www.chaijs.com/api/assert/)), available via `m.assert`. These methods will call `m.pass()` or `m.fail()` for you.

#### `m.pass()`
Marks a unit test as passing. If the test case has previously been marked as failed, then this results in a no-op.

Example:
```brightscript
m.it("passes", sub()
    m.pass()
end sub)
```

#### `m.fail()`
Marks a unit test as failing. Only the first failure encountered in a test case will be reported, and it will override any prior calls to `m.pass`.

Example:
```brightscript
m.it("fails", sub()
    m.fail()
end sub)
```

#### `m.assert.equal(actual, expected, errorMessage)`
Compares two **primitive** values for equality.

Usage:
```brightscript
m.it("test case", sub()
    m.assert.equal("foo", "foo", "foo should not equal foo")
end sub)
```

#### `m.assert.notEqual(actual, expected, errorMessage)`
Opposite of `m.assert.equal`. Compares two **primitive** values.

Usage:
```brightscript
m.it("test case", sub()
    m.assert.notEqual("foo", "not foo", "foo should not equal foo")
end sub)
```

#### `m.assert.isTrue(value, errorMessage)`
Checks to see if a **boolean** value is `true`.

Usage:
```brightscript
m.it("test case", sub()
    foo = true
    m.assert.isTrue(foo, "foo should be true")
end sub)
```

#### `m.assert.isFalse(value, errorMessage)`
Checks to see if a **boolean** value is `false`.

Usage:
```brightscript
m.it("test case", sub()
    foo = false
    m.assert.isFalse(foo, "foo should be false")
end sub)
```

#### `m.assert.isInvalid(value, errorMessage)`
Checks to see if a value is `invalid`.

Usage:
```brightscript
m.it("test case", sub()
    foo = invalid
    m.assert.isInvalid(foo, "foo should be invalid")
end sub)
```
