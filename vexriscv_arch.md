# VexRiscv architecture
[VexRiscv](https://github.com/SpinalHDL/VexRiscv) is an open-source RISC-V processor written in [SpinalHDL](https://github.com/SpinalHDL/SpinalHDL). It was awarded first place in the RISC-V Foundation's [SoftCPU contest](https://riscv.org/blog/2018/12/risc-v-softcpu-contest-highlights/) in 2018 for achieving the highest performance on both Lattice and Microsemi FPGAs. It can be scaled to fit a wide range of use cases, from highly constrained bare-metal embedded applications to multi-core Linux with the same code base.

VexRiscv takes a uniquely software-inspired approach to hardware description. Virtually every component is a __plugin__ object con\nected to one or more stages in the pipeline. Plugins can insert data into the pipeline at one stage, and VexRiscv will automatically propagate those signals through down the pipeline to be accessed by later stages. Plugins can also provide __services__, which are essentially APIs for other plugins to interact with. The result is a highly configurable CPU where everything down to the register file is interchangeable.

- [VexRiscv architecture](#vexriscv-architecture)
  - [SpinalHDL](#spinalhdl)
    - [Supplementary materials](#supplementary-materials)
    - [Suggestions](#suggestions)
  - [CPU pipeline](#cpu-pipeline)
  - [Plugins](#plugins)
  - [Services](#services)

## SpinalHDL
SpinalHDL is a Scala-based hardware description language. Traditional HDLs offer few high-level abstractions, which leads to long and repetitive descriptions of wiring interconnects. Developing complex hardware in those languages is very time consuming, and maintenence can be difficult. SpinalHDL allows developers to leverage high-level abstractions, as well as object-oriented and functional programming patterns, to create concise self-documenting hardware.

A frequently asked question regarding SpinalHDL is __how does it compare to Chisel__? The language was inspired by, though not quite "forked" from, Chisel 2. The Chisel ecosystem exists to support ASIC design and manufacturing. At the time of this writing, it's under active development with frequent API changes. SpinalHDL targets FPGAs, and is essentially "frozen" at this point. It is still actively maintained (i.e., bugs are fixed quickly and new features are added on occasion), but the API is fixed for the foreseeable future.

### Supplementary materials
An overview of the language with illustrations is available [here](https://cdn.jsdelivr.net/gh/SpinalHDL/SpinalDoc@master/presentation/en/presentation.pdf). A great starting point for learning is the [workshop](https://github.com/SpinalHDL/SpinalWorkshop), and the [docs](https://spinalhdl.github.io/SpinalDoc-RTD) are thorough and up-to-date.

### Suggestions
- Start with the workshop and inspect the Verilog output. It is important to have an intuition for how SpinalHDL translates to hardware when working with larger projects.
- Always be aware of what is pure Scala and what is part of the SpinalHDL libraries. It is sometimes desirable to specify a component and then duplicate or connect it with functional programming techniques. This can lead to unexpected results for the inexperienced user.
- You do __not__ need to install SpinalHDL from source to work with VexRiscv. The Scala package manager will handle this automatically when building the CPU.

## CPU pipeline

## Plugins

## Services