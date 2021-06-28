## Building and Emulation

The building of codes for the physical FPGA chips is a time consuming process. Firstly it is useful to have a powerful system for undertaking the actual build process, which is the case for this testbed, and also vendors provide emulation support such that users can much more quickly build and test their codes on the CPU without having to do the full build for the FPGA.

### Xilinx toolchain

You should first load the _vitis_ module, `module load vitis`. This will make available Xilinx specific tooling, including _v++_, _vivado_, _vitis_hls_, _vitis_, and _vitis_analyser_ into your environment, as well as setting up the correct OpenCL environment for hardware emulation. We have developed a simple sum example to demonstrate using the Xilix toolchain on the testbed, this can also be used as a skeleton structure for more complex FPGA codes if that is helpful.

```console
[username@nextgenio-login2 ~]$ git clone https://github.com/FPGAtestbed/xilinx_sum_example.git
[username@nextgenio-login2 ~]$ cd xilinx_sum_example
```

The example contains a makefile which provides configurations for building for software emulation, hardware emulation, and physical hardware. 

#### Building and running for software emulation

Software emulation is the quickest, both in terms of build time (typically less than a minute) and runtime. It is however the least accurate as the code is not synthesised into RTL, but instead the C++ is compiled into a CPU executable and run. There can be times when SW emulation runs OK but then hardware emulation does not (e.g. provides the wrong answer or hangs), but regardless this is a useful first step at-least for checking kernel logic.

```console
[username@nextgenio-login2 ~]$ make host
[username@nextgenio-login2 ~]$ make device TARGET=sw_emu 
.....
[username@nextgenio-login2 ~]$ export XCL_EMULATION_MODE=sw_emu
[username@nextgenio-login2 ~]$ bin/host bin/sum_kernel.sw_emu.xclbin
```

#### Building and running for hardware emulation

Hardware emulation takes longer to build for, typically around 10 minutes, and runtime is extremely slow (sometimes up to a million times slower than the FPGA itself!) This is because hardware emulation is cycle accurate, where the programmer's C++ code is synthesised by the HLS tooling to the target RTL as part of the build process and this is then executed as it would be on the physical FPGA chip. Therefore it is commonplace to run hardware emulation with substantially reduced data sizes in order for it to complete in a timely fashion. It should be noted that, whilst the execution is hardware accurate and can give some indication of performance, some important aspects such as external memory access can have a large performance impact on the physical FPGA and not always accurately measured here. Therefore, it is our suggestion that if code runs correctly in hardware emulation then you can be confident that it will run on the actual FPGA, but for performance measurements then you should always do that on the FPGA itself rather than relying on emulation. 

The console snippet below illustrates running hardware emulation, note that if the host binary already exists then you do not need to rebuild this. At the first line we are issuing `make emconfig`, this creates a platform specific descriptor (we are using the U280 here) which is used as input to the emulation tool. Note that once this is created then it need not be created each time.

```console
[username@nextgenio-login2 ~]$ make emconfig
[username@nextgenio-login2 ~]$ make host
[username@nextgenio-login2 ~]$ make device TARGET=hw_emu 
.....
[username@nextgenio-login2 ~]$ export XCL_EMULATION_MODE=hw_emu
[username@nextgenio-login2 ~]$ bin/host bin/sum_kernel.hw_emu.xclbin
```
