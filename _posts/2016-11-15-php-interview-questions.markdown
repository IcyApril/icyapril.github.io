---
layout: post
title:  "PHP Software Engineer/Developer Interview Questions"
date:   2016-11-15 09:10:00 -0500
categories: programming php
---

Over the previous few years I've been involved in many development interviews; mainly as the interviewer but occasionally as the interviewee. Alongside this, through the wider PHP Community, I've heard some real disaster stories when it comes to poor interviews. From the whole interviewing team entirely lacking development knowledge to engineers being accidentally hired as a result of poor internal process - in some companies, I've even seen interviews where the whole process is a sales process due to them being unable to hire developers as a result of serious internal corporate issues or simply offering too low of a salary for the skill set they require.

For my own amusement, I came up with a list of technical questions that I believe a good senior PHP software engineer working on Enterprise software development should be able to answer. That said, don't try and ask all these in a single interview, that'd just be painful.

By all means this isn't the knowledge set that everyone on your team needs (your junior developer probably won't know much of this). Your requirements may also alter (you may have particular technologies or frameworks in your stack). Above all, this is just a set of questions; pairing with your candidate to write code and engage in discussion is important - so don't seek to just run down this entire list of long questions, instead I prefer to engage in programming tasks and discussion (which you can base around these questions). The ability to learn is far more important than the knowledge you already possess.

## Testing

I am a firm believer that software engineering is not merely a discipline of software construction; it is vital that engineers must be able to design and test their solutions. Without the knowledge required to effectively test code effective refactoring becomes a pipe dream, let alone adopting the "Refactor Mercilessly" principle as described in Extreme Programming. A key tenant of writing resilient code is self-testing code.

**Why is Unit Testing important?**

There are a few different reasons for this, your candidate will likely list off the fact they allow for refactoring to be performed easier which can help prevent the worst extremities of Technical Debt. In addition to this, when implementing Continuous Integration, self-testing code can help identify bugs rapidly. In order for code to be self-testing, you need a suite of automated tests that can check a large part of the code base for bugs. TDD can also give confidence as to when you've written enough code, which in turn can help enforce the KISS Principle. Amongst the other reasons, good unit testing can help document what a given function should do.

**Should Unit Tests cover private methods?**

This is a clear no for me (though some may disagree). Indeed testing private methods is not possible in PHPUnit without some leg work. Unit tests exist to test the public interface of a class, not what goes on behind the scenes. If private methods become so big that they require separate tests, this is an indication they should be moved to a separate class (can help ensure compliance of the Single Responsibility Principle).

**What process do you follow after identifying a bug?**

Bugs can come about through a failure of testing. To ensure developers firmly understand as to what the issue they need to debug is, Acceptance Tests should be written. Once the failing Acceptance Test is ascertained, developers should then write appropriate Unit Tests to ensure they know when the bug is solved (by running those Unit Tests). Having good Unit Tests ensures this problem is less likely to re-occur. When the code is fixed such that the unit tests now work, the Acceptance Tests can be re-run.

**What is your experience with BDD/UI/API testing?**

This is more an open-ended discussion regarding how such testing is done. Your candidate may talk about there experiments with Behat and Selenium or indeed could talk even more abstractly about how they achieve UI or API testing. Your candidate may argue that TDD is enough and it is often interesting to hear the various justifications for this.

**How do you debug a PHP Application?**

Your candidate should be able to walk you through using XDebug, PHPDBG or another debugging tool to debug PHP projects. The horrors of filling code with ````var_dump```` and ````print_r```` are inexcusable nowadays.

I once worked with a developer who indeed went to the extent of altering his code such that if a particular GET variable was passed into the production environment all variables would be dumped onto the screen (yes, including secure information).

## Object Oriented Programming

**What is polymorphism? Why are interfaces useful in them?**

