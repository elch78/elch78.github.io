---
layout: post
title: "Rethinking Testing: Speed and Behavior matter more than nomenclature"
date: 2026-01-09 10:00:00 +0000
categories: testing architecture
---

Thankfully tests are not optional anymore nowadays. But how to test is still a big topic in my view and the terms used
are not really clear. Over the years, my approach to software testing has evolved quite a bit. Like many developers, I
started with
extensive fine-grained unit testing and mock-heavy test suites. Today, I've moved toward a more pragmatic testing
strategy that prioritizes behavior over implementation details.

## The Problem with Fine-Grained Unit Testing

Earlier in my career, I invested heavily in comprehensive unit testing. I tested controllers, services and repositories
in isolation. With Spring Boots horizontal slicing support and the easy injection of mock beans that was straight
forward.
Although this approach tightly couples the tests to the implementations and makes refactorings harder.

Since then I've learned that tests are not only to prevent regressions and move faster, but they also serve as
documentation for what the expected behavior of the system is (if the tests are well written). I've always treated test
code as first class citizen applying the same clean code principles to them as to all other code. After this insight
though I
moved away from testing each class to testing more E2E focusing on the behavior of the system and less on how it is
implemented.

So my standard approach is to test the happy paths of an API end to end. E.g. a Rest API together with the service layer
and a DB in a testcontainer. If there is code that needs more detailed testing because it contains more logic I test that
piece in isolation.

Of course this makes tests significantly slower than pure in memory tests.

There's an excellent [talk by Ted Young on testable architecture](https://www.youtube.com/watch?v=dT3jORSx6C4&t=6062s)
that
resonates with my experience. One key insight: the distinction between "unit test" and "integration test" matters less
than we think. What truly matters is **test speed**. He distinguishes tests by if they use IO or not and gives good advice
how to isolate the code from IO in order to make it more testable.

## Speed Is the Real Metric

The value of any test is directly related to how quickly it provides feedback. Ideally, feedback should be
instantaneous—like compiler errors in modern development environments. (Remember when compilation itself took noticeable
time? We've come a long way.)

Fast feedback accelerates development. Slow tests, regardless of their classification, create friction in the
development loop.

The challenge, of course, is that tests naturally become slower as the unit under test grows larger and we need to find
a balance between testing end to end with IO vs. small units and in memory. But I think the classic testing pyramid is still good advice
only the labels (Unit-, Integration-, E2E-Test) on the pyramid have less importance. It's more like the slower the tests the less we should have of them.

## The Testing Pyramid: Balance, Not Dogma

What we're really testing are **interfaces, contracts, and expectations**. Here's the crucial insight: a "unit" can be
anything that exposes behavior we want to verify. It might be:

- A single class with a few methods
- A module or package with a public API
- An entire service layer
- A REST API endpoint
- A complete microservice

I've found that traditional testing nomenclature often creates artificial boundaries. I try not to get hung up on
whether something is a "unit test" or an "integration test." Instead, I focus on what behavior I'm validating and how
quickly I can verify it.

The key is maintaining balance across different abstraction levels:

- **Many detailed tests at lower levels**: Testing smaller units with comprehensive coverage
- **Fewer tests at higher levels**: Validating integrated systems end-to-end

## Testing at Multiple Levels: A Practical Example

Take a REST API endpoint, for instance. I've found value in testing it at multiple levels simultaneously:

**Level 1: End-to-End Workflow Test**
Spin up a test container with a real database, make an HTTP request, and verify the user-facing outcome. This test
focuses on whether the user can successfully complete their task and receive the expected result. It's slower but
provides high confidence that the feature actually works from the user's perspective.

**Level 2: Controller-Focused Test**
Tests just the controller layer with mocked dependencies. This test runs in milliseconds and focuses on:

- Input validation (are invalid requests rejected correctly?)
- Error handling (do errors produce appropriate HTTP status codes and messages?)
- Output formatting (is the response structure correct?)

Another test could, as Ted explains in his talk, test a domain class in detail in memory, covering edge cases and
business rules exhaustively without any external dependencies. Both tests have value in my workflow. The end-to-end test
catches integration issues and verifies the system works as a whole focusing on the happy path. The controller test provides fast feedback during
development and lets me thoroughly exercise edge cases that would be tedious to set up end-to-end.

## A Real-World Example

Let me illustrate with a concrete example from a project involving legacy system workflows. The expected user behavior
was clear, but the implementation was opaque. We faced tight deadlines and significant detective work to understand the
existing codebase. There was no time for detailed tests anyway but the e2e test to prove that the feature works as expected
was non negotiable.

We initially pursued what seemed like the right approach—and it turned out to be completely wrong. We had to pivot and
reimplement a good portion of the code.

Here's where our testing strategy saved us: because we had adopted behavior-driven development (BDD) principles, our
tests validated behavior rather than implementation details. When we rewrote the implementation, **the tests didn't need
to change**. They continued to verify that the system met user expectations, regardless of how we achieved those
outcomes internally.

This experience reinforced a valuable lesson: tests should serve as living documentation of what the system should do,
not how it currently does it.

## Conclusion

Modern testing isn't about choosing between unit tests and integration tests—it's about building a balanced test suite
that:

1. Provides fast feedback
2. Tests behavior and contracts, not implementation
3. Enables confident refactoring

I've found that focusing on these principles rather than rigid testing categories helps me build more maintainable
systems with test suites that actually support development rather than slowing it down. Your mileage may vary, but this
approach has served me well.
