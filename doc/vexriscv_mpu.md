# VexRiscv MPU implementation
A prototype implementation of the RISC-V MPU was developed for the VEDLIoT project, which can be found on the `mpu` branch of `lindemer`'s fork of VexRiscv, [here](https://github.com/lindemer/VexRiscv/blob/mpu/src/main/scala/vexriscv/plugin/PmpPlugin.scala). Note that the MPU features were integrated directly into the PMP plugin's source code, because they share a significant amount of hardware logic. This implementation is based on a *simplified* version of the MPU proposal, which is described on this page. It provides the minimum features required to implement secure enclaves in three privelege levels (M/S/U).

- [VexRiscv MPU implementation](#vexriscv-mpu-implementation)
  - [Specification](#specification)
  - [Programming guide](#programming-guide)
    - [RISC-V privilege levels](#risc-v-privilege-levels)
    - [Trap handling](#trap-handling)
    - [Privilege override](#privilege-override)
    - [Exception codes](#exception-codes)
    - [Trap delegation](#trap-delegation)
  - [Integration with LiteX](#integration-with-litex)

## Specification
The concept and motivation behind this extension is described in [this paper](https://ascslab.org/conferences/secriscv/materials/papers/paper_15.pdf) and the [accompanying presentation](https://ascslab.org/conferences/secriscv/materials/presentations/presentation_paper_15.pdf). Since the actual [MPU specification](https://docs.google.com/document/d/1x7esOSBFfpcbDHaRPpe5NEWmav1_8der_nB25Hd5hqs/edit#) is still a draft and constantly changing, we implemented only the key features. 

In the VexRiscv implementation, the MPU behaves almost identically to PMP, except that the `mpucfg` and `mpuaddr` CSRs can be accessed by *both* M- and S-mode. The `mpucfg` CSRs have an `L` bit like `pmpcfg`, but the meaning is different. In `pmpcfg`, the `L` bit causes the permissions to be enforced on M-mode and prevents the region from being changed until a full device reset. The `L` bit in `mpucfg` can *always* be changed by both M- and S-mode, and it does not cause the permissions to be enforced on S-mode. Instead, it fully revokes S-mode's access to that region, but continues to enforce the specified permissions on U-mode. This is similar to how an MMU prevents the kernel from executing code located in userspace RAM. 

This implementation only supports NAPOT addressing, and the registers are *not* ordered by priority. If there are multiple matching regions corresponding to a memory access, the access will be granted if *any* of those regions allow it. The same limitations are present in VexRiscv's PMP implementation. The `mpucfg` registers are numbered `0x900` to `0x903` and the `mpuaddr` registers are numbered `0x910` to `0x91f`.

## Programming guide
The MPU unit tests on VexRiscv can be found [here](https://github.com/lindemer/VexRiscv/blob/mpu/src/test/cpp/raw/mpu/src/crt.S). These provide examples on how to configure it in Assembly. 

### RISC-V privilege levels

- **M-mode** can control both the PMP and MPU. Only locked PMP regions apply.
- **S-mode** can control the MPU. Locked and unlocked PMP regions apply. Locked MPU regions block access entirely, regardless of what the R/W/X permissions are.
- **U-mode** is restricted by both the PMP and the MPU. 

### Trap handling
By default, all exceptions will cause the CPU to jump to the M-mode trap handler, so on startup, one of the first actions is to declare its location. This is done by writing the `mtvec` CSR:
```
__start:
    la x1, mtrap
    csrw mtvec, x1

mtrap:
    csrr x1, mcause
    // do something
    csrw mepc, x2
    mret
```
Inside the trap handler, a few other CSRs are useful:
- `mcause` contains a code indicating what caused the exception.
- `mepc` contains the address that the CPU will jump back to when `mret` is called.

`mret` is a special instruction which tells the CPU to return and switch to the previous privilege level. For example, if U-mode caused the exception, `mret` will switch the CPU back to U-mode. It's not possible to discover the current privilege level from software in RISC-V, which is an intentional design choice to enhance security.

### Privilege override
The CPU uses the 2-bit `MPP` field in the `mstatus` register to determine which privilege level to switch to when `mret` is called. These bits are set automatically when an exception occurs, so trap handlers can simply call `mret` to return to the originating level. In order to switch to a *specific* privilege level, these bits must be set manually before calling `mret`. M-mode is `0b11`, S-mode is `0b01`, and U-mode is `0b00`. 

For example, after some startup configuration in M-mode, the CPU will typically need to jump into S-mode and boot the OS. This can be done with the following sequence (assuming `x1` holds the start address of the OS kernel):
```
    csrw mepc, x1
    li x1, 0x1800
    csrc mstatus, x1
    li x1, 0x0800
    csrs mstatus, x1
    mret
```
This clears the `MPP` bits and sets only the LSB of that field, in order to indicate S-mode. From S-mode, the jump to U-mode is quite similar. (Refer to the specification for more detailed information.):
```
    csrw sepc, x1
    li x1, 0x80
    csrc sstatus, x1
    sret
```

### Exception codes
The MPU is intended to be an alternative to the MMU, and has thus re-used its exception codes. These are:
- 12 for instruction violations (i.e., jumping to a location where no matching MMU region has the `X`-bit set)
- 13 for load violations (i.e., reading a word from a location where no matching MMU region has the `R`-bit set)
- 15 for store violations (i.e., writing a word from a location where no matching MMU region has the `W`-bit set)
It is significant that the PMP and MMU have distinct exception codes, because the PMP violations should typically be handled by M-mode software, while the MMU violations should be handled by S-mode software.

### Trap delegation
One of the key use cases for the RISC-V MPU is running an embedded OS in S-mode with its own exception handler. Indeed, an operating system *needs* an exception handler. This way, the OS can use the MPU for thread isolation and abort threads that violate their permissions. To do this, the CPU must be configured at startup to send MPU exceptions to the S-mode trap handler, but leave other exceptions (e.g., PMP violations) to the M-mode trap handler.

The `mdeleg` CSR indicates which exceptions will be handled by the next-highest privilege level, in this case S-mode. The MPU throws page fault exceptions, which are numbered 12, 13 and 15. To delegate these to the S-mode trap handler, the corresponding bits must be set in `mdeleg` (i.e., `0xb000`). (Note that all CSRs starting with the letter `m` are accesible only to M-mode.) After entering S-mode, the trap handler for those exceptions is declared by writing `stvec`. (Note that most of the M-mode CSRs have siblings in S-mode, i.e., `scause`, `sstatus`, etc.)

## Integration with LiteX