Polymorphism is effectively the ability for multiple different objects to match a single given interface. For example, you can have an interface called ````Shape```` with a ````draw()```` method, both a ````Circle```` and ````Square```` class can fill this interface by containing a ````draw()```` method. Therefore any implementation doesn't need to worry about exactly what the shape is, it knows that it can run the ````draw()```` method on it, providing it implements the ````Shape```` interface.

Instead of messy switch statements for each object type, you can treat them all the same. Polymorphism can often be subdivided into Ad Hoc and Parametric polymorphism.

**How does Composition compare with Inheritance?**

Instead of extending a class via Inheritance, Composition means that one class actually contains an instance of another class that it wants to extend. An argument in favor of the "Favor 'object composition' over 'class inheritance'" principle (stated in the Design Patterns book by the Gang of Four) is that it provides more precise control as to the interface of the extended class. Some programming languages (notably Go, formerly GoLang) exclusively use Composition.

**What is Dependency Injection?**

In it's most basic form, it is injecting other objects into a class.

## Design Patterns

**What are advantages of the Publish-Subscribe Pattern?**

In the Publish-Subscribe Pattern, publishers are abstracted away from their subscribers. The subscribers need not even know of the existence of the publishers. This pattern therefore allows for better Scalability and Loose Coupling.

**How can the Prototype Pattern be implemented in PHP**

The Prototype Pattern is a Creational Design Pattern. In PHP it can be implemented through cloning objects:

```
$clone = clone $original;
```

PHP even contains a ````__clone()```` magic method which is run after an object is cloned, to customize the behavior further.

**What is the Composite Design Pattern?**

This is an easy Structural Pattern to get your head around. Effectively this pattern allows us to treat individual objects (a leaf) and a group of those objects (a composition) identically. Suppose we have a Song as the individual object and a Playlist as the group of objects - these would be treated identically. In this case, we can achieve this by both implementing the same Music interface with the same play, pause, stop, rewind, forward buttons.

**How can the Observer Design Pattern be implemented in PHP?**

A nice example of a Behavioral pattern; there are multiple ways of implementing this - but the PHP SPL contains two interfaces (SplObserver and SplSubject) which allow this to be implemented in a standardized way. An Observer is attached to the Subject, when the Subject wishes to notify the Observer it calls the Observer's ````update```` method.

## Code Construction

**What are some anti-patterns you've encountered?**

An Anti-Pattern is a common response to a given problem that is usually ineffective and risks being highly counter-productive. There are so many different Anti-Patterns - they are commonly re-invented but always poor solutions. Many are debatable, but it's still a fun exercise. I have a few favorites here.

Code:

* Not Invented Here Syndrome - believing that something is always better if built in-house, creating your own solution rather than using a third party solution.
* God Objects - An object that has too many methods or properties (usually in violation of the Single Responsibility Principle), usually out of a reluctance to create more classes.
* Singletons - instead of using  tightly-coupled, dependency-hiding, hard-to-test Singletons - try using Dependency Injection instead.
* Database-as-IPC - your database isn't a message queue for queueing jobs, there are a host of reasons why this is bad practice (row locks, constant cache invalidation, etc); use a proper MQ for that instead.
* Indefinitely running Cron Jobs - a Cron Job is a scheduled job that will run at a predetermined time, not something that operates a service for you. Damon your services properly.
* Blind Faith - Deploying bug-fixes or features without correctly testing your code.

Project Management:

* Design By Committee - many contributors working to solve a problem with poor judgement, inadequate information or no unifying vision.
* Bike Shedding - giving disproportionate weight to trivial decisions.
* Lack of Technical Standards - Developers writing poor code

**What is your version control workflow?**

There are many of these from Feature Branch to GitFlow and GitHub Flow. Really you want to ensure that someone has exposure to one such methodology and understands the concepts of branching/forking.

**What are the PSR-FIG Standards?**

PSR-FIG Standards are PHP Community coding standards that outline various standards for how code should be written consistently. Popular standards include ````PSR-1```` (basic coding style), ````PSR-2```` (advanced coding style) and ````PSR-4```` (succeeding ````PSR-0```` as the Auto-Loading Standard).

