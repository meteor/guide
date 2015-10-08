## Meteor Guide: Package Testing with TinyTest

After reading this guide, you'll know:

1. What Tinytest is used for.
1. How to include tests in your package.
1. How to structure your test file(s).
1. When to use synchronous or asynchronous tests.
1. How to run your tests.
1. What assertions and utility methods are available to you.
1. How to test specific types of functionality.

## What is Tinytest?

Tinytest is the official Meteor package test runner.

It's used to test *local* packages. You cannot test non-local packages (for example packages you have included using `meteor add some:package-name`) unless you clone their repos and make them available locally. The assumption is, of course, that you are testing packages under development.

Tinytest is specifically designed for *unit testing*. From [wikipedia](https://en.wikipedia.org/wiki/Unit_testing):

> ... Intuitively, one can view a unit as the smallest testable part of an application. In procedural programming, a unit could be an entire module, but it is more commonly an individual function or procedure. In object-oriented programming, a unit is often an entire interface, such as a class, but could be an individual method.

> ... Ideally, each test case is independent from the others. Substitutes such as method stubs, mock objects, fakes, and test harnesses can be used to assist testing a module in isolation.

The object of a unit test is to prove functionality you have written. It is not to prove third-party or other package or library code which may form a part of your codebase. This may require that you head up your tests with *stubs, mocks* or *fakes* which satisfy the demands of your code, but are non-functional (or have limited functionality). Alternatively, instead of placing the stubs in your test file, you could put them into a separate `stubs.js` file, which you `api.addFiles`, or even create and add a stubs package which you then `api.use`.

From [martinfowler.com](http://www.martinfowler.com/articles/mocksArentStubs.html):

> Fake objects actually have working implementations, but usually take some shortcut which makes them not suitable for production (an in memory database is a good example).

> Stubs provide canned answers to calls made during the test, usually not responding at all to anything outside what's programmed in for the test. Stubs may also record information about calls, such as an email gateway stub that remembers the messages it 'sent', or maybe only how many messages it 'sent'.

> Mocks [...]: objects pre-programmed with expectations which form a specification of the calls they are expected to receive.

## Adding Tinytest to Your Package

Tinytest is included through configuration in the `package.js` file using `Package.onTest()`.

```javascript
// Define test section
Package.onTest(function(api)
  // We will need the following packages to run our tests.
  // Note that we also need to include the package to be tested (myname:mypackage).
  api.use(['tinytest', 'underscore', 'ecmascript', 'myname:mypackage']
  // In v1.2 test-packages no longer include any packages globally.
  // You may need to make some exports global for your tests to run, for example:
  api.imply('underscore'
  // This file contains the tests we want to run (these will run on the client and the server)
  api.addFiles('tests.js'
  // You can add as many test files as you want and choose where they are to run:
  api.addFiles('server-tests.js', 'server'
  // and these only run on the client:
  api.addFiles('client-tests.js', 'client');
});
```

## Structuring the Test File

The test file is normally Javascript/ES2015 (but may be anything if the appropriate transpiler is included with `api.use('some-transpiler');` within `Package.onTest`). Examples in this guide assume ES2015, for which you will have to include an `api.use('ecmascript');` line in your `Package.onTest`.

A "test" contains the code you want to test, plus zero or more *assertions* wrapped within a `Tinytest.add` or `Tinytest.addAsync` callback. The `name` of the test is specified in the `add` or `addAsync` call. An assertion is a call to a method used to verify an actual result (or some property of a result) vs a expected result. For example, "is the result === 1?", or "is the result a JavaScript `Number`?".

If a test has no assertions it is assumed to pass unless an exception is caught. However, it is more usual to explicitly specify assertions.

In any single test all assertions must pass for the test as a whole to pass.

The `name` of the test may also be used to generate a hierarchical (grouped) result page. Hierarchical levels are split using " - " as a separator in the name. The name of the package should be at the head of the `name`:

    'mypackage - functional tests - test 1'
    'mypackage - functional tests - test 2'
    'mypackage - performance tests - test 1'
    'mypackage - performance tests - test 2'

Would be good `name`s to use for reporting test results in a hierarchy.

Multiple test files may be included in `package.js`. They will be processed in the order declared. However, test results are presented in the order in which they appear in their groupings. The pass/fail status of tests is updated in order, including waiting for *asynchronous* tests to complete.

## Synchronous and Asynchronous Tests

Tests may be *synchronous* or *asynchronous*.

A synchronous test is one in which the flow of code *does not* need to wait for the result of an external action to be made available through an on-completion callback.

Conversely, an asynchronous test is one in which the flow of code *does* need to wait for the result of an external action to be made available through an on-completion callback.

Synchronous tests are more common, but asynchronous tests are useful occasionally. With the introduction of Meteor Promises (promises running in fibers), we can now successfully write synchronous tests containing asynchronous code. However, examples of this usage are outside the scope of this section of the guide.

### Synchronous Tests

```javascript
Tinytest.add(name, function(test) {
  // test body
});
```

ES6/2015 (recommended):

```javascript
Tinytest.add(name, (test) => {
  // test body
});
```

### Asynchronous Tests

```javascript
Tinytest.addAsync(name, function(test, onComplete) {
  someAsyncRequest(function(error, result) {
    // test body
    onComplete(); // invoke when async function completes.
  }
})
```

ES6/2015 (recommended):

```javascript
Tinytest.addAsync(name, (test, onComplete) => {
  someAsyncRequest((error, result) => {
    // test body
    onComplete(); // invoke when async function completes.
  }
});
```

## Executing the Tests

Tests are executed by running the (development) application with the `test-packages` command. By default, all packages are tested, but specific packages can be tested by specifying them by name.

`meteor test-packages` runs tests on all (local) packages in the app.

`meteor test-packages her:package his:package` runs tests only on `her:package` and `his:package`.

The results are presented on the browser (port 3000) as usual. When run as shown, the file watcher runs, so code may be edited and the tests will be re-run automatically. If this is undesirable, you may add the `--run-once` switch.

## Assertions

The `test` object in the above callbacks is used to add assertions to the test.

```javascript
Tinytest.add('mypackage - basic tests', (test) => {

  // Get the result from our function under test.
  const result = myFunction(1);

  // Do the test ... calling myFunction with 1 should return 2.
  test.equal(result, 2);
});
```

Tests may include multiple assertions. The total number of successes and failures for each test are reported.

### Assertions with optional fail messages

An optional `message` may be added to the assertions which will change a failure report from `fail - <failure-test>` to `fail - <failure-test> - message <optional message>`

#### equal

test.equal(actual, expected[, message[, not]]);

`not` is a boolean (`true`/`false`) parameter which may be used to switch the sense (`equal` becomes `notEqual`). Used internally by `notEqual`. **Don't use it in your tests - it just makes your code harder to grok.**

Compares by type and value, so 2 is not equal to "2", for example. Is a better choice for checking something is strictly `true` or `false` than `test.isTrue` or `test.isFalse` (see below).

#### notEqual

test.notEqual(actual, expected[, message]);

Internally, calls `equal` with the `not` flag set to `true`.

#### matches

test.matches(actual, regexp[, message]);

Checks `regexp.test(actual)`

#### notMatches

test.notMatches(actual, regexp[, message]);

Checks `!regexp.test(actual)`

#### isTrue

test.isTrue(actual[, message]);

Checks that `actual` is truthy (does not check type is `Boolean`).

#### isFalse

test.isFalse(actual[, message]);

Checks that `actual` is falsey (does not check type is `Boolean`).

#### isNull

test.isNull(actual[, message]);

Checks `actual === null`

#### isNotNull

test.isNotNull(actual[, message]);

Checks `!(actual === null)`

#### isUndefined

test.isUndefined(actual[, message]);

Checks `actual === undefined`

#### isNotUndefined

test.isNotUndefined(actual[, message]);

Checks `!(actual === undefined)`

#### isNaN

test.isNaN(actual[, message]);

Checks `isNaN(actual)`

#### isNotNaN

test.isNotNaN(actual[, message]);

Checks `!(isNaN(actual))`

#### length

test.length(obj, expected_length[, message]);

Checks `obj.length === expected_length`

#### instanceOf

test.instanceOf(obj, klass[, message]);

Checks `obj instanceof klass`

#### notInstanceOf

test.notInstanceOf(obj, klass[, message]);

Checks `!(obj instanceof klass)`

#### include

test.include(haystack, needle[, message[, not]]);

`haystack` may be an `array`, `object` or `string`. Correspondingly, `needle` may be an `element` (simple elements only), `key` (method or property name) or `substring`.

`not` is a boolean (`true`/`false`) parameter which may be used to switch the sense (`include` becomes `notInclude`). Used internally by `notInclude`. **Don't use it in your tests - it just makes your code harder to grok.**

#### notInclude

test.notInclude(haystack, needle[,message]);

Internally, calls `include` with the `not` flag set to `true`.

#### _stringEqual

test._stringEqual: function (actual, expected[, message]);

EXPERIMENTAL way to compare two strings that results in a nicer display in the test runner, e.g. for multiline strings.

### Assertions without optional fail messages

test.throws(func, expected);

`expected` can be:

- `undefined`: accept any exception.
- `string`: pass if the string is a substring of the exception message.
- `regexp`: pass if the exception message passes the regexp.
- `function`: call the function as a predicate with the exception.

Note: Node's `assert.throws` also accepts a constructor to test whether the error is of the expected class.  But since JavaScript can't distinguish between constructors and plain functions and Node's `assert.throws` also accepts a predicate function, if the error fails the `instanceof` test with the constructor then the constructor is then treated as a predicate and called (!)

The upshot is, if you want to test whether an error is of a particular class, use a predicate function.

## Utility Methods

#### runId

test.runId();

Returns a unique string id for this test (for example 'ZmXxMPyoWGFy5wEiB').

#### exception

test.exception(exception);

**Should only be used with asynchronous tests**

Call this to fail the test with an exception that occurs inside asynchronous callbacks in tests. If you call this function, you should make sure that:

- the test doesn't call its callback (onComplete function).
- the test function doesn't directly raise an exception.

#### expect_fail

test.expect_fail();

May be used to change a fail to a qualified pass. The test is counted as a pass, but clicking on it reveals that the underlying test failed:

    - expected_fail — assert_equal - expected xxx - actual yyy - not

## How do I test call/method?

Remember, you are testing a package, so unless your `call`s and `method`s are both in this package, you shouldn't be doing this. Similarly, you are writing *unit tests*, so there should be no need to test that `call`/`method` operates correctly between client and server. You should be testing that your call is functionally correct and that your method is functionally correct.

### How do I test my call?

It is often unnecessary to test a simple call. Consider:

```javascript
Meteor.call('mymethod'); // Not really testable!
```

However, it is sensible to test the correct behavior of dependent code:

```javascript
Meteor.call('mymethod', value, (error, result) => {
  let answer;
  if (error) {
    // something nasty happened
  } else {
    if (_.isNumber(result)) {
      answer = result * 2;
    }
  }
});
```

In the above example, we only really need to test that the code in the `else` block is correct: remember, a unit test should not be verifying that the method is being called on the server (we have done that for you). However, the above snippet is not structured well for unit testing, because the functional code is buried within a conditional within a callback within a `Meteor.call`. If the code looked like this:

```javascript
MyPackageNamespace.someFunction = function(result) {
  if (_.isNumber(result)) {
    return result * 2;
  }
};

Meteor.call('mymethod', value, (error, result) => {
  let answer;
  if (error) {
    // something nasty happened
  } else {
    answer = MyPackageNamespace.someFunction(result);
  }
});
```

We get a unit of functionality (`MyPackageNamespace.someFunction`) which *can* be unit tested.

```javascript
Tinytest.add('mypackage - test someFunction', (test) => {
  // test body
  test.equal(MyPackageNamespace.someFunction(1), 2); // Double the number
  test.equal(MyPackageNamespace.someFunction(-10), -20); // Double the number
  test.isUndefined(MyPackageNamespace.someFunction('abc')); // Should not give a value
  test.isUndefined(MyPackageNamespace.someFunction({})); // Should not give a value
  }
});
```

### How do I test my method?

Follow the same principle as with testing `call`. Ensure that the `method`'s functionality is encapsulated in testable components. It's likely you will be getting data from "elsewhere" in a method (for example, a `collection` or a REST endpoint). For these use cases you should *fake* the expected result(s). Remember, unless you are testing the `mongo` package, you are not testing that a `Mongo.Collection` actually is connected to a physical MongoDB collection!

Don't use:

```javascript
let list = MyCollection.find().fetch();
let total = 0;
_.each(list, (element) => {
  total += list[element].age;
});
```

Use:

```javascript
let list = [
  {_id: 'abcd', name: 'Bob', age: 30},
  {_id: 'bcde', name: 'Carol', age: 31}
];
let total = 0;
_.each(list, () => {
  total += list[element].age;
});
```

This ensures predictable data for testing, which means if appropriate, it's easy to supply in-range, out-of-range and edge-case data.

### How do I (do what I shouldn't and) test call/method end-to-end

The normal use case for `Meteor.call` and `Meteor.methods` is that a client `call`s a server `method`. However, `call`s and `methods` may exist on client and/or server. This means there are four basic use cases. In addition, a server may `call` a `method` on another server. In this use case, the `call`ing server is treated as a client (but this is definitely outside the scope of a package test).

`call`s may be made with or without completion callbacks, but the "safest common denominator" is to use the callback form, which will work as expected on the client or the server. A callback implies an asynchronous test, so the most basic form for this test is:

```javascript
Tinytest.addAsync('test name', (test, onComplete) => {
  Meteor.call('methodName', (error, result) => {
    if (error) {
      // Throw a test exception
      test.exception(error);
    } else {
      // Check result is as expected
      test.equal(result, 1234);
      // Return through the onComplete callback
      onComplete();
    }
  });
});
```

## How do I test pub/sub?

TBD

## How do I test my UI?

TBD

## How do I test ...

TBD
