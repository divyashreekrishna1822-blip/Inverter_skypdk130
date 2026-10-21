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
To view the simulation graphs of ngspice, xterm is required and can be installed using:
```bash
sudo apt-get update
sudo apt-get install xterm
```

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

Now that I have all the necessary tools installed, 

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
<img width="1366" height="768" alt="Screenshot (118)" src="https://github.com/user-attachments/assets/e3a18b52-1fc5-4f48-a077-734a9edec13c" />


**Checking if Xschem works:**
```bash
xschem
```
<img width="1366" height="768" alt="Screenshot (124)" src="https://github.com/user-attachments/assets/72460c2d-d2ec-485f-8aa1-63ba7c510e55" />


**Checking if Netgen works:**
```bash
netgen
```
<img width="1366" height="768" alt="Screenshot (120)" src="https://github.com/user-attachments/assets/9c4f4749-4f3e-4c73-ba2c-43e88784155d" />


**Checking if Ngspice works:**
```bash
ngspice
```


---

### Creating inverter schematic using Xschem
An initial schematic `inverter18.sch` is made by placing components from the `sky130_fd_pr` library:
- **PMOS (`XM2`)**: `sky130_fd_pr__pfet_01v8`, $W = 3\,\mu\text{m}$, $L = 0.18\,\mu\text{m}$, $nf = 3$
- **NMOS (`XM1`)**: `sky130_fd_pr__nfet_01v8`, $W = 4.5\,\mu\text{m}$, $L = 0.18\,\mu\text{m}$, $nf = 3$
- Input and output pin labels: `in`, `out`, `vdd`, `vss`

1)applying the nmosfet values
<img width="1366" height="768" alt="Screenshot (125)" src="https://github.com/user-attachments/assets/064054e0-5202-4def-8d23-ee489ed01d1d" />
2)applying the pmosfet values
<img width="1366" height="768" alt="Screenshot (126)" src="https://github.com/user-attachments/assets/7bdcdcba-5f78-4320-9935-da71773ee291" />
3)Final circuit 
<img width="1366" height="768" alt="Screenshot (128)" src="https://github.com/user-attachments/assets/22d27340-2b2f-4eb7-ada0-cb83b7c7d8be" />



### Creating the Inverter Symbol
Convert the schematic into a modular symbol `inverter18.sym` with matching pin names (`in`, `out`, `vdd`, `vss`).

<img width="1366" height="768" alt="Screenshot (133)" src="https://github.com/user-attachments/assets/602b647d-6af7-4439-bc30-6f3f7a9ef021" />

### Creating and simulating testbench Schematic
Using the symbol, an independent testbench `inverter18_tb.sch` was built to verify inverter operation.  
Transient simulation was configured with a pulse input on `in` and DC $1.8\,\text{V}$ on `vdd`.

After successfully creating the schematic and applying the required changes for the cicuit i generated the netlist and simulation the resulted transition waveform as shown below
<img width="1366" height="768" alt="Screenshot (134)" src="https://github.com/user-attachments/assets/a1d2d906-885d-44a0-bf80-db62bf8467c4" />




The circuit is simulated in Ngspice:
```spice
.control
tran 1n 1u
plot V(in) V(out)
.endc
```
The output confirms clean inverter switching behavior from $1.8\,\text{V}$ down to $0\,\text{V}$:

<img width="1366" height="768" alt="Screenshot (135)" src="https://github.com/user-attachments/assets/89d8abc6-7246-4ebf-9763-edb9912d2e55" />


---

### Creating Inverter Layout in Magic and exporting its netlist
The schematic netlist `inverter18.spice` is imported directly into Magic:
```tcl
readspice inverter18.spice
```
Magic automatically instantiates the parameterised transistor cells (`XM1` NMOS and `XM2` PMOS) and sets up the port pins.

1)The imported spice file looks like below with improper placement of the cells
<img width="1366" height="768" alt="Screenshot (139)" src="https://github.com/user-attachments/assets/62924217-7bd8-45a4-858f-7b003c04cfee" />
2)After the correct position replacement it looked like this
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/e864bf66-21a0-445f-ac5b-27ecedfd70a4" />



#### Layout Placement & Routing:
- PMOS placed on top connected to $V_{DD}$; NMOS placed at the bottom connected to $V_{SS}$.
- Input gates connected across fingers and routed to the **`in`** pin using `metal1`.
- Drain diffusions connected together and routed to the **`out`** pin using `metal1`.
- Top power rail ($V_{DD}$) and bottom ground rail ($V_{SS}$) routed across substrate/well taps.
- Layout design rule check verified with **`DRC = 5`**!
- <img width="1366" height="768" alt="Screenshot (147)" src="https://github.com/user-attachments/assets/2c4c521d-fb6f-41ca-a509-ca7564afa0a3" />
<img width="1366" height="768" alt="Screenshot (155)" src="https://github.com/user-attachments/assets/cfd80d22-70c6-447d-92b1-baa3740c319a" />
 3)Observation : The crosslines in the magic window in the pmos and nmos present because in ubuntu the magic -d XR and magic -d OGL command is executing in my laptop
  As soon as i enter the  magic -d XR  and enters the magic window pops and closes automatically and same happend for the magic -d OGL for this command



---

### Performing LVS checks on testbench and layout netlists