## Security

**What Cross-Site Request Forgery?**

CSRF is whereby a request is where the origin of a request isn't validated, such that it can be forged to emerge from a different host. For example; one site has submits a ````POST```` request to an endpoint on another site without authorization; this action could be anything from a transfer of money to publishing a blog post. As the victim site only checks the user is logged-in (and not the source of the ````POST```` request), this attack is possible. A crude mitigation is checking the referrer when processing the action, a better check would be to contain a hidden field on a form that contains a random string (stored in, say, a session variable) which is checked when request is processed.

**How do you prevent SQL Injection attacks?**

If using the MySQLi or PDO packages, then you can use Prepared Statements; alternatively an ORM like Doctrine or Eloquent can help manage this security burden for you. The key is not to run any user submitted variables as SQL statements.

**How do you securely store a password in a database?**

Simply: PBKDF2 or BCrypt. Merely hashing the password leaves them open to Rainbow Table attacks. Passwords can be salted (with a unique salt for each password) to mitigate this. PBKDF2 and BCrypt are better because they account for computational difficulty when hashing. BCrypt can be claimed to be better than PBKDF2 as it is more resistant to GPU Acceleration.

## HTTP/Networking

Most developers work on the HTTP layer, but it is largely abstracted away from them. In an increasing amount of environments, a basic understanding of HTTP is required. Whilst I wouldn't necessarily expect a developer to understand the ins-and-outs and BGP, you can expect them to differentiate between HTTP methods and use cURL.

**What is an RFC?**

Your candidate may either talk about IETF RFCs or PHP RFCs (or if they're savy, both). This question is about ascertaining whether the developer understands how a standardization process works. This can aid the developer in preventing NIH Syndrome and understanding how to look elsewhere for pre-existing solutions. E.g. Rather than trying to invent their own TOTP system, they can look to the IETF RFC on the matter.

PHP RFCs undergo a standard voting process to be approved, IETF RFCs roughly undergo a Internet Draft->RFC->Internet Standard process.

**What is the ````OPTIONS```` HTTP method typically used for?**

This HTTP method exposes what HTTP methods a particular endpoint can accept. This method also has uses in CORS for Preflight Requests. I mainly ask this question (or a similar one asking what the purpose of the ````PUT```` method is), to ascertain that the developer actually understands that there are methods other than ````POST```` and ````GET````.


**How would you compare and contrast Microservices against a Service Oriented Architecture?**

I tend to the follow the Martin Fowler school of thought, whereby Microservices are a subset of SOAs. The key differentiation you want your candidate to express is the idea that Microservices are specialized web services which perform one purpose alone and interconnect through a common format (e.g. HTTP REST).

**What is SNI?**

Server Name Indication is a technology to allow for VirtualHosting of TLS, effectively allowing multiple TLS certificates to be served from a single IP Address.

## Computer Science

A strong understanding of Computer Science never goes amiss, these questions can help assess developers who understand the basics.

**What is concurrency?**

Concurrency is effectively running multiple tasks concurrently (together at the same time), therefore a long running process can't block another process that will be complete faster. There are multiple ways of achieving this in PHP, including the Icicle library. Languages like Go also allow for easy concurrency.

**What is Big-O Notation?**

Big-O Notation is a mechanism to identify the difficulty of an algorithm, where ````O(n)```` is linear and ````O(2^n)```` is exponential. Getting your candidate to assess the difficulty of a given sorting algorithm can be useful, to assess how they practically apply this.

**What is the CAP Theorem?**

The CAP Theorem states that it is impossible for a distributed computer system to simultaneously guarantee:

* Consistency - Every request receives the latest information that has been written, or an error.
* Availability - Every request gets a request (which doesn't necessarily have to be the latest information).
* Partition Tolerance - When an arbitrary number of messages being dropped between nodes, the system continues to operate correctly.
