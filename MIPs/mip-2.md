---
mip: 2
title: Standard for MEM public functions detection
author: Darwin (@charmful0x)
type: Standards
discussions-to: n/a (internal)
status: Final
created: 09-08-2023
---

## Abstract
This MIP proposes the introduction of a `publicFunctions` property in the MEM contract state to define MEM contract functions along with their corresponding input object arguments. This enhancement aims to automate the capturing of public callable functions in MEM contracts, alleviating the manual burden on contract developers and the DevX of MEM contracts IDEs and explorers.

## Motivation
Currently, contract developers are required to manually define the properties that represent public callable functions along with their input object arguments every time involving calling a function from the IDE UI. This process can be error-prone and time-consuming, leading to potential inconsistencies and issues across MEM contracts. The proposed `publicFunctions` property aims to provide a standardized and automated way to define these functions and arguments, reducing developer effort and enhancing contract consistency.

## Proposal
Introduce a reserved property called `publicFunctions` in the MEM contract state that will define the public callable functions along with their input object arguments. The structure of `publicFunctions` will be a dictionary, where the keys represent the function names, and the corresponding values are arrays of strings representing the input object properties (function arguments).

Example: `state.json`
```json
{
  "publicFunctions": {
    "addUser": ["username", "bio", "id"]
  }
}
```

In this example, the `addUser` function is defined with three input object properties: `username`, `bio`, and `id`.

## Rationale
Introducing the `publicFunctions` property simplifies capturing public callable functions and their input object arguments in MEM contracts. It provides a clear and standardized structure that helps contract developers avoid manual errors and maintain consistency across contracts during contract writing and testing.

## Implementation
The implementation of this proposal will involve adding a parsing mechanism for the `publicFunctions` property in the MEM contracts IDE to auto-detect a contract's public callable function with its properties. The code will parse the `publicFunctions` dictionary and use the provided function names and input object properties to generate the necessary contract function definitions.

## Backwards Compatibility
This proposal maintains backward compatibility as it introduces a new reserved property that will not affect existing contract state structures. Existing contracts can continue to function without using the `publicFunctions` property but they will not have the in-IDE functions calling feature.

## Copyrights
Copyright and related rights waived via [CC0](../LICENSE).
