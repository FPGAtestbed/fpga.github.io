## Software available on the testbed

Throughout the testbed lifetime we will install a variety of FPGA software and libraries for use by programmers, this page will be updated with details. Typically this software is made available via the module environment, and you should execute `source /home/nx08/shared/fpga/fpga_modules.sh` to make these available (or add this to your .bashrc file, more details [here](https://fpga.epcc.ed.ac.uk/docs/getting_started.html)).

### Xilinx Vitis 

The core Xilinx Vitis toolchain which provides command line (_v++_) and GUI (_vitis_, _vitis_hls_, _vivado_, _vitis_analyzer_) tools for application development on the Alveo and Versal. Loading the Vitis module will also automatically set-up the Xilinx OpenCL environment on the front-end node for hardware emulation.

| Software  | Version | How to access |
| ------------- | ------------- | ------------- |
| Vitis  | 2020.2  | module load vitis | 
| Vitis  | 2021.1  | module load vitis/2021.1 |

### Intel Quartus

The Intel Quartus toolchain providesthe Quartus Prime Design Software, along with the high level synthesis (HLS) compiler, the OpenCL framework, and other tools. Loading the intelFPGA_pro module will configure the system to utilise these.

| Software  | Version | How to access |
| ------------- | ------------- | ------------- |
| Quartus  | 20.4  | module load intelFPGA_pro | 

### Intel OneAPI

OneAPI, from Intel, includes a data parallel programming lanugage, DPC++, that is able to build applications for FPGAs. DPC++ is an implementation of the SYCL framework, and as such can port across a range of computing hardware, including GPUs. It integrates with Quartus tools to create bitstreams for Intel FPGAs, and there is potential to use other versions of SYCL alongside DPC++ to port to Xilinx FPGAs as well. OneAPI DPC++ functionality is accessed through the compiler and dpc modules on the system.

| Software  | Version | How to access |
| ------------- | ------------- | ------------- |
| DPC++ | 2021.1.1  | module load compiler/2022.0.2; module load dpl/2021.1.1  | 
| DPC++ | 2021.1.4  | module load compiler/2022.0.2; module load dpl/2021.1.4  | 
 
