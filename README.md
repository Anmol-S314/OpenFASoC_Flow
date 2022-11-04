# OpenFASoC_Flow
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

Note: Miniconda installation might help in easy installation

Make sure to download and install all the required dependencies as mentiioned in the link above.

If you are facing issues in installating Klayout, follow below steps:

1. Go to the below mentioned website and install the suitable klayout version package for your OS https://www.klayout.de/build.html.
2. Follow one of the below for installation.

```
sudo dpkg -i <packagename>.deb
cd klayout-*
./build.sh
```
If `./build.sh` doesn't work, try `sudo ./build.sh` or `sudo bash ./build.sh`

OR
```
sudo apt-get install <packagename>.deb
```
## Understanding Routing
Routing is the penultimate step in the flow wherein we connect the components. During routing, wire-like pathways known as traces are placed in the design.
Routing is also divided into two phases: global routing and detailed routing. Right before global routing, OpenFASoC calls `pre_global_route.tcl`.
This script sources two other files: `create_routable_power_net.tcl`, which adds an NDR rule to the VIN net to improve routes that connect both voltage domains,
and `create_custom_connections.tcl`, which creates the connection between the VIN net and the HEADER instances.

At the end, OpenROAD Flow will output its logs under `flow/reports/`, and its results under `flow/results/`.
Note: All the script files are found in `temp-sense-gen/flow/scripts`.
### Global Routing
First we load the files `4_cts.odb` and `4_cts.sdc`. With this information pre-global routing is initiated. In this stage rotable power nets are added with the `create_routable_power_net.tcl` file and creates connections custom to the temp-sense-gen design using `create_custom_connections.tcl` file.

Global-routing is carried out using the `route.guide` which has matrices for every pin specification in the design.

In the next step all the report_checks are carried out such as **report_tns**, **report_wns** and all the results are stored in the file `5_1_grt.odb`.

### Detailed Routing
The input files are `4_cts.odb` and `5_1_grt.odb` which is acquired after performing global routing.
The final output is dumped in the files `5_route.sdc` and `5_2_route.odb`.
