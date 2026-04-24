# Chapter 11  : Writing Automated Tests
<img width="364" height="157" alt="Chapter 11 topics" src="https://github.com/user-attachments/assets/a26dee1e-8a6a-49b5-8c5f-1ac79327cb64" />


**Why do we write tests**
- Simple reason to ascertain whether or not an operation, a series of tasks, a function call or any e2e event operates as desired.

Tests in rust are basic functions that verify, whether or not the non-test / target code is functioning in an expected manner
Test functions typically perform these 3 actions:

- Set up any needed data or state.
- Run the code you want to test.
- Assert that the results are what you expect.


## How to write tests

Lets say you want to test a simple add function.
```bash
 cargo new adder --lib 
```
this creates the crate below

### Structuring Test Functions

src/lib.rs
```rust
pub fn add(left: u64, right: u64) -> u64 {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }
}
```
1. #[cfg(test)] :
   1.  excluded from normal builds (like cargo build), 
   2.  module (in this case tests) is only compiled when running tests
2. #[test] annotation: This attribute indicates this is a test function, so the test runner knows to treat this function as a test. 
3. The tests are run when you run `cargo test` in the terminal 
4. use super::* ->  this line in tests module marks it as a regular module that follows the usual visibility rules 

   
```bash

PS C:\Users\patra\Desktop\rust_daily_prapti\adder> cargo test 
   Compiling adder v0.1.0 (C:\Users\patra\Desktop\rust_daily_prapti\adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.72s
     Running unittests src\lib.rs (target\debug\deps\adder-804645d3c37b0252.exe)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```
*Explaining the output*

- This shows that the one test was compiled, and it ran successfully as 1 passed and 0 failed
- Doc-tests adder is for the results of any documentation tests. Rust can compile any code examples that appear in our API documentation, and keeps your docs and your code in sync, documentation is in Chapter 14

*Lets customize our tests to see what failure looks like*
```rust
    #[test]
    fn another() {
        panic!("Make this test fail");
    }
```
here, if you add this right below the it_works test inside `mod tests{}` 
- panic! is a macro that terminates execution, it causes the test to fail and prints an error message.
- The output difference is what it looks like when a test fails


### Checking Results with assert! 
assert! macro is to ensure that some condition in a test is true. 
We give the `assert!` macro an argument that returns a boolean value. If the value is `true`, then the test passes. If the value is `false`, the assert! macro internally calls panic! to cause the test to fail. 
- Using the assert! macro helps us check that our code is functioning in the way we intend.

**Lets use an example**
in src/lib.rs

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

// above is a struct Rectangle, and an implementation on it
// below are the tests which is explained below
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }
} 
```
We have created a test larger_can_hold_smaller to test the functionality of can_hold fxn to see whether or not, this function returns a correct value, for the 2 Rectangle structs with custom values, which should give true.

running cargo test will return:
```bash

running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```
since a 5,1 can fit in 7,2 based on the functions, here we used custom values to check whether or not the function runs properly.

There is also an easy way to check the opposite that it should be false just by using ! (not) so: assert!(!<condition>)

### Testing Equality with assert_eq! and assert_ne!
Not all functions simply return a true or false, sometimes we want to check whether 2 values are exactly the same or even slightly different, based on the function, and how they should operate, according to us, this is a common situation: so a common test that the standard library provides a pair of macros:
- `assert_eq!`and `assert_ne!` : to perform this test more conveniently.
- These macros compare two arguments for equality or inequality, respectively. 

They’ll also print the two values if the assertion fails, which makes it easier to see why the test failed; conversely, the assert! macro only indicates that it got a false value for the == expression, without printing the values that led to the false value.

```rust
pub fn add_two(a: u64) -> u64 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        let result = add_two(2);
        assert_eq!(result, 5);
    }
}
```
This should get false, and in the terminal we see clear:

```bash

running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----

thread 'tests::it_adds_two' panicked at src\lib.rs:12:9:
assertion `left == right` failed
  left: 4
 right: 5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

```
it clearly shows left: right: to show what the values were with an additional RUST_BACKTRACE = 1 suggestion to see the exact point of failure

**assert_ne!**
- The assert_ne! macro will pass if the two values we give it are not equal and will fail if they are equal. This macro is most useful for cases when we’re not sure what a value will be, but we know what the value definitely shouldn’t be.
  
ex: if there is a function which is random, but the value can never be 2, then we do an assert_ne!(fxn(), 2) to check that the fxn didnt return 2

*Note: Under the surface, the assert_eq! and assert_ne! macros use the operators == and !=, respectively.*


- When the assertions fail, these macros print their arguments using debug formatting, which means the values being compared must implement the `PartialEq` and `Debug` traits. All primitive types and most of the standard library types implement these traits.
-  For structs and enums that you define yourself, you’ll need to implement PartialEq to assert equality of those types. You’ll also need to implement Debug to print the values when the assertion fails. Because both traits are derivable traits in a simple: `#[derive(PartialEq, Debug)]` on top of a struct.

