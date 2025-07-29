Next Steps for fud2
===================

[fud2][] has reached a point where it is now the right "default choice" for the Calyx ecosystem, mostly replacing OG fud.
There are many [engineering improvements][issues] to be done, but this document is about proposed next steps for its high-level design.

[issues]: https://github.com/calyxir/calyx/issues?q=is%3Aissue%20state%3Aopen%20label%3A%22C%3A%20fud2%22
[fud2]: https://docs.calyxir.org/running-calyx/fud2/index.html
[tracker]: https://github.com/calyxir/calyx/issues/1878


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


Some Engineering Preliminaries
------------------------------

With that in mind, here are some engineering (not very deep) improvements we might want to make to "set the table" for interesting data model work:

* Manual plan specification. We should think about how to do this in a simple, obvious way that is not necessarily convenient for the user.
* Pre-baked plan reuse. To make these manual plans actually usable, let's provide a way to embed a few pre-specified plans inside the fud2 executable. Then, we'll build some kind of CLI option to select one of these pre-existing plans.


Desiderata for the Data Model
-----------------------------

Here are some big things I think we could do with the data model.

### Finish Off the Hypergraph, Without a Planner

There's a lot of latent functionality in fud-core already for handling ops with multiple inputs and multiple outputs.
Let's take this to its natural conclusion.

In particular, let's focus on a handful of fud2 use cases that are currently hampered by the single-input, single-output restriction:

* As described in [the tracker][tracker], the need to use `-s sim.data=foo.json` is an annoyance. We would like to have a way to provide `foo.futil` and `foo.json` as coequal "arguments" to a two-input plan.
* The recently-merged [YXI-based AXI flow][yxi] is an annoyingly multi-step process, requiring 5 separate `fud2` invocations and a temporary file to produce an executable. The root cause is that the complete plan is a dag, not a path. Let's make this one command.
* The [FIRRTL backend][firrtl] requires some larger-than-would-be-ideal ops to deal with the need to combine user-provided code with the primitives library. (My memory of this one is hazier than the other two.)

In the past, the obstacle to addressing these use cases has roughly been:
designing a perfectly flexible hypergraph-aware planner is kinda interesting and hard;
and the CLI for interacting with this planner is hard to think about (what does `--through` mean exactly?).
I suggest that we sidestep the planning problem and just make this stuff work using the inconvenient pre-baked plan idea.
Even if we don't have a planner and accompanying UI that will work for every single circumstance, some of these use cases are so common that it seems OK to hand-write a plan for them.
And this way, we can iron out any additional kinks in the data model and execution engine to support all this stuff.

[yxi]: https://docs.calyxir.org/running-calyx/fud/xilinx.html#wip-calyx-native--fud2-xilinx-workflows
[firrtl]: https://docs.calyxir.org/running-calyx/firrtl.html

### Parametric States

fud2 has a lot of states, and some of them feel quite obviously like workarounds for... *something*.
For example, fud2 currently has all of these states for Verilog code:

* `verilog`
* `verilog-noverify`
* `synth-verilog`
* `verilog-refmem`
* `verilog-refmem-noverify`

And for FIRRTL, we have:

* `firrtl`
* `firrtl-noverify`
* `firrtl-with-primitives`
* `firrtl-with-primitives-noverify`

Clearly, we're headed for a combinatorial explosion here.
Every new option for RTL code requires doubling the number of states!
But the problem goes beyond just state proliferation;
distinct states also require distinct ops.
If you want to write an op that can deal with any Verilog code, for example, you need to make 5 copies of that op so it can handle any of these not-quite-the-same Verilog states.

Roughly speaking, what we want is for states to be parametric.
For example, maybe the general Verilog state is written `verilog[verify?, synth?, refmem?]`.
Then, a concrete instance of that state could be `verilog[true, false, false]` or whatever.
Crucially, ops should be able to specify, for each parameter to a polymorphic state, whether it cares about that parameter or not.
For example, a general Verilog transformer could accept `verilog[?, ?, ?]` as input, indicating that it can handle any form of Verilog at all.
A more specific one might accept `verilog[?, true, ?]` to say that any synthesizable-mode Verilog will work.

Let's design the data model for these parametric states and ops, without worrying about the scripting interface or the planner.
*If* we can come up with something useful, maybe we'll have an interesting planning problem on our hands for later.
