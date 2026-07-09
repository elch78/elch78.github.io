---
layout: post
title: "Cohesion Matters: Why Organizing Code by Feature Beats Organizing by Layer"
date: 2026-07-09 10:00:00 +0000
categories: software-engineering architecture design
---

Often, when I come to a new project and check out the repository, what meets me is this:

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

I've learned to recognize this as a red flag, though, because it is a clear sign of low cohesion. What this structure
shows in the first second is that the project lacks modularization.

## Cohesion: The Other Side of Separation of Concerns

We talk a lot about separation of concerns in software engineering. Keep your HTTP logic separate from your business
logic. Keep your business logic separate from your database access. That's all good advice that is usually found one
level deeper in the folders in the form of a User-/OrderController and User-/OrderService.

But separation of concerns has a sibling principle that gets less attention: **cohesion**. While separation of concerns
is about keeping different things apart, cohesion is about keeping related things together. David Farley captures both
in a single line he borrows from Kent Beck in *Modern Software Engineering*:

> Pull the things that are unrelated further apart, and put the things that are related closer together.

Cohesion is the *"put the related things closer together"* half. It is the basis for modules — separation of concerns
applied one level above the class. And it is a *fractal* idea: the same principle governs a method, a class, a package,
a deployable module, and a whole service. You just keep applying it at the next level up.

In software engineering, cohesion measures how closely related and focused the responsibilities of a module are. High
cohesion means the elements within a module belong together, they work toward a common purpose and change together for
the same reasons.

When cohesion is low, your modularization is also low. There's no separation of concerns at the
package level. Your orders code, payment code, and user management code are all tangled together across the same three
directories. You've separated technical layers, but you haven't separated business capabilities.

The controller/service/repository structure is so common that we rarely question it. It groups code by *what it is*
rather than *what it does*. This is a **technology-oriented split**, and it's telling that *Team Topologies* singles
out exactly this front-end/back-end/data-tier division as "historically the most common split — and usually a poor
one," because it cuts straight across business value and forces hand-offs. The split that holds up is the
**business-domain bounded context** (in Domain-Driven Design terms) — which is just a precise name for organizing
*by feature*.

## Why Cohesion and Separation of Concerns Matter

Both principles serve the same underlying goal: **managing complexity** — which David Farley names as one of the two
core competencies of software engineering (the other being optimizing for learning).

Cohesion and separation of concerns on a module level create seams in the project that separate it into isolated
compartments. Each compartment has a clear responsibility and well-defined boundaries. You can open one compartment,
understand what's inside, make changes, and close it again without needing to hold the entire system in your head.
This reduces the cognitive load and makes large codebases more manageable.

Also, compartments let you hide things inside. Just as a class protects its internal structure from the outside (and
vice versa), a module can hide its internal complexity — for example by making a repository package-private, so it is
inaccessible to code outside the package. With a mindful public API the module presents a simpler interface to the
outside, hiding the internal complexity. This is **information hiding and abstraction**: from the outside you neither
need nor want to know how the thing behind the seam works, only how to use it.

Clean modularization on the package level also makes refactoring easier. If you want to replace a functionality you don't
have to search for the part. Or, if you want to extract a functionality into a microservice, with high cohesion this 
is a much easier task.

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

## Making the Transition

If you're maintaining a layer-first codebase, start small. Pick one self-contained feature like "reviews" or "
notifications" and refactor it into its own package. See if the concept works for your team.

As you work on features, consider consolidating them into feature-based packages. Implementing a new order feature? Try
putting everything in `orders/` rather than scattering it across layers.

It's okay to have a hybrid structure during transition. Some features organized by layer, others by feature. Over time,
as you touch code, you can consolidate it.
Don't let perfect be the enemy of better.

When I start a new feature or project I usually start with no packages at all and let it evolve. When I notice high
cohesion between two classes, for example a controller and its DTOs, that is the time when I create a package for them
to
coexist.

## The other extreme: Microservices from day 1

This is another anti-pattern in my opinion, that I've encountered more than once, that a team decides to start with
microservices from the beginning.

Microservices are modules two levels up from the package level (artifact->deployment unit). As mentioned earlier, when I
start with a new codebase I start with no modules at all and let them grow as I learn what I need. A modulith - a
modular monolith - is the ideal starting point for fast exploration with minimal overhead. Microservices on the other
hand come with high costs associated with changing interfaces due to marshalling, deployment dependencies, multiple
codebases. It helps to be precise about *why*: a service boundary doesn't remove coupling — as Farley points out, you
can't eliminate coupling at all, because perfectly decoupled components can't talk to each other. It only relocates
that coupling across a network, where it becomes the very thing that most directly limits how easily you can change and
ship. A refactoring in a modulith is done automatically by the IDE in most cases; the same change across service
boundaries is a coordinated, multi-repo, versioned affair.

Finding the right seams to cut and the right interfaces between them is hard and needs time — *Team Topologies* calls
these seams **fracture planes**, and warns that cutting in the wrong place gives you a **distributed monolith**:
services that are physically separate but so interdependent that almost every change has to be coordinated across
several of them, the worst of both worlds. Make your life easy as long as you can and use a modulith. Only use
microservices if you have a good reason. Usually that is when the team has grown so big that it becomes impractical to
work on one codebase — which is really Conway's law talking: you split the system because you need to split the *teams*.
With good automated tests and Continuous Delivery a modulith goes a very long way. In fact I haven't been in the
situation that would mandate microservices often. Maybe once or twice at most.

## The Core Principle

Organizing by feature instead of by layer minimizes the distance between related code.

When code that changes together lives together, you spend less time navigating, build better mental models, reduce
accidental coupling, and make changes with confidence.

Code organization isn't about aesthetics. It's about making the codebase reveal its intent and making changes easy.

---

I've worked with teams that significantly improved their development speed by reorganizing from layers to features. Not
because the code itself became better, but because developers could find and understand it faster. That's not just
anecdote: the *Accelerate*/DORA research finds **loosely coupled architecture** to be one of the capabilities that
predict software delivery performance — the ability of teams to change their part of the system without depending on
others is a measurable driver of both throughput and stability.

High cohesion isn't just a theoretical ideal. It's a practical tool that makes your daily work easier, your codebase
more maintainable, and your team more productive.

It can start with one simple shift: next time you create a class, ask yourself whether you're grouping by what it is, or
by what it does.

**Group by what it does.**

---

*P.S. — a small perk you notice once you have it: the package structure doubles as a control surface. Debugging the
orders module? `logging.level.com.myproject.orders=DEBUG` catches the controller, service, and repository in one line.
With a layer-first layout that prefix doesn't exist — you'd tweak each class individually across three packages.*

---

*Further reading:*

- David Farley, *Modern Software Engineering: Doing What Works to Build Better Software Faster* (Addison-Wesley, 2021)
  — especially the chapters on modularity, cohesion, separation of concerns, and managing coupling.
- Matthew Skelton & Manuel Pais, *Team Topologies* (IT Revolution) — fracture planes, team-sized boundaries, and the
  distributed-monolith trap.
- Nicole Forsgren, Jez Humble & Gene Kim, *Accelerate* (IT Revolution, 2018) — the DORA research on loosely coupled
  architecture and delivery performance.
