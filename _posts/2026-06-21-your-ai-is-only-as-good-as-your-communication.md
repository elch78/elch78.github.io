---
layout: post
title: "Your AI Is Only as Good as Your Communication — Just Like People"
date: 2026-06-21 10:00:00 +0000
categories: software-engineering ai
---

It started as a showerthought, but it hasn't left me since: working with AI is just like working with
people. If you communicate badly, the results are bad. If your requirements are vague, contradictory, or
buried in noise, you get vague, contradictory, noisy output. The model isn't the bottleneck. The
communication is.

We like to think of large language models as alien intelligences with their own rules. But the more I work
with them, the more I notice how human their failure modes are. The things that make a human colleague
struggle make an LLM struggle too. And the things that help a human do good work help the model just as
much.

## Bad Communication In, Bad Results Out

Hand a human developer a tangled mess of unstructured code and a pile of half-formed, conflicting
requirements, and you'll get a poor result. Not because they're incapable, but because they can't figure
out what you actually want. They'll guess, and they'll guess wrong.

LLMs behave exactly the same way. Feed them unclear instructions and messy context, and they produce
unclear, messy work. The quality of what comes out is bounded by the quality of what goes in. This is the
old "garbage in, garbage out" principle, except now it applies to intent and communication, not just data.

So the lever isn't a smarter model. The lever is clearer communication: code that is understandable and
well structured, and requirements that are coherent and easy to follow. The same things we've always known
make software easier for *people* to work with turn out to make it easier for AI to work with too.

## Cognitive Load Is a Problem for LLMs Too

Here's the part that I find most striking. Cognitive load isn't just a human limitation. It's an AI
limitation as well.

A person can only hold so much in their head at once. Give them a problem that fits in working memory, with
the relevant details close at hand, and they can focus on what matters. Overload them with irrelevant
detail and they lose the thread.

An LLM has the same constraint, just with a different name: the context window. The problem the model is
working on has to fit inside that window, the same way a problem has to fit inside a human's working memory.
If it doesn't fit, the model can't concentrate on the essentials. It can't reason about what it can't hold.

This is exactly why clean, boring code helps AI. I [wrote before]({% post_url 2026-01-13-boring-is-exciting %})
that good abstractions manage cognitive load so you can understand one piece without holding the entire
system in your head. That property isn't only a gift to your human teammates. Well-named abstractions and
cohesive modules let an AI grasp a piece of the system without loading the whole thing into its context.
Cognition-friendly code is cognition-friendly for *any* mind, silicon or carbon.

## Fill the Context Window With Signal, Not Noise

If the context window is the model's working memory, then what you put in it matters enormously. It should
be filled with clear, non-contradictory information that is relevant to the task at hand.

This is where I learned to get noticeably better results. When you plan a feature, there's a lot of back
and forth: options weighed, ideas proposed and rejected, trade-offs argued. That discussion is valuable
while you're planning. But once you move to implementation, dragging that entire debate along into the
context window hurts more than it helps.

Picture handing a human developer not the final, agreed-upon requirements, but the complete transcript of
every meeting where the team argued about what to build, including all the ideas that were ultimately
thrown out. They'd be confused. Which decision was final? Was that rejected approach actually rejected, or
is it still on the table? The contradictions in the record become contradictions in their understanding.

An LLM reacts the same way. If the context during implementation still contains the whole planning
discussion, with all its for-and-against and abandoned directions, it confuses the model, exactly as it
would confuse a person facing unclear requirements. The model can't tell the difference between a decision
and a discarded option if both are sitting in front of it with equal weight.

The fix is the same one good teams already use with each other: distill. Condense the messy planning
conversation into a focused plan. That plan can absolutely carry forward the important discussion points —
including the pitfalls to avoid and why — but boiled down to the essence rather than the full transcript.
Then give the implementation step *that*, not the raw debate. Clear the noise. Keep the signal.

## Small Steps and Fast Feedback Fit AI Perfectly

There's one more practice that turned out to be a surprisingly good fit for working with AI: continuous
delivery. Working in small steps, each one verified by tests before moving on, isn't just a discipline that
makes humans safer. It's almost tailor-made for collaborating with a model.

Think about why. A small step is a small problem, and a small problem fits comfortably into the context
window with room to spare for focus. You're not asking the model to hold an entire feature in its head at
once — just the next increment. And tests give you the constant control loop: after each step you find out,
immediately and objectively, whether the change did what it was supposed to. That's exactly the
[fast feedback]({% post_url 2026-01-09-Rethinking-Testing-Speed-and-Behavior-matter-more-than-nomenclature %})
I value when working on my own, except now it also keeps the AI honest.

This matters even more with a model than with a human, because an LLM will confidently produce something
plausible-looking whether or not it's correct. Small steps with tests catch the drift early, before it
compounds into a tangled mess that no longer fits anyone's head. Tight communication and a tight feedback
loop turn out to be the same idea applied at two different timescales.

What I find rewarding is that this is the way of working I'd already had great experiences with long before
AI — good tests simply make you fast — and it carries over to working with a model almost unchanged. The
nicest part is the shift in where my time goes: less of it spent typing and chasing bugs, more of it spent
actually thinking.

## The Takeaway

None of this is really new advice. We've always known that good software is understandable and well
structured, that requirements should be clear and consistent, and that you can't do focused work while
drowning in irrelevant detail. What's new is realizing how directly these same principles apply to AI.

And yet, in my daily work I still run into imprecise communication far too often: requirements that
contradict each other, code that hides its intent, plans that never quite get to the point, and the same
concept going by three different names depending on who's talking. That last one is exactly why
Domain-Driven Design pushes for a ubiquitous language and why a good domain-specific language pays off — a
shared, precise vocabulary so that humans and machines mean the same thing by the same word. It was already
costing us when only humans were on the receiving end. Now that AI is part of the team, the cost of unclear
communication just doubled — and so did the payoff for getting it right. The good news is that this is a
skill you can practice and get good at. It has always been worth investing in. Today it pays off twice.

If I had one wish for our craft, it would be that we get more serious about communication and treat it as a core
engineering skill rather than a soft extra. Requirements that don't contradict themselves. Names so that
everyone — human or machine — means the same thing. Code structured so it can be understood. We've tolerated
sloppy communication for a long time because humans are good at filling in the gaps. AI is far less
forgiving, and I've come to see that as a gift: it holds up a mirror and shows us, immediately, where our
communication was unclear all along.

So these days I try to treat the model a bit like a capable colleague who can only do good work when I
communicate well. Clean code to work with, coherent requirements, a problem that fits into the context
window, filled with clear and non-contradictory information. When I manage that, the results are good. When
I don't, I tend to get back roughly what I'd hand a confused human.

That symmetry is what stuck with me from the original showerthought, and it keeps proving true. With AI, it
really is a lot like with people.
