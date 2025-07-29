Next Steps for fud2
===================

[fud2][] has reached a point where it is now the right "default choice" for the Calyx ecosystem, mostly replacing OG fud.
There are many [engineering improvements][fud2 issues] to be done, but this document is about proposed next steps for its high-level design.

[fud2 issues]: https://github.com/calyxir/calyx/issues?q=is%3Aissue%20state%3Aopen%20label%3A%22C%3A%20fud2%22
[fud2]: https://docs.calyxir.org/running-calyx/fud2/index.html


Design Overview
---------------

In my view, fud-core consists of these high-level components:

* *Data model:* The data structures for specifying driver functionality, i.e., states and ops.
* *Scripting interface:* How you produce the above functionality for your domain of choice.
* *Planner:* Given some evidence about what the user wants to do, the planner concocts a series of steps using the available states and ops.
* *Execution engine:* The thing that actually carries out plans by generating Ninja and invoking it.

fud2 is basically a collection of scripts that use fud-core's scripting interface to produce functionality expressed in fud-core's data model.
Everything else is generically provided by fud-core.


Focusing on the Data Model
--------------------------

All these pieces are fairly interlinked, so sometimes it can be hard to think about how to improve any one of them---new ideas can seem to require new ideas across multiple components.
For example, making the scripting interface better might require rethinking the data model, which necessarily requires changes to the planner.

To frame more reasonable goals, I think we should try to focus on projects that mostly focus on one component and, as much as possible, avoid solving hard problems in the other components at the same time.
In my opinion, the most important thing we can focus on is the core data model.
That's the central piece of the design that affects what all the rest of the components are capable of.
To have the most impact, then, it makes sense to focus on that first, see what we can accomplish, and *then* worry about how to adapt the other components.

However, that strategy implies that we need a way to experiment with the data model in semi-isolation.
I suggest this philosophy:

* Don't do anything interesting to the scripting interface; just provide really clunky/basic ways to construct the data model.
* Let's build a "no-op" planner: i.e., an *inconvenient* way for users to manually specify an entire plan. This seems nice for testing and other situations where `--through` doesn't cut it anyway. Bonus points if the system can ship with a few built-in, pre-baked plans that you can select between with a CLI option or something.
* Changes to the execution engine may be inevitable.

Using that philosophy, we can more freely make progress on the core data model.
We can return to interesting planners and scripting interfaces in a future phase.
