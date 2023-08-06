## Summary 📌

The report aims to optimize the project's codebase using these software paradigms:

- **Divide and Conquer (DaC)**
- **Separation of Responsibilities (SoR)**

---

## Vulnerability Details 🔍

### A STRUCTURAL CHANGE ☢️

Monolithic architectures might offer some advantages in terms of gas efficiency, but with growing project complexity, it's vital to prioritize maintainability and flexibility. A clear separation of concerns, coupled with a DaC approach, can offer significant benefits.

**Recommendations**:

1️⃣ **Break down the responsibilities** of `Lender.sol` into two distinct contracts:

- **PoolsManager.sol**: Pool-related operations and management.
- **Lender.sol**: Lending-specific operations and management.

2️⃣ For **further modularity** adoption:

- **ILender.sol & IPoolsManager.sol**: Modularize events and easily grasp a view of the 2 contracts functions.

By adopting this restructured approach, each module will excel in its function, leading to a more robust and manageable system.

<details> <summary> Modularized Code Template 🗺️ </summary>

### Modularized Code Template 📜

> 🚧 **Note** ⚠️: The code below is just a template to guide you. It has not been tested and should not be relied upon. Moreover, while the intent is to simplify and reorganize, this restructuring might not be the most gas-efficient approach.

_**`Lender.sol`**_:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;
// Needed imports...

contract Lender is ILender, Ownable {
    uint256 public constant MAX_INTEREST_RATE = 100000;
    uint256 public constant MAX_AUCTION_LENGTH = 3 days;
    uint256 public lenderFee = 1000;
    uint256 public borrowerFee = 50;
    address public feeReceiver;

    // 🟢 Now we would get pools this way --> PoolsManager.getPool(poolId);
    PoolsManager public poolsManager; // <------- 🟢 To access pools info
    Loan[] public loans;

    constructor(address _poolsManager) Ownable(msg.sender) {
        poolsManager = PoolsManager(_poolsManager); // <-- 🟢 Constructor should change etc etc
    }

    function setLenderFee(uint256 _fee) external onlyOwner {}
    //REST OF FUNCS RELATED TO LENDING OPERATIONS
}
```

_**`PoolsManager.sol`**_:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;
// Needed imports...

contract PoolsManager is IPoolsManager, Ownable {
    uint256 public constant MAX_INTEREST_RATE = 100000;
    uint256 public constant MAX_AUCTION_LENGTH = 3 days;

    mapping(bytes32 => Pool) public pools;

    //  🟢 HERE!! Notice Owner should be the Lender.sol contract
    constructor() Ownable(msg.sender) {}

    function getPoolId(address lender, address loanToken, address collateralToken)
        public
        pure
        returns (bytes32 poolId)
    {}

    // 🟢 HERE!! NOTICE THE NEED FOR A NEW FUNCTION THAT ALLOWS INTERACTION BETWEEN LENDER AND POOLSMANAGER CONTRACTS
    function getPool(bytes32 poolId) public view returns (Pool memory) {
        return pools[poolId];
    }

    // REST OF FUNCS RELATED TO POOL MANAGEMENT OPERATIONS ...
}
```

_**`ILender.sol`**_:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;
// + Some extra imports...
import "../src/utils/Structs.sol";

contract ILender {
    // Lender related events...
    event Borrowed();
    event Repaid();
    // Function signatures + documentation. NatSpec used is recommended.
}
```

_**`IPoolsManager.sol`**_:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;
// + Some extra imports...

contract IPoolsManager {
    // PoolManagement related events...
    event PoolCreated(bytes32 indexed poolId, Pool pool);
    event PoolUpdated(bytes32 indexed poolId, Pool pool);
    // Function signatures + documentation.
}
```

</details>

---

## Impact 📈

DaC and SoR offer multiple benefits, including:

- They make error detection more traceable, easier debugging.
- Better readability when declaring the intentions of each part of the codebase.
- If an error occurs, these paradigms can help prevent its propagation throughout the system.

---

## Tools Used 🛠️

- Manual audit.
- Solidity Visual Developer VSPlugin for visualizing function dependencies.

---

## Recommendations 🎯

Allocate dedicated time during the development phase to strategic codebase design, emphasizing clarity and scalability. Such an approach, though possibly more time-consuming initially, long-term its more efficient for extensive projects like this one.
