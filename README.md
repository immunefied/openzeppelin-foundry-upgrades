# OpenZeppelin Foundry Upgrades

Foundry library for deploying and managing upgradeable contracts, which includes upgrade safety checks.

> **Warning**
> Experimental code. This is provided as a technology preview and its functionality may be subject to change.
> **Use at your own risk.**

## Installing

Run these commands:
```
forge install OpenZeppelin/openzeppelin-foundry-upgrades
forge install OpenZeppelin/openzeppelin-contracts-upgradeable@v5.0.0
```

Set the following in `remappings.txt`, replacing any previous definitions of these remappings:
```
@openzeppelin/contracts/=lib/openzeppelin-contracts-upgradeable/lib/openzeppelin-contracts/contracts/
@openzeppelin/contracts-upgradeable/=lib/openzeppelin-contracts-upgradeable/contracts/
```

## Version limitations

This library only supports proxy contracts and upgrade interfaces from OpenZeppelin Contracts 5.0 or higher.

## Prerequisites

This library uses the [OpenZeppelin Upgrades Core CLI](https://docs.openzeppelin.com/upgrades-plugins/1.x/api-core) for upgrade safety checks. In order for safety checks to work, the following are required:
- Node.js must be installed.
- Configure your `foundry.toml` according to the [CLI Prerequisites](https://docs.openzeppelin.com/upgrades-plugins/1.x/api-core#foundry).
- If you are upgrading your contract from a previous version, add the `@custom:oz-upgrades-from <reference>` annotation to the new version of your contract according to [Define Reference Contracts](https://docs.openzeppelin.com/upgrades-plugins/1.x/api-core#define-reference-contracts) or specify the `referenceContract` option when calling the library's functions.
- Run `forge clean` before running your Foundry script or tests.
- Include `--ffi` in your `forge script` or `forge test` command.

If you do not want to run upgrade safety checks, use the `unsafeSkipAllChecks` option when calling the library's functions. Note that this is a dangerous option meant to be used as a last resort.

## Usage

Import the library:
```
import {Upgrades} from "openzeppelin-foundry-upgrades/Upgrades.sol";
```

Then call functions from [Upgrades.sol](src/Upgrades.sol) in your Foundry scripts or tests to run validations, deployments, or upgrades.

### Examples

Deploy a UUPS proxy:
```
address proxy = Upgrades.deployUUPSProxy(
    "MyContract.sol",
    abi.encodeCall(MyContract.initialize, ("arguments for the initialize function"))
);
```

Deploy a transparent proxy:
```
address proxy = Upgrades.deployTransparentProxy(
    "MyContract.sol",
    INITIAL_OWNER_ADDRESS_FOR_PROXY_ADMIN,
    abi.encodeCall(MyContract.initialize, ("arguments for the initialize function"))
);
```

Call your contract's functions as normal, but remember to always use the proxy address:
```
MyContract instance = MyContract(proxy);
instance.myFunction();
```

Upgrade a transparent or UUPS proxy and call an arbitrary function (such as a reinitializer) during the upgrade process:
```
Upgrades.upgradeProxy(
    transparentProxy,
    "MyContractV2.sol",
    abi.encodeCall(MyContractV2.foo, ("arguments for foo"))
);
```

Deploy an upgradeable beacon:
```
address beacon = Upgrades.deployBeacon("MyContract.sol", INITIAL_OWNER_ADDRESS_FOR_BEACON);
```

Deploy a beacon proxy:
```
address proxy = Upgrades.deployBeaconProxy(
    beacon,
    abi.encodeCall(MyContract.initialize, ("arguments for the initialize function"))
);
```

Upgrade a beacon:
```
Upgrades.upgradeBeacon(beacon, "MyContractV2.sol");
```

### Deploying

Run your script with `forge script` to broadcast and deploy. See Foundry's [Solidity Scripting](https://book.getfoundry.sh/tutorials/solidity-scripting) guide.

> **Important**
> Include the `--sender <ADDRESS>` flag for the `forge script` command when performing upgrades, specifying an address that owns the proxy or proxy admin. Otherwise, `OwnableUnauthorizedAccount` errors will occur.

> **Note**
> Include the `--verify` flag for the `forge script` command if you want to verify source code such as on Etherscan. This will verify your implementation contracts along with any proxy contracts as part of the deployment.

## Contributing

### Running tests

```
forge clean && forge test --ffi
```

### Running example script

You can simulate deployments and upgrades of the various kinds of proxies by running the script:

```
forge clean && forge script test/Upgrades.s.sol --ffi
```
