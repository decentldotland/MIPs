---
mip: 3
title: Standard for MEM source code sharing
author: K (@kay_is)
type: Standards
status: Draft
---

## Abstract

This MIP proposes the introduction of a `Contract-Src` transaction tag for resuing already
deployed contract source code with multiple contract instances that all have their own initial state.

## Motivation

Currently, contract developers must redeploy their contract source code every time they want to create a new contract instance with a different initial state. This is not only inefficient but also costly as it requires the developer to pay fees every time they want to create a new contract instance.

Right now, it's also hard to check the source code of a contract that has already been deployed. This makes it difficult to verify the contract's functionality and security.

## Proposal

Introduce a reserved tag called `Contract-Src` that contains the TX ID of an already deployed MEM contract.

When executing a contract with this tag, the MEM executor will ignore the `contractSrc` property in the transaction data and instead use the `contractSrc` of the MEM contract linked in the `Contract-Src` tag.

### Example

First, an existing transaction A, which contains a `contractSrc` in its data.

Tags:

```javascript
[
  {"name": "Content-Type", "value": "application/javascript"},
  {"name": "Owner", "value": ""},
  {"name": "App-Name", "value": "EM"},
  {"name": "Type", "value": "Serverless-Function"},
  {"name": "EM-Bundled","value": "true"},
  {"name": "Size", "value": "9999"}
],
```

Data:

```json
{
  "contractOwner": "",
  "contentType": "application/javascript",
  "contractSrc": [...],
  "initState": "{}"
}
```

Then, a new transaction B, which contains a `Contract-Src` tag that points to transaction A. Transaction B has no `contractSrc` in its data and no `Size` tag.

Tags:

```javascript
[
  {"name": "Content-Type", "value": "application/javascript"},
  {"name": "Owner", "value": ""},
  {"name": "App-Name", "value": "EM"},
  {"name": "Type", "value": "Serverless-Function"},
  {"name": "EM-Bundled","value": "true"},
  {"name": "Contract-Src", "value": "<TRANSACTION_A_ID>"}
],
```

Data:

```json
{
  "contractOwner": "",
  "contentType": "application/javascript",
  "initState": "{}"
}
```

## Rationale

Introducing the `Contract-Src` tag lets developers reuse already deployed contracts, so they can lower their transaction size and leverage existing contracts without implementing their own.

It also gives contract consumers and the MEM executor a standardized way to check that new contracts use a specific source code.

## Implementation

The implementation of this proposal requires adding a fetching mechanism for the transaction linked in the `Contract-Src` tag, so the MEM executor can load the code that is required to run the contract.

## Backwards Compatibility

This proposal maintains backward compatibility as it introduces a new reserved tag that will not affect existing contract state structures. Existing contracts can continue to function without using the `Contract-Src` property.

New contracts deployed with the new tag can also reuse the code of contracts deployed before the creation of this proposal.

## Copyrights

Copyright and related rights waived via [CC0](../LICENSE.md).
