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


Implicated Technical Debt
-------------------------

TK notes about:

* `ref` memories instead of `@external`
* `dyn_mem`
* old vs. new testbenches
