# RISC-V PMP
The RISC-V ISA is developed through open community discussions. At the time of this writing, one of the most actively-discussed topics is security, and a number of new extensions in this area have been proposed. This page provides an overview of the existing proposals and their current status.

- [RISC-V PMP](#risc-v-pmp)
  - [Getting involved](#getting-involved)
  - [Physical Memory Protection (PMP)](#physical-memory-protection-pmp)
    - [VexRiscv implementation](#vexriscv-implementation)
  - [IOPMP](#iopmp)
  - [PMP enhancements (formerly ePMP)](#pmp-enhancements-formerly-epmp)
  - [Memory Protection Unit (MPU, formerly sPMP)](#memory-protection-unit-mpu-formerly-spmp)

## Getting involved
All of the extensions described on this page are discussed in mailing lists and virtual meetings via the RISC-V Foundation. It is free to become a member, either as an individual or as an organization. Once you're a member, you can join any of the [mailing lists](https://lists.riscv.org/g/main). After an extension is sufficiently well-defined and agreed upon, addition to the official ISA is done via pull request on the ISA's [GitHub page](https://github.com/riscv/riscv-isa-manual).

## Physical Memory Protection (PMP)
PMP is configurable in RISC-V's highest privelege level, machine mode (M), and controls memory access by software running in supervisor (S) and user (U) mode. Small devices may only have M- and U-mode implemented. Machines running Linux, for example, require S-mode, because that is the privilege level which controls the MMU. VexRiscv has a PMP and an MMU plugin, but they are not compatible. Please refer to the RISC-V privileged specification for details.

### VexRiscv implementation
A highly optimized implementation of PMP was developed for the VEDLIoT project. This proved challenging because combinatorial logic with many inputs and outputs does not map well to FPGA fabric. In RISC-V, the PMP region bounds are written in an *encoded* format to the CPU's control status registers (CSRs). In order to enforce memory restrictions, these must be somehow *decoded* into a format that can be efficiently compared against memory addresses requested by software. The trivial approach to doing this is to instantiate a decoder, in hardware, for each CSR. Indeed, this is how other open-source implementations have done it (e.g., [Rocket](https://github.com/chipsalliance/rocket-chip/blob/86a2f2cca699f149bcc082ef2828654a0a4e3f4b/src/main/scala/rocket/PMP.scala), [CVA6](https://github.com/openhwgroup/cva6/blob/d24287e957981fb847fdbdaee9e318c2b502b412/src/pmp/src/pmp.sv), [NEORV32](https://github.com/stnolting/neorv32/blob/2be3da7f1ef60e3ab2125b25de913e1063ffa0f7/rtl/core/neorv32_cpu_control.vhd)). The VexRiscv implementation, instead, has a single decoder unit that decodes the CSRs over several CPU cycles when they are modified. This significantly reduces the hardware overhead of the extension.

**Limitations** The VexRiscv implemenation does not fully comply with the specification, because some features were identified to be of minimal utility for embedded applications and not worth the cost. Specifically:
1. The regions are not ordered by priority. In the specification, the permissions associated with the lowest-numbered PMP CSR that matches the current access are applied. In VexRiscv, permissions are granted if *any* matching region have them enabled. (E.g., if three matching regions have R--, -W-, and -WX permissions enabled, respectively, the memory access will be granted RWX access.)
2. Only the NAPOT addressing mode is implemented. The minimum granularity is 8 bytes, but this can be made coarser in the configuration file.

## IOPMP

## PMP enhancements (formerly ePMP)
A number of proposed enhancements to PMP are currently awaiting ratification. The latest draft, at the time of this writing, is avialable [here](https://docs.google.com/document/d/1Mh_aiHYxemL0umN3GTTw8vsbmzHZ_nxZXgjgOUzbvc8/edit#). These changes are motivated by some known vulnerabilities in emedded systems. Please refer to the original document for more information. In a nutshell, the proposal is to allow PMP to place restrictions on lower privilege levels while removing all access to M-mode. See below:

| `pmpcfg` settings | M-mode | S/U-mode |
|:-----------------:|:------:|:--------:|
| `0000` | `----` | `----` |
| `0001` | `----` | `---X` |
| `0010` | `-RW-` | `-R--` |
| `0011` | `-RW-` | `-RW-` |
| `0100` | `----` | `-R--` |
| `0101` | `----` | `-R-X` |
| `0110` | `----` | `-RW-` |
| `0111` | `----` | `-RWX` |
| `1000` | `L---` | `L---` |
| `1001` | `L--X` | `----` |
| `1010` | `L--X` | `L--X` |
| `1011` | `LR-X` | `LR-X` |
| `1100` | `LR--` | `----` |
| `1101` | `LR-X` | `----` |
| `1110` | `LRW-` | `----` |
| `1111` | `LR--` | `L--R` |

## Memory Protection Unit (MPU, formerly sPMP)
The RISC-V TEE WG is actively discussing, and hoping to finalize, a so-called *memory protection unit* for RISC-V in 2021. Its former name, supervisor PMP (sPMP), was perhaps more descriptive. The idea is to duplicate PMP, but give control to S-mode software. This scheme allows developers to place an embedded OS in S- instead of M-mode without giving up access to the memory protection hardware required to isolate userspace threads. Then, a security monitor (i.e., hypervisor) can run in M-mode, which uses the existing PMP hardware to separate the OS from TEEs on the same device. The latest revision for RISC-V MPU, as of this writing, is available [here](https://docs.google.com/document/d/1x7esOSBFfpcbDHaRPpe5NEWmav1_8der_nB25Hd5hqs/edit#). The MPU specification incorporates features of ePMP, which is also on the agenda for finalization ion 2021.