The netlist of the completed layout is extracted in Magic TkCon:
```tcl
extract do local
extract all
ext2spice lvs
ext2spice
```
<img width="1366" height="768" alt="Screenshot (159)" src="https://github.com/user-attachments/assets/b42abda4-f9e9-4c12-b61b-8c60b19b5eeb" />

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
Final result is not came as per my expectation
```
Netgen confirms that the layout Top level cell failed pin matching.

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/2e279750-c413-40af-97f5-18a2611f109f" />


---
## Chapter 3 - DRC Issues & Debugging Analysis

During the layout execution of `inverter18`, several practical DRC challenges were encountered and analyzed:

### 1. Initial High DRC Count (`DRC=20` to `DRC=45`)
- **Cause**: Upon running `readspice`, the 4 default pin squares (`vdd`, `in`, `out`, `vss`) were placed at fixed default coordinates $(0, -400, -800, -1200)$, overlapping the active diffusion of `XM1`.
- **Accidental Layer Painting**: When switching to `Tool: wiring`, clicking near contacts inadvertently placed `locali` (local interconnect) and contact cuts across the PMOS N-well boundary, triggering spacing and enclosure violations.
- **Resolution**:
  - Replaced wiring tool routing with precise Box Tool commands (`paint metal1`).
  - Cleared erroneous `locali` and `nwell` patches using `erase locali` and `erase nwell`.
  - DRC count successfully dropped from 45 down to 5.

### 2. Remaining Violations (`DRC=5`)
- **Analysis**: Running `drc why` highlighted gate-to-tap spacing and contact minimum enclosure rules at the bottom of the NMOS device.
- **Root Cause**: The ground rail ($V_{SS}$) was painted slightly below the NMOS substrate tap, leaving the tap contacts without complete `metal1` coverage.
- **Correction Required**: Extending the $V_{SS}$ and $V_{DD}$ `metal1` boxes to fully overlap the tap contact strips eliminates these 5 violations and brings the layout to **`DRC = 0`**.

---

## Chapter 4 - LVS Analysis & Failure Debugging

After layout extraction using Magic:
```tcl
extract do local
extract all
ext2spice lvs
ext2spice
```

Netgen was executed to compare layout against schematic:
```bash
netgen -batch lvs "../mag/inverter18.spice inverter18" "../xschem/inverter18.spice inverter18" /usr/share/pdk/sky130A/libs.tech/netgen/sky130A_setup.tcl lvs.log
```

### Netgen LVS Output Summary:
```text
Contents of circuit 1: Circuit: 'inverter18' (Layout)
Circuit inverter18 contains 2 device instances.
  Class: sky130_fd_pr__pfet_01v8 instances: 1
  Class: sky130_fd_pr__nfet_01v8 instances: 1
Circuit contains 4 nets.

Contents of circuit 2: Circuit: 'inverter18' (Schematic)
Circuit inverter18 contains 2 device instances.
  Class: sky130_fd_pr__pfet_01v8 instances: 1
  Class: sky130_fd_pr__nfet_01v8 instances: 1
Circuit contains 4 nets.

Circuit 1 contains 2 devices, Circuit 2 contains 2 devices.
Circuit 1 contains 4 nets,   Circuit 2 contains 4 nets.

Final result:
Top level cell failed pin matching.
```

### Root Cause Analysis (Why Pin Matching Failed):

Inspection of the detailed comparison log `comp.out` revealed three critical discrepancies:

```text
Subcircuit pins:
Circuit 1: inverter18 (Layout)       | Circuit 2: inverter18 (Schematic)
-------------------------------------|--------------------------------------
vss                                  | vdd **Mismatch**
in                                   | in
(no pin, node is vss)                | out
(no pin, node is vss)                | vss
----------------------------------------------------------------------------
```

1. **Net Shorting Between `out` and `vss` in Layout:**
   - In Circuit 1 (Layout), `out` was not extracted as an independent port. Instead, Netgen reported `(no pin, node is vss)`.
   - **Reason**: The drain `metal1` connection extending towards the `out` pin accidentally touched the bottom $V_{SS}$ ground rail. This merged the output and ground nodes into a single physical net.
2. **Substrate & Well Isolation (`VSUBS` and `XM2/B`):**
   - In Circuit 1, Netgen extracted `VSUBS` (substrate) and `XM2/B` (N-well bulk) as unassigned nets rather than connecting them to `vss` and `vdd`.
   - **Reason**: The power rails were drawn adjacent to, rather than directly overlapping, the tap contact strips. Consequently, Magic extracted the bulk nodes as floating/isolated bodies.
3. **Port Ordering Discrepancy:**
   - Because `out` was shorted into `vss`, Netgen could not match the 4 distinct terminal ports of the schematic (`vdd`, `in`, `out`, `vss`), resulting in **`Top level cell failed pin matching.`**

### Required Corrections for Full LVS Match:
1. **Separate Output from Ground Rail**: Break the contact between the drain wire and the ground rail so `out` is isolated.
2. **Enclose Well/Substrate Taps**: Extend the top $V_{DD}$ `metal1` rail across the PMOS N-well tap contacts, and the bottom $V_{SS}$ `metal1` rail across the NMOS P-substrate tap contacts.
3. **Re-extract & Verify**:
   ```tcl
   extract do local
   extract all
   ext2spice lvs
   ext2spice
   ```
   Re-running Netgen will resolve the port partition and yield the clean target result:
   ```text
   Circuits match uniquely.
   ```


