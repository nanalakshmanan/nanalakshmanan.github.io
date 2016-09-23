---
layout: post
title: "In God We Trust..."
modified:
categories: articles
excerpt:
tags: ['General']
image:
  feature:
date: 2016-09-22T08:08:50-04:00
---
...everyone else must write unit tests.

In this changing world of services, [unit testing](https://en.wikipedia.org/wiki/Unit_testing) has become an often [debated topic](https://www.reddit.com/r/programming/comments/1zqjk8/why_most_unit_testing_is_waste/). The intent of this article is not to join the bandwagon but to enumerate certain good design principles that will help write good unit tests.

Before starting it is good to provide the context behind why this is being written now and why this matters. There are a lot of quality criteria that are applied to product code - unit tests, design guidelines, style guidelines, performance guidelines, localization guidelines, etc. However, more often than not, test code does not have any quality criteria associated with it. This leads to a number of problems:
* There are random failures which need to be evaluated in every test run
* Test suites carry forward state making it difficult to run many individually
* Tests are environment specific causing issues (*for instance some tests don't run in localized builds*)
* It is difficult to refactor since tests will fail and it is hard to figure out why (*though this may sound funny, there have been numerous cases this is the actual reason why people fear refactoring bad code*)

In order to avoid those problems, tests must adhere to certain overall quality criteria (*as someone in my team liked to call it - tests for tests*). The following are some of the principles that can be applied to test suites - which are a collection of related tests

**1. Isolated**
Every test suite should setup the environment it needs. This includes all mocking as well as any global environment settings, temporary state or other artifacts it may need. A good way to ensure if a suite conforms to this is to run the suite in isolation in a clean environment (*unlike as part of a full test run*) and check if it succeeds.

**2. Cleanup**  
 Every test suite should cleanup the environment and clear any temporary state/artifacts that it has created. This is to ensure that no other test suite takes an accidental dependency on the same (*i.e if another test suite is not isolated, it might still succeed due to the existing environment*). A good way to ensure if a suite conforms to this is to snapshot the environment before and after the test run and compare the same.

**3. Deterministic**
Test suites should be deterministic - reliable and repeatable. Sometimes tests randomly fail - mostly because they are badly written. For example, if a test waits for a period of time instead of waiting for an event, it might randomly fail depending on the load on a system. Such tests waste precious developer time when they have to determine why tests randomly fail. A quick way to ensure test suites are reliable is to run them a few times in a loop and in a few different environments

**4. Platform Completeness**
Test suites should account for the variety of platforms they can run on. For instance many test suites may be expected to run in different versions of Windows like Windows server and Nano Server or may be expected to run cross platform like on Windows and Linux. Tests need to ensure that they are written correctly to accommodate the same. If tests do not belong to a particular platform they have to be skipped. Tests also have to ensure that they use common API sets to ensure they work correctly on all supported platforms. Running tests on all supported platforms using a CI system will help identify suites that are not platform complete quickly.

**5. Maintainability**
This one cannot be emphasized enough. Given that tests never get to production, there is a tendency to not write maintainable test code. Having test suites that are modular and easy to maintain go a long way in enhancing developer productivity. When dealing with open source projects (*even non open source projects for that matter*), please remember that other developers will have to extend and contribute to these tests. 

**6. Coverage**
Any test suite should have an acceptable level of coverage. What is acceptable varies across various software projects. However, not having enough coverage can often lead to not enough testing. Having good coverage also ensures that refactoring can be done effectively since tests will find any regressions introduced by way of refactoring. 

These are some of the high level guidelines. If there are other important guidelines please feel free to discuss them in the comments. 



