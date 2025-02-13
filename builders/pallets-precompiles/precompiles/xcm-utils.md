---
title:  XCM-Utils Precompile Contract
description:  Learn the various XCM related utility functions available to smart contact developers with Moonbeam's precompiled XCM-Utils contract.
keywords: solidity, ethereum, xcm, utils, moonbeam, precompiled, contracts
---

# Interacting with the XCM-Utils Precompile

![Precomiled XCM-Utils Banner](/images/builders/pallets-precompiles/precompiles/xcm-utils/xcm-utils-banner.png)

## Introduction {: #xcmutils-precompile}

The XCM-utils precompile contract gives developers XCM-related utility functions directly within the EVM. This allows for easier transactions and interactions with other XCM-related precompiles. 

Similar to other [precompile contracts](/builders/pallets-precompiles/precompiles/){target=_blank}, the XCM-utils precompile is located at the following addresses:

=== "Moonbeam"
     ```
     {{networks.moonbeam.precompiles.xcm_utils }}
     ```

=== "Moonriver"
     ```
     {{networks.moonriver.precompiles.xcm_utils }}
     ```

=== "Moonbase Alpha"
     ```
     {{networks.moonbase.precompiles.xcm_utils}}
     ```

--8<-- 'text/precompiles/security.md'

## The XCM-Utils Solidity Interface {: #xcmutils-solidity-interface } 

[XcmUtils.sol](https://github.com/PureStake/moonbeam/blob/master/precompiles/xcm-utils/XcmUtils.sol){target=_blank} is an interface to interact with the precompile.

!!! note
    The precompile will be updated in the future to include additional features. Feel free to suggest additional utility functions in the [Discord](https://discord.gg/PfpUATX){target=_blank}.

The interface includes the following functions:

 - **multilocationToAddress**(*Multilocation memory* multilocation) — read-only function that returns the multilocation-derivative account from a given multilocation
 - **weightMessage**(*bytes memory* message) — read-only function that returns the weight that an XCM message will consume on the chain. The message parameter must be a SCALE encoded XCM versioned XCM message
 - **getUnitsPerSecond**(*Multilocation memory* multilocation) — read-only function that gets the units per second for a given asset in the form of a `Multilocation`. The multilocation must describe an asset that can be supported as a fee payment, such as an [external XC-20](/builders/interoperability/xcm/xc20/xc20){target=_blank}, or else this function will revert

The `Multilocation` struct in the XCM-utils precompile is built the [same as the XCM-transactor](/builders/interoperability/xcm/xcm-transactor#building-the-precompile-multilocation){target=_blank} precompile's `Multilocation`.

## Using the XCM-Utils Precompile {: #using-the-xcmutils-precompile } 

The XCM-Utils precompile allows users to read data off of the Ethereum JSON-RPC instead of having to go through a Polkadot library. The functions are more for convenience, and less for smart contract use cases. 

For `multilocationToAddress`, one example use case is being able to allow transactions that originate from other parachains by whitelisting their multilocation-derived addresses. A user can whitelist a multilocation by calculating and storing an address. EVM transactions can originate from other parachains via [remote EVM calls](/builders/interoperability/xcm/remote-evm-calls).  

```solidity
// SPDX-License-Identifier: GPL-3.0-only
pragma solidity >=0.8.3;

import "https://github.com/PureStake/moonbeam/blob/master/precompiles/xcm-utils/XcmUtils.sol";

contract MultilocationWhitelistExample {
    XcmUtils xcmutils = XcmUtils(0x000000000000000000000000000000000000080C);
    mapping(address => bool) public whitelistedAddresses;

    modifier onlyWhitelisted(address addr) {
        _;
        require(whitelistedAddresses[addr], "Address not whitelisted!");
        _;
    }

    function addWhitelistedMultilocation(
        XcmUtils.Multilocation calldata externalMultilocation
    ) external onlyWhitelisted(msg.sender) {
        address derivedAddress = xcmutils.multilocationToAddress(
            externalMultilocation
        );
        whitelistedAddresses[derivedAddress] = true;
    }

    ...
}
```
