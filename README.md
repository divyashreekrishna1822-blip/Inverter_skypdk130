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
```
More info can be found at [http://opencircuitdesign.com/magic/index.html](http://opencircuitdesign.com/magic/index.html)

### Netgen
Netgen is a tool for comparing netlists, a process known as LVS, which stands for "Layout vs. Schematic".  
**Install steps:**
```bash
git clone git://opencircuitdesign.com/netgen
cd netgen
./configure
make
sudo make install
```
More info can be found at [http://opencircuitdesign.com/netgen/index.html](http://opencircuitdesign.com/netgen/index.html)

### Xschem
Xschem is a schematic capture program.  
**Install steps:**
```bash
git clone https://github.com/StefanSchippers/xschem.git xschem_git
cd xschem_git
./configure
make
sudo make install
```
More info can be found at [http://repo.hu/projects/xschem/index.html](http://repo.hu/projects/xschem/index.html)

### Ngspice
ngspice is the open-source spice simulator for electric and electronic circuits.  
**Install steps:**
```bash
tar -zxvf ngspice-37.tar.gz
cd ngspice-37
mkdir release
cd release
../configure --with-x --with-readline=yes --disable-debug
make
sudo make install
```
Please note that to view the simulation graphs of ngspice, xterm is required and can be installed using:
```bash
sudo apt-get update
sudo apt-get install xterm
```
More info can be found at [https://ngspice.sourceforge.io/index.html](https://ngspice.sourceforge.io/index.html)

### open_pdk
Open_PDKs is distributed with files that support the Google/SkyWater sky130 open process description. Open_PDKs will set up an environment for using the SkyWater sky130 process with open-source EDA tools.  
**Install steps:**
```bash
git clone git://opencircuitdesign.com/open_pdks
cd open_pdks
./configure --enable-sky130-pdk
make
sudo make install
```

Now that we have all the necessary tools installed, let's understand the design flow!

---

## Chapter 1 - Understanding the design flow

### Verifying the open_pdk installation
An initial working directory can be made by copying the required setup files as follows:
```bash
mkdir -p inverter18/mag inverter18/netgen inverter18/xschem
cd inverter18/xschem
cp /usr/share/pdk/sky130A/libs.tech/xschem/xschemrc .
cp /usr/share/pdk/sky130A/libs.tech/ngspice/spinit .spiceinit
cd ../mag
cp /usr/share/pdk/sky130A/libs.tech/magic/sky130A.magicrc .magicrc
cd ../netgen
cp /usr/share/pdk/sky130A/libs.tech/netgen/sky130A_setup.tcl setup.tcl
```

#### Tool Verification Checks:

**Checking if Magic works:**
```bash
magic -T sky130A
```
![mag_test](Resources/Lab1/mag_test.png)

**Checking if Xschem works:**
```bash
xschem
```
![xschem_test](Resources/Lab1/xschem_test.png)

**Checking if Netgen works:**
```bash
netgen
```
![netgen_test](Resources/Lab1/netgen_test.png)

**Checking if Ngspice works:**
```bash
ngspice
```
![spice_test](Resources/Lab1/spice_test.png)

---

### Creating inverter schematic using Xschem
An initial schematic `inverter18.sch` is made by placing components from the `sky130_fd_pr` library:
- **PMOS (`XM2`)**: `sky130_fd_pr__pfet_01v8`, $W = 3\,\mu\text{m}$, $L = 0.18\,\mu\text{m}$, $nf = 3$
- **NMOS (`XM1`)**: `sky130_fd_pr__nfet_01v8`, $W = 4.5\,\mu\text{m}$, $L = 0.18\,\mu\text{m}$, $nf = 3$
- Input and output pin labels: `in`, `out`, `vdd`, `vss`

![xschem_inv](Resources/Lab1/xschem_inv.png)

### Creating the Inverter Symbol
Convert the schematic into a modular symbol `inverter18.sym` with matching pin names (`in`, `out`, `vdd`, `vss`).

![xschem_sym](Resources/Lab1/xschem_sym.png)

### Creating and simulating testbench Schematic
Using the symbol, an independent testbench `inverter18_tb.sch` was built to verify inverter operation.  
Transient simulation was configured with a pulse input on `in` and DC $1.8\,\text{V}$ on `vdd`.

![xschem_tb](Resources/Lab1/xschem_tb.png)

The circuit is simulated in Ngspice:
```spice
.control
tran 1n 1u
plot V(in) V(out)
.endc
```
The output confirms clean inverter switching behavior from $1.8\,\text{V}$ down to $0\,\text{V}$:

![xschem_sim](Resources/Lab1/xschem_sim.png)

---

### Creating Inverter Layout in Magic and exporting its netlist
The schematic netlist `inverter18.spice` is imported directly into Magic:
```tcl
readspice inverter18.spice
```
Magic automatically instantiates the parameterised transistor cells (`XM1` NMOS and `XM2` PMOS) and sets up the port pins.

![mag_import](Resources/Lab1/mag_import.png)

#### Layout Placement & Routing:
- PMOS placed on top connected to $V_{DD}$; NMOS placed at the bottom connected to $V_{SS}$.
- Input gates connected across fingers and routed to the **`in`** pin using `metal1`.
- Drain diffusions connected together and routed to the **`out`** pin using `metal1`.
- Top power rail ($V_{DD}$) and bottom ground rail ($V_{SS}$) routed across substrate/well taps.
- Layout design rule check verified with **`DRC = 0`**!

![mag_layout_clean](Resources/Lab1/mag_layout_clean.png)

---

### Performing LVS checks on testbench and layout netlists

The netlist of the completed layout is extracted in Magic TkCon:
```tcl
extract do local
extract all
ext2spice lvs
ext2spice
```

Intermediate `.ext` files can be cleaned up using:
```bash
rm *.ext
```

The schematic netlist and layout netlist are compared using LVS by Netgen:
```bash
netgen -batch lvs "../mag/inverter18.spice inverter18" "../xschem/inverter18.spice inverter18" /usr/share/pdk/sky130A/libs.tech/netgen/sky130A_setup.tcl lvs.log
```

#### Verification Result:
```text
Circuits match uniquely.
```
Netgen confirms that the layout perfectly matches the schematic netlist with zero mismatches, completing the physical verification!

![netgen_lvs](Resources/Lab1/lvs.png)

---

## Chapter 2 - DRC and LVS Fundamentals

### Design Rule Checking (DRC)
Make sure the design layout meets all silicon foundry geometric and electrical design rules for mask generation.

### Layout vs. Schematic (LVS)
Make sure the physical layout matches the electrical schematic connectivity and device parameters.
