---
layout: post
title:  "Higher Reliability APIs with Design by Contract (using Go and Sentry)"
date:   2019-04-15 14:13:00 +0000
categories: go programming
---

API reliability extends beyond uptime, it is also essential that APIs perform their functionality correctly and in particular when changes are being made to underlying data. Whilst we use testing methodologies when developing software to ensure software is reliable when exploring certain test cases, sometimes things fall through the cracks of the use-cases we devise for testing our software. 

API contracts commonly refer to the definition of an API endpoint, the schema of what the request and the response should look like. Whilst this definition works fine for entirely stateless APIs, where only `GET` requests are used and there are no changes made to underlying data, this is not sufficient when APIs make changes to data.

I wrote a previous blog post on the [Design by Contract](/go/programming/2019/03/25/a-pattern-for-validating-design-by-contract-in-Production-with-go-and-sentry.html) software development methodology, where pre-conditions and post-conditions are used to assert changes by a given piece of software. Some assertions are run on software prior to the function being run and then some assertions are run after the function has been called, this allows us to determine when software has been executed properly. Design by Contract is typically used in an Object Oriented environment by running pre-conditions, post-conditions (and sometimes invariant assertions) to assess the correctness of a function. Instead of using smoke tests, Design by Contract allows us to assert what should be correct for all valid use-cases of the software. 

In this blog post I'd like to discuss applying this methodology in a wider scope, to APIs instead of individual functions in software. This allows us to precisely define what an overall API endpoint does, what changes are made when a request is made. This allows us to prevent violations to data integrity and ensuring APIs correctly report what they're doing.

## Example Integrity Violation

Let's use a simple example here. Here we have a simple users API which has 3 endpoints:

* GET `/users` - lists all users
* GET `/users/{username}` - returns user by username, else a 404 if none exists
* POST `/users` - creates a user with a given username and password, in a JSON body.

For example, to list all users, we can do the following:

```sh
curl -X GET http://localhost:8000/users -H 'Content-Type: application/javascript' | jq '.'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   135    0   135    0     0  23092      0 --:--:-- --:--:-- --:--:-- 27000
{
  "5c8acf486969a": {
    "username": "IcyApril",
    "email": "junade@example.com"
  },
  "5c8acf8e108f1": {
    "username": "Alice",
    "email": "hello@example.com"
  }
}
```

Note that the API endpoint contains a fatal flaw, despite a record with the username `IcyApril` existing, the API still allows us to create an additional record with the same username such as:

```sh
$ curl -X POST \
>   http://localhost:8000/users \
>   -H 'Content-Type: application/json' \
>   -d '{
> "username": "IcyApril",
> "email": "icyapril@example.com"
> }' | \
> jq '.'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   359    0   300  100    59  46332   9111 --:--:-- --:--:-- --:--:-- 50000
{
  "username": "IcyApril",
  "email": "icyapril@example.com"
}
```

We can see that now, duplicate records exist for the username `IcyApril`:

```sh
$ curl -X GET \
>   http://localhost:8000/users \
>   -H 'Content-Type: application/javascript' | \
>   jq '.'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   206    0   206    0     0  31397      0 --:--:-- --:--:-- --:--:-- 34333
{
  "5c8acf486969a": {
    "username": "IcyApril",
    "email": "junade@example.com"
  },
  "5c8acf8e108f1": {
    "username": "Alice",
    "email": "hello@example.com"
  },
  "5cb1f6ac233a0": {
    "username": "IcyApril",
    "email": "icyapril@example.com"
  }
}
```

Whilst this is a simple example, it does demonstrate an example of an integrity problem. Duplicate values result in the creation of an endpoint. This issue can be further exacerbated if the API returns an error code but continues to enter the new record in. For example, even if there is a HTTP error code returned, the API still continues to insert the record into the database because of some kind of regression. 

This problem can cause further issues as a downstream API gateways or clients (be they human or software) could try to repeat the action (especially if the API returns a generic `500` server error status code instead of a `400` client error). This can cause a dramatic integrity integrity issues of malformed data being inserted into a database.

## Composing a Contract

This issue can be rectified by formulating a contract using pre-conditions and post-conditions. The pre-condition can check that a user with that username does not exist and the post-condition can be used to verify, that when the API returns back saying a user has been created, that the user account has been created.

In it's most simple form, a pre-condition and a post-condition to prevent the integrity problem detailed above can look as follows (using the Go programming language):

```go
func userPreCondition(target string, username string) {
	resp, _ := http.Get(target + "/users/" + username)
	gobycontract.Require(resp.StatusCode == 404, "User record already exists for POST /users")
}

func userPostCondition(target string, username string) {
	resp, _ := http.Get(target + "/users/" + username)
	gobycontract.Ensure(resp.StatusCode == 200, "User was not created after POST /users")
}
```

In short, the pre-condition fires off a check to see if the username does not already exist by checking to see if the individual user search endpoint returns a `404` error. The post-condition will verify that the user has been created by checking the endpoint returns a `200` when the username is queried again.

The functions above use the [Go by Contract library](https://github.com/IcyApril/gobycontract) that I discussed in my previous blog post on [Validating Design by Contract Assertions in Production](/go/programming/2019/03/25/a-pattern-for-validating-design-by-contract-in-Production-with-go-and-sentry.html). In essence, this library allows developers to be able to able to run disable panics in production environments and instead push error messages to Sentry - thereby allowing contracts to continue to be validated in production environments instead of being outright disabled. In a development or staging environment, traffic can be disrupted when a contract is violated but a reporting approach is likely better suitable in a production environment (for example, allowing a downstream API to handle violations in pre-conditions itself). 

We can implement such contract enforcement using a reverse proxy. For this, we'll use the native Reverse Proxy functionality in Go - this pattern can be extended to perform API gateway functionality or other tasks associated with a reverse proxy (authentication, etc). Similarly, this pattern can be included in other API gateway solutions. 

Note that the following example is somewhat crude for the purpose of brevity - for example, error handling should be improved and additionally it may be worth abstracting such contracts out to a separate package to improve code cleanliness as more endpoints are added. Additionally, depending on the load of the service, you may wish to implement a form of sampling to ensure that only a small percentage of requests have pre-conditions and post-condition assertions checked (if desired or necessary due to economics), additionally it may also be desirable to add additional error information (such as the request body) to the error message that is logged. With these warnings in mind, here's the example code in Go:

```go
package main

import (
	"log"
	"net/http"
	"net/http/httputil"
	"net/url"
	"github.com/IcyApril/gobycontract"
	"encoding/json"
	"io/ioutil"
	"bytes"
)

func main() {
	http.HandleFunc("/", handleRequest)
	log.Fatal(http.ListenAndServe(":9000", nil))
}

func userPreCondition(target string, username string) {
	resp, err := http.Get(target + "/users/" + username)
	if err != nil {
		log.Fatal(err)
	}
	
	gobycontract.Require(resp.StatusCode == 404, "User record already exists for POST /users")
}

func userPostCondition(target string, username string) {
	resp, err := http.Get(target + "/users/" + username)
	if err != nil {
		log.Fatal(err)
	}
	
	gobycontract.Ensure(resp.StatusCode == 200, "User was not created after POST /users")
}

type individualUser struct {
	Username string `json:"username"`
	Email    string `json:"email"`
}

func handleRequest(w http.ResponseWriter, r *http.Request) {
	// This URL can be changed to a given upstream API or set dynamically
	rawTarget := "http://localhost:8000"
	target, err := url.Parse(rawTarget)
	if err != nil {
		log.Fatal(err)
	}

	if r.Method == "POST" && r.RequestURI == "/users" {
		buf, _ := ioutil.ReadAll(r.Body)
		bufReader := ioutil.NopCloser(bytes.NewBuffer(buf))
		r.Body = ioutil.NopCloser(bytes.NewBuffer(buf))

		body, _ := ioutil.ReadAll(bufReader)
		var requestNewUser = new(individualUser)
		json.Unmarshal(body, &requestNewUser)

		userPreCondition(rawTarget, requestNewUser.Username)
		defer userPostCondition(rawTarget, requestNewUser.Username)

	}

	proxy := httputil.NewSingleHostReverseProxy(target)
	proxy.ServeHTTP(w, r)
}
```

In addition to the contracts, the above creates a HTTP server and a request handler (`handleRequest`). The request handler defines the upstream API in the `rawTarget` variable - this is validated and parsed to be used as the upstream party. There's some code in an `if` block that checks that the HTTP method is indeed a POST request and that it heads to the endpoint being considered. Note there is some usage of `NopCloser` to allow the `body` to not be wiped after being read for the contract assertions, the same code block contains some logic to run the contracts. The `userPostCondition` method is deferred to allow contract to run after the upstream request to the API service itself is run. The remaining code creates a simple reverse proxy that can be used to relay requests back upstream to the original API using the `NewSingleHostReverseProxy` method.

To demonstrate what happens when a contract assertion is violated, I'm going to configure the application not to panic, but to simply report contract violations to Sentry and then run the reverse proxy:

```sh
$ export "GOBYCONTRACT_DONTPANIC=true"
$ export SENTRY_DSN="https://<key>:<secret>@sentry.io/<project>"
$ go run main.go 
```

From here, I can simply attempt to create a duplicate entry for a user record, the API returns the JSON response without error: 

```sh
junade$ curl -X POST \
>   http://localhost:9000/users \
>   -H 'Content-Type: application/json' \
>    -d '{
>  "username": "IcyApril",
>  "email": "icyapril@example.com"
>  }' | \
>  jq '.'
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   129  100    67  100    62    135    125 --:--:-- --:--:-- --:--:--   135
{
  "username": "IcyApril",
  "email": "icyapril@example.com"
}
```

However when I look in my Sentry dashboard, I can a pre-condition violation has been logged with the message we described in the reverse proxy - this provides a lightweight example but in practice, I'd definitely recommend elaborating and adding example request information into the request that you log in Sentry:  

![Sentry API Contract Pre-Condition Violoation](/images/2019-04-15-higher-reliability-apis-with-design-by-contract/sentry_api_contract_precondition_violation.png)

## Summary

This blog post has described how it is possible to use contract-based specifications to improve reliability of APIs. By adding simple logic to a reverse proxy written in Go, we are able to check that a pre-conditions and post-conditions are met when running an API request. Such a Design by Contract approach can be set to report data to a logging service like Sentry to allow the logging-only behaviour of contract violations in a production environment. 