---
layout: post
title:  "Refactoring Legacy PHP"
date:   2016-12-24 17:45:00 -0000
categories: programming php
---

Like most PHP Developers, part of my career was built on the need to refactor legacy code to counter technical debt. Due to bad design in software, delivering value to the client became ever more difficult and stressful until bad design decisions were rectified. For developers working on badly designed projects, making repayments on technical debt, whilst also delivering value is a key skill - in this post I'd like to explain how it is possible to square-the-circle and both deliver value whilst paying down technical debt.

There are multiple bad practice approaches which can lead to a project being filled with technical debt; for example, source code with a complex and tangled control structure can be referred to as "Spaghetti Code". I remember a colleague once exclaiming "I have so much spaghetti, I could open an Italian restaurant!"

Martin Fowler used the following pseudo-graph (plotting delivered functionality vs time on two stereotypical programming project) to describe [the phenomenon of technical debt](http://martinfowler.com/bliki/DesignStaminaHypothesis.html):

![Design Stamina Graph](/images/2016-12-23-refactoring-legacy-php/designStaminaGraph.gif)

Short-term, the software project with no design is able to yield slightly more code in a time constrained environment, however once the "design payoff line" is passed - the project with well designed code is able to deliver functionality far faster.

As more functionality is delivered, the codebase with no design deteriorates, meaning it is harder to modify and deliver valuable functionality. To the contrary, the project with good design is able to maintain a linear delivery of valuable functionality, by maintaining great design through Agile practices. Fundamentally technical debt damages your ability to deliver value to your client.

The cost of never paying down this technical debt is clear; eventually the cost to deliver functionality will become so slow that it is easy for a well-designed competitive software product to overtake the badly-designed software in terms of features. In my experience, badly designed software can also lead to a more stressed engineering workforce, in turn leading higher staff churn (which in turn affects costs and productivity when delivering features). Additionally, due to the complexity in a given codebase, the ability to accurately estimate work will also disappear. In cases where development agencies charge on an feature-to-feature basis, the profit margin for delivering code will eventually deteriorate.

## Testing

Before refactoring any code, we have to ensure that we don't cause damage or waste time by introducing bugs. Not only can these cause bugs going into production, they can also cost time having to constantly play whack-a-mole with bugs. Being careful and guessing software will continue to function is simply not enough - we need a scientific way to test software works before deploying. Additionally it provides for a tangible, executable specification of what our software should do.

Manual testing is simply too prone to not being thorough, additionally it is time consuming and far too costly. In order to be effective, tests which can be automated should be automated.

Automated testing is often described with a pyramid similar to the one displayed below, with Unit Tests being the largest in quantity to the bottom with manual session based tests being the very fewest at the top of the pyramid:

![Ideal Automated Testing Pyramid](/images/2016-12-23-refactoring-legacy-php/idealautomatedtestingpyramid.png)

Unfortunately we can't reach this ideal easily - there is some really messy PHP out there. There are cases where it is hard to implement Unit Tests in PHP before refactoring starts, instead we can start automating things like GUI tests or API tests. Any app, regardless of how bad its codebase is, can have some automated testing baked in.

The PHPUnit framework has support for browser regression testing through Selenium WebDriver. By using PHPUnit with Selenium you are able to automatically run browser-level tests for your application, which can provide an indication as to what has or hasn't broken. PHPUnit have put together great documentation on [using PHPUnit with Selenium](https://phpunit.de/manual/4.8/en/selenium.html), it is even possible to capture a browser screenshot when a given test fails. Similarly, if you're refactoring an API backend, you can use PHPUnit together with a HTTP client like Guzzle to create Automated API Tests.

You may be desperate to go in and start tidying up the codebase as it stands, but adding these tests will ultimately allow you to reach a faster speed when doing so. As the refactoring code converges to a more Object-Oriented or functional design, you can start adding low-level Unit Tests which provide much faster feedback.

## Speed up the Release Train

You need to be able to release regularly and rapidly. If you can't do this, you have a major problem. The chances of releases encountering problems increases proportionally to the amount of change you deploy; less can go wrong with smaller releases. Having a regular release schedule is vital to countering technical debt - much of the effort I had to spend with new clients was breaking down the barriers to achieving a systematic release train.

You should start with the end goal of being able to release continuously, whenever there is value to be added by performing a release. Amazon are able to [deploy every 11.6 seconds](https://www.youtube.com/watch?v=dxk8b9rSKOo), though you almost certainly will start from a position where you aren't able to. I would recommend you start by picking a day of the week where you deploy (no, not Friday) and deploy every week from then on. No matter how small the change, you deploy.

Once your release train can cruise on a weekly release track, switch up the speed and start deploying more frequently; twice a week and then daily. In order to achieve this frequency in releases you will find that you need to automate more and more of your operational approach and converge towards DevOps. This is perfectly healthy.

Preventing damage is merely one benefit of frequent releases, Jez Humble wrote an excellent post on the ThoughtWorks blog on [the business case for CI](https://www.thoughtworks.com/insights/blog/case-continuous-delivery). Push-button releases will require further automation work, implementing a tool like Bamboo to achieve automated feedback on the production readiness of software will become vital. You will also need development environments which are similar to production as possible, meaning tests can be run in a manor which will avoid lethal edge-case situations. To this end, having a solid development stack that's fast to spin-up and converges to production will help.

Automation of repetitive tasks is an excellent practice, humans are the most fallible part of the development process. Preventing a developer from manually having to enter dangerous commands into an SSH terminal will help your deployment operational resilience.

## What to Refactor?

As you run through the code, it is vital you understand what to refactor - to avoid inserting Anti-Patterns which take the codebase in the wrong direction. Code Smells are often used to find bad practices which root themselves in deep design issues within the software. Issues may also reside outside the codebase itself, architectural issues will need to be resolved through the implementation of well-suited Architectural Patterns.

To start with it is often a great idea to get your code in shape so you can start adding Unit Tests. The process of making your code more testable can usually also help iron out some core issues. For example; restricting the use of global states within your application won't just help make it more testable, but will also make it more resilient and easier to change.

Ensuring your application converges to Object-Orientation is also a great step, extracting smaller methods from a massively bloated functions allows us to prevent code reuse. The SOLID principles were originally referred to by Robert C. Martin as "first five principles" of Object-Orientation, it is a great idea to ensure your code complies with them:

* **Single Responsibility Principle** - a class should only have a single responsibility
* **Open/Closed Principle** - a class should be open for extension but closed for modification
* **Liskov Substitution Principle** - objects should be replaceable by their subtypes without affecting the correctness of a program
* **Interface Segregation Principle** - many client-specific interfaces are better than one general purpose interface
* **Dependency Inversion Principle** - depend on abstractions, not concretions

In particular, approaches such as favouring inheritance over composition and programming to interfaces rather than implementations will serve you well. With the basics of Object-Orientation nailed down, you can start moving Anti-Pattern behaviour towards better suited Design Patterns. For example; replacing hard-coded notifications with the Observer Design Pattern or replacing one/many distinctions with the Composite Design Pattern.

Code Smells, such as complex logical statements should be replaced with better suited approaches, such as Polymorphism. There are many resources available to describe what Code Smells you should seek to mitigate, I have covered these extensively in Chapter 7 of my book [Mastering PHP Design Patterns](https://www.packtpub.com/application-development/mastering-php-design-patterns). I would also strongly recommend the books [Refactoring to Patterns](https://www.amazon.co.uk/gp/product/B001TKD4RQ/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&camp=1634&creative=6738&creativeASIN=B001TKD4RQ&linkCode=as2&tag=icya-21) and [Refactoring: Improving the Design of Existing Code](https://www.amazon.co.uk/gp/product/0201485672/ref=as_li_tl?ie=UTF8&camp=1634&creative=6738&creativeASIN=0201485672&linkCode=as2&tag=icya-21) (though not PHP specific).

There are far too many approaches to cover here, but I'd highly recommend you research more into Code Smells and how to refactor code towards well suited patterns. Additionally, a tool like [PHPMD](https://phpmd.org/) will help you find code smells.

## Deliver Value

Businesses are built on the need to deliver value to clients - one of the massive benefits of refactoring over total rewrites is the fact that you can continue to add value to your existing codebase whilst refactoring it.

When starting to work on an existing part of the codebase, it is perfectly acceptable to start by adding tests (to accelerate the development of software). With these in place, when encountering a new part of a codebase, refactoring will help you gain an understanding how the codebase operates, and will indeed allow you to take the simplest route to achieve your intended logic.

Mitigating Technical Debt through refactoring your codebase doesn't slow down development, to the contrary it speeds it up.

> Refactor mercilessly to keep the design simple as you go and to avoid needless clutter and complexity. Keep your code clean and concise so it is easier to understand, modify, and extend. Make sure everything is expressed once and only once. In the end it takes less time to produce a system that is well groomed.

-- "[Refactor Mercilessly](http://www.extremeprogramming.org/rules/refactor.html)" is a key principle of Extreme Programming.

## Recognise Institutional Problems

You may often find the problem with bad code rooted in organisational failures within your company. It is vital to ensure you that you not only achieve buy-in for refactoring, but that the process is not hindered by cargo-cult practices or internal political disputes. This may well mean you have to have a heart-to-heart conversation with your manager to ensure these barriers are removed.

Badly implemented Scrum is often known of railroading proper engineering practices, no matter which management methodology you subscribe to - the need to subscribe to proper engineering practices is vital. Without ensuring code is properly tested or that code is constantly refactored, you risk falling back into the same trap of technical debt. I would highly recommend reviewing the practices described in Extreme Programming to see what you can implement from there.
