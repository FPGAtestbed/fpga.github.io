## VCK5000 Building and Emulation

The VCK5000 contains a VC1902 Versal AI-core ACAP and 16GB of DDR-DRAM. This is an interesting architecture as it provides AI engines which are 7-way VLIW processors which can perform vectorised fixed-point and (single precision) floating-point arithmetic. It is suggested that you are already familiar with programming the non-Versal Alveo cards (e.g. the U280 or U250) before working with the VCK5000 as the concepts there will be required.

### Versal chip specifications

The VC1902 Versals we have are of the AI-core family and combine Programmable Subsystem (PS), Programmable Logic (PL), and AI engines. The PS comprises a dual-Core ARM Cortex-A72 and dual-Core ARM Cortex-R5, the PL provides 900K LUTs, 1968 DSPs, 4.2MB BRAM, and 16.3MB URAM. There are 400 AI engines each of which is running at 1.2GHz. The board itself also provides 16GB of DDR-DRAM and PCIe-Gen4.

For more details about AIE development then see the [programmers guide](https://docs.xilinx.com/r/en-US/ug1076-ai-engine-environment) and [best practice guide](https://docs.xilinx.com/r/en-US/ug1079-ai-engine-kernel-coding).

## Xilinx toolchain

You should first load the _vitis/2022.1_ module, `module load vitis/2022.1`. This will make available Xilinx specific tooling, including _v++_, _vivado_, _vitis_hls_, _vitis_, and _vitis_analyser_ into your environment, as well as setting up the correct OpenCL environment for hardware emulation. It is important to use the 2022.1 version of the tooling here, as previous versions do not fully support the VCK5000. Furthermore there are some incompatibilities with GNU and associated modules. By default the _vitis/2022.1_ module will load the _gnu7/7.3.0_ which works fine, however other versions of GCC on the machine indexed under _/home/software/modulefiles_ load modules which result in bitstreams that are not compatible.

We have developed a simple number sum example [here](https://github.com/FPGAtestbed/vck5000_sum_example) to demonstrate using the Xilix VCK5000 toolchain on the testbed, this can also be used as a skeleton structure for more complex FPGA codes if that is helpful. It should be noted that this is more involved than building for the Alveos due to the inclusion of the AI engines.

```console
[username@nextgenio-login2 ~]$ git clone https://github.com/FPGAtestbed/vck5000_sum_example.git
[username@nextgenio-login2 ~]$ cd vck5000_sum_example
```

### Building for the AI engines

The _aiengine_ directory in this example contains the code, makefile, and test-files for the AI engines. Two modes of building and simulation are supported, x86 and native. We will build for x86 first, which can be thought of as software emulation - namely that it is a bit faster but doesn't provide the same level of in-depth testing.

```console
[username@nextgenio-login2 ~]$ cd aiengine
[username@nextgenio-login2 ~]$ make aie_compile_x86
......
[username@nextgenio-login2 ~]$ make aie_simulate_x86
````

Where the _make aie_compile_x86_ builds the code for x86 simulation, and `make aie_simulate_x86` will call the x86 simulator to execute the built code. If you look in the _data_ directory you will see two files, _input.txt_ and _num_elements.txt_, it is data from these which are fed into the AIE graph during simulation. The result of simulation can be found in _x86simulator_output/data/output.txt_. 

Next is to build the AIE code natively, this is the format not only used for native testing but also for programming the actual AIEs themselves.

```console
[username@nextgenio-login2 ~]$ make aie_compile
......
[username@nextgenio-login2 ~]$ make aie_simulate
````

Again, those input files are used to drive the simulation and the result of the run can be found in _aiesimulator_output/data/output.txt_. You will see a file named _libadf.a_ in the top level aiengine directory. This is the build AIE code which we will deploy on the VCK5000's AI engines.

### Building the host code

The _sw_ directory contains the host code for driving this example. The code itself, and execution model, is pretty similar to what one would expect when programming other, non-Versal, Alveo cards. It can be seen that the main interaction is with the _xclbin_ file (that we will build in a minute), deploying this as usual to the FPGA and then setting up kernels, buffers, and marshalling their execution. It should be noted that there is no explicit interaction with the AIE graph here, whilst the XRT API does provide calls for this it is not supported when host code is driven by the host CPU (compared to when it's deployed on the PS).

From the top level _vck5000_sum_example_ directory:

```console
[username@nextgenio-login2 ~]$ cd sw
[username@nextgenio-login2 ~]$ make host
````

After executing this step you will see a new _bin_ directory at the top level, this will contain the built host application code.

### Building the PL code

Building the HLS code for the PL is very similar to non-Versal Alveo cards, with the only difference being that there is an additional packaging step required which combines the PL and AIE aspects togther. In this example we have two HLS kernels, _device_mm2s.cpp_ which streams data onto the AIEs and _device_s2mm.cpp_ which streams data off the AIEs. 

>**NOTE:**  
> If you look in _device_mm2s.cpp_ you will see that the number of elements and add value are streamed into the AIE code at the start of execution, and in the corresponding AIE code you can see how this is retrieved. Whilst the AIEs provide a better way of doing this by runtime modifyable parameters, on the VCK5000 as we can not manipulate the AIE graph from the host this approach is the only one possible.

Assuming you have built the AIE code natively as above, then from the top level _vck5000_sum_example_ directory:

```console
[username@nextgenio-login2 ~]$ cd hw
[username@nextgenio-login2 ~]$ make device TARGET=hw_emu
````

This will result in the file _sum.hw_emu.xclbin_ in the top level _bin_ directory which is ready to run in HW emulation.

## Hardware emulation

Once you have built the required files then there should be two files in the top level _bin_ directory. These are _host_ which is the host application driver, and _sum.hw_emu.xclbin_ which contains the PL and AIE codes. Once _sum.hw_emu.xclbin_ is loaded by the host code then it will configure the PL with your kernels and programme the AIEs based on the code that has been built.

Running for hardware emulation is the same as other Alveo cards that support Vitis:

```console
[username@nextgenio-login2 ~]$ cd bin
[username@nextgenio-login2 ~]$ export XCL_EMULATION_MODE=hw_emu
[username@nextgenio-login2 ~]$ ./host sum.hw_emu.xclbin 16
````

Where we provide the name of the bitstream and number of elements to sum (which is always rounded up to the nearest multiple of eight due to 8-way vectorisation of single precision floating point operations on the AIEs). 

>**NOTE:**  
> When emulating for the VCK5000, the hardware emulator doesn't tend to terminate cleanly after execution has completed. The host code will finish and it will inform that the hardware emulator is shutting down, but then it hangs. You can Ctrl-C out of it, but you should then issue _ps_ to ensure there are no zombie processes running. If there are you can issues _source kilall.sh_ to kill them (you can get this by loading the _common_fpga_ module).

## Building for hardware and quick builds

To build for hardware, similarly to hardware emulation the AIE code must be built natively (e.g. _make aie_compile_). Then in the _hw_ directory issue _make device TARGET=hw_ . Remember to unset the _XCL_EMULATION_MODE_ environment variable when then running it. A suggested flow is:

```console
[username@nextgenio-login2 ~]$ nohup make device TARGET=hw &> output &
[username@nextgenio-login2 ~]$ tail -fn2000 output
```

This will build the bitstream as a background process, and by tailing the _output_ file then you can view in real time the progress. Vitis creates numerous sub-processes and a disadvantage of this nohup approach is if you then want to terminate the build, for instance if you realise there is an error in the code that needs corrected, then it can be very time consuming to kill each individual process. Instead, we have provided a _killall_ script in the common_fpga module which will kill all the processes running either foreground or background in the current terminal.

```console
[username@nextgenio-login2 ~]$ module load common_fpga
[username@nextgenio-login2 ~]$ source killall.sh
```

### Quick builds

If you modify your AIE code and rebuilt it, then as long as you haven't changed the interface between the PL and AIE (e.g. the number or configuration of streams) then you don't need to fully rebuild the bitstream. This is advantageous as it will cut out the (potentially very) time consuming process required by a full build. Instead, if you already have the _.xsa_ file (e.g. you have done a full build previously) then you can simply to a repackage which only takes a few seconds.

To do this for our example here, in the _hw_ directory you can use the _repackage_ rule, e.g. by issuing _make repackage TARGET=hw_emu_ (this also works when building for hardware too).

## Troubleshooting

If you are using the native XRT C++ API, and on building get an error like below, this is because the XRT libraries have been compiled without using the C++11 ABI, therefore it expects _std::string_ whereas the C++11 string is _std::__cxx11::basic_string_ and these things are seen as incompatible by the linker. To address this you need to compile your host code with _D_GLIBCXX_USE_CXX11_ABI=0_ passed as a command line argument which will build the code without the C++11 ABI and hence make it compatible with the XRT built library.

```console
ost_overlay.cpp:(.text+0x11a7): undefined reference to `xrt::device::load_xclbin(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)'
```
