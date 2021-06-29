## Software available on the testbed

Throughout the testbed lifetime we will install a variety of FPGA software and libraries for use by programmers, this page will be updated with details. Typically this software is made available via the module environment, and you should execute `source /home/nx08/shared/fpga/fpga_modules.sh` to make these available (or add this to your .bashrc file, more details [here](https://fpga.epcc.ed.ac.uk/docs/getting_started.html)).

### Xilinx Vitis 

The core Xilinx Vitis toolchain which provides command line (_v++_) and GUI (_vitis_, _vitis_hls_, _vivado_, _vitis_analyzer_) tools for application development on the Alveo and Versal. Loading the Vitis module will also automatically set-up the Xilinx OpenCL environment on the front-end node for hardware emulation.

| Software  | Version | How to access |
| ------------- | ------------- | ------------- |
| Vitis  | 2020.2  | module load vitis | 
| Vitis  | 2021.1  | module load vitis/2021.1 |
