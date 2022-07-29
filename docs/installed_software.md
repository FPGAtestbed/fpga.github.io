## Software available on the testbed

Throughout the testbed lifetime we will install a variety of FPGA software and libraries for use by programmers, this page will be updated with details. Typically this software is made available via the module environment, and you should execute `source /home/nx08/shared/fpga/fpga_modules.sh` to make these available (or add this to your .bashrc file, more details [here](https://fpga.epcc.ed.ac.uk/docs/getting_started.html)).

### General

There are some general modules provided which apply to both Xilinx and Intel technologies

| Software  | Version | Notes | Module name |
| ------------- | ------------- | ------------- | ------------- | 
common | 1.0 | Contains utility scripts | common_fpga/1.0 |
ocl-icd | 2.2.12 | OpenCL ICD loader for emulation under Xilinx and Intel | ocl-icd/2.2.12 |
host_support | 0.1 | Library for host code support functionality | host_support/0.1 |

### Xilinx Vitis 

The core Xilinx Vitis toolchain which provides command line (_v++_) and GUI (_vitis_, _vitis_hls_, _vivado_, _vitis_analyzer_) tools for application development on the Alveo and Versal. Loading the Vitis module will also automatically set-up the Xilinx OpenCL environment on the front-end node for hardware emulation.

| Software  | Version | Notes | Module name |
| ------------- | ------------- | ------------- | ------------- | 
| Vitis  | 2020.1  | Required for TCP/IP stack | vitis/2020.1 |
| Vitis  | 2020.2  | Default | vitis/2020.2 |
| Vitis  | 2021.1  |  | vitis/2021.1 |
| Vitis  | 2021.2  | | vitis/2021.2 |
| Vitis  | 2022.1  | Required for programming VCK5000 | vitis/2022.1 |

| Software  | Version | Notes | Module name |
| ------------- | ------------- | ------------- | ------------- | 
| Vitis Library | 2021.2  | Compatible with previous versions of Vitis | vitis_libraries/2021.2 |

| Software  | Version | Notes | Module name |
| ------------- | ------------- | ------------- | ------------- | 
| XRT | 2.11.634  | Default | xrt/2.11.634 |
| XRT | 2.12.447  |  | xrt/2.12.447 |
| XRT | 2.13.446  |  | xrt/2.13.446 |
| XRT | 2.13.478  | Required for VCK5000 | xrt/2.13.478 |

*Note*: Loading Vitis module will also automatically load GCC 10.2, ocl-icd, host_support, common_fpga, and XRT. The exception is Vitis/2022.1 which will load GCC 7.3 rather than GCC 10.2 due the compatability issues.

### Intel Quartus

The Intel Quartus toolchain providesthe Quartus Prime Design Software, along with the high level synthesis (HLS) compiler, the OpenCL framework, and other tools. Loading the quartus module will configure the system to utilise these.

| Software  | Version | Notes | Module name |
| ------------- | ------------- | ------------- | ------------- |
| Quartus  | 19.4  | | module load quartus/19.4 | 
| Quartus  | 20.2  | | module load quartus/20.2 | 
| Quartus  | 20.4  | Default, should be used for Stratix-10 | module load quartus/20.4 | 

| Software  | Version | Notes | Module name |
| ------------- | ------------- | ------------- | ------------- | 
| Bittware  | s10mx  | Support for Stratix-10 MX | bittware/s10mx | 

*Note*: Loading Quartus module will also automatically load GCC 10.2, ocl-icd, bittware/s10mx, and common_fpga

### Intel OneAPI

OneAPI, from Intel, includes a data parallel programming lanugage, DPC++, that is able to build applications for FPGAs. DPC++ is an implementation of the SYCL framework, and as such can port across a range of computing hardware, including GPUs. It integrates with Quartus tools to create bitstreams for Intel FPGAs, and there is potential to use other versions of SYCL alongside DPC++ to port to Xilinx FPGAs as well. OneAPI DPC++ functionality is accessed through the compiler and dpc modules on the system.

| Software  | Version | How to access |
| ------------- | ------------- | ------------- |
| DPC++ | 2021.1.1  | module load compiler/2022.0.2; module load dpl/2021.1.1  | 
| DPC++ | 2021.1.4  | module load compiler/2022.0.2; module load dpl/2021.1.4  | 
 
