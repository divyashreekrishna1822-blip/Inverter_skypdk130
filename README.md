# Physical Verification Using SKY130 PDK

This repository contains the documentation and design files for the physical verification workshop using SkyWater 130nm Technology and open-source EDA tools.

## Index
* [Chapter 0 - Getting the Tools](#chapter-0---getting-the-tools)
* [Chapter 1 - Understanding the Design Flow](#chapter-1---understanding-the-design-flow)
* [Chapter 2 - DRC and LVS Fundamentals](#chapter-2---drc-and-lvs-fundamentals)

---

## Chapter 0 - Getting the Tools

To utilize the open-source SkyWater 130nm PDK and design flow, the following tools were installed:
- **Magic** (VLSI Layout Tool)
- **Netgen** (LVS Comparison Tool)
- **Xschem** (Schematic Capture)
- **Ngspice** (SPICE Simulator)
- **open_pdks** (SkyWater 130nm PDK)

### 1. Magic
```bash
git clone git://opencircuitdesign.com/magic
cd magic
./configure
make
sudo make install
```

### 2. Netgen
```bash
git clone git://opencircuitdesign.com/netgen
cd netgen
./configure
make
sudo make install
```

### 3. Xschem (v3.4+)
```bash
sudo apt install -y flex bison libxaw7-dev libcairo2-dev
git clone https://github.com/StefanSchippers/xschem.git xschem_git
cd xschem_git
./configure
make -j$(nproc)
sudo make install
```

### 4. Ngspice
```bash
sudo apt update
sudo apt install -y ngspice libngspice0 xterm
```

### 5. open_pdks
```bash
git clone git://opencircuitdesign.com/open_pdks
cd open_pdks
./configure --enable-sky130-pdk
make
sudo make install
```

---

## Chapter 1 - Understanding the Design Flow

### 1. Setting Up the Working Directory
```bash
mkdir -p Lab1_inverter/{mag,netgen,xschem}
cd Lab1_inverter/xschem
ln -s /usr/share/pdk/sky130A/libs.tech/xschem/xschemrc
ln -s /usr/share/pdk/sky130A/libs.tech/ngspice/spinit .spiceinit
cd ../mag
ln -s /usr/share/pdk/sky130A/libs.tech/magic/sky130A.magicrc .magicrc
cd ../netgen
ln -s /usr/share/pdk/sky130A/libs.tech/netgen/sky130A_setup.tcl setup.tcl
```
<img width="1366" height="768" alt="Screenshot (75)" src="https://github.com/user-attachments/assets/230fe3a1-6813-4bff-b47d-8a3b7da71379" />

### 2. Inverter Schematic in Xschem
Created the CMOS inverter using primitive Sky130 devices:
- **NMOS:** `sky130_fd_pr/nfet_01v8.sym` ($W=4.5\,\mu\text{m}, L=0.18\,\mu\text{m}, \text{nf}=3$)
- **PMOS:** `sky130_fd_pr/pfet_01v8.sym` ($W=3.0\,\mu\text{m}, L=0.18\,\mu\text{m}, \text{nf}=3$)

<img width="1366" height="768" alt="Screenshot (95)" src="https://github.com/user-attachments/assets/eccac568-c583-48c8-8533-0080ac6e5114" />


### 3. Symbol Creation
Generated the `inverter2.sym` symbol from the schematic to instantiate in testbenches.

<img width="1366" height="768" alt="Screenshot (98)" src="https://github.com/user-attachments/assets/5931e4c2-03eb-4d71-ae9f-d7689e776470" />


### 4. Testbench Creation & Transient Simulation
Created `inverter2_tb.sch` with:
- DC supply voltage: $V_2 = 1.8\,\text{V}$
- Input pulse signal: $V_1 = \text{PWL}(0\text{ }0\text{ }20\text{n }0\text{ }900\text{n }1.8)$
- Model inclusion: `.lib /usr/share/pdk/sky130A/libs.tech/ngspice/sky130.lib.spice tt`
- Control script for transient simulation:
  ```spice
  .control
  tran 1n 1u
  plot V(in) V(out)
  .endc
  ```

<img width="1366" height="768" alt="Screenshot (104)" src="https://github.com/user-attachments/assets/7067bc16-282f-45c3-b6e4-99a3332b12b2" />


### 5. Inverter Layout in Magic
1. Placed primitive devices using Magic's **Devices 1** menu.
2. Routed polysilicon gate and metal1 drains and power rails (`VDD`, `GND`).
3. Added port labels for `in`, `out`, `vdd`, `vss`.

![Inverter Layout](Resources/Lab1/mag_p1.png)

### 6. Netlist Extraction & LVS with Netgen
Extracted the layout netlist in Magic:
```tcl
extract do local
extract all
ext2spice lvs
ext2spice
```
Ran LVS verification comparing schematic and layout:
```bash
netgen -batch lvs "../mag/inverter2.spice inverter2" "../xschem/inverter2.spice inverter2"
```

unfortunately due to errors in saving the files and downloading aaand importing the spice files to magic i failed
