---
layout: post
title: "Seed Your System With the Test DSL, Not With Dumps"
date: 2026-11-14 10:00:00 +0000
categories: testing software-engineering ai
---

I needed a save game with a large factory in it. My hobby project is a voxel factory tower defense game written in
Rust, and I wanted to see how the frame rate holds up once hundreds of buildings, belts and towers are running. Building
that factory by hand in the game would have taken an evening. Writing a dump file by hand was out of the question.

So I asked whether the acceptance test scenario could build it. Less than a hundred lines later it did: ten rows of six
production lines, each a drill, a smelter, a workbench and a tower joined by belts, all four kinds of tower in turn,
sixty repair stations filled with ingots. The tool lets the simulation run for thirty seconds, checks that every tower
received ammunition, and writes the save. It takes seven seconds. The file loaded in the game on the first try, and the
factory was running.

{% include figure.html
   src="/assets/images/scenario-dsl-test-map.jpg"
   full="/assets/images/scenario-dsl-test-map-full.jpg"
   alt="View from above onto a landscape of grey stone blocks. In the middle lies a large levelled square with sixty small production lines in ten rows of six. Each line is a dark belt with coloured buildings on it: a blue drill at the west end, a brown smelter, an orange workbench and a tower at the east end, with a small repair station beside it. Around the square the ground rises in grey steps, and patches of rust-coloured iron ore and green copper ore show on its surface."
   caption="The test map from above. Each line runs from west to east: drill, belt, smelter, belt, workbench, belt, tower, and a repair station beside it." %}

It is the second time this approach worked right away. In an earlier project I used the same scenario DSL to inject
demo data into running systems. There, an AI used the DSL to feed the system with data, step by step, the way a user
would.

## The scenario DSL in short

In an [earlier post]({% post_url 2026-02-05-clean-tests-with-builder-and-mother-object-patterns %}) I described
builders, object mothers and, at the end, a `Scenario` class: a facade that offers tests reusable steps in the language
of the domain. Builders and mothers keep the details of single objects out of the test. The scenario keeps the
mechanics of the system out of it.

The idea of the DSL is Dave Farley's. He structures acceptance tests in four layers: test cases written in the language
of the problem domain, a domain-specific language that turns them into steps, protocol drivers that talk to the system
the way a user or another system would, and the system under test itself. The scenario is the DSL layer of that model;
in the game, the commands it submits are the protocol.

In the game, a scenario step looks like this:

```rust
/// Builds a drill facing east on the cell, drilling into an iron ore block put below it.
pub fn given_drill_at(&mut self, cell: Position) -> Position {
    self.simulation_mut()
        .0
        .set_block(cell.below(), Block::Ore(Ore::Iron));
    self.given_built(Command::BuildDrill {
        position: cell,
        drills_into: Direction::Down,
        facing: Direction::East,
    });
    cell
}
```

The step does not write a drill into some internal list. It hands the player the drill and submits the same command a
click in the game submits. The simulation decides whether the drill can stand there, exactly as it would for a player.

## The map is a scenario too

The tool that builds the map uses only steps like that one:

```rust
#[test]
#[ignore = "builds the test map; run it on purpose"]
fn test_map() {
    let mut client = ClientScenario::given_generated_world_with_shipped_parameters(SEED);
    let base = client.base_location();
    client.given_level_ground(base, LEVELLED);
    let west = base.x - LINES_ACROSS * LINE_SPACING / 2;
    let mut towers = Vec::new();
    for row in 0..ROWS {
        for line in 0..LINES_ACROSS {
            let start = Position::new(
                west + line * LINE_SPACING,
                base.y + 4 + row * ROW_SPACING,
                base.z,
            );
            let kind = TowerKind::ALL[(row * LINES_ACROSS + line) as usize % TowerKind::ALL.len()];
            let tower = given_production_line(&mut client, start, kind);
            client.given_filled_repair_station_at(tower.moved(Direction::East, STATION_OFFSET));
            towers.push(tower);
        }
    }
    client.run_ticks(RUNNING);
    // ... check that every tower got ammunition, then save
}

fn given_production_line(client: &mut ClientScenario, start: Position, kind: TowerKind) -> Position {
    let mut cell = start;
    client.given_drill_at(cell);
    cell = belt_after(client, cell);
    client.given_smelter_at(cell);
    cell = belt_after(client, cell);
    client.given_workbench_at(cell);
    cell = belt_after(client, cell);
    client.given_tower_of_kind_at(kind, cell)
}
```

It is an ignored test, so the normal test run skips it, and it runs with one command when I want a fresh map. Most of
the steps existed already, because the acceptance tests needed them. A few were new: levelling ground, building a drill
or a belt at a given cell, filling a repair station, saving into a file. Each of them is a step of the domain, not a
trick for this one tool, so the tests can use them as well.

The map holds no enemies; I start a wave in the game when I want the towers to have something to do.

{% include figure.html
   src="/assets/images/scenario-dsl-test-map-attack.jpg"
   full="/assets/images/scenario-dsl-test-map-attack-full.jpg"
   alt="The same factory from above during an attack. North of the production lines, a loose crowd of small red enemies walks towards the factory. Between the lines in the middle stand groups of small dark figures, the units of enemy swarms that have already arrived. Next to them, an artillery shell explodes as a large glowing orange ball."
   caption="A wave attacks the test map. Enemies come in from the north, swarms already stand between the lines, and a shell explodes among them." %}

## Why not a dump

I learned this lesson about fifteen years ago, around 2011, when I had to test complex rule sets. Test data from
database dumps kept falling behind the rules. The data that held up was the data the system created itself, because it
always followed the rules as they were at that moment.

A dump is a snapshot of the state at one moment. The system goes on changing; the snapshot does not. On the day I built
the map, the shape of the save game changed five times: enemies got two new values, the terrain started to be saved
as the changes made to it, waves got new parameters, and the save started to list which parts of the now 1024 blocks
wide world are generated. A dumped map would have broken five
times, and each time someone would have had to repair or rebuild it. The scenario just runs again and produces a map
of the current shape.

The same holds for demo data in a business system. A database dump of a demo environment ages with every migration and
every new mandatory field. Data created through the system's own entry points follows the system, because it is made
the same way production data is made.

There is a second difference. A dump says what the state is, not how it came to be. The scenario says it in the
language of the domain: a drill on iron ore, a belt, a smelter. When the map looks wrong, I read the scenario, not a
464 KB file. And because every step goes through the rules of the system, the map cannot contain a state the game
would never produce, such as a drill on stone or a tower inside a block.

The tool also checks itself. After thirty seconds of simulation it asserts that every tower holds ammunition. When a
change breaks the production lines, building the map fails, instead of handing me a broken map that I would only notice
in the game.

## A DSL is a good interface for an AI

In this project an AI wrote the map tool, in the same language as the tests. In the earlier project an AI used the
scenario DSL to seed running systems with demo data, the way a user would click through the application. In both cases
the AI did not need to know the database schema, the save format or the internals of the services. It needed the
vocabulary of the domain, and the system's own rules caught every step that made no sense. I only told the AI how the
demo data should look like. It invented the necessary data, used the DSL to create the necessary objects and feed
them into the system - even through a three-step workflow with different user roles.

That fits what I wrote about [communication with AI]({% post_url 2026-06-21-your-ai-is-only-as-good-as-your-communication %}):
a clear interface helps a model for the same reasons it helps a colleague. A scenario DSL is such an interface. It is
small, it speaks the language of the domain, and it cannot be used to put the system into a state it would not reach
on its own.

## What makes it work

The steps of the scenario have to be steps of the domain just like in Gherkin. A step that only one test needs, or that checks something,
belongs into that test. Otherwise the scenario fills up with special cases, and a tool like the map builder finds
nothing it can reuse.

The steps have to go through the system's real entry points: commands, services, the API a user calls. A step that
writes into internal state directly is faster to write, but it creates data the system could never have produced, and
it does not follow the system when the system changes.
