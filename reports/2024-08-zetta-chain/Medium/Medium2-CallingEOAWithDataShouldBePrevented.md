
## Relevant Context

Calling an EOA with data regardless `msg.value` returns true and the EOA receives the value sent.

## Description

If a user mistakenly sends `native coin + call` to an EOA with data, the `_execute()` function in `GatewayEVM` will just execute and send the `msg.value` to the EOA.

## Impact Explanation

Funds sent to a mistaken EOA will be lost when it can be easily avoided. The protocol should revert if someone tries to call data on an EOA.

Some might say this is a `Low` issue because of user mistake. But I think the protocol can and clearly should stop executing to avoid using unnecessary gas and prevent a clear loss of funds.

Notice that calling this with `msg.value > 0` is posible if coming from `execute()` function in `GatewayEVM` contract.

```solidity
    function execute(
        address destination,
        bytes calldata data
    )
        external
        payable // ◀️🟢👁️
        onlyRole(TSS_ROLE)
        whenNotPaused
        nonReentrant
        returns (bytes memory)
    {
        if (destination == address(0)) revert ZeroAddress();
        bytes memory result = _execute(destination, data);

        emit Executed(destination, msg.value, data);
        return result;
    }
```

## Proof of Concept

Paste this in remix and call it with a known EOA and some data like:

- EOA == [0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045](https://etherscan.io/address/0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045)
- Data == 0x696969 
- `msg.value` == any amount, 1 ether for example

```solidity 
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

contract TestEOACall {
    bytes public data;
    address public called;
    uint256 public balanceB;
    uint256 public balanceAfter;

    // Function to send Ether and data to a specified address
    function sendEtherAndData(address destination, bytes memory _data) public payable returns(bytes memory){
        // Attempt to call the destination with Ether and data

        uint256 balance = address(this).balance;
        (bool success, bytes memory result) = destination.call{value: msg.value}(_data);
        uint256 balance_aft = address(this).balance;

        if(success){
            data = _data;
            called  = destination;
            balanceB = balance;
            balance_aft = balance_aft;
        }

        return result;
    }
}
```

## Recommendation

At the `_execute()` mentioned, revert if someone tries to call data on an EOA. This is not just useless execution but it can highly likely signal sending funds to a wrong address leading to its loss.

The check looks like: `require(destination.code.length > 0 && data.length > 0, "Destination is not a contract");`
