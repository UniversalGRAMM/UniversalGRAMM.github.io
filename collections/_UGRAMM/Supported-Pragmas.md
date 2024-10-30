---
title: supported-pragmas
layout: default
nav_order: 5
description: "Pragma support guide for UGRAMM"
permalink: /UGRAMM-Documentations/Supported-Pragmas
---

# Pragma Support:

In the GRAMM architecture, `FuncCell` units like ALUs are equipped with defined lists of supported operations. For instance, a floating-point ALU might handle operations such as `FADD`, `FMUL`, `FSUB`, and `FDIV`. Different ALU types are configured with specific capabilities—`ALU1`, for example, may be restricted to integer operations by defining only integer operations in its supported operations list, while `ALU2` might be set up exclusively for floating-point operations by similarly specifying only floating-point functions. This differentiation enables the application kernel to map operations onto hardware based on each `FuncCell`’s unique capabilities mentioned in the supported operations.

The question then is, how exactly are these supported operations specified within the UGRAMM framework? The answer lies in `Pragmas`, defined at the top of application, and device-model`.dot` files. Pragmas are structured as multi-line comments, followed by the main graph content within these files. 

Application `.dot` files can be found in the `Kernels/` directory, while device-model `.dot` files can be generated using the `device-model-gen.py` script located under `Scripts/`. For more information, refer to the [UGRAMM documentation on device model files](/UGRAMM-Documentations/device-model-file).

### **Example device-model pragma** :
```
/*
{
    "ALU" : ["FADD", "FMUL", "FSUB", "FDIV"],
    "MEMPORT" : ["input", "output"],
    "Constant" : ["const"]
}
*/
```
### **Example application-graph pragma** :
```
/*
{
    "ALU" : ["FADD", "FMUL"],
    "MEMPORT" : ["input", "output"],
    "Constant" : ["const"],
    "Any2Pins" : "inPinA,inPinB"
}
*/
```

As you can see from the examples of these pragmas from device-model, and application graph files, they are defined in typical [JSON format](https://www.w3schools.com/js/js_json_syntax.asp). The reason for using JSON format here, the configuration input for UGRAMM is defined in JSON fomrat and just to keep everything consistent we have used same JSON format here as well. Following section describes the features of the Pragmas in these dot files:

## Supported operations for function cells:

Each `FuncCell` type (e.g., `ALU`, `MEMPORT`, `Constant`) in UGRAMM can configure to set of operations defined in JSON-based pragmas, enabling specific mapping of kernel operations to hardware.

- **Device Model Syntax**: `"[FuncCell-Name]" : ["operation1", "operation2", ...]`
- **Application Graph Syntax**: `"[FuncCell-Name]" : ["operation1", "operation2", ...]`

Application graph operations are always a **subset** of the device model. UGRAMM checks this using `readDeviceModelPragma()` and `readApplicationGraphPragma()`, ensuring compatibility and correctness between hardware and kernel definitions.

## Alias for attributes:

In UGRAMM, aliases simplify repeated attribute definitions, such as specifying loadPins across multiple connections. Instead of listing each pin every time, an alias can be defined in the application graph pragma to represent a group of pins, improving readability and maintainability.

### Instead:

```
Load_1 -> FMUL_10  [driver=outPinA, load="inPinA,inPinB"];
Load_2 -> FMUL_11  [driver=outPinA, load="inPinA,inPinB"];
```

### Do Alias way!!

```
 /*
{
    "ALU" : ["FADD", "FMUL"],
    "MEMPORT" : ["input", "output"],
    "Constant" : ["const"],
    "Any2Pins" : "inPinA,inPinB"
}
*/
//Any2Pins is an alias which is defined in the pragma of the application.dot
Load_0 -> FMUL_9  [driver=outPinA, load=Any2Pins]; 
Load_1 -> FMUL_10  [driver=outPinA, load=Any2Pins];
Load_2 -> FMUL_11  [driver=outPinA, load=Any2Pins];
```