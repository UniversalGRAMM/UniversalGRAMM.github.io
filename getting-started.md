---
title: Getting started
layout: default
nav_order: 2
description: "Getting started guide for UGRAMM"
permalink: /getting-started
---

# Table of Contents

- [Getting Started](#getting-started)
- [Demo](#demo)
  - [Running UGRAMM](#running-ugramm)

---

# Getting started:

- Clone the UGRAMM repository:
    - `git clone https://github.com/UniversalGRAMM/UGRAMM.git`
- Compile the source code:
    - `make`
- Above step will produce a `UGRAMM` binary file in the root directory.

```
❯ ./UGRAMM --help
[UGRAMM] allowed options =:
  -h [ --help ]         Print help messages
  --seed arg            Seed for the run
  --verbose_level arg   0: info, 1: debug, 2: trace
  --dfile arg           Device model file
  --afile arg           Application graph file
  --config arg          UGRAMM config file
```

- UGRAMM primarily accepts three types of files:
    - **afile** [REQUIRED]: An application-graph file in Graphviz dot format [(ref)](https://graphviz.org/doc/info/lang.html). This file describes the input application to be mapped onto the device model (mapping here refers to checking if this smaller graph is a minor of the larger device-model graph).
    - **dfile** [REQUIRED]: A device-model file in Graphviz dot format. This file represents the device model of the hardware, and the tool checks if the application graph (H) is a minor of this device model graph (G).
    - **config** [OPTIONAL]: An optional configuration file that can be used in scenarios such as skipping the placement of specific nodes or locking certain nodes placement.

- We have created a helper script (`scripts/device_model_gen.py`) to generate device model files for CGRA architectures such as RIKEN and ADRES.
- The UGRAMM repository includes a variety of benchmarks for CGRA applications in the `Kernels/` folder, featuring Stencil, Convolution, and FFT benchmarks.
- While UGRAMM is tested for the CGRA mapping process, it can also be used with other applications, provided that users ensure the input to GRAMM is in a format compatible with UGRAMM, as specified in this documentation.
    - We have designed UGRAMM with generalized features to accommodate a wide range of applications. However, if you need specific functionality or have any questions, please create an issue related to your requirements on the UGRAMM GitHub repository.

---

# Demo:

- A helper script (`run_ugramm.sh`) has been developed to simplify running UGRAMM. The operations of this script are described below. You can run UGRAMM using this script or directly.

> ./run_ugramm.sh $1 $2 $3 $4 $5

- Argument information about this script:
    - $1 [Seed] = 0,1,15 etc...
    - $2 [NR] = 8 //Number of rows in the CGRA architecture
    - $3 [NC] = 8 //Number of columns in the CGRA architecture
    - $4 [ApplicationGraph] = Kernels/Conv_Balance/conv_nounroll_Balance.dot 
    - $5 [grammConfigFile]  = config.json //Config file for the UGRAMM 

- This helper script will generate a specific CGRA device model file based on user input; Compile, and run UGRAMM with the generated device model file as input. It will also convert the output of UGRAMM into PNG format for easier debugging and visualization.
  - To generate the device model using the external script:
      ```
      cd scripts && ./device_model_gen.py -NR $2 -NC $3 -Arch RIKEN && cd ..
      ```
  - To execute UGRAMM and produce the mapping result in `mapping_output.dot`:
      ```
      make && ./UGRAMM --seed $1 --verbose_level 0 --dfile $device_model_output --afile $4 --config $5
      ```
  - Finally, to convert the mapped output DOT files into PNG format:
      ```
      neato -Tpng positioned_dot_output.dot -o positioned_dot_output.png
      dot -Tpng unpositioned_dot_output.dot -o unpositioned_dot_output.png
      ```
  - The successful mapping results will be saved in `unpositioned_dot_output.png` and `positioned_dot_output.png`.

## Running UGRAMM:

> ./run_ugramm.sh 15 8 8 Kernels/Conv_Balance/conv_nounroll_Balance.dot config.json

- After running above command the mapped output results should be present in the root directory of UGRAMM.

- **positioned_dot_output**: 
    - This dot-file contains user provided co-ordinates of the device-model node cells.
<div style="text-align: center;">
    <img src="assets/positioned_dot_output.png" alt="Fig 1. positioned_dot_output" style="border: 1px solid black; width: 400px;">
    <figcaption style="font-size: 14px; color: #555;">Fig 1. positioned_dot_output</figcaption>
</div>

- **unpositioned_dot_output**: 
    - This dot-file does not contains user provided co-ordinates of the device-model node cells. 
    - This file also displays the input application graph for which UGRAMM was executed.
<div style="text-align: center;">
    <img src="assets/unpositioned_dot_output.png" alt="Fig 2. unpositioned_dot_output" style="border: 1px solid black; width: 600px;">
    <figcaption style="font-size: 14px; color: #555;">Fig 2. unpositioned_dot_output</figcaption>
</div>
