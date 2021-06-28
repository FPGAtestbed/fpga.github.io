## Building and Emulation

The building of codes for the physical FPGA chips is a time consuming process. Firstly it is useful to have a powerful system for undertaking the actual build process, which is the case for this testbed, and also vendors provide emulation support such that users can much more quickly build and test their codes on the CPU without having to do the full build for the FPGA.

## Xilinx toolchain

You should first load the _vitis_ module, `module load vitis`. This will make available Xilinx specific tooling, including _v++_, _vivado_, _vitis_hls_, _vitis_, and _vitis_analyser_ into your environment, as well as setting up the correct OpenCL environment for hardware emulation. 

```console
[username@nextgenio-login2 ~]$ git clone https://github.com/FPGAtestbed/xilinx_sum_example.git
[username@nextgenio-login2 ~]$ cd xilinx_sum_example
```
