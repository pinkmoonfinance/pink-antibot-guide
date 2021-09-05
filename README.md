## PinkAntiBot integration guide

1. Add this interface to your codebase:

`IPinkAntiBot.sol`
```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.5.0;

interface IPinkAntiBot {
  function setTokenOwner(address owner) external;

  function onPreTransferCheck(
    address from,
    address to,
    uint256 amount
  ) external;
}
```

2. Update your token contract:

```diff
+import "path/to/IPinkAntiBot.sol";

contract MyToken {
+ IPinkAntiBot public pinkAntiBot;

  constructor(
    string memory name_,
    string memory symbol_,
    uint8 decimals_,
    uint256 totalSupply_,
+   address pinkAntiBot_ 
  ) {
    ... omitted for clarity

    // Create an instance of the PinkAntiBot variable from the provided address
+   pinkAntiBot = IPinkAntiBot(pinkAntiBot_);
    // Register the deployer to be the token owner with PinkAntiBot. You can
    // later change the token owner in the PinkAntiBot contract
+   pinkAntiBot.setTokenOwner(msg.sender);
  }

  // Inside ERC20's _transfer function:
  function _transfer(
    address sender,
    address recipient,
    uint256 amount
  ) internal virtual {
    require(sender != address(0), "ERC20: transfer from the zero address");
    require(recipient != address(0), "ERC20: transfer to the zero address");
+   pinkAntiBot.onPreTransferCheck(sender, recipient, amount);
  }
}
```

3. (Optional) If you want more control over how `PinkAntiBot` is enabled or disabled, you can do it inside your contract instead of relying on `PinkAntiBot` contract's configuration:


```diff
import "path/to/IPinkAntiBot.sol";

contract MyToken {
  IPinkAntiBot public pinkAntiBot;
+ bool public antiBotEnabled;

  constructor(
    string memory name_,
    string memory symbol_,
    uint8 decimals_,
    uint256 totalSupply_,
    address pinkAntiBot_ 
  ) {
    ... omitted for clarity

    // Create an instance of the PinkAntiBot variable from the provided address
    pinkAntiBot = IPinkAntiBot(pinkAntiBot_);
    // Register the deployer to be the token owner with PinkAntiBot. You can
    // later change the token owner in the PinkAntiBot contract
    pinkAntiBot.setTokenOwner(msg.sender);
+   antiBotEnabled = true;
  }

  // Use this function to control whether to use PinkAntiBot or not instead
  // of managing this in the PinkAntiBot contract
+ function setEnableAntiBot(bool _enable) external onlyOwner {
+   antiBotEnabled = _enable;
+ }

  // Inside ERC20's _transfer function:
  function _transfer(
    address sender,
    address recipient,
    uint256 amount
  ) internal virtual {
    require(sender != address(0), "ERC20: transfer from the zero address");
    require(recipient != address(0), "ERC20: transfer to the zero address");
    // Only use PinkAntiBot if this state is true
+   if (antiBotEnabled) {
      pinkAntiBot.onPreTransferCheck(sender, recipient, amount);
+   }
  }
}
```