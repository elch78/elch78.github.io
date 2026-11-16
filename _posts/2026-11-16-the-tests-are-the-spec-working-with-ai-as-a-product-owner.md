---
layout: post
title: "The Tests Are the Spec: Working With AI as a Product Owner"
date: 2026-11-16 10:00:00 +0000
categories: testing software-engineering ai
---

Two weeks ago I started a hobby project: a voxel factory tower defense game. Drills extract ore, belts carry it to
smelters, workbenches turn ingots into ammunition, and towers shoot the enemies that come for the factory. It is written
in Rust, and an AI writes the code. Since the first commit on 3 November there have been more than 200 commits and more than
550 acceptance tests.

Rust is not my language; I have spent most of my career with Java. Two things delight me. The practices I rely on in
Java carried over without any change. And my own role shifted: working with the AI feels much more like being the
product owner than being the developer.

## A test from the game

Here is one test. Enemies damage buildings, and a repair station sends out a small flying unit that repairs them with
iron ingots. A broken smelter stops working; a repaired one has to work again:

```rust
#[test]
fn a_repaired_smelter_smelts_again() {
    // Given
    let mut scenario = Scenario::given_new_world();
    let station = scenario.build(a_repair_station());
    let smelter = scenario.build(a_smelter().near(station));
    scenario.given_inventory(Item::IronOre, 1);
    scenario.load(smelter, Item::IronOre, 1);
    scenario.break_down(smelter);
    scenario.given_material_for(smelter, station);

    // When
    scenario.run_ticks(scenario.time_to_repair(smelter, station) + scenario.duration_of(smelter));

    // Then
    assert_eq!(scenario.output_of(smelter), [(Item::IronIngot, 1)]);
}
```

You do not need to know Rust to read it. A world, a repair station, a smelter near it, loaded with one piece of iron
ore. The smelter breaks down, the station gets the ingots the repair costs. After the time the repair and one round of
smelting take, an iron ingot lies in the smelter.

## Builders, mothers and the scenario proved themselves again

I have written about [builders and object mothers]({% post_url 2026-02-05-clean-tests-with-builder-and-mother-object-patterns %})
and about the [scenario DSL]({% post_url 2026-11-14-seed-your-system-with-the-test-dsl-not-with-dumps %}) before. In
this project they do the same job they do in my Java projects.

The post on builders used a constructed example with products, because the tests I write at work belong to my clients
and I cannot share them. The game is my own. The test above is real code from a real project, and so is every other
example in this post.

`a_smelter()` is an object mother. It returns a builder with everything a smelter needs, so the test only says what
matters to it: `.near(station)`. Where exactly the smelter stands, which way it faces, which order it works on, the
test leaves open. The `Scenario` is the facade over the simulation. Its steps speak the language of the game: build,
load, break down, run. Each step submits the same commands a click in the game submits, so a test cannot create a state
the game would never reach.

The test also does not know how long a repair takes or how many ingots it costs. It asks the world:
`time_to_repair(smelter, station)`, `given_material_for(smelter, station)`. The tests never run on the game's shipped
configuration. A fixture draws every tuning value at random within a sensible range, and a test sets only the values
it is about. When I rebalance the game, no test breaks.

The scenario grows with the game. Each feature adds the steps its tests need, and the next feature finds them there.
When the way the scenario places buildings had to change, the change stayed inside the scenario, and the tests that
used it got shorter, not longer.

## The same rules in a different language

The AI works by a set of rules that I give it in every project: write the acceptance test first, use builders and
object mothers with random values, never test on the shipped configuration, cut the code by feature, commit small and
complete steps to trunk and keep it green. I wrote these rules with Java and Spring in mind. Their examples are Java.

In the game, they apply to Rust without a single change. Rust is quite unlike Java, but none of these rules depends on
the language. A test that comes first, a scenario that speaks the domain and a trunk that stays green work the same in
any of them.

Continuous delivery carried over as well. Every change goes to the main branch in small commits. Before each commit,
formatting, the linter and the full test suite run. A mistake costs one small commit, not an afternoon.

## Working with the AI feels like product ownership

What I bring to a session is almost always a requirement. Damage is an example. I decided that a damaged building is
not rebuilt but repaired, by mobile units, and that a repair costs resources and nothing else. That is a decision
about the game. How the unit finds the nearest damaged building, how it carries ingots, how the simulation stores its
flight: that is the implementation, and the AI does it.

In a session, the AI writes a plan, asks what is unclear, and I answer and correct. My corrections are about the game:
a factory is not a straight line, because filters have outputs to the side; an enemy that reaches the base should not
stand there forever. They are rarely about the code. At the start of the project I reviewed every acceptance test
before any code was written. By now I usually let the AI explain its plan in a few sentences, and it still writes the
test first.

The AI then works through the cycle the rules prescribe: write the test, see it fail, write the code, see it pass,
review the change against the architecture rules, commit. My job is to know what the game should do, and to notice
when a plan does not match it. That is the job of a product owner. With this setup, working with an AI comes down to
delivering requirements, and I am the bottleneck. The AI implements a requirement faster than I can think the next one
through.

## The tests are the specification

The requirement does not get lost on its way into the code. Every test file starts with a short statement of who wants
the feature, what it does and why. This is the beginning of the one for repairs:

```rust
//! Who: a player whose factory was damaged by enemies.
//! What: the player builds a repair station and loads it with iron ingots, by hand or from a belt or storage in
//!       front of it; it takes nothing else, and no more ingots than it holds at most. [...] Its unit flies to the
//!       nearest damaged building within the station's range and repairs it, at a cost of ingots per point of
//!       health. A repaired building works again: a machine smelts, a drill drills, a tower shoots, a storage hands
//!       over and a belt carries. Without ingots, or out of range, nothing is repaired.
```

Below it follow the tests, one per capability, each named after what it shows:
`a_repair_station_does_not_repair_a_building_out_of_its_range`, `a_repaired_smelter_smelts_again`,
`a_unit_flies_home_once_it_has_used_up_its_ingots`. A product owner can read the header and the test names and say
whether this is what the game should do. The test run checks whether the game does it. 73 test files of the project
start with such a header.

So there is no separate specification that someone has to keep in step with the code, and no Gherkin layer in between.
The test is the specification, written in the words of the domain and executed on every commit. It is the place where
my requirement and its proof meet. That is what lets me stay at the level of requirements: I do not need to read the
implementation to know whether the game does what I asked for. I read the test.

## What it takes

This way of working does not come for free. The rules have to be written down before an AI can follow them. The
scenario has to exist, and it only exists because every feature added the steps it needed. And the habit behind it,
test first, small steps, a green trunk, is older than any of this; I worked that way long before AI.

What changed is who profits from it. In
[Your AI Is Only as Good as Your Communication]({% post_url 2026-06-21-your-ai-is-only-as-good-as-your-communication %})
I wrote that what helps a human colleague helps a model too. Clear rules, a domain language and tests that say what
the system should do were always worth having. With an AI writing the code, they are what lets me spend my time on what
the game should do and leave the how to the model.

Continuous delivery already made me much faster than a workflow with feature branches and pull requests, where every
change waits for a review before it reaches the main branch. DORA, the research programme behind the State of DevOps
reports, measures this as lead time for changes: the time from a commit to production. Small commits to a green trunk
keep it short. With an AI, the time before the commit has shrunk as well. From the idea to a tested, committed change
now takes minutes.
