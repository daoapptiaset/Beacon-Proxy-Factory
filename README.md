# Beacon-Proxy-Factory
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Beacon {
    address public implementation;

    constructor(address _impl) {
        implementation = _impl;
    }

    function upgradeTo(address newImpl) public {
        // Add onlyOwner in production
        implementation = newImpl;
    }
}

contract BeaconProxy {
    address public beacon;

    constructor(address _beacon) {
        beacon = _beacon;
    }

    fallback() external payable {
        address impl = Beacon(beacon).implementation();
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}
