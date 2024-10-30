---
title: kernels
layout: default
nav_order: 7
description: "Kernels list for UGRAMM"
permalink: /UGRAMM-Documentations/kernels
---

# Kernels

For experimentation and testing of the UGRAMM software, developers used HPC benchmarks, including FFT, stencil, and convolution operations. These benchmarks were unrolled with factors from 1 to 10 to increase kernel size, effectively stress-testing the UGRAMM mapper.

The release includes the following benchmark operations:
- Conv_Balance
- Conv_nonBalance
- FFT-Radix-4
- FFT-Radix-5
- Stencil_Balance
- Stencil_nonBalance

Additionally, each benchmark includes two versions: `Any` and `Specific`. The `Any` version signifies that the `funcCell` is cumulative, allowing input signals to map to any pin of the `funcCell`. Conversely, the `Specific` version implies a non-cumulative `funcCell`, where input signals must map to specific pins as defined in the benchmark attributes.