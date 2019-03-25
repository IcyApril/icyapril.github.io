---
layout: post
title:  "A Pattern for Validating Design by Contract Assertions in Production (with Go and Sentry)"
date:   2019-03-25 19:40:00 +0000
categories: go programming
---

In basic terms, Design by Contract is a software correctness methodology that focuses on using conditions to assert what functions should do. Such contracts specify what the obligations and benefits are of a function, they define how the function must be called and what the function itself should return. There is much material available on the topic including in some original work by [Bertrand Meyer](http://se.inf.ethz.ch/~meyer/publications/computer/contract.pdf), who originally coined the phrase.

An example of such a contract in the Eiffel programming language may be found in Eiffel Software's [Building bug-free O-O software: An Introduction to Design by Contract](https://www.eiffel.com/values/design-by-contract/introduction/):

```eiffel
put (x: ELEMENT; key: STRING) is
		-- Insert x so that it will be retrievable through key.
	require
		count <= capacity
		not key.empty
	do
		... Some insertion algorithm ...
	ensure
		has (x)
		item (key) = x 
		count = old count + 1
	end
``` 

In the code above detailing the `put` routine, the `require` block asserts pre-conditions that must be true for the software in the `do` block to run. The `ensure` block contains post-conditions that are run to determine what the correct output of the routine should be. Another assertion that can be used is Invariants, though we won't utilise this in this blog post, the work here can be easily extended to work with such assertions also.

This post discusses how Design by Contract can be used to test assertions in production software environments, without impacting user experience.

## How to Handle Failure

Many modern implementations of Design by Contract (particularly in generalist languages), advocate running software with contract enforcement in development environments but removing such contracts for production software builds.

There are two key arguments that are often used to advocate against testing contracts in a production environment. Often performance concerns are mentioned, but with modern computing the performance overhead of testing contracts in most applications is minimal. This is especially true when considering the readily available scalable compute power and HTTP caching for web applications. Indeed, I've seen Design by Contract implementations that continue to compute whether conditions are passed or failed but merely suppress the exceptions or panics when in production.

Secondly, and more relevantly, the concern of failure is mentioned. Contract violations are designed to fail hard, exposing programming mistakes to end-users in an unorganised way. This can be especially concerning when the software itself may have well continued to function despite contractual failures.

Despite this, disabling contracts in a production environment come at the cost of losing a valuable source of data. Whilst in a development or staging environment, contract violations will come from either software being run through automated tests or manual testing, in production environments there is real user data to check if the application meets its specification. By running real user requests through contracts, we are able to achieve a continuous and wide-spread approach to testing the assertions in our software.  

Contracts do not replace automated tests or vice-vera. Contracts instead represent a form of internal self-tests that detect errors both when running tests but also in production code during a test-phase. They can catch mistakes that smoke tests alone cannot catch by using a wider range of inputs and use-cases.

The SPARK Ada language (often used on high integrity embedded systems) offers using such specifications to formally verify the correctness for a significant proportion of the software (and using traditional testing tools for the remaining coverage). Rigorous software design practices help ensure the appropriateness of such contracts. Unfortunately, such tooling is simply not available in other high-level languages and it is not currently feasible to use rigorous design practices for most software without high integrity requirements.

Fortunately, error tracking services and software provides us an opportunity for capturing such contract violations in a production environment without disturbing users. Aggregation and analysis of such contract violations can later allow for the creation of tests to catch such violations in development, and the rectification of defective software.

## Go by Contract

To demonstrate this, I've created a simple implementation for Design by Contract behaviour in a Go package called [Go by Contract](https://github.com/IcyApril/gobycontract) - this is a primitive binding that allows for validating pre-conditions and post-conditions. It additionally allows for suppression of panics and sending information on the failure of contracts to [Sentry](https://sentry.io/) an online error tracking service (they also ship an open-source version which can be used as a drop-in replacement). Note that this binding was created to demonstrate this pattern, so accordingly may need some additional functionality or clean-up to be production-ready.

Take the following simple application - it uses a simple function to convert seconds into seconds and minutes - note that there exists a precondition that the number of seconds be a positive value and a number of postconditions to check the returned values are positive and that the amount of seconds does not exceed 59:

```go
package main

import (
"github.com/IcyApril/gobycontract"
"fmt"
"strconv"
)

func main() {
	minutes, seconds := SecondsToSecondsAndMinutes(125)
	fmt.Println(strconv.Itoa(minutes) + " minutes " + strconv.Itoa(seconds) + " seconds")
}

func SecondsToSecondsAndMinutes(seconds int) (minutes int, remainingSeconds int) {
	gobycontract.Require(seconds >= 0, "Input seconds must be positive")

	minutes = seconds/60
	remainingSeconds = seconds % 60

	gobycontract.Ensure(minutes >= 0, "Output minutes must be positive")
	gobycontract.Ensure(remainingSeconds >= 0, "Output remaining seconds most be positive")
	gobycontract.Ensure(remainingSeconds < 60, "There can be no more than 59 remaining seconds")

	return
}
```

The output of the application is as expected, for an input of 125 seconds:

```text
2 minutes 5 seconds
```

Now note what happens when we set a negative value in place of the 125 second calculation:

```go
minutes, seconds := SecondsToSecondsAndMinutes(-5)
```

Upon running this software, as the first pre-condition fails, we accordingly receive a panic stating that the input amount of seconds should be a positive value:

```go
panic: Pre-Condition not met: Input seconds must be positive

goroutine 1 [running]:
github.com/IcyApril/gobycontract.Require(0xc000354000, 0x13f225c, 0x1e)
	/Users/junade/go/src/github.com/IcyApril/gobycontract/gobycontract.go:37 +0xf2
main.SecondsToSecondsAndMinutes(0xfffffffffffffffb, 0x2, 0xc0003540e0)
	/Users/junade/go/src/github.com/IcyApril/testbed/main.go:15 +0x47
main.main()
	/Users/junade/go/src/github.com/IcyApril/testbed/main.go:10 +0x3c

Process finished with exit code 2
```

However, let's look at what happens if we suppress the ability of the application to panic and additionally instruct the application to report failures to Sentry. 

```sh
$ export "GOBYCONTRACT_DONTPANIC=true"
$ export SENTRY_DSN="https://<key>:<secret>@sentry.io/<project>"
$ go run main.go 
``` 

This time, the application operates without panicking and provides a negative output:

```text
0 minutes -5 seconds
```

However, this time our application has fired off appropriate notifications to Sentry notifying us of the contract violations:

![Sentry Contract Violations](/images/2019-03-25-a-pattern-for-validating-design-by-contract-in-Production-with-go-and-sentry/sentry_contract_violations.png)

## Conclusion

That's all there is to this pattern. By being able to toggle a production mode on our Design by Contract implementation, we are able to switch out panicking in production with sending messages to an error tracking service. With this approach we are able to monitor production reliability issues without user interference but continue to gain vital information about the resiliency of our application.