AArch64 Generic Interrupt Controller (v3/v4) Example
====================================================

Introduction
============
This example demonstrates the use of the Generic Interrupt Controller (GIC) in a baremetal environment.


Notice
=======
Copyright (C) Arm Limited, 2019 All rights reserved.

The example code is provided to you as an aid to learning when working 
with Arm-based technology, including but not limited to programming tutorials. 
Arm hereby grants to you, subject to the terms and conditions of this Licence, 
a non-exclusive, non-transferable, non-sub-licensable, free-of-charge licence, 
to use and copy the Software solely for the purpose of demonstration and 
evaluation.

You accept that the Software has not been tested by Arm therefore the Software 
is provided “as is”, without warranty of any kind, express or implied. In no 
event shall the authors or copyright holders be liable for any claim, damages 
or other liability, whether in action or contract, tort or otherwise, arising 
from, out of or in connection with the Software or the use of Software.


Requirements
============
* DS-5 Ultimate Edition (5.29 or later) or Arm Development Studio
https://developer.arm.com/tools-and-software/embedded/arm-development-studio

* Armv8-A Base Platform FVP, FVP_Base_RevC-2xAEMv8A
https://developer.arm.com/tools-and-software/simulation-models/fixed-virtual-platforms


File list
=========
 <root>
  |-> headers
  |   |-> generic_timer.h
  |   |-> gicv3_basic.h
  |   |-> gicv3_lpis.h
  |   |-> gicv3_registers.h
  |   |-> gicv4_virt.h
  |   |-> sp804_timer.h
  |   |-> system_counter.h
  |
  |-> src
  |   |-> el3_vectors.s      Minimal vector table.
  |   |-> generic_timer.s    Helper functions for Generic Timer.
  |   |-> gicv3_basic.c      Helper functions for GICv3.1.
  |   |-> gicv3_cpuif.S      GICv3 CPU interface functions.
  |   |-> gicv3_lpis.c       Helper functions for GICv3.1 physical LPIs.
  |   |-> main_basic.c       Example showing usage of SPIs and PPIs.
  |   |-> main_gicv31.c      Example showing usage of GICv3.1 Extended PPI and SPI ranges.
  |   |-> main_lpi.c         Example showing usage of physical LPIs.
  |   |-> sp804_timer.c      SP804 Dual Timer.
  |   |-> startup.s          Minimal reset handler.
  |   |-> system_counter.c   Helper functions for System Counter.
  |
  |-> Makefile               Makefile for use with GNU make and Arm Compiler 6.
  |-> ReadMe.txt             This file.
  |-> scatter.txt            Memory layout scatter file for linker.


Description
===========
The package includes a number of small example programs, each demonstrating a different aspect of using the GIC.

These examples work with both GICv3.x and GICv4.x:

  image_basic
    This example shows the basic set up of a GICv3/4 interrupt controller, including the PPI and SPI interrupt types.

  image_lpi
    This example shows the setup and use of physical LPIs and the ITS.


These examples require GICv3.1:

  image_gicv31
    This example shows the use of the GICv3.1 extended PPI and SPI ranges.



Building the examples from the command line
==========================================
To build the examples:
- Open a DS-5 or Arm Developer Studio command prompt, then navigate to the location of the example

The make command depends on the version of the GIC architecture implemented.

To build the examples for GICv3.x:
    - Run "make"
    
To build the examples for GICv4.x:
    - Run "make GICV=GICV4"

Optionally, adding "DEBUG=TRUE" results in additional logging messages being printed.

Note:
When a GIC implements GICv3.x, the redistributors occupy 128K of address space.  When GICv4.x is implemented, they occupy 256K.  The example requires the redistributor size at build time, which is why the GIC version is passed as an argument to make.


Explanation of model parameters
===============================
These examples are intended to run on the Armv8-A Base Platform FVP (FVP_Base_RevC-2xAEMv8A).
The FVP_Base_RevC-2xAEMv8A model takes a number of parameters to configure how the GIC appears to software.
Some of the examples require non-default values for a number of parameters. These parameters and values are as follows:

-C cluster0.gicv3.extended-interrupt-range-support=1
  Controls whether the PE supports the GICv3.1 extended range, default 0.  the image_gicv31 examples requires this to be set to 1.


-C gic_distributor.extended-ppi-count=<n>
  Controls how many (if any) GICv3.1 extended PPIs are available, default 0.  The image_gicv31 example requires this to be set to 32.


-C gic_distributor.extended-spi-count=<n>
  Controls how many (if any) GICv3.1 extended SPIs are available, default 0.  The image_gicv31 example requires this to be set to 32.


-C gic_distributor.ITS-count=<n>
  Controls how many (if any) ITSs are present, default 0. Most of the examples require it to be set to 1.


-C gic_distributor.ITS-use-physical-target-addresses=<n>
  Controls the value of GITS_TYPER.PTA, default 1.  Most of the examples requires it to be set to 0.


-C gic_distributor.virtual-lpi-support=<bool>
  Whether the GIC supports GICv4.x, default FALSE.  The GICv4.x examples require it to be set to TRUE.


-C gic_distributor.GITS_BASER<x>-type=<n>
  The type of table for each GITS_BASER<n> register.  Most of the examples require this to be set as follows:
    GITS_BASER0 = 1
    GITS_BASER1 = 4
    GITS_BASER2 = 2 (GICv4.x examples only)


-C gic_distributor.ITS-vmovp-bit=<bool>
  Controls the value of GITS_TYPER.VMOVP, default 0.  GICv4.x examples require 1.



Running the example on GICv3.0:
===============================
The image_basic and image_lpi examples are compatible with GICv3.0.

- Open a command prompt, then navigate to the location of Base Platform FVP executable
- To run on the AEM Base Platform Model:

  FVP_Base_RevC-2xAEMv8A -C cluster0.NUM_CORES=1 -C cluster1.NUM_CORES=0 -C gic_distributor.GITS_BASER0-type=1 -C gic_distributor.GITS_BASER1-type=4 -C bp.secure_memory=0 -C gic_distributor.ITS-count=1 -C gic_distributor.ITS-use-physical-target-addresses=0 -C gic_distributor.GITS_BASER6-type=0 --application=<PATH TO EXAMPLE>

- To run on the Cortex-A53x1 Base Platform Model (shipped with Arm DS)

  FVP_Base_Cortex-A53x1 -C gic_distributor.GITS_BASER0-type=1 -C gic_distributor.GITS_BASER1-type=4 -C bp.secure_memory=0 -C gic_distributor.ITS-count=1 -C gic_distributor.ITS-use-physical-target-addresses=0 -C gic_distributor.GITS_BASER6-type=0 --application=<PATH TO EXAMPLE>


Running the example on GICv3.1:
===============================
Note: These examples require the AEM Base Platform model

The image_basic, image_lpi and image_gicv31 examples are compatible with GICv3.0.

- Open a command prompt, then navigate to the location of Base Platform FVP executable
- Run:

  FVP_Base_RevC-2xAEMv8A -C cluster0.NUM_CORES=1 -C cluster0.gicv3.extended-interrupt-range-support=1 -C cluster1.NUM_CORES=0 -C gic_distributor.GITS_BASER0-type=1 -C gic_distributor.GITS_BASER1-type=4 -C bp.secure_memory=0 -C gic_distributor.ITS-count=1 -C gic_distributor.ITS-use-physical-target-addresses=0 -C gic_distributor.GITS_BASER6-type=0 -C gic_distributor.extended-spi-count=32 -C gic_distributor.extended-ppi-count=32 --application=<path_to_example>
