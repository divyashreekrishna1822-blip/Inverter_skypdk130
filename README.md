# Physical Verification Using SKY130 PDK

This repository contains the documentation and design files for the physical verification workshop carried out by VSD using SkyWater 130nm Technology and open-source EDA tools.

The workshop describes the physical verification process that takes place during an RTL to GDSII flow using Skywater 130nm Technology, including DRC and LVS tests. This helps the participants get ready for tape-out and is particularly helpful for chip fabrication. The workshop assists in locating and fixing violations made during the physical verification stage.

More information about the workshop can be found at [https://www.vlsisystemdesign.com/physical-verification-using-sky130/](https://www.vlsisystemdesign.com/physical-verification-using-sky130/)

A special thanks to Tim, Kunal, and Sumanto for creating and helping out during this fantastic workshop:
- [R. Timothy Edwards](https://github.com/RTimothyEdwards)
- [Kunal Ghosh](https://github.com/kunalg123)

---

## Index
* [Chapter 0 - Getting the tools](#chapter-0---getting-the-tools)
* [Chapter 1 - Understanding the design flow](#chapter-1---understanding-the-design-flow)
* [Chapter 2 - DRC and LVS Fundamentals](#chapter-2---drc-and-lvs-fundamentals)
* [Chapter 3 - DRC Issues](#chapter-3---drc-issues)
* [Chapter 4 - LVS](#chapter-4---lvs)
* [Additional Content - OpenLANE Design Flow](#additional-content---openlane-design-flow)

---

## Chapter 0 - Getting the tools

To adequately utilize the open-source skywater130 pdk and understand the design flow, we first require to install all the tools, which are:
- open_pdk
- magic
- ngspice
- xschem
- netgen

*Note: open_pdk has to be installed last so it can correctly associate the xschem and magic directories.*  
*Note: if the configure step fails during any process, it is most likely due to missing additional packages, and they need to be installed to complete the installation.*

### Magic
Magic is an open-source VLSI layout tool.  
**Install steps:**
```bash
git clone git://opencircuitdesign.com/magic
cd magic
./configure
make
sudo make install

