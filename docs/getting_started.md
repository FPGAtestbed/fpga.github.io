## Getting started

This section describes how to access both the command line and GUI of the testbed system. The testbed is part of a larger machine, NextGenIO, with testbed users typically building and emulating their FPGA codes on a specific front-end node, and then running on the FPGAs which are part of back-end nodes. The front-end node contains two 48-core Cascade Lake Xeon Platinum CPUs, 512GB RAM, and files are stored on the Lustre high performance parallel filesystem which is also visible to the back-end nodes.

### Command line access

The testbed system is only accessible via the _hydra-vpn_ proxy host, and in the previous _applying for access_ step you will have signed up for accounts on both _hydra-vpn_ and _NextGenIO_. To access the testbed you need to therefore first ssh to _hydra-vpn_ via `ssh username@hydra-vpn.epcc.ed.ac.uk` , the first time that you access this you will need to use the password automatically allocated via the SAFE system (it can be retrieved via SAFE) and will be prompted to change this.

Once you have logged into hydra-vpn then you can access the testbed login node via `ssh username@nextgenio-login2` . This is the node  flow is illustrated below.

```console
username@localhost:~$ ssh username@hydra-vpn.epcc.ed.ac.uk
[username@hydra-vpn ~]$ ssh username@nextgenio-login2
[username@nextgenio-login2 ~]$
```
>**ADVICE:**  
> It is possible to automate the proxy step by setting this up in your ssh .config file as a proxyjump

### Desktop access

<img src="/docs/images/x2go_settings.png" width="400" height="400" align="right"/>

The lightweight XFCE desktop is installed on the front-end of the testbed system, which is especially useful for programming FPGAs as much of the tooling has a graphical component to it. The front-end is also running X2GO which tends to provide much better performance than vanilla X forwarding. Therefore we strongly suggest accessing the desktop via X2GO, with users just needing to install the client program which is available [here](https://wiki.x2go.org/doku.php/download:start). 

Once the client is installed, create a new profile with settings matching those as illustrated below (assuming that you use the same username and password for both _hydra-vpn_ and the _NextGenIO_ node.

>**NOTE:**  
> Whilst it is possible to run the individual graphical tools directly via X2GO, we strongly suggest doing this via the XFCE desktop environment as find that this provides a much better user experience.

### FPGA specific tooling

FPGA tooling and useful associated applications are available on the front-end via the module environment. There are a default set of modules made available on login (which can be viewed via `module available`) and a set automatically into the user environment (which can be viewed via `module list`). More general details about the module environment can be found [here](https://linux.die.net/man/1/module). 

The FPGA specific modules are not available by default and must be loaded, this can be done via the command below

```console
[username@nextgenio-login2 ~]$ source /home/nx08/shared/fpga/fpga_modules.sh
[username@nextgenio-login2 ~]$ module available
----------------- /home/nx08/shared/fpga/modulefiles -----------------
   common_fpga/1.0       intelFPGA_pro/20.3  (D)    vitis/2020.2 (D)
   intelFPGA_pro/19.4    ocl-icd/2.2.12             vitis/2021.1
```

>**ADVICE:**  
> You can add this command into your _.bashrc_ file which will then make these modules available automatically.

There are more details about the specifics of these modules on other pages, however _common_fpga/1.0_ deserves some discussion. This contains common applications that are generally useful when programming FPGAs. Currently this makes available firefox, GUI, and GUI-GUI.
