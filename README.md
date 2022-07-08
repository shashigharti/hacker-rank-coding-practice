# Solution to Coding Challenges in Hacker Rank

[![Coverage Status](https://coveralls.io/repos/github/hyp0th3rmi4/hacker-rank-coding-practice/badge.svg?branch=master)](https://coveralls.io/github/hyp0th3rmi4/hacker-rank-coding-practice?branch=master)

This repository contains the solutions to some of the coding challenges published in Hacker Rank. I do coding challenges in my spare time and I keep my solutions all in one place.

In order to speed up the writing and testing of the solution during development I have added a couple of wrapper classes simplify the execution of test cases via JUnit. The main method of the solution is a function that has the same signature (except for `public static` which became `protected`) of the method stub provided in the Hacker Rank's code editor to solve the challenge. Hence, it can be easily copied in.


## Running Tests

You need to have Java and Maven installed (JDK 11 and Maven 3.0.2 will do).

```
    mvn test
```

## Object Model

### Challenge Solution

This interface abstracts away the contract of a solution and it is specialised by two template types: `ChallengeInput` and `ChallengeResult`.

```java

public interface ChallengeSolution<I extends ChallengeInput, R extends ChallengeResult> {

    public boolean accept(String testCaseId);
    public R execute(I input);
}
```

There are two main methods in this interface: 

- `accept(String)` which can be used to skip some test-cases by a given implementation (for instance in order to avoid long-runs or marginal cases not covered by a solution); 
- `execute(I)` which unpacks the input executes the method that implements the solution and packs the output into a `ChallengeResult` implementation.

The `ChallengeInput` and `ChallengeResult` interfaces act only as type constraints and are used to provide a generic approach to manage the
variety of parameters required by the specific challenges from within the testing environment.

This interface is implementedd by `DefaultChallengeSolution` which provides a default implementation for `accept(String)` always returning `true` and `DefaultChallengeResult<V>` for the result class. This class implements a simple container for the one single parameter and overrides the `equals(Object)` method to perform equality check by checking that the wrapped value is equal or both of the two values are `null`. 

This leaves the implementation of a specific problem solution limited to define the `execute(Input input)` and the solution method itself. An example of the pattern is the following:

```java

public class MySolution extends DefaultChallengeSolution<Input, Integer> {


    public DefaultChallengeResult<Integer> execute(Input input) {

        int result = this.mySolutionMethod(input.getN(), input.getS());
        return new DefaultChallengeResult<>(result);
    }

    protected int mySolutionMethod(int n, String s) {... }
}
```

Here `Input` is defined as a simple pojo with two attribute (`n` and `s`) and the associated getters:

```java

public class Input implements ChallengeInput {

    protected int n;
    protected String s;
    
    public Input(int n, String s) {
       this.n = n;
       this.s = s;
    }
    
    public int getN() { return this.n; }
    public String getS() { return this.s; }
}
```


### Unit Test Automation

The unit test automation is primarily implemented via the class `SolutionTestBase`. The class is defined below:

```java

public abstract class SolutionTestBase<I extends ChallengeInput, R extends ChallengeResult> {


    protected InputStream getTestCase(String testCaseId) { ... }

    protected void runTestCase(String testCaseId) throws IOException { .... }

    protected abstract TestCase<I,R> readTestCase(InputStream testInput) throws IOException;
    
    protected abstract ChallengeSolution<I,R> getSolution();
}

```

The first two methods provide capabilities for retrieving the input stream associated to a test case file and execute the test with the given test case. The method `runTestCase(String)` does the heavy lifting of:

- accessing the input stream associated to the test case identifier
- retrieving an instance of `ChallengeSolution<I,R>` and verifying that the current instance can execute the specified test case
- invoking the method `readTestCase(InputStrem)` which produces an instance of `TestCase<I,R>` which contains both the input and the expected output
- invoking `ChallengeSolution.execute(I)` with the given input and verifying that the result matches the expected result.

The methods `readTestCase(InputStream)` and `getSolution()` are extension points for concrete test classes to provide the logic to read the content of the file in an instance of `TestCase<I,R>` and the instance of the solution under test.  A `TestCase` instance is container for both the test input data and the expected result as defined below:

```java

public class TestCase<I extends ChallengeInput, R extends ChallengeResult> {
    
    protected I input;
    protected R result;
    
    public TestCase(I input, R result) {
        this.input = input;
        this.result = result;
    }
    
    public I getInput() { return this.input; }
    public R getResult() { return this.result; }
}
```


To implement a specific test class for a given solution it is sufficient to implement the two extension points and define a `@Test` method for each of the test cases to be run, which simply invoke `runTestCase(String)`. An example follows below:

```java 

public class MySolutionText extends SolutionTestBase<Input, DefaultChallengeResult<Integer>> {

    @Test
    public void testCase0() throws IOException {
        this.runTestCase("my-solution/testCase0.test");
    }

    @Override
    protected TestCase<I, DefaultChallengeResult<Integer>> readTestCase(InputStream testInput) throws IOException {


            // logic to create a TestCase instance from the input stream
            // the example is based on the previous definition of input
            // which takes one integer and one string, and the result 
            // wrapping an integer (i.e. expected)

            return new TestCase<>(new Input(n,s), new DefaultChallengeResult<>(expected));
    }

    @Override 
    protected ChallengeSolution<Input, DefaultChallengeResponse<Integer>> getSolution() {
        return new MySolution();
    }
}
```

The string passed to the `runTestCase(String)` method is interpreted as a path relative to the root of the `src/test/resources` folder.test
