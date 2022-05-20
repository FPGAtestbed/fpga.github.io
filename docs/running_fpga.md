## Running on the FPGAs

The FPGAs are hosted as back-end nodes of NEXTGenIO and currently accessible via ssh but the plan is to integrate these with the batch queue system. Therefore to access specific FPGA(s) you should ssh to the associated node, where all the nodes mount the Lustre filesystem and have available all the same tools as the login nodes.

| Node  | FPGAs | 
| ------------- | ------------- | 
| nextgenio-amd01  | Alveo U280  | 
| nextgenio-amd02  | Alveo U250  | 
| nextgenio-amd03  | Stratix-10 MX  | 
| nextgenio-icx | Unallocated |

## Xilinx environment

After connecting to the Xilinx nodes issue `module load vitis` and this will bring into your environment the nescesary settings to run on the Xilinx FPGAs and you can then run your host-side executable.

You can use the `xbutil` utility to manage the FPGAs, for instance if your code locks up then `xbutil reset` (likely in another terminal) will forcibly reset the FPGA and quick the code executing on it. `xbutil examine` will list available FPGA devices, and `xbutil examine --device 0000:d4:00.5 --report all` will report the status of the FPGA (where _0000:d4:00.5_ is the device ID obtained by the initial _examine_ command).

## Intel environment

After connecting to the Intel node issue `module load quartus` which will bring into your environment the settings to run on the Stratix-10 and you can then run your host-side executable. Currently the HPC OpenCL image is written onto the board, it is possible to load the MAX image which enables the QSP28 network ports and for this then please contact us.

You can use `aocl diagnose` to report the status of the FPGA(s) present