### Adding Custom Failure Messages
Generally lets say there are a lot of tests in your codebase, which are huge functions, so simply knowing whether or not the test failed is not enough, so we also have the ability to add custom messages, as optional arugments in all:  `assert!`, `assert_eq!`, and` assert_ne!`macros.  

**Example:**

Lets take a src/lib.rs, the function greeting will obv just return "Hello", so the assert! will definitely fail, so the format is
- assert!(condition, "Error message"); 
```rust
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}

#[cfg(test)]
mod tests{
    use super::*
    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(
            result.contains("Carol"),
            "\nGreeting did not contain name, value was `{result}` \n"
        );
    }
}
```
Below is a section of the output from running cargo test

```bash
Running unittests src\lib.rs (target\debug\deps\adder-804645d3c37b0252.exe)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----

thread 'tests::greeting_contains_name' panicked at src\lib.rs:11:9:

Greeting did not contain name, value was `Hello!`

note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

```
here you see the line : Greeting did not contain name, value was `Hello!`

Which shows that it clearly works and that even variable can be passed in those messages as part of the error message.


### Checking for panics with attribute should_panic
It’s important to check that our code handles error conditions as we expect, to check whether panics! macros run when we expect it to.

This test should_panic passes if the code inside the function panics; the test fails if the code inside the function doesn’t panic.
-  #[should_panic] : this attribute basically marks a test as a must panic test.   
- We place the #[should_panic] attribute after the #[test] attribute and before the test function it applies to. 
Lets take this example: src/lib.rs

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {value}.");
        }
        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
        // if the val is >100 or <1 new() will panic
    }
}
```
Part of the output on cargo test:

```bash
running 1 test
test tests::greater_than_100 - should panic ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

As you can see above, the fxn did panic! and as a result, the test passed

Now if we were to make a minor change
```rust
Guess::new(80);
```
then it wont panic and as a result the terminal output on running the test is:

```bash
running 1 test
test tests::greater_than_100 - should panic ... FAILED

failures:

---- tests::greater_than_100 stdout ----
note: test did not panic as expected at src\lib.rs:20:8

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```
- We don’t get a very helpful message in this case, but when we look at the test function, we see that it’s annotated with #[should_panic]. The failure we got means that the code in the test function did not cause a panic.



- Tests that use `should_panic` can be imprecise. A `should_panic` test would pass even if the test panics for a different reason from the one we were expecting. To make `should_panic` tests more precise, we can add an optional `expected` parameter to the `should_panic` attribute. The test harness will make sure that the failure message contains the provided text. 
- The expected parameter allows you to check the panic message: **it’s a substring match**
```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                "Guess value must be greater than or equal to 1, got {value}."
            );
        } else if value > 100 {
            panic!(
                "Guess value must be less than or equal to 100, got {value}."
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "blah blah")]
    fn greater_than_100() {
        Guess::new(200);
    }
}

```
This will fail since the panic message does not contain "blah blah"
```bash

running 1 test
test tests::greater_than_100 - should panic ... FAILED

failures:

---- tests::greater_than_100 stdout ----

thread 'tests::greater_than_100' panicked at src\lib.rs:12:13:
Guess value must be less than or equal to 100, got 200.
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
note: panic did not contain expected string
      panic message: "Guess value must be less than or equal to 100, got 200."
 expected substring: "blah blah"

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

```
- The failure message indicates that this test did indeed panic as we expected, but the panic message did not include the expected string.
- When part of the panic message has : "blah blah", then the test will pass.

So then, we can figure out where our bug is.

### Using Result<T, E> in Tests
The Result enum is a very useful and extensively used, we can modify our tests to return Ok(()) or Err() type. 

```rust
pub fn add(left: u64, right: u64) -> u64 {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() -> Result<(), String> {
        let result = add(2, 2);

        if result == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```
The output below clearly shows, that the test pass, so we can use a Result enum in place of the panic! or something else.


```bash
running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

```

- The it_works function now has the Result<(), String> return type. In the body of the function, rather than calling the assert_eq! macro, we return Ok(()) when the test passes and an Err with a String inside when the test fails.

- Writing tests so that they return a Result<T, E> enables you to use the question mark operator in the body of tests, which can be a convenient way to write tests that should fail if any operation within them returns an Err variant.

You can’t use the `#[should_panic]` annotation on tests that use `Result<T, E>`. To assert that an operation returns an Err variant, don’t use the question mark operator on the `Result<T, E>` value. Instead, use `assert!(value.is_err())`

Now that you know several ways to write tests.
Lets see what happens when we run our tests and explore the different options we can use with cargo test.



## Controlling How Tests Are Run

