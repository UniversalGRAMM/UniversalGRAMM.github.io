---
title: node-locking
parent: config-file-input
layout: default
nav_order: 2
description: "Node locking guide for UGRAMM"
permalink: /UGRAMM-Documentations/config-file/node-locking
---

## Node Locking:

Locking is a feature in UGRAMM that allows specific application DFG vertices to be mapped to specific device model graph nodes. Since UGRAMM can target various architectures—whether realistic or abstract—the locking of nodes is achieved using the string names of the application DFG and device model graph nodes. To implement this, a tag called `lock-nodes` must be included in the config file, followed by a list of locked nodes in string format. The structure for locked nodes is as follows: `hName::gName`. Here, `hName` represents the application DFG node name, and `gName` represents the device model graph node name. The `::` is used to separate the two names. Without this, UGRAMM will not be able to identify the two names in the string. Do node that the values provided in the JSON file are case-insensitive. However, the key `lock-nodes` is case-sensitive to lower case characters. 
```
"lock-nodes" : [<-String list of locked nodes->]
```

The following shows a node locking of some application DFG nodes on the RIKEN CGRA architecture. 
```
"lock-nodes" : ["FMUL_9::pe.w32.c4.r6.alu", "FMUL_11::pe.w32.c4.r4.alu", "FMUL_13::pe.w32.c3.r3.alu"]
```

Since locking depends on the names of the nodes in both the application DFG and the device model graph, it is essential for users to ensure that the node names are correct and exist in their respective directed graphs. If a node name does not exist within the corresponding graph, UGRAMM will be unable to properly lock the nodes.

---
## Locking Terminology

When discussing node locking, a few key terms need to be introduced. Throughout this documentation and in the code, we frequently refer to `Fully Locked Nodes`. `Fully Locked Nodes` represent a one-to-one mapping between an application DFG node and a device model graph node. In other words, the application DFG node can only be locked to a single, specific node in the device model graph.

In contrast, `Partially Locked Nodes` refer to nodes that perform a one-to-many locking. This means that the application DFG node can be locked to multiple device model graph nodes. This terminology is directly tied to the wildcard feature, which is explained further below.

`Non-locked Nodes` are nodes that are not locked to a specific node in the device model graph and can be mapped freely to any suitable location within the device model graph. Unlike Fully Locked or Partially Locked Nodes, these nodes are not defined in the config file giving the UGRAMM mapper full flexibility in determining where they should be placed.

---
## Skip over fully locked nodes when mapping

When mapping `Non-locked Nodes`, UGRAMM has an option to skip over `Fully Locked Nodes` in the device model graph during the graph minor search. This feature was added to improve the runtime of the mapping process, especially when users know certain nodes are unavailable for mapping. However, keep in mind that using this feature may result in a non-optimal mapping solution.

To enable this feature, users must set the `skipFullyLockedNodes` define statement in `UGRAMM.h` to `1`. This will allow UGRAMM to bypass `Fully Locked Nodes` during the mapping process. If set to `0`, UGRAMM will consider `Fully Locked Nodes` during the mapping process.
```
skipFullyLockedNodes 1
```

As this adjustment modifies a define statement, it is highly recommended to run `make clean` before running `make` again to avoid the executable from crashing.

---
## Wildcard locking

UGRAMM also provides users the option to introduce wildcards, `*`, in their node locks, creating `Partially Locked Nodes`. This allows UGRAMM to map the application DFG node to one of several possible locations in the device model graph. Since node locking is string-based, using a wildcard will force the application DFG node to be mapped to a device model graph node whose name contains the specified substring defined in the `Partially Locked Nodes declaration` in the configuration file. The final device model graph node for the Partially Locked Node will be chosen based on the lowest mapping cost.

In the following example, the application DFG node `FMUL_13` will be mapped to a device model node whose name contains the substrings `r3` and `alu`. One possibility for mapping `FMUL_13` to a device model graph node is `pe.w32.c3.r3.alu`, as it contains both substrings mentioned in its name. Another potential location could be `pe.w32.c5.r3.alu`, which also satisfies the required substrings in its name. UGRAMM will choose the final mapping based on the lowest routing cost between these candidates.

```
"lock-nodes" : ["FMUL_13::*r3*alu"]
```

It’s important to note that not all locked nodes in the config file need to use wildcards when the wildcard feature is enabled. You can include a mix of `Fully Locked Nodes` and `Partially Locked Nodes`. For example, as shown below, `FMUL_9` and `FMUL_11` are `Fully Locked Nodes` and do not use wildcards, while `FMUL_13` is a `Partially Locked Node`, as it uses the wildcard `*`. 
```
"lock-nodes" : ["FMUL_9::pe.w32.c4.r6.alu", "FMUL_11::pe.w32.c4.r4.alu", "FMUL_13::**r3*alu"]
```

To enable wildcard locking, users must set the `allowWildcardInLocking` define statement in `UGRAMM.h` to `1`. This activates the wildcard locking feature in UGRAMM. If set to `0`, UGRAMM will not use wildcard locking and will only support `Fully Locked Nodes`. It is important to note that if a `*` (wildcard) is used in the locked node while `allowWildcardInLocking` is disabled, UGRAMM will crash due to the mismatch in the configuration.

```
allowWildcardInLocking 1
```


As this adjustment modifies a define statement, it is highly recommended to run `make clean` before running `make` again to avoid the executable from crashing.
