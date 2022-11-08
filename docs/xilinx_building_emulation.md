## Xilinx Building and Emulation

The building of codes for the physical FPGA chips is a time consuming process. Firstly it is useful to have a powerful system for undertaking the actual build process, which is the case for this testbed, and also vendors provide emulation support such that users can much more quickly build and test their codes on the CPU without having to do the full build for the FPGA.

### Xilinx toolchain

You should first load the _vitis_ module, `module load vitis`. This will make available Xilinx specific tooling, including _v++_, _vivado_, _vitis_hls_, _vitis_, and _vitis_analyser_ into your environment, as well as setting up the correct OpenCL environment for hardware emulation. We have developed a simple number sum example [here](https://github.com/FPGAtestbed/xilinx_sum_example) to demonstrate using the Xilix toolchain on the testbed, this can also be used as a skeleton structure for more complex FPGA codes if that is helpful.

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
[username@nextgenio-login2 ~]$ bin/host bin/sum_kernel.sw_emu.xclbin 100
Found Platform
Platform Name: Xilinx
INFO: Reading bin/sum_kernel.sw_emu.xclbin
Loading: 'bin/sum_kernel.sw_emu.xclbin'
Total runtime : 1.110740 sec, (0.403078 xfer on, 0.638774 execute, 0.068888 xfer off) for 100 elements
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
[username@nextgenio-login2 ~]$ bin/host bin/sum_kernel.hw_emu.xclbin 10
Found Platform
Platform Name: Xilinx
INFO: Reading bin/sum_kernel.hw_emu.xclbin
Loading: 'bin/sum_kernel.hw_emu.xclbin'
INFO: [HW-EMU 01] Hardware emulation runs simulation underneath. Using a large data set will result in long simulation times. It is recommended that a small dataset is used for faster execution. The flow uses approximate models for DDR memory and interconnect and hence the performance data generated is approximate.
Total runtime : 1003.154663 ms, (1.948597 ms xfer on, 1000.808350 ms execute, 0.397680 ms xfer off) for 10 elements
INFO: [HW-EMU 06-0] Waiting for the simulator process to exit
INFO: [HW-EMU 06-1] All the simulator processes exited successfully
```

If your run fails like this when you try to execute the compiled binary:
```console
CRITICAL WARNING: [SW-EM 09-0] Unable to find emconfig.json. Using default device "xilinx:pcie-hw-em:7v3:1.0"
terminate called after throwing an instance of 'std::_Nested_exception<HostSupportException>'
  what():  No devices found matching name 'u280' for platform with name 'Xilinx'
Aborted (core dumped)
```

Then you may need to create a configuration file for the FPGA platform you are emulating. You can do this as follows:
```console
[username@nextgenio-login2 ~]$ emconfigutil --platform xilinx_u280_xdma_201920_3
[username@nextgenio-login2 ~]$ cp emconfig.json bin/
```

#### Building for the FPGA

Building for actual hardware will take around 90 minutes for this example, which can be done via `make device TARGET=hw`. One challenge is that this building process is then connected to the console, so closing that will terminate the building of the bitstream. There are a number of ways around this, for instance by using the graphical desktop then when you quit out of X2GO it will typically keep the session open and running. However, a more complete way is to use the _nohup_ command. We would generally suggest redirecting stderr and tailing the output file too.

```console
[username@nextgenio-login2 ~]$ nohup make device TARGET=hw &> output &
[username@nextgenio-login2 ~]$ tail -fn2000 output
```

This will build the bitstream as a background process, and by tailing the _output_ file then you can view in real time the progress. Vitis creates numerous sub-processes and a disadvantage of this nohup approach is if you then want to terminate the build, for instance if you realise there is an error in the code that needs corrected, then it can be very time consuming to kill each individual process. Instead, we have provided a _killall_ script in the common_fpga module which will kill all the processes running either foreground or background in the current terminal.

```console
[username@nextgenio-login2 ~]$ module load common_fpga
[username@nextgenio-login2 ~]$ source killall.sh
```

## Troubleshooting

If you are using the native XRT C++ API, and on building get an error like below, this is because the XRT libraries have been compiled without using the C++11 ABI, therefore it expects _std::string_ whereas the C++11 string is _std::__cxx11::basic_string_ and these things are seen as incompatible by the linker. To address this you need to compile your host code with _D_GLIBCXX_USE_CXX11_ABI=0_ passed as a command line argument which will build the code without the C++11 ABI and hence make it compatible with the XRT built library.

```console
ost_overlay.cpp:(.text+0x11a7): undefined reference to `xrt::device::load_xclbin(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)'
```
