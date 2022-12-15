# OpenFASoC_Flow_Routing
## What is OpenFASoC
FASoC stands for Fully Autonomous System-on-Chip

The FASoC Program is focused on developing a complete system-on-chip (SoC) synthesis tool from user specification to GDSII. FASoC leverages a differentiating technology to automatically synthesize “correct-by-construction” Verilog descriptions for both analog and digital circuits and enable a portable, single pass implementation flow. The SoC synthesis tool realizes analog circuits, including PLLs, power management, ADCs, and sensor interfaces by recasting them as structures composed largely of digital components while maintaining analog performance. They are then expressed as synthesizable Verilog blocks composed of digital standard cells augmented with a few auxiliary cells generated with an automatic cell generation tool. By expanding the IPXACT format and the Socrates tool from ARM, we then enable composition of vast numbers of digital and analog components into a single correct-by-construction design. This project is led by a team of researchers at the Universities of Michigan, Virginia, and Arm.
Tool installation steps can be found here:

https://openfasoc.readthedocs.io/en/latest/getting-started.html#installation

You can either follow the express installation or the manual installation steps.

The following tools will be required

1. OpenRoad
2. Magic
3. Yosys
4. Klayout
5. Open_PDK
6. Netgen

## Installation

### OpenFASOC:
The command used to install OpenFASOC are 
```
git clone https://github.com/idea-fasoc/openfasoc
cd openfasoc
pip install -r requirements.txt
```
For the complete steps of installing OpenFASOC, refer Manual Installation from [here](https://github.com/idea-fasoc/OpenFASOC/blob/main/docs/source/getting-started.rst). 

### OpenROAD: 
OpenROAD is an integrated chip physical design tool that takes a design from synthesized Verilog to routed layout. OpenROAD uses the OpenDB database and OpenSTA for static timing analysis. Documentation is also available [here](https://openroad.readthedocs.io/en/latest/main/README.html).


The commands to install OpenROAD are,
```
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD.git
cd OpenROAD
./etc/DependencyInstaller.sh
./etc/DependencyInstaller.sh -run
./etc/DependencyInstaller.sh -dev
mkdir build
cd build
cmake ..
make
```

### Klayout
Downlaod the latest version of the Klayout from [here](https://www.klayout.de/build.html). Install the following dependencies: qt5-default, qttools5-dev, libqt5xmlpatterns5-dev, qtmultimedia5-dev, libqt5multimediawidgets5 and libqt5svg5-dev.
```
sudo apt-get install -y libqt5widgets5
sudo dpkg -i klayout_0.27.11-1_amd64.deb
```

### Netgen
To install Netgen, 
```
sudo add-apt-repository ppa:ngsolve/ngsolve
sudo apt-get update
sudo apt-get install ngsolve
```

### Yosys
The software used to run gate level synthesis is Yosys. Yosys is a framework for Verilog RTL synthesis. It currently has extensive Verilog-2005 support and provides a basic set of synthesis algorithms for various application domains. Yosys can be adapted to perform any synthesis job by combining the existing passes (algorithms) using synthesis scripts and adding additional passes as needed by extending the Yosys C++ code base.


To install yosys, install the prerequisites using the following command 
 ```
 sudo apt-get install build-essential clang bison flex \
	libreadline-dev gawk tcl-dev libffi-dev git \
	graphviz xdot pkg-config python3 libboost-system-dev \
	libboost-python-dev libboost-filesystem-dev zlib1g-dev
```
To install latest Version of Yosys, 
```
git clone https://github.com/YosysHQ/yosys.git
make
sudo make install 
make test
```

### Magic
Run following commands one by one to fulfill the system requirement.
#### Prerequisites for magic
```
sudo apt-get install m4
sudo apt-get install tcsh
sudo apt-get install csh
sudo apt-get install libx11-dev
sudo apt-get install tcl-dev tk-dev
sudo apt-get install libcairo2-dev
sudo apt-get install mesa-common-dev libglu1-mesa-dev
sudo apt-get install libncurses-dev
```
#### Install magic
```
git clone https://github.com/RTimothyEdwards/magic
cd magic/
./configure
sudo make
sudo make install
```
type `magic` terminal to check whether it installed succesfully or not. Type `exit` to exit magic.

## Understanding OpenFASoC Flow

We have considered the Temp-Sense Generator in building the schematic for the flow.

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207885968-f8f6b824-6614-451d-a266-cffd0fb73887.png" width="500" alt="accessibility text">
</p>
<p align="center">
    <em>OpenFASoC Flow</em>
</p>

The OpenFASOC Design flow starts by taking the design specifications in the form of `.json` format. Then the OpenFASOC generator determines the number of auxillary cell to be added to optimize the design. The generator also uses the model file to automatically determine the number of aux-cells to be added based on user-specifications. Here, the model file is saved in the form of a `.csv` file.

In other words, the generator iteratively searches the the model file to find the optimum number of auxillary cells to be added. After finding the optimum structure, the behavioral verilog description is created. Then we use `yosys` to map behavioral code to gate level code. This is then followed by PNR. Place and Route is performed by OpenROAD tool. To further ensure the authenticity of the design OpenRoad does DRC, LVS AND PEX checks. Unlike traditional Analog-Design flow, in OpenFASOC flow, the complete process of aux-cell generation to layout generation is automated which significantly reduced the design time. Hence this process is also called Automatic Place and Route.



<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207887250-5a311351-d3cc-4d44-938d-cbc59c3b24c3.png" width="700" alt="accessibility text">
</p>


Since OpenROAD was developed with digital designs in mind, some features do not natively support analog or mixed-signal designs for now. Hence, in the temperature sensor, the physical implementation does not get successfully generated with the original flow.

Some changes are then made to customize the OpenROAD Flow repo and generate a working physical design, summarized in the diagram  given above.


## Understanding Routing
<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207877996-6789179e-0ee9-4689-98a3-98f1f38087c6.png" width="600" alt="accessibility text">
</p>


Routing is the penultimate step in the flow wherein we connect the components. During routing, wire-like pathways known as traces are placed in the design.
Routing is also divided into two phases: global routing and detailed routing. Right before global routing, OpenFASoC calls `pre_global_route.tcl`.
This script sources two other files: `create_routable_power_net.tcl`, which adds an NDR rule to the VIN net to improve routes that connect both voltage domains,
and `create_custom_connections.tcl`, which creates the connection between the VIN net and the HEADER instances.

APR is run using the `temp-sense-gen.py` python script, present at the following location: [temp-sense-gen.py](https://github.com/idea-fasoc/OpenFASOC/blob/main/openfasoc/generators/temp-sense-gen/tools/temp-sense-gen.py)
<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/200106954-b0239209-4b73-4469-94b2-59d1cce4a043.png" width="500" alt="accessibility text">
</p>
<p align="center">
    <em>Automatic Placement and Routing in .py script</em>
</p>

At the end, OpenROAD Flow will output its logs under `flow/reports/`, and its results under `flow/results/`.
Note: All the script files are found in [temp-sense-gen/flow/scripts](https://github.com/idea-fasoc/OpenFASOC/tree/main/openfasoc/generators/temp-sense-gen/flow/scripts).
### Global Routing
First we load the files `4_cts.odb` and `4_cts.sdc`. With this information pre-global routing is initiated. In this stage routable power nets are added with the `create_routable_power_net.tcl` file and creates connections custom to the temp-sense-gen design using `create_custom_connections.tcl` file.

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/200106958-8dc4f0e1-7d1f-467b-9499-5847ab18ae8e.png" width="500" alt="accessibility text">
</p>
<p align="center">
    <em>Global pre-routing</em>
</p>
Global-routing is carried out using the route.guide which has matrices for every pin specification in the design.

In the next step all the report_checks are carried out such as **report_tns**, **report_wns** and all the results are stored in the file `5_1_grt.odb`.

### Detailed Routing
The input files are `4_cts.odb` and `5_1_grt.odb` which is acquired after performing global routing.
The final output is dumped in the files `5_route.sdc` and `5_2_route.odb`.

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/199914547-86820bc8-97d6-453b-bc59-a03ef4cf1943.png" width="500" alt="accessibility text">
</p>
<p align="center">
    <em>Final Layout After Routing</em>
</p>

## Auxillary Cells

### Temp-Sense Generation

The physical implementation of the analog blocks in the circuit is done using two manually designed standard cells (auxillary cells):
- HEADER cell, containing the transistors in subthreshold operation;
- SLC cell, containing the Split-Control Level Converter.

The gds and lef files of HEADER and SLC cells are pre-created before the start of the Generator flow. The aux cells are generated by writing a spice netlist and the outputs are taken with the model files that contain the temperature requirements for the functioning of the cell.

#### Header Cell Layout

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207873270-8482bc02-1249-4be2-8d87-bd6e54f9c4b5.png" width="700" alt="accessibility text">
</p>

#### SLC Cell Layout

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207873312-f6f962bd-0d8e-4d5e-90d0-b3d04f48f6ea.png" width="700" alt="accessibility text">
</p>

### Phase Locked Loop Generation

A Phase Locked Loop (PLL) consists of

- Phase Detector
- Charge Pump
- Voltage Controlled Oscillator
- Frequency Divider

The PLL tries to match the frequency with the given input signal. Here the Charge Pump and the VCO are the auxillary cells whereas the Frequency Divider and the Phase Detector are the digital blocks.
These aux cells are generated using ALIGN tool. This tools gives us the gds and lefs file which are integrated into the flow of the design.
The gds and lef files of HEADER and SLC cells are pre-created before the start of the Generator flow.

#### Charged Pump Cell Layout

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207874885-728612d8-b7ae-44ba-acef-26508f352eef.png" width="700" alt="accessibility text">
</p>


#### Voltage Controlled Cell Layout

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207874918-885c10e3-da96-4241-983c-b9f7f8d3d7cd.png" width="700" alt="accessibility text">
</p>


## Routing using OpenFASOC Flow

### Temperature Sensor Generation

#### Global Routing Area and Power

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207872071-9791ae26-4e30-49b1-9d04-e85b64bb4537.png" width="550" alt="accessibility text">
</p>

#### Global Route Timing Reports

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207872101-ca1097a8-a02e-4f0c-99c6-cded8810169d.png" width="500" alt="accessibility text">
</p>

#### Finished Power and Area Report

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207872111-ed16e485-11be-4d16-ae74-088850fa4049.png" width="550" alt="accessibility text">
</p>

#### Final Routed Design

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207872764-c1a770ab-1ff2-41ce-b665-c6cd2bd97767.png" width="700" alt="accessibility text">


### PLL Generator

#### Global Routing Area and Power

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207875063-4ea8a713-9c0a-4349-9a17-5b996ca4e5b6.png" width="550" alt="accessibility text">
</p>


#### Global Route Timing Reports

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207875121-82dd3163-47c0-4145-8142-589c925a2508.png" width="500" alt="accessibility text">
</p>


#### Final Routed Design

<p align="center">
 <img src="https://user-images.githubusercontent.com/78084271/207875293-dec497bc-5dd0-4bdd-a270-aee7e78a5a72.png" width="700" alt="accessibility text">
 
 ## Future Works


- The errors faced during routing are bypassed by not creating crucial components in the flow such as creating a routable net for VIN, removing path to `create_routable_power_net.tcl` which results in incorrect routing of the macro cells which can be seen in the aux cell placement in the final.gds layout. Further rectification is required.
- Crashing during the routing stage, which is bypassed by removing the path to create_routable_power_net.tcl. The cause of the crash is yet to be rectified.
- Warnings are generated in routing for missing connections. Need to verify whether this is due to the errors bypassed in the routing or any new issue.
- Model File updation for more cases is required to achieve the pll generation for different input requirements that achieves best performance.
- The gds files generated from the ALIGN tool gives us error during placement due to undefined metal layers (M3, M2) as they must be defined in a specific manner. This mismatch must be further looked at.

## Author

- **Anmol Shetty**

## Acknowledgments

- Kunal Ghosh, Director, VSD Corp. Pvt. Ltd.
- Madhav Rao, Associate Professor, IIIT Bangalore
- Tejas BN, MTech Student, IIIT Bangalore
- Mahati Basavaraju, MTech Student, IIIT Bangalore
- Vinay Rayapati, MTech Student, IIIT Bangalore



