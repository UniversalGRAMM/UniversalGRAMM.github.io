---
title: input-application
parent: UGRAMM
nav_order: 1
---

# Input application for UGRAMM:

<div style="text-align: center;">
    <img src="assets/dfgExample.png" alt="Fig 1. Data Flow Example" style="border: 1px solid black; width: 150px;">
    <figcaption style="font-size: 14px; color: #555;">Fig 1. Data Flow Example</figcaption>
</div>

- In a CGRA, the input application is usually represented as a Data Flow Graph (DFG), where vertices correspond to operations like ADD, SUB, and MUL, and the edges between these vertices indicate the dependencies among these operations.

- UGRAMM comes with various CGRA benchmarks designed for HPC applications, including Convolution, FFT, and Stencil, with both Balanced and Unbalanced versions. Additionally, each benchmark type supports different unrolling factors ranging from 1 to 10.

```
├── Kernels
│   ├── Conv_Balance
│   │   ├── conv_nounroll_Balance.dot
│   │   ├── conv_nounroll.dot
│   │   ├── conv_unroll2_Balance.dot
│   │   ├── conv_unroll2.dot
│   │   ├── conv_unroll3_Balance.dot
│   │   └── conv_unroll3.dot
│   ├── Conv_nonBalance
│   ├── FFT-Radix
│   │   ├── FFT-Radix-4
│   │   └── FFT-Radix-5
│   ├── Stencil_Balance
│   └── Stencil_nonBalance
```
