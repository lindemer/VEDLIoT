## Overview
This page provides installation instructions for the [LiteX](https://github.com/enjoy-digital/litex) build system on Linux. 

- [Overview](#overview)
- [Xilinx Vivado](#xilinx-vivado)
  - [Installation](#installation)
  - [Running](#running)
- [LiteX](#litex)

## Xilinx Vivado
Vivado is a commercial design and synthesis tool. The LiteX build system uses this sofrware as a backend to generate bitstreams for Xilinx FPGAs.

### Installation
1. Create an installation directory with:
```
sudo mkdir /opt/Xilinx
sudo chown $USER:$USER /opt/Xilinx
```
2. Create a Xilinx account [here](https://www.xilinx.com/registration/create-account.html).
3. Download a Linux self-extracting installer for Vivado [here](https://www.xilinx.com/support/download.html). This tutorial is based on version 2018.2, but newer versions may also work.
4. Unpack the installer, run `chmod +x` on the `.bin` file and run the installer.
5. Customize the installation according to the screenshots shown below.

![Vivado edition](vivado_edition.png)
![Vivado customization](vivado_customization.png)
![Vivado path](vivado_path.png)

### Running
It should now be possible to run Vivado from the command line. First, run `source /opt/Xilinx/Vivado/2018.2/settings64.sh` (substituting your Vivado version), then run `vivado`. Consider adding the first command to your `~/.bashrc`, as it is required by the LiteX build system after every new login.
   
## LiteX
