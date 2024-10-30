---
title: DRC
layout: default
nav_order: 6
description: "DRC guide for UGRAMM"
permalink: /UGRAMM-Documentations/DRC
---

# DRC:

The DRC is an extension in the UGRAMM mapper that pre-verifies all input files, including the application DFG, device model graph, and config files, for syntax and logical errors. Essentially, the DRC assesses whether the inputs are compatible with the requirements for graph minor mapping. If it detects any issues that would prevent successful mapping or program execution, UGRAMM exits early and provides detailed error messages, saving time by flagging issues before the UGRAMM mapper runs fully.

By default, UGRAMM includes a set of general DRC rule checks to ensure the mapper operates correctly. Additional custom rules can be added to the DRC depending on specific use cases.

---
## DRC Terminology and Code Structure Flow

In UGRAMM, both the application graph (`H`) and the device model graph (`G`) are represented as directed graphs. These graphs are constructed and managed within the mapper using the Boost Graph Library (BGL), which provides efficient data structures and algorithms for handling graph-based operations. This enables UGRAMM to perform complex mapping and traversal tasks across both graphs efficiently.

UGRAMM also utilizes two key data structures, `hConfig` and `gConfig`, which are maps storing node configuration details for the application graph and device model graph, respectively. In these maps, the key represents the node ID of the respective graph, and the value is a `NodeConfig` structure containing the configuration parameters and attributes associated with each node. A comprehensive list of configuration parameters and attributes can be found within the `NodeConfig` structure defined in `UGRAMM.h`.

In all DRC rule-check functions, a pointer is provided for either the application graph (`H`) or the device model graph (`G`), along with a pointer to either `hConfig` or `gConfig`, based on the specific rule’s scope. For instance, if a rule check targets only the application graph, the function will include pointers to the `H` graph and the `hConfig` map. Similarly, if a rule pertains solely to the device model graph, the function will include pointers to the `G` graph and the `gConfig` map.

When a rule applies to both the application graph and the device model graph, the function will need pointers to all structures — `H`, `G`, `hConfig`, and `gConfig` — to ensure that all relevant data is available for a comprehensive check.

Additionally, each DRC function includes a pointer to a boolean value called errorDetected. This pointer serves to flag any detected errors and will signal for an early exit after DRC execution is complete. Note that DRC will only terminate the execution after all DRC rules have been processed, allowing all potential errors to be printed simultaneously, thereby providing comprehensive feedback in a single execution.

---
## Default DRC Rules

GRAMM includes a set of default DRC rule checks. These checks ensure that the UGRAMM mapper can operate correctly with the provided input application graph, device model graph, and config files, verifying compatibility and minimizing the risk of runtime errors during mapping. Below is a list of the general DRC rules currently supported by UGRAMM:

| **Rule Name**                    | **Graph**          | **Rule Description**                                                                                                                                                                                                                                                                                                                                     |
|----------------------------------|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Verify Pin Nodes                 | Device Model Graph | Check that each input PinCell node in the application graph has one fanout edge connected to a funcCell, and each output PinCell node in the device model graph has one fanin edge also connected to a funcCell.                                                                                                                                         |
| Check Floating Nodes             | Device Model Graph | Check that all nodes in the device model graph are connected, flagging any floating node without connections as a critical error since it will not be accessed during the mapping process.                                                                                                                                                               |
| Check Graph is Weakly Connected  | Device Model Graph | Check that the device model graph is weakly connected, meaning all nodes should be reachable as undirected, and flags any graph with two or more disconnected subgraphs as a critical error, since it would prevent the mapping process from establishing connections across these disjointed sections and disrupt the application's functionality.      |
| Check FuncCell Connectivity      | Device Model Graph | Check that all funcCells in the device model graph are accessible from other funcCells, triggering a warning for any that are unreachable. Do acknowledge that some may be intentionally isolated due to design constraints, resulting in an informational warning rather than a critical error.                                                         |
| Check Device Model Attributes    | Device Model Graph | Check that all mandatory attributes in the device model graph DOT file are correctly instantiated in the G graph, flagging any missing or incorrectly defined attributes to prevent compatibility issues during the mapping process.                                                                                                                     |
| Check Floating Nodes             | Application DFG    | Check that all nodes in the application graph are connected, flagging any floating node without connections as a critical error since it will not be accessed during the mapping process.                                                                                                                                                                |
| Check Pin Names                  | Application DFG    | Check that both the driver and load pin names are included in the pragma, either as aliases or supported pins, ensuring that the referenced pins in the configuration are valid and recognized by the device model graph.                                                                                                                                |
| Check Graph is Weakly Connected  | Application DFG    | Check that the application graph is weakly connected, meaning all nodes should be reachable as undirected, and flags any graph with two or more disconnected subgraphs as a critical error, as this would prevent the mapping process from establishing connections across disjointed sections and potentially disrupt the application's functionality.  |
| Check Device Model Attributes    | Application DFG    | Check that all mandatory attributes in the application graph DOT file are correctly instantiated in the H graph, flagging any missing or incorrectly defined attributes to prevent compatibility issues during the mapping process.                                                                                                                      |
| Check Dupplication In Lock Nodes | Application DFG    | Check if the configuration file specifies two application graph nodes locked to the same device model node in the device model graph, flagging it as an error since each device model node should only be mapped to a single application graph node unless otherwise specified.                                                                          |
| Check Lock Node Type             | Application DFG    | Check that the locked device model node has the required capabilities and attributes to support the mapped application graph node, flagging it as an error if the device model node lacks the necessary functionality or configuration, as this incompatibility would prevent correct mapping and execution.                                             |


