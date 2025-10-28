---
layout: post
title: "Cohesion Matters: Why Organizing Code by Feature Beats Organizing by Layer"
date: 2025-10-28 10:00:00 +0000
categories: software-engineering architecture design
---

What I often see when I've cloned a new repository is this:

```
src/
├── controllers/
├── services/
└── repositories/
```

Familiar, right? You expand the folders and find `UserController`, `OrderController`, `PaymentController` in one
directory, their corresponding services in another, and repositories in a third.

Everything looks clean and organized. The codebase follows the classic three-tier architecture. You can tell immediately
which files handle HTTP requests, which contain business logic, and which talk to the database.

I've learned to recognize this as a red flag, though, that jumps right into my face in many repositories. From the first
glance I suspect, that I can expect low cohesion.

## Cohesion: The Other Side of Separation of Concerns

We talk a lot about separation of concerns in software engineering. Keep your HTTP logic separate from your business
logic. Keep your business logic separate from your database access. That's all good advice that is usually found one
level deeper in the folders in the form of a UserController and an OrderController.

But separation of concern has a sibling principle that gets less attention: **cohesion**. While separation of concerns
is about keeping
different things apart, cohesion is about keeping related things together. It is the basis for modules, i.e. separation
of concern on the next level above the class level.

In software engineering, cohesion measures how closely related and focused the responsibilities of a module are. High
cohesion means the elements within a module belong together, they work toward a common purpose and change together for
the same reasons.

When cohesion is low, your modularization is also low. There's no separation of concerns at the
package level. Your orders code, payment code, and user management code are all tangled together across the same three
directories. You've separated technical layers, but you haven't separated business capabilities.

The controller/service/repository structure is so common that we rarely question it. It groups code by *what it is*
rather than *what it does*.

## Why Cohesion and Separation of Concerns Matter

Both principles serve the same underlying goal: **managing the complexity**.

Cohesion and separation of concern on a module level create seams in the project, that separate the project into
isolated compartments. Each compartment has a clear responsibility and well-defined boundaries. You can open
one compartment, understand what's inside, make changes, and close it again without needing to hold the entire system in
your head.

This is what reduces the cognitive load and makes large codebases more manageable.

Creating the compartment allows to hide things inside. For example the repository can be private instead of public,
making it impossible to reach it from outside the module. Or, if you want to extract a responsibility into a
microservice, with high cohesion this is a much easier task.

## A Different Way

Consider organizing the same codebase by feature:

```
ecommerce/
├── orders/
│   ├── OrderController.java
│   ├── OrderService.java
│   ├── OrderRepository.java
│   ├── Order.java
│   └── OrderCancellationService.java
├── products/
│   ├── ProductController.java
│   ├── ProductService.java
│   ├── ProductRepository.java
│   └── Product.java
├── users/
│   ├── ...
└── reviews/
    ├── ...
```

## "But What About Shared Code?"

The most common objection: "What if multiple features need the same utility or validation?"

It depends on what kind of sharing.

True infrastructure like logging, authentication, database connections belongs in shared utilities:

```
ecommerce/
├── orders/
├── users/
└── common/
    ├── logging/
    ├── auth/
    └── database/
```

These are genuinely cross-cutting concerns used everywhere. They're stable and provide technical capabilities rather
than business logic.

For shared business logic, create a focused module. If both orders and subscriptions need tax calculation, create a
`billing/` module with `TaxCalculator.java`. The `billing` module becomes its own feature with high cohesion around
billing concerns.

Both `orders/` and `subscriptions/` depend on it. That's fine, , it's an explicit, intentional dependency.

A word of caution though: resist extracting shared modules too early. It's okay to duplicate simple validation logic
initially. Only extract shared code when the logic is truly identical, stable, and unlikely to diverge.

Premature abstraction can hurt cohesion just as much as scattering code across layers and creating shared abstractions
creates additional dependencies that make changes more difficult. That is something I also learned recently: DRY must
not be applied blindly.

## Making the Transition

If you're maintaining a layer-first codebase, start small. Pick one self-contained feature like "reviews" or "
notifications" and refactor it into its own package. See if the concept works for your team.

As you work on features, consider consolidating them into feature-based packages. Implementing a new order feature? Try
putting everything in `orders/` rather than scattering it across layers.

It's okay to have a hybrid structure during transition. Some features organized by layer, others by feature. Over time,
as you touch code, you can consolidate it.
Don't let perfect be the enemy of better.

When I start a new feature or project I usually start with no packages at all and let it evolve. When I notice high
cohesion between two classes, for example a controller and its DTOs, that is the time when I create a package for them to
coexist.

## The Core Principle

Organizing by feature instead of by layer minimizes the distance between related code.

When code that changes together lives together, you spend less time navigating, build better mental models, reduce
accidental coupling, and make changes with confidence.

Code organization isn't about aesthetics. It's about making the codebase reveal its intent and making changes easy.

---

I've worked with teams that significantly improved their development speed by reorganizing from layers to features. Not
because the code itself became better, but because developers could find and understand it faster.

High cohesion isn't just a theoretical ideal. It's a practical tool that makes your daily work easier, your codebase
more maintainable, and your team more productive.

It can start with one simple shift: next time you create a class, ask yourself whether you're grouping by what it is, or
by what it does.

**Group by what it does.**
