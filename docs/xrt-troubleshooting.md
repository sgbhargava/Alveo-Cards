﻿<table class="sphinxhide">
 <tr>
   <td align="center"><img src="https://www.xilinx.com/content/dam/xilinx/imgs/press/media-kits/corporate/xilinx-logo.png" width="30%"/><h1>Alveo Debug Guide</h1>
   </td>
 </tr>
</table>

# XRT Troubleshooting
The Xilinx Runtime library (XRT) is an open-source easy to use software stack that facilitates management and usage of Alveo accelerator cards. Users use familiar programming languages like C/C++ or Python to write host code which uses XRT to interact with the FPGA on the Alveo card. For more information on XRT, please go to the [XRT Github page](https://xilinx.github.io/XRT/master/html/index.html). If you are just starting to debug, please consult the [main Alveo Debug Guide page](../README.md) to determine if this is the best page for your purposes.

## This Page Covers
This page covers issues users have reported when using XRT. If your issue is not covered, please post on the [Xilinx forums](https://forums.xilinx.com/t5/Alveo-Accelerator-Cards/bd-p/alveo).

## You Will Need
Before beginning debug, you need to:

- Have [Root/sudo permissions](common-steps.md#root-sudo-access)
- [Confirm System Compatibility](check-system-compatibility.md)

## Common Cases 

- - -
### Driver build succeeds

During XRT package installation, the XRT drivers are compiled and added to the linux kernel.  Look for messages that the drivers are built and registered with the linux kernel.

The following example is the expected output for a successful installation:
<br>

```
Building initial module for 4.15.0-118-generic
Done.

xocl:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/4.15.0-118-generic/updates/dkms/

xclmgmt.ko:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/4.15.0-118-generic/updates/dkms/

depmod...

DKMS: install completed.
Finished DKMS common.postinst
Loading new XRT Linux kernel modules
```

- - -
### Install hits an issue while apt/yum are running

If there are one or more error messages while apt or yum are running the install of XRT or the installation fails without an error message, there is an issue with the package manager.

Next step:

- Go to [Package manager](package-manager.md)
   * Install issues may need local IT involvement

- - -
### xdma driver on the system

If the xdma driver is installed and talking to the Alveo card this prevents XRT from talking to the card. This can be confirmed either with the  `lsmod | grep xdma` command showing the xdma driver running or `lspci` showing the xdma driver attached to the card. Examples of the output with each command are shown below.

- `lsmod` would show
```
:~> lsmod | grep xdma
xdma                 194893
```

- `lspci` would show
```
db:00.0 Memory controller: Xilinx Corporation Device 6987
Subsystem: Xilinx Corporation Device 1351
Flags: bus master, fast devsel, latency 0, NUMA node 1
Memory at 387f02000000 (64-bit, prefetchable) [size=32M]
Memory at 387f04010000 (64-bit, prefetchable) [size=64K]
Capabilities: <access denied>
Kernel driver in use: xdma
Kernel modules: xdma, xclmgmt
```

Next step:

  - Reload the drivers via the following commands
    - `modprobe -r xdma`
    - `modprobe xclmgmt`
    - `modprobe xocl`

- - -
### XRT reports unknown driver version

If XRT shows that `XOCL: unknown` or `XCLMGMT: unknown`, XRT is not seeing the driver kernel modules. Without the drivers loaded, XRT will not be able to communicate with the card. This can be seen with the command `xbutil --version` below:

```
:~> xbutil --version
     XRT Build Version: 2.8.333
     Build Version Branch: master
     Build Version Hash: af0982d05118b8f53b17c86f8d7230de55726253
     Build Version Hash Date: Wed, 19 Aug 2020 23:51:40 -0700
     Build Version Date: Thu, 20 Aug 2020 08:32:40 +0000
                   XOCL: unknown
                XCLMGMT: unknown
```

Next steps:

- See if the drivers are present on the system with `lsmod`
   * `lsmod | grep xocl`
   * `lsmod | grep xclmgmt`
- If both drivers are present [Reload the XRT drivers](common-steps.md#unload-reload-xrt-drivers)
- If a driver is missing go to [Driver not installed into Kernel](#drivers-not-installed-into-kernel) (below)

- - -

### Drivers not installed into kernel

If the XRT drivers aren't visible on `lsmod` with the `lsmod | grep xocl` and `lsmod | grep xclmgmt` commands, there was an issue with the installation. During XRT package installation, the XRT drivers are compiled and added to the linux kernel. Review the installation messages in the console for messages like those below:

```
DKMS failed to install XRT drivers.
```
or

```
Error!  Build of xocl.ko failed for: 3.10.0-1127.19.1.el7.x86_64 (x86_64)
Consult the make.log in the build directory
/var/lib/dkms/xrt/2.8.333/build/ for more information
```
Note: the tail end of the install messaging will look OK. XRT installation does not report errors at the end.

```
Installed:
  xrt.x86_64 0:2.8.333-1

Complete!
```

Next steps:

- Confirm the XRT version is supported by this OS release
   * [Determine the linux release](common-steps.md#determine-linux-release)
   * [Determine kernel headers](common-steps.md#determine-linux-kernel-and-header-information) match the release, and are installed correctly
- [Remove XRT](common-steps.md#remove-xrt)
- Download the latest XRT from the [Alveo landing page](https://www.xilinx.com/products/boards-and-kits/alveo.html)
- Re-install XRT
   * Check for dependency issues
   * For RH/Centos, follow the steps on the [Xilinx Runtime install](https://xilinx.github.io/XRT/master/html/install.html) to enable the optional-rpms, or epel-release
- If issue persists capture error messages and post on the [Xilinx forums](https://forums.xilinx.com/t5/Alveo-Accelerator-Cards/bd-p/alveo).

- - -

### No card is found

If the `xbmgmt flash --scan --verbose` command does not recognize the platform on the card it displays `No card is found!` as shown below.  

```
:~> sudo xbmgmt flash --scan --verbose
No card is found!
```

If this occurs and `lspci` does recognize the card (displays `Kernel driver in use: xclmgmt` shown below), there is a communication issue between XRT and the card. Example output below:


```
:~> sudo lspci -vd 10ee:
83:00.0 Processing accelerators: Xilinx Corporation Device 5020
        Subsystem: Xilinx Corporation Device 000e
        Physical Slot: 4
        Flags: bus master, fast devsel, latency 0, NUMA node 1
        Memory at 1232000000 (64-bit, prefetchable) [size=32M]
        Memory at 1234020000 (64-bit, prefetchable) [size=128K]
        Capabilities: [40] Power Management version 3
        Capabilities: [60] MSI-X: Enable+ Count=32 Masked-
        Capabilities: [70] Express Endpoint, MSI 00
        Capabilities: [100] Advanced Error Reporting
        Capabilities: [1c0] #19
        Capabilities: [e00] Access Control Services
        Capabilities: [e10] #15
        Capabilities: [e80] Vendor Specific Information: ID=0020 Rev=0 Len=010 <?>
        Kernel driver in use: xclmgmt
        Kernel modules: xclmgmt
```

Next steps:
- If card was last used in a Vivado flow, XRT will not be able to communicate to the card. Revert the card to the golden image using [AR 71757](https://www.xilinx.com/support/answers/71757.html).
- If card wasn't used in Vivado flow, [Remove XRT](common-steps.md#remove-xrt)
- Download the latest XRT from the [Alveo landing page](https://www.xilinx.com/products/boards-and-kits/alveo.html)
- Re-install XRT
- Install the desired deployment platform
  - Use the [Confirm XRT/Platform compatibility](common-steps.md#confirm-xrt-platform-compatibility) to ensure the platform and XRT are compatible
- [Determine platform on card and system](common-steps.md#determine-platform-and-sc-on-card-and-system)
- If the card does not have the desired platform [Flash the card with the deployment platform](common-steps.md#flash-the-card-with-a-deployment-platform)

- - -

### Nothing else has worked

- [Unload and reload the XRT drivers](common-steps.md#unload-reload-xrt-drivers)
- Look for similar issues on [Xilinx forums](https://forums.xilinx.com/t5/Alveo-Accelerator-Cards/bd-p/alveo)
- [Log machine state](common-steps.md#log-machine-state) and post on the [Xilinx forums](https://forums.xilinx.com/t5/Alveo-Accelerator-Cards/bd-p/alveo)

- - -

### Xilinx Support

For additional support resources such as Answers, Documentation, Downloads, and Alerts, see the [Xilinx Support pages](http://www.xilinx.com/support). For additional assistance, post your question on the Xilinx Community Forums – [Alveo Accelerator Card](https://forums.xilinx.com/t5/Alveo-Accelerator-Cards/bd-p/alveo). 

Have a suggestion, or found an issue please send an email to alveo_cards_debugging@xilinx.com .

### License

All software including scripts in this distribution are licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License.

You may obtain a copy of the License at
[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

All images and documentation, including all debug and support documentation, are licensed under the Creative Commons (CC) Attribution 4.0 International License (the "CC-BY-4.0 License"); you may not use this file except in compliance with the CC-BY-4.0 License.

You may obtain a copy of the CC-BY-4.0 License at
[https://creativecommons.org/licenses/by/4.0/]( https://creativecommons.org/licenses/by/4.0/)


Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

<p align="center"><sup>XD027 | &copy; Copyright 2021 Xilinx, Inc.</sup></p>