---
## Adding Custom DRC Rules

To tailor UGRAMM to specific user needs, users can add custom DRC (Design Rule Check) rules based on their use cases. Adding these custom rules can enhance UGRAMM's versatility and applicability. Here are the steps needed to create a custom DRC rule:

1. Declare your DRC rule function in the `drc.h` header file. For consistency and readability, place your DRC function declaration in the appropriate section under either the application graph or device model graph, as relevant to your function’s purpose. Before proceeding, please refer to the `DRC Terminology and Code Structure Flow` section above for guidance on the required and allowed variables in each DRC function parameters, ensuring your function parameters align with the UGRAMM standards.
2. Write the logic of DRC rule function in `drc.cpp` file. For consistency and readability, place your DRC function declaration in the appropriate section under either the application graph or device model graph, as relevant to your function’s purpose.
3. The DRC files utilize the [SPDLogger](https://github.com/gabime/spdlog) library to print statements to the console with different verbosity levels based on message importance. Here’s how each level works:
    - Info Messages (drcLogger->info("")): Use for general information or status updates that don't indicate any issues.
    - Warning Messages (drcLogger->warn("")): Use for messages that indicate potential issues or important notices that don’t halt execution.
    - Error Messages (drcLogger->error("")): Use for critical messages where execution needs to halt due to serious issues.
Select the appropriate verbosity level based on the severity of each message to provide clear and effective feedback during DRC operations.
4. To indicate an error, please raise the *errorDetected flag. This informs UGRAMM that one of the DRC rule checks has failed, prompting the program to exit early. This ensures any detected issues are addressed before proceeding further in the execution.
5. To execute a DRC rule function, declare the function within the `runDRC` function in `drc.cpp`. You can also comment out specific functions in runDRC if you’d like UGRAMM to skip certain DRC rule checks. This flexibility allows you to enable or disable checks as needed.


---
## DRC Verbosity

The DRC files utilize the [SPDLogger](https://github.com/gabime/spdlog) library to print statements to the console with different verbosity levels based on message importance. Here’s how each level works:
- Info Messages (drcLogger->info("")): Use for general information or status updates that don't indicate any issues.
- Warning Messages (drcLogger->warn("")): Use for messages that indicate potential issues or important notices that don’t halt execution.
- Error Messages (drcLogger->error("")): Use for critical messages where execution needs to halt due to serious issues.

Users can set the verbosity level of DRC print statements by using the `--drc_verbose_level` flag when executing the command. This flag allows users to control the type of messages displayed based on the severity level (e.g., info, warning, error). An example usage of the flag is shown below:

```./UGRAMM --seed $SEED --verbose_level 1 --dfile device_model.dot --afile application.dot --config config.json --drc_verbose_level 1```

By default, the `--drc_verbose_level` flag is set to `1`, which will display only warning and error messages. Users can adjust this verbosity level as follows:
- Set to `0`: Displays only error messages.
- Set to `1`: Displays only warning and error messages.
- Set to `2`: Displays info, warning, and error messages.

This flexibility allows users to control the output based on their need for detail during debugging 

---
## Disable DRC

Users can disable the DRC during UGRAMM execution by setting the `--drc_disable` flag. This option skips all DRC checks, allowing UGRAMM to run without performing syntax or logical verification on the input files. By default, UGRAMM will automatically run the DRC before executing the mapper.

Here is an example of how to use the flag:
``` ./UGRAMM --seed $SEED --verbose_level 1 --dfile device_model.dot --afile application.dot --config config.json --drc_disable ```

This command will bypass the DRC checks, allowing the mapper to proceed directly to execution.