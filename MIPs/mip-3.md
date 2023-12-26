---
mip: 3
title: Upgradeable MEM Functions
author: Darwin (darwin@decent.land)
type: Standards
discussions-to: n/a (internal)
status: Final
created: 26-12-2023
---

## Abstract
This MIP introduces new standards outlining rules for upgrading MEM serverless functions, encompassing both state and source code modifications.

## Motivation
At present, MEM developers lack a defined standard for creating upgradeable functions. The capability to upgrade functions is immensely valuable, enabling developers to implement significant changes to deployed functions, be it alterations to the source code or state, without disrupting integrations with other third-party entities linked to the upgraded serverless function ID, such as Apps, APIs, or serverless function ID integration/references.

## Proposal
This proposal outlines two aspects of function upgradeability: one for the function's state (JSON state) and the other for the function's source code. Both solutions empower developers to seamlessly upgrade their code or state (by overwriting), eliminating the need to disrupt any applications utilizing the function's unique 43-character ID. In essence, this approach ensures that functions remain entirely upgradeable without necessitating a change to the originally distributed function ID.

In order for both types of upgradeability to function properly, developers need to define specific state properties before deploying the function. These properties are essential prerequisites for ensuring the function's validity as an upgradeable entity.


| Upgradeability Type | state property 1 | state property 2 |
|:-----------------:|:-----------------:|:-----------------:|
| source code | `"mip_3_type" : "source_code"` | `"mip_3_admin": "ADMIN_EOA"` |
| state | `"mip_3_type" : "state"` | `"mip_3_admin": "ADMIN_EOA"` (optional) |


In this proposal, ECDSA has been implemented to verify signed messages, particularly for source code upgrading requests. Consequently, state property 2 mandates ADMIN_EAO to be a valid Ethereum Externally Owned Account (EOA).

### Functions Registry 
This specific aspect applies solely to functions that facilitate source code upgradeability. Unlike functions responsible for state upgrades, which do not generate a new function ID (43 characters), updating a function's source code does result in a new function ID. This new ID should be associated back to the original version (function ID) initially deployed for the function:

`mapping => latest_function_id --> initial_function_id ` 

The `Functions Registry`'s function represents the technical implementation of this MIP.

## Rationale
The MEM Improvement Proposal (MIP) aims to elevate the Developer Experience (DX) and introduce the possibility of upgradeable functions. However, end users must be informed when interacting with an upgradeable function. Trust in the deployed function's admin (deployer) is essential for ensuring a risk-free usage, relying on their integrity and credibility in maintaining the upgradeable function.

## Implementation
This proposal's implementation will be structured into two parts, following the outlined `Proposal` structure.

### Upgradeable state
In the context of upgradeable functions, it becomes standard practice for developers to update (overwrite/migrate) their function's state—either partially or entirely—by leveraging external data sources through the DeterministicFetch feature.

```json
// function state

{
    "mip_3_type": "state",
    "admin": "ADMIN_EOA" // recommended
}
```

And for function's state upgrading, the recommendation involves retrieving data from Arweave. This approach ensures the availability of data over an extended period.

In this proposal, I've integrated a previously tested code snippet from the ANS function source code. This snippet was utilized during the protocol's migration from EXM to MEM to upgrade the function's state.

```js
// function source code

    if (input.function === "importState") {
      ContractAssert(!state.isImported, "ERROR_STATE_IMPORTED");
      const importedState = (await EXM.deterministicFetch(`https://arweave.net/3s6cviQM40DvucydzrS8-167myxYKXfo5vb4WWTieVQ`)).asJSON();
      importedState.isImported = true;
      importedState.molecule_endpoints.ar_mem = `http://ar.molecule.sh/ar-auth`;
      importedState.sig_messages.push("ans-mainnet-mem::");
      importedState.signatures = [];
      state = importedState;

      return { state };
    }
```

Observably, the function verifies whether the importState function has been previously invoked; subsequent calls to importState will fail if it has. Depending on your function's requirements and code logic, you can incorporate caller-gating checks into your state upgrading function. For instance, authenticating the caller using the EVM molecule could be an option.

### Upgradeable Source Code

Similar to the previously mentioned upgradeability type, functions featuring upgradeable source code should include the following state properties. These properties enable the `Functions Registry` to ascertain the function's upgradeability status and validate requests for upgrades (such as function ID mapping) made by the function deployer.

```json
{
    "mip_3_type": "source_code",
    "admin": "ADMIN_EOA" // mandatory
}
```

#### Functions Registry Implementation
This segment introduces the MEM Functions Registry, dedicated to:

- Receiving source code upgrading requests from function deployers.
- Verifying function upgradeability and authenticating deployer-submitted upgrade requests.
- Mapping the initial function ID to the most recent submitted upgrade request while maintaining a log of upgrade history.

This registry function acts as a routing function IDs API, ensuring they're always updated with the latest version of any MEM upgradeable function deployed in production. The code snippets below are the Functions Registry initial state and source code. As you can notice, this function does not implement this proposal (MIP-3).



```json
// state.json

{
  "registry": {},
  "sig_message": "mip-3-registry::",
  "counter": 0,
  "evm_molecule_endpoint": "http://evm.molecule.sh",
  "signatures": [],
  "publicFunctions": {
    "upgrade_function": ["old_id", "new_id", "sig"]
  }
}
```

```js

// functions_registry.js
export async function handle(state, action) {
  const input = action.input;

  if (input.function === "upgrade_function") {
    const { old_id, new_id, sig } = input;

    _validateFunctionId(old_id);
    _validateFunctionId(new_id);

    ContractAssert(old_id !== new_id, "ERROR_INCORRECT_UPGRADE");

    const old_admin = await _validateUpgrade(old_id);
    const new_admin = await _validateUpgrade(new_id);

    ContractAssert(old_admin === new_admin, "ERROR_INVALID_CALLER");

    await _moleculeSignatureVerification(new_admin, sig);

    if (!(old_id in state.registry)) {
    	state.registry[old_id] = {};
      state.registry[old_id].latest = new_id;
      state.registry[old_id].logs = [new_id];

      return { state };
    }

    state.registry[old_id].latest = new_id;
    state.registry[old_id].logs.push(new_id);

    return { state };
  }

  function _validateFunctionId(id) {
    ContractAssert(/[a-z0-9_-]{43}/i.test(id), "ERROR_INVALID_MEM_FUNC_ID");
  }

  async function _validateUpgrade(new_id) {
    try {
      const state = (
        await EXM.deterministicFetch(`https://api.mem.tech/api/state/${new_id}`)
      )?.asJSON();

      // check for MIP-3 state properties
      ContractAssert(
        state?.mip_3_type === "source_code",
        "ERROR_FUNCTION_MISSING_MIP3_PROPERTY",
      );
      ContractAssert(state?.mip_3_admin, "ERROR_MIP3_ADMIN_NOT_FOUND");

      return state?.mip_3_admin;
    } catch (error) {
      throw new ContractAssert("ERROR_DF");
    }
  }

  async function _moleculeSignatureVerification(caller, signature) {
    try {
      const encodedMessage = btoa(`${state.sig_message}${state.counter}`);
      ContractAssert(
        !state.signatures.includes(signature),
        "ERROR_SIGNATURE_ALREADY_USED",
      );

      const isValid = await EXM.deterministicFetch(
        `${state.evm_molecule_endpoint}/signer/${caller}/${encodedMessage}/${signature}`,
      );
      ContractAssert(isValid.asJSON()?.result, "ERROR_UNAUTHORIZED_CALLER");
      state.signatures.push(signature);
      state.counter += 1;
    } catch (error) {
      throw new ContractError("ERROR_MOLECULE.SH_CONNECTION");
    }
  }
}

```

The Functions Registry testing contract has been deployed here: [lhTHcou2Y-PUx4Sn-xzDhk8bXJ2dCz9OvlMD_Wv2gfg](https://api.mem.tech/api/state/lhTHcou2Y-PUx4Sn-xzDhk8bXJ2dCz9OvlMD_Wv2gfg)

### Testing the Functions Registry

First, let's define two testing functions: one labeled as `old` and the other as `new`. The `new` function represents an upgraded version of the `old` function. Both functions are required to implement MIP-3 (Note: both functions have the same initial state in this example):

```json
// old.json
{
	"count": 0,
	"mip_3_type": "source_code",
	"mip_3_admin": "0x197f818c1313dc58b32d88078ecdfb40ea822614"
}
```

```js
// old.js
export async function handle(state, action) {
  const input = action.input;

  if (input.function === "increment") {
    state.count += 1;
    return { state };
  }
}
```

```js
// new.js
export async function handle(state, action) {
  const input = action.input;

  if (input.function === "increment") {
    state.count += 1;
    return { state };
  }

  if (input.function === "decrement") {
    state.count -= 1;
    return { state };
  }
}
```

After deploying both the `old` and `new` contracts on the MEM protocol, the resulting function IDs are:

- `old.js` function ID: [ltKTJCTQc5jvbqwgB1dgw3Sn30ZKT_U1eMZ7dvxHkVY](https://api.mem.tech/api/state/ltKTJCTQc5jvbqwgB1dgw3Sn30ZKT_U1eMZ7dvxHkVY)
- `new.js` function ID: [7pBQVa_J69Smt-kBGzoENqihKjqc5lsy6pqzfSYoCHE](https://api.mem.tech/api/state/7pBQVa_J69Smt-kBGzoENqihKjqc5lsy6pqzfSYoCHE)

Now that we have both the old and new function IDs, the next step involves generating a signature to authenticate the upgrading request within the Functions Registry. The message required for this authentication can be retrieved from the Functions Registry function state, utilizing the `counter` and `sig_message` parameters.

![image_2023-12-16_18-07-29](https://github.com/charmful0x/mip3/assets/77340894/56ed08be-a219-4644-a6eb-dff3104c2b6d)

The final step is interacting with the Functions Registry to submit the upgrade request:

```js
async function sendRequest() {
  try {
    const upgrade = [
      {
        input: {
          function: "upgrade_function",
          old_id: "ltKTJCTQc5jvbqwgB1dgw3Sn30ZKT_U1eMZ7dvxHkVY",
          new_id: "7pBQVa_J69Smt-kBGzoENqihKjqc5lsy6pqzfSYoCHE",
          sig: "0xcb6f64a5b95bfac2c4fb2a0f2cc8cb056c9160aa5944408be2b1b222313eda000249f6608b4413c2a87ad2acb6e4dae11dfdc132a7bfc44c5d30d2cb4fbbe4811c",
        },
      },
    ];

    const functionsRegistryId = "lhTHcou2Y-PUx4Sn-xzDhk8bXJ2dCz9OvlMD_Wv2gfg";

    const req = await axios.post(
      "https://api.mem.tech/api/transactions",
      {
        functionId: functionsRegistryId,
        inputs: upgrade,
      },
      {
        headers: {
          "Content-Type": "application/json",
        },
      },
    );

    console.log(JSON.stringify(req?.data, null, 4));
    return req?.data;
  } catch (error) {
    console.log(error);
  }
}

```


## Backwards Compatibility
This proposal maintains backward compatibility by introducing new reserved properties that will not affect existing functions state structures. Existing functions can continue to function without using this MIP but they will not have the MEM function upgradeability feature.

## Copyrights
Copyright and related rights waived via [CC0](../LICENSE.md).
