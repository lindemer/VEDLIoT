# VexRiscv-LiteX integration
LiteX includes a handful of pre-built VexRiscv CPU variants, as well as the Scala scripts used to generate them. Typical LiteX users will use the pre-built copies and do not need a Scala installation. This page gives an overview of how to integrate custom variants of VexRiscv into the LiteX build system.

- [VexRiscv-LiteX integration](#vexriscv-litex-integration)
  - [Adding new CPU variants](#adding-new-cpu-variants)
    - [Customizing the build flags](#customizing-the-build-flags)
    - [Litex integration](#litex-integration)
  - [Loading software](#loading-software)

## Adding new CPU variants
After following the instructions in the [getting started](quickstart.md) guide, you should have a directory containing the primary LiteX repository and several community repositories. The one containing the relevant VexRiscv files is called [pythondata-cpu-vexriscv](https://github.com/litex-hub/pythondata-cpu-vexriscv). In this repository, the build files can be found in `pythondata_cpu_vexriscv/verilog`. This directory has its own README file, which explains how to link your local VexRiscv clone for rebuilding the CPU. (You will need to do this to integrate a modified version of the CPU into LiteX.)

### Customizing the build flags
The script used to generate the CPU variants is called `GenCoreDefault.scala`, which can be found [here](https://github.com/litex-hub/pythondata-cpu-vexriscv/blob/master/pythondata_cpu_vexriscv/verilog/src/main/scala/vexriscv/GenCoreDefault.scala). Notice that this is similar to the `src/main/scala/vexriscv/demo` configuration files, except that this will add a Wishbone interface. This script takes command line arguments to determine the arrangement and properties of plugins added to the CPU.

The Scala build script is called by the [Makefile](https://github.com/litex-hub/pythondata-cpu-vexriscv/blob/master/pythondata_cpu_vexriscv/verilog/Makefile) to generate the set of CPUs that come pre-built with LiteX. Notice how the command line flags here interact with the configuration options in the Scala build script. Here, you can define a new variant with its own set of properties. To rebuild all the CPUs, simply run `make`.

Alternatively, you can build a specific variant. For example:
```
sbt compile "runMain vexriscv.GenCoreDefault --csrPluginConfig secure --pmps 16 --outputFile VexRiscv_MyVariant"
```

### Litex integration
After defining a new variant and building it, you'll want to reference it in the top-level LiteX build script. The one for VexRiscv can be found [here](https://github.com/enjoy-digital/litex/blob/13979a43b716295f941cd85c1aac773a1fd217aa/litex/soc/cores/cpu/vexriscv/core.py). After adding your new variant by following the examples already in the file, you will be able to reference it from the FPGA synthesis scripts. For example:
```
./arty.py --cpu-type=vexriscv --cpu-variant=my+variant --build
```
