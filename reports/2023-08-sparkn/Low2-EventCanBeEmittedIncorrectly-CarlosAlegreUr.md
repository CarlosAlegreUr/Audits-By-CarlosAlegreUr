<hr/>

## **Summary 📌**

Solidity's low-level function **_`call()`_** usage might lead to **incorrect event emissions**.

---

## **Vulnerability Details 🔍**

In the process of delegating calls for prize distribution, the **_`_delegate()`_** function eventually gets called, triggering **_`proxy.call(data)`_**.

The `data` in the call may not always pertain to the **`Distributor.sol`** contract's **_`distribute()`_** function. For instance, it might eventually call the **_`getConstants()`_** function and still result in a successful call.

This inconsistency would lead the **`Proxy.sol`** contract to emit the **`Distributed(proxy, data)`** event, even if no actual distribution took place.

<details><summary> Proof of Concept (PoC) 🕵️ </summary>

> 🚧 **Note** ⚠️: The test is a variant of the team's original test: `testSucceedsIfAllConditionsMet (ProxyFactoryTest.t.sol)`. Adding event checks and adjusted data value. New additions are highlighted in 🟢.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.18;

import {MockERC20} from "../mock/MockERC20.sol";
import {ECDSA} from "openzeppelin/utils/cryptography/ECDSA.sol";
import {Test, console} from "forge-std/Test.sol";
import {StdCheats} from "forge-std/StdCheats.sol";
import {ProxyFactory} from "../../src/ProxyFactory.sol";
import {Proxy} from "../../src/Proxy.sol";
import {Distributor} from "../../src/Distributor.sol";
import {HelperContract} from "../integration/HelperContract.t.sol";

contract FakeEventTest is StdCheats, HelperContract {
    // 🟡 This is set-up, new test is marked with 🟢
    event Distributed(address indexed proxy, bytes data);

    bytes32 constant _SOMEID = keccak256(abi.encode("Jason", "001"));

    function setUp() public {
        // set up balances of each token belongs to each user
        if (block.chainid == 31337) {
            // deal ether
            vm.deal(factoryAdmin, STARTING_USER_BALANCE);
            vm.deal(sponsor, SMALL_STARTING_USER_BALANCE);
            vm.deal(organizer, SMALL_STARTING_USER_BALANCE);
            vm.deal(user1, SMALL_STARTING_USER_BALANCE);
            vm.deal(user2, SMALL_STARTING_USER_BALANCE);
            vm.deal(user3, SMALL_STARTING_USER_BALANCE);
            vm.deal(TEST_SIGNER, SMALL_STARTING_USER_BALANCE);
            // mint erc20 token
            vm.startPrank(tokenMinter);
            MockERC20(jpycv1Address).mint(sponsor, 100_000 ether); // 100k JPYCv1
            MockERC20(jpycv2Address).mint(sponsor, 300_000 ether); // 300k JPYCv2
            MockERC20(usdcAddress).mint(sponsor, 10_000 ether); // 10k USDC
            MockERC20(jpycv1Address).mint(organizer, 100_000 ether); // 100k JPYCv1
            MockERC20(jpycv2Address).mint(organizer, 300_000 ether); // 300k JPYCv2
            MockERC20(usdcAddress).mint(organizer, 10_000 ether); // 10k USDC
            MockERC20(jpycv1Address).mint(TEST_SIGNER, 100_000 ether); // 100k JPYCv1
            MockERC20(jpycv2Address).mint(TEST_SIGNER, 300_000 ether); // 300k JPYCv2
            MockERC20(usdcAddress).mint(TEST_SIGNER, 10_000 ether); // 10k USDC
            vm.stopPrank();
        }

        // labels
        vm.label(organizer, "organizer");
        vm.label(sponsor, "sponsor");
        vm.label(supporter, "supporter");
        vm.label(user1, "user1");
        vm.label(user2, "user2");
        vm.label(user3, "user3");
    }

    ///////////////////////
    // Modifier for test //
    ///////////////////////
    modifier setUpContestForJasonAndSentJpycv2Token(address _organizer) {
        vm.startPrank(factoryAdmin);
        bytes32 randomId = keccak256(abi.encode("Jason", "001"));
        proxyFactory.setContest(_organizer, randomId, block.timestamp + 8 days, address(distributor));
        vm.stopPrank();
        bytes32 salt = keccak256(abi.encode(_organizer, randomId, address(distributor)));
        address proxyAddress = proxyFactory.getProxyAddress(salt, address(distributor));
        vm.startPrank(sponsor);
        MockERC20(jpycv2Address).transfer(proxyAddress, 10000 ether);
        vm.stopPrank();
        // console.log(MockERC20(jpycv2Address).balanceOf(proxyAddress));
        assertEq(MockERC20(jpycv2Address).balanceOf(proxyAddress), 10000 ether);
        _;
    }

    function createDataGetter() public pure returns (bytes memory data) {
        data = abi.encodeWithSelector(Distributor.getConstants.selector);
    }

    // 🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢🟢
    function testFailWrongEventEmitted() public setUpContestForJasonAndSentJpycv2Token(organizer) {
        // before
        assertEq(MockERC20(jpycv2Address).balanceOf(user1), 0 ether);
        assertEq(MockERC20(jpycv2Address).balanceOf(stadiumAddress), 0 ether);

        bytes32 randomId_ = keccak256(abi.encode("Jason", "001"));
        bytes memory data = createDataGetter(); // 🟢

        bytes32 salt = keccak256(abi.encode(organizer, randomId_, address(distributor)));
        address prxyAdd = proxyFactory.getProxyAddress(salt, address(distributor));
        vm.expectEmit(true, true, true, true, address(proxyFactory)); // 🟢
        emit Distributed(prxyAdd, data);

        vm.warp(9 days);
        vm.startPrank(organizer);
        proxyFactory.deployProxyAndDistribute(randomId_, address(distributor), data);
        vm.stopPrank();

        // after
        assertEq(MockERC20(jpycv2Address).balanceOf(user1), 9500 ether);
        assertEq(MockERC20(jpycv2Address).balanceOf(stadiumAddress), 500 ether);
    }
}
```

</details>

---

## **Impact 📈**

The impact is relatively minimal. However, **users can emit misleading events**. Ensuring **accurate logging of system actions** and intentions **is important, especially** considering SPARKN's **targeted adoption by government and public institutions**, emphasizing the need for transparency.

---

## **Tools Used 🛠️**

- Manual audit.

---

## **Recommendations 🎯**

Two possible fixes are proposed:

- **1️⃣ Introduce a filter to inspect the data parameter's function call. If it deviates from distribute(), the function should revert.**

    <details> <summary> Implementation Example 💻 </summary>

  > 🚧 **Note** ⚠️: This code is illustrative and has not been tested.

  ```solidity
          // 🟢 Getting selector of distribute function
        bytes4 private constant APPROVED_SELECTOR_1 = Distributor.distribute.selector;

      // 🟢 filterCall() will analyze the data received
        function filterCall(bytes calldata data) external {
          // 🟢 extractSelector() is defined below
            bytes4 selector = extractSelector(data);

            require(
                selector == APPROVED_SELECTOR_1,
                "Function not approved"
            );

            // 🟢 If the selector is approved, perform the call
            proxy.call(data);
            // rest of code...
        }

        function extractSelector(bytes memory data) public pure returns (bytes4) {
            require(data.length >= 4, "Data too short");
            bytes4 selector;
            // solhint-disable-next-line no-inline-assembly
            assembly {
                selector := mload(add(data, 0x20))
            }
            return selector;
        }
  ```

    </details>

- **2️⃣ Examine the return data. If distribute() gets invoked, the output will be 0; otherwise, the function should revert.**

    <details><summary> Implementation Example 💻 </summary>
    
    > 🚧 **Note** ⚠️: This code is illustrative and has not been tested.

  ```solidity
    function _distribute(address proxy, bytes calldata data) internal {

          // 🟢 Notice we are checking the returned output just in case
          (bool success, bytes memory result) = proxy.call(data);
          require(result.length == 0, "Data was returned, distribute was not called, reverting.");

          if (!success) revert ProxyFactory__DelegateCallFailed();
          emit Distributed(proxy, data);
      }
  ```

    </details>

Given its straightforwardness and the absence of any other function in **`Distributor.sol`** returning nothing, **solution 2️⃣** stands out as the preferable choice.

If in the future the **`Distributor.sol`** gets more complex and has more functions returning something, implementation 1️⃣ should be considered.
