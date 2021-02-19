---
layout: page
title: Unit Testing with Jest
permalink: /tutorials/week5-unit-testing
parent: Tutorials
nav_order: 1
---
This tutorial covers the basics on unit testing with Jest. By the end of this tutorial, you will have an introduction to unit testing with jest, best practices, and some handy tricks and tips to use in your tests. 

Contents:
* [Unit Testing Basics](#unit-testing-basics)
* [Spies, Stubs, and Mocks](#spies-stubs-mocks)
* [Handling Promises in Tests](#handling-promises-in-tests)
* [Mocking Timers](#mocking-timers)

# Unit Testing Basics

For understanding the basics on unit testing, let us assume we have a file called calculator.ts present in the directory src/services/math/calculator.ts. Let us consider this file contains a class called Calculator, with a method for add() as below:
- ```ts
    // Contents of src/services/math/calculator.ts

    export class Calculator {

      public add(num1: number, num2: number): number {
        const result: number = num1 + num2;
        console.log('The result is: ', result);
        return result;
      }

    }
  ```

Let us write some tests for this code using jest.

## Suite

All spec/test files start with a suite. A suite is a collection of tests (or a logical grouping of tests). In jest, a suite is created by using the function `describe()`. The suite takes 2 arguments, 1st being the description of the suite, and the second being a callback function. Additionally, suites can be nested to form logical groups.
Suites can further be broken down into 3 components which we will explore in detail shortly:
1. Setup
2. Teardown
3. Test

Syntax:
- ```ts
    describe('Description of suite', () => {
      // The tests go here.
    });
  ```

Suites can be used make debugging easier in large test suites. Here is one recommended suite hierarchy:
- Top level describe should contain the file path after src.
- Second descrbe should contain the name of the Class/File being tested.
- Other describe blocks should contain the name of the function being tested.

Thus, the test file for the above example would look as follows:
- ```ts
    describe('services > math', () => {
      describe('Calculator', () => {

        describe('add()', () => {
          // Tests for add() go here.
        });

      });
    });

  ```

## Specs

A spec is an actual test (which executed some code and asserts some result). A test is created using the keyword `it()` or `test()`. Similar to `describe()`, `it()` takes 2 arguments, first being the description of the test, and the second being a callback. Generally, we want to describe what the code *should* do in the description of `it()` and assert the same within the test. Each test can be broken down into 3 parts (Assemble, Act, Assert) which makes up the AAA pattern. Optionally, there may be a clean-up/teardown step after the assert.

Syntax:
- ```ts
    it('should check a specific behaviour', () => {

    });
  ```

Let us write a simple test for our add() method to check 1 + 1 = 2.
Let us start by adding a spec to the suite we created previously.

- ```ts
    describe('services > math', () => {
      describe('Calculator', () => {

        describe('add()', () => {
          
          it('should return 2 when inputs are 1 and 1', () => {

            // Assemble

            // Act

            // Assert

          });

        });

      });
    });

  ```

### Assemble

In order to run a test, we need to first assemble it. This may include creating instances of classes/variables, setting up test data for inputs, setting up spies/stubs/mocks (which will be covered in subsequest sections), or setting up the expected output. In simple cases, one may not need to assemble the test. This phase is very similar to the setup phase.

In our example, let us create an instance of the Calculator class as part of assembling the test.

- ```ts
    import Calculator from './calculator';

    describe('services > math', () => {
      describe('Calculator', () => {

        describe('add()', () => {
          
          it('should return 2 when inputs are 1 and 1', () => {

            const calculator: Calculator = new Calculator();

            // Act

            // Assert

          });

        });

      });
    });

  ```

### Act 

In this step, we actually execute the function under test with required inputs and get the returned result (if any). 

In our example, we will invoke the add() method with inputs (1, 1) and get the result.

- ```ts
    import Calculator from './calculator';

    describe('services > math', () => {
      describe('Calculator', () => {

        describe('add()', () => {
          
          it('should return 2 when inputs are 1 and 1', () => {

            const calculator: Calculator = new Calculator();

            const result: number = calculator.add(1, 1);

            // Assert

          });

        });

      });
    });

  ```

### Assert

Assertion is a statement which validates the behaviour of our code by comparing the actual result against the expected results. There are many assertions provided by Jest and some useful assertions we will use throughout our tests are below:
- `expect(actual).toEqual(expected)` // Expects both entities to have the same value.
- `expect(actual).toBe(expected)` // Expects both entities to be the same.
- `expect(spy/stub/mock).toHaveBeenCalled()` // Expects a function being spied/stubbed/mocked to be invoked.
- `expect(spy/stub/mock).toHaveBeenCalledWith([arguments])` // Expects a function being spied/stubbed/mocked to be invoked with specified arguments.
- `expect(actual).toBeDefined()` // Expects the entity to be defined.
- `expect(actual).not.` // Negates the assertion. Can be chained with any matchers above

A full list of matchers can be found [here](https://jestjs.io/docs/en/expect)

In our example, we can use the .toEqual() matcher.

- ```ts
    import Calculator from './calculator';

    describe('services > math', () => {
      describe('Calculator', () => {

        describe('add()', () => {
          
          it('should return 2 when inputs are 1 and 1', () => {

            const calculator: Calculator = new Calculator();

            const result: number = calculator.add(1, 1);

            expect(result).toEqual(2);

          });

        });

      });
    });

  ```

## Setup and Teardown

Often in tests, we need some things to happen before a test actually runs and some things after it. This may include resetting/initializing values, setting up test data, setting up spies/stubs/mocks, or cleaning up variables after a test, and resetting spies/stubs/mocks. Sometimes these setps may need to be repeated for each test. This is where the setup and teardown can be especially useful.

Jest Provides 2 methods for setup and 2 methods for teardown:
- beforeAll(): Runs one time before all the tests in a suite.
- beforeEach(): Runs before every test in a suite.
- afterEach(): Runs after every test in a suite.
- afterAll(): Runs once after all tests in a suite.

In our example, notice we created an instance of calculator in our Assemble phase. We will probably have multiple tests for the calculator which will require this instance. In order to avoid repeating this in every step, let us move this to the setup phase and add a teardown to clear this after all tests.

*Note:* Use beforeEach()/afterEach() if the function/class stores state and we need a clean instance for each test. In our case, calculator does not store any state, and we can share the same instance across tests with out any side effects. Hence, we will use beforeAll()/afterAll().

- ```ts
    import Calculator from './calculator';

    describe('services > math', () => {
      describe('Calculator', () => {

        describe('add()', () => {

          let calculator: Calculator;

          beforeAll(() => {
            calculator = new Calculator();
          });

          afterAll(() => {
            calculator = undefined;
          });

          it('should return 2 when inputs are 1 and 1', () => {

            const result: number = calculator.add(1, 1);

            expect(result).toEqual(2);

          });

        });

      });
    });

  ```

Let us add another test to cover a different scenario, such as adding negative numbers.

- ```ts
    import Calculator from './calculator';

    describe('services > math', () => {
      describe('Calculator', () => {

        describe('add()', () => {

          let calculator: Calculator;

          beforeAll(() => {
            calculator = new Calculator();
          });

          afterAll(() => {
            calculator = undefined;
          });

          it('should return 2 when inputs are 1 and 1', () => {

            const result: number = calculator.add(1, 1);

            expect(result).toEqual(2);

          });

          it('should return -2 when inputs are -1 and -1', () => {

            const result: number = calculator.add(-1, -1);

            expect(result).toEqual(-2);

          });

        });

      });
    });

  ```


# Spies, Stubs, and Mocks

As a project grows, so do the interdependencies in the project. A function under test can have dependencies from various external entities, this may include other functions, network requests, database connections, of built-in connections. Spies, Stubs, and Mocks are ways of dealing with such external dependencies. You can read more on what you can do with spies/stubs/mocks [here](https://jestjs.io/docs/en/mock-function-api)

## Spy

A spy is a watcher on a function which tracks various proterties of the function being spied on. This can return information such as whether a function was invoked, how many times it was invoked, and what argument it was invoked with. A spy on a function is created using the syntax `const spy = jest.spyOn(object, 'methodName');` 

*Note:* The function being spied on actually executes.

In our example, we have an external dependency on console.log(). Let us add a spy and test for it.

- ```ts
    import Calculator from './calculator';

    describe('services > math', () => {
      describe('Calculator', () => {

        describe('add()', () => {

          let calculator: Calculator;

          beforeAll(() => {
            calculator = new Calculator();
          });

          afterAll(() => {
            calculator = undefined;
          });

          it('should invoke console.log() with the result 2 for inputs 1 and 1', () => {

            const logSpy = jest.spyOn(window.console, 'log');

            const result: number = calculator.add(1, 1);

            expect(logSpy).toHaveBeenCalledWith('The result is: ', result);

            logSpy.mockRestore();

          });

        });

      });
    });

  ```

## Mock

A mock is function which replaces an existing function. In our example, if we wanted to change the behavior of console.log() for our tests, we can do so using a mock. A mock implementation can be substituted for a spy or a jest.fn(). The syntax is as below:
- ```ts
    spy.mockImplementation(() => {
      // new function body goes here.
    });
  ```

*Note:* The function being mocked does not execute.
In our example, if we wanted to replace the behavior of console.log(), we can do so as shown:

- ```ts
    it('should invoke console.log() with the result 2 for inputs 1 and 1', () => {

      const logSpy = jest.spyOn(window.console, 'log');
      logSpy.mockImplementation(() => {
        // This will no longer print to console.
      });

      const result: number = calculator.add(1, 1);

      expect(logSpy).toHaveBeenCalledWith('The result is: ', result);

      logSpy.mockRestore();

    });
  ```

*Warning:* Watch out for circular dependencies in mock implementations.

## Stub

A stub is a special kind of mock which does not require an alternate implementation, but instead returns some value which we specify. When a stub gets invoked, it does not invoke the actually function, but returns the desired value instead. The syntax is as below: 
- ```ts
    spy.mockReturnValue(someValue);
  ```

To return a promise, we can use:
- ```ts
    spy.mockResolvedValue(someValue);
  ```
This can be especially handy when stubbing Axios requests.

Using a stub in our example simply prevent console.log() from being executed since it does not return a value anyway.
- ```ts
    it('should invoke console.log() with the result 2 for inputs 1 and 1', () => {

      const logSpy = jest.spyOn(window.console, 'log');
      logSpy.mockReturnValue();

      const result: number = calculator.add(1, 1);

      expect(logSpy).toHaveBeenCalledWith('The result is: ', result);

      logSpy.mockRestore();

    });
  ```

# Handling Promises in Tests


# Handling Timers