1. cargo run compiles your code and then runs the resultant binary. 
2. cargo test compiles your code in test mode and runs the resultant test binary. 

The default behavior of the binary produced by cargo test is to run all the tests in parallel and capture output generated during test runs, preventing the output from being displayed and making it easier to read the output related to the test results.

We can, however, specify command line options to change this default behavior.

**Separators --**

cargo test followed by the separator -- and then the ones that go to the test binary. 

Running cargo test --help displays the options you can use with cargo test, and running cargo test -- --help displays the options you can use after the separator. These options are also documented in the “Tests” section of The rustc Book.

### Running Tests in Parallel or Consecutively
When you run multiple tests, by default they run in parallel using threads, meaning they finish running more quickly and you get feedback sooner.

Because the tests are running at the same time, you must make sure your tests don’t depend on each other or on any shared state, including a shared environment, such as the current working directory or environment variables.

If you don’t want to run the tests in parallel or if you want more fine-grained control over the number of threads used, you can send the --test-threads flag and the number of threads you want to use to the test binary. Take a look at the following example:

```bash
cargo test -- --test-threads=1
```

We set the number of test threads to 1, telling the program not to use any parallelism.

Running the tests using one thread will take longer than running them in parallel, but the tests won’t interfere with each other if they share state.

### Showing Function Output

If a test passes, the test library captures anything printed to standard output. Like, if we call println! in a test and the test passes, we won’t see the println! output in the terminal, we’ll see only the line that indicates the test passed. If a test fails, we’ll see whatever was printed to standard output with the rest of the failure message.

Lets take a function that prints the value of its parameter and returns 10, as well as a test that passes and a test that fails.

src/lib.rs
```rust

fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {a}");
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(value, 10);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(value, 5);
    }
}

```

The output to this: 

```bash
running 2 tests
test tests::this_test_will_pass ... ok
test tests::this_test_will_fail ... FAILED

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8

thread 'tests::this_test_will_fail' panicked at src\lib.rs:20:9:
assertion `left == right` failed
  left: 10
 right: 5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

```
The tests which fails
The output from the test that failed, "I got the value 8", appears in the section of the test summary output, which also shows the cause of the test failure.

Now, if we want to see printed values for passing tests as well, we can tell Rust to also show the output of successful tests with --show-output:

cargo test -- --show-output

and then we get the output for both passes and failure, as below:
```bash
running 2 tests
test tests::this_test_will_pass ... ok
test tests::this_test_will_fail ... FAILED

successes:

---- tests::this_test_will_pass stdout ----
I got the value 4


successes:
    tests::this_test_will_pass

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8

thread 'tests::this_test_will_fail' panicked at src\lib.rs:19:9:
assertion `left == right` failed
  left: 10
 right: 5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```

Above we can see "I got the value 4" and success alongside it, since the test passed.


### Running a Subset of Tests by Name
Running a full test suite can sometimes take a long time, especially if you are working on only a certain small sections, and just want to check if those sections work.

To show, how to only run certain tests, lets take this ex:
```rust
pub fn add_two(a: u64) -> u64 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        let result = add_two(2);
        assert_eq!(result, 4);
    }

    #[test]
    fn add_three_and_two() {
        let result = add_two(3);
        assert_eq!(result, 5);
    }

    #[test]
    fn one_hundred() {
        let result = add_two(100);
        assert_eq!(result, 102);
    }
}
```
Now to run lets say just the one_hundred test, we can use the command: cargo test one_hundred and then the other tests will be filtered out, and only 1 test will run, as you can see below.

```bash

$ cargo test one_hundred     
   Compiling adder v0.1.0 (C:\Users\patra\Desktop\rust_daily_prapti\adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.78s
     Running unittests src\lib.rs (target\debug\deps\adder-804645d3c37b0252.exe)

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out; finished in 0.00s

```

We can’t specify the names of multiple tests in this way; only the first value given to cargo test will be used. But there is a way to run multiple specific tests.

**Filtering to Run Multiple Tests**

We can specify part of a test name, and any test whose name matches that value will be run. For example, because two of our tests’ names contain add, we can run those two by running cargo test add.
```bash
$ cargo test add

Finished `test` profile [unoptimized + debuginfo] target(s) in 0.01s
Running unittests src\lib.rs (target\debug\deps\adder-804645d3c37b0252.exe)

running 2 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s
```

This allows you to run 2 tests beginnin with add and filter out the one which wasn't so.


### Ignoring Tests Unless Specifically Requested

Sometimes you dont want to run specific tests, for any number of reasons, from them being time consuming or for future implementations, so to exclude them during most runs of cargo test, we have an attribute called `#[ignore]` to ignore them.

Lets take an example: src/lib.rs

```rust
pub fn add(a : u64, b : u64 ) -> u64 {
    a+b
}

#[cfg(test)]
mod tests {
    use std::{thread::sleep, time::Duration};

    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }

    #[test]
    #[ignore]
    fn expensive_test()->Result<(), String> {
        // code that takes long to run
        sleep(Duration::new(10,0)); // pauses execution for 10 secs
        Ok(())
    }
}
```
Now on running the code above with a simple cargo run it gives the below section, i.e. ignores the test
```bash
$ cargo test    
   Compiling adder v0.1.0 (C:\Users\patra\Desktop\rust_daily_prapti\adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.55s
     Running unittests src\lib.rs (target\debug\deps\adder-804645d3c37b0252.exe)

running 2 tests
test tests::expensive_test ... ignored
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out; finished in 0.00s

```

Now to run the ignored test, we use `cargo test -- --ignored`
when we run that, the ignored test(s) are run.
```bash
$ cargo test -- --ignored
Finished `test` profile [unoptimized + debuginfo] target(s) in 0.01s
Running unittests src\lib.rs (target\debug\deps\adder-804645d3c37b0252.exe)

running 1 test
test tests::expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 10.00s

Doc-tests adder

running 0 tests
```
By controlling which tests run, you can make sure your cargo test results will be returned quickly. When you’re at a point where it makes sense to check the results of the ignored tests and you have time to wait for the results or have finished implementing those sections, then you can run it at your own leisure.

## Test Organization

Testing is a complex discipline, and different people use different libraries and organization techniques. The Rust community thinks about tests in terms of two main categories: unit tests and integration tests. 

1. Unit tests:  Small and more focused, testing one module in isolation at a time, and can test private interfaces
2. Integration tests:  Entirely external to your library and use your code in the same way any other external code would, using only the public interface and potentially exercising multiple modules per test.

Writing both kinds of tests is important to ensure that the pieces of your rust projects are doing what you wanted them to do, both separately and together.

### Unit Tests
As said previously, purpose of unit tests is to test each unit of code in isolation from the rest of the code to quickly pinpoint where code is and isn’t working as expected.

The convention is to create a module named tests in each file to contain the test functions and to annotate the module with cfg(test).

**The tests Module and #[cfg(test)]**
The `#[cfg(test)]` annotation on the tests module tells Rust to compile and run the test code only when you run `cargo test`, not when you run `cargo build`. 
- This saves compile time when you’re only building the library and saves space in the resulting compiled artifact because the tests are not included.
- `cfg` stands for *configuration* and tells Rust that the following item should only be included given a certain configuration option. In this case, the configuration option is `test`.

**Private Function Tests**
There’s differe opinions in the testing community, about whether or not private functions should be tested directly.
But Rust does allow you to test private functions.

Example: src/lib.rs
```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```
Here, `internal_adder` is not marked as `pub`, 
But because the `tests` module is a child of the root module,
we can use `use super::*;` to bring it into scope and test it directly.

### Integration Tests
Integration tests are entirely external to your library. They use your library in the same way any other code would, which means they can only call functions that are part of your library’s public API. Their purpose is to test whether many parts of your library work together correctly.

**The tests Directory**
We create a `tests` directory at the top level of our project directory, next to `src`. Cargo knows to look for integration test files in this directory.

Example structure:
```text
adder
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

tests/integration_test.rs
```rust
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```
- We don’t need `#[cfg(test)]` in `tests/integration_test.rs` because Cargo treats the `tests` directory specially and compiles files in this directory only when we run `cargo test`.
- Each file in the `tests` directory is compiled as its own separate crate, which is useful for creating separate scopes to more closely imitate the way end users will be using your crate.

**Submodules in Integration Tests**
As you add more integration tests, you might want to share some setup code. If you create a file like `tests/common.rs`, Rust will treat it as a test crate and try to run tests in it.
To avoid this, we use the older naming convention for modules: create `tests/common/mod.rs` instead.
- Files in subdirectories of the `tests` directory don’t get compiled as separate crates or have sections in the test output.

**Integration Tests for Binary Crates**
If our project is a binary crate that only contains a `src/main.rs` and no `src/lib.rs`, we can’t create integration tests in the `tests` directory and bring functions defined in `src/main.rs` into scope with a `use` statement.
- This is one reason why Rust projects usually have a straightforward `src/main.rs` that calls logic that lives in `src/lib.rs`.

## Summary
Rust’s testing features provide a way to specify how code should function to ensure that it continues to work as you expect.

1. We learnt multiple macros and attributes like: assert! assert_eq!, assert_ne!, #[cfg(test)],  #[test], #[ignore], explained =, using Result enum with the tests.

2. Different ways of structuring tests, grouping and running only certain ones, and many more of rusts features.

Unit tests exercise different parts of a library separately and can test private implementation details. 

Integration tests check that many parts of the library work together correctly, and they use the library’s public API to test the code in the same way external code will use it. 
