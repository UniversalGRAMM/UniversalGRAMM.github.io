---
title: config-file-input
layout: default
nav_order: 4
description: "Config file guide for UGRAMM"
permalink: /UGRAMM-Documentations/config-file
---

## UGRAMM config file:
UGRAMM allows users to inclde an input JSON file to configure the graph minor mapping through various parameters. As of now, UGRAMM supports the following parameters. However, more configuration support will be released in future releases.
    - [Skip Placement](/UGRAMM-Documentations/config-file/skip-placement)
    - [Node Locking](/UGRAMM-Documentations/config-file/node-locking)

---
## JSON File Support:
The config file is format interms of a JSON. This Jason file will be parsed in UGRAMM to retrieve the different configuration parameters. UGRAMM is using an open source parser called [JSON by nlohmann](https://github.com/nlohmann/json). For ease of use, UGRAMM has the library downloaded in the code and is used in the application through `#include "../lib/json.h"` statement. 

