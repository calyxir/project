YXI: The Calyx Interface Compiler
=================================

What is YXI?
------------

YXI is Calyx's in-progress infrastructure for declaring and implementing interfaces between coarse-grained hardware units.
At an extremely high level, the idea is very roughly analogous to [an `.mli` module interface file for ML][mli]:
an YXI file declares the memory-level interface for interacting with a Calyx design.

While all kinds of hardware interfaces are in scope,
the most urgent reason why we want to develop the YXI system is for *host/accelerator interfaces*.
The root problem is that running Calyx programs only in RTL simulation is unsatisfying; we really really want to run them on real FPGAs (and, eventually, ASICs).
Execution on FPGAs requires wrapping the Calyx code in some kind of memory-bus-like interface such as [AXI][] that lets the host send data in, read data out, and manage the execution lifecycle of the accelerator.
(Such an interface is also potentially useful for interactions between hardware units, but that's not our focus right now.)

The Calyx compiler has an existing (pre-YXI) way to add AXI wrappers compatible with [Xilinx's XRT host stack][xrt].
However, it works by generating Verilog ASTs from a Rust backend and has proven hard to maintain and improve.
The current focus of the YXI project is to replace this legacy AXI wrapper generator with a new, more hackable alternative.

Beyond host/accelerator interfaces, we also want YXI to be useful for other purposes:
for example, for driving testbenches and for supporting other FPGA and ASIC toolchains.


What We Have Now
----------------

The new YXI system has these components:

* A really simple YXI file format, which is just a JSON document describing the externally exposed memories for a Calyx program.
* [A little tool, called `yxi`,][yxi-tool] that can extract an YXI spec from a Calyx program with `ref` memories in its main component.
* A [separate thing][yxi-xml] that can generate Xilinx's `kernel.xml` file, which is the spiritual equivalent of YXI for the XRT ecosystem (i.e., it describes the memory-level interface to a hardware design).
* **This is the big, complicated part:** AXI interface generators. These work by taking an YXI spec and generating *Calyx code* that implements AXI logic for the specified interface. There are actually two of these (demonstrating YXI's role as an abstraction over the interface implementation):
    * A simple "bulk-synchronous" version that works by loading all the input data into on-chip memories, running the wrapped Calyx program, and then receiving all the output data.
    * A "dynamic" version that just forwards every memory request from the Calyx program directly to the host via AXI.
* fud2 components to orchestrate all of the above.
* [Cocotb][]-based testbenches for AXI-wrapped Calyx programs.

[mli]: https://ocaml.org/docs/modules#interfaces-and-implementations
[axi]: https://en.wikipedia.org/wiki/Advanced_eXtensible_Interface
[xrt]: https://github.com/Xilinx/XRT
[yxi-tool]: https://github.com/calyxir/calyx/tree/main/tools/yxi
[yxi-xml]: https://github.com/calyxir/calyx/blob/main/yxi/xml/xml_generator.py
[cocotb]: https://www.cocotb.org


Toward a Viable End-to-End System
---------------------------------

As a first order of business, let's aim for something that works end-to-end as a complete replacement for the old Verilog generator for AXI interfaces.
The main things we need are:

* Testing and CI improvements.
    * We are pretty close to having Cocotb-based end-to-end correctness tests for AXI-wrapped Calyx code. Let's set this up in Runt and test a range of small Calyx programs this way, in CI. [Nathaniel's Zulip message][nathaniel-zulip] has pointers about how to do this.
    * Let's extend the Cocotb testbench to support Xilinx's `s_axi_control` management interface. This will give us a way to test the whole system with all open-source tools. Let's enable this in CI too.
* Debug the current AXI generators. One or both of the existing AXI wrapper generators have some correctness issues; armed with our testing infrastructure above, let's find and fix those bugs.
* Validate the new AXI interfaces on our real Xilinx FPGAs. Once things are all working using the Cocotb testbench, let's run it (a) in Xilinx's cosimulation environment and (b) on real hardware. It's unlikely we can get anything CI-like working with Xilinx tools, but just doing this manually will be satisfying.
* Finally, we can deprecate or remove the old Verilog generator and officially recommend YXI as the "one true way" to run Calyx code on real hardware.

[nathaniel-zulip]: https://calyx.zulipchat.com/#narrow/channel/445339-calyx-yxi/topic/AXI.20Controller/near/531247840


More Researchy Directions
-------------------------

Once we have something minimally viable, there are a few more legitimately researchy next steps to try.
These are in no particular order; we can pick any of these to address next.

### Calyx Language Improvements

The Calyx-implemented AXI interfaces stretch the Calyx programming language in new ways that do not arise in other frontends.
The core reason is the need for cycle-level timing guarantees in the presence of dynamic latencies.
(Calyx has historically focused on *either* cycle-level timing *or* dynamic latencies, but combining them has not been a focus.)
A notable example that (I think?) is implemented and working is [the `@fast` attribute][fast attr] for transitioning between dynamic and static control.

The goal in this project would be to extend the Calyx IL and compiler to support stuff in the AXI interfaces that is currently inconvenient to express.
Basically, the idea is to make notes about things that are not ideal in the current language, design language extensions, discuss them with the Calyx community, and implement them in the compiler.

[fast attr]: https://github.com/calyxir/calyx/issues/1828

### Fancier Host/Accelerator Interfaces

The "big idea" with YXI is that it is an abstraction layer over hardware interfaces.
It generically describes the data that needs to be exchanged;
then, a concrete interface *mechanism* decides how to exchange that data.
We currently have two interface implementations:
a "fully eager" one that exchanges all data in bulk,
and a "fully lazy" one that only fetches/sends data on demand.
Most real programs probably want something in the middle.
So this project would focus on implementing "interesting" AXI exchange strategies that strike a better balance between the two.

A simple place to start would be with a cache: i.e., a small on-chip SRAM that stores *N* recently-accessed values.
But there are surely more creative ways to implement this interface that could improve on the complexity and efficiency of a straightforward cache.

### Performance Measurement

As part of the above, let's get serious about measuring the performance of host/accelerator interfaces.
We'll need an array of benchmarks with different data-access characteristics and a systematic way to understand the data transfer cost and the end-to-end execution time.

## Other Targets

Xilinx FPGAs are not the entire world (surprisingly!).
Let's try to use the YXI abstraction layer to implement interfaces for Altera FPGAs or even some ASIC target we cook up.


Implicated Technical Debt
-------------------------

TK notes about:

* `ref` memories instead of `@external`
* `dyn_mem`
* old vs. new testbenches
