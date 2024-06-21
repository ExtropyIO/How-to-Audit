# Boomer token audit


## Project Details
Token Address: [0xcde172dc5ffc46d228838446c57c1227e0b82049](https://basescan.org/address/0xcde172dc5ffc46d228838446c57c1227e0b82049)\
Token Code: https://vscode.blockscan.com/base/0xcde172dc5ffc46d228838446c57c1227e0b82049

## Findings

- [Boomer token audit](#boomer-token-audit)
  - [Project Details](#project-details)
  - [Findings](#findings)
    - [\[HIGH\] Centralization risks](#high-centralization-risks)
    - [\[MEDIUM\] `swapBack()` is useless if `buyTotalFees` or `sellTotalFees` are not set at a certain point in time](#medium-swapback-is-useless-if-buytotalfees-or-selltotalfees-are-not-set-at-a-certain-point-in-time)
    - [\[MEDIUM\] Transferring ownership should be a two steps process](#medium-transferring-ownership-should-be-a-two-steps-process)
    - [\[LOW\] `swapBack()` timing is predictable](#low-swapback-timing-is-predictable)
    - [\[LOW\] Priviliges are not revoked for old owner when ownership is transferred](#lowpriviliges-are-not-revoked-for-old-owner-when-ownership-is-transferred)
    - [\[MEDIUM\] Missing input validation in setNewFees()](#medium-missing-input-validation-in-setnewfees)
    - [\[INFO\] Sandwich attack in `swapTokensForEth()`](#info-sandwich-attack-in-swaptokensforeth)
    - [\[INFO\] Dead code](#info-dead-code)

### [HIGH] Centralization risks
- **Location(s):** BOOMER.sol#1171-1176, BOOMER.sol#1168

- **Description:** The functions `rescueETH()` and `rescueERC20()` allow the contract owner to withdraw all ETH and ERC20 tokens from the contract at any time.

    ```solidity
    function rescueETH(uint256 weiAmount) external onlyOwner {
        require(weiAmount > 0, "Amount must b over 0");
        payable(owner()).transfer(weiAmount);
    }

    function rescueERC20(address tokenAdd, uint256 amount) external onlyOwner {
        require(tokenAdd != address(0), "wrong token address");
        require(amount > 0, "Amount must be > 0");

        IERC20(tokenAdd).transfer(owner(), amount);
    }
    ```
     Furthermore, the function `swapBack` swaps the entire contract's token balance to ETH if it goes above the `tokenSwapThreshold` and then sent to `devWallet` causing a possible centralization risk. 

    ```solidity
        function swapBack() private {

        ... // code
       
        (success, ) = address(devWallet).call{value: address(this).balance}("");
    }
    ```


- **Recommendation:**  Consider implementing the following solutions:
  - Multi-signature wallet with at least (2/3 keys) for critical roles like `owner` (e.g. Gnosis safe)
  - Time-lock contract with reasonable latency hours for critical operations (e.g. Compound TimeLock)
  - A medium/blog link for sharing the timelock contract and multi-signers addresses information with the public audience.



### [MEDIUM] `swapBack()` is useless if `buyTotalFees` or `sellTotalFees` are not set at a certain point in time
- **Location(s):** BOOMER.sol#1149

- **Description:** The `swapBack()` function is called within `_transfer()` but does nothing, due to a return statement, if one between `contractBalance` or `totalTokensToSwap` is zero. 

    ```solidity
     function swapBack() private {
        uint256 contractBalance = balanceOf(address(this));
        uint256 totalTokensToSwap =  taxedTokens;

        if (contractBalance == 0 || totalTokensToSwap == 0) {
            return;
        }

        ... // other code
    ```
    Particularly `totalTokensToSwap` reflects the current value of the state variable `taxedTokens` which is incremented with the value of `fees` computed on the amount being transferred in/out from the pair, which means sell/buy operations.

    ```solidity
    function _transfer(address from, address to,uint256 amount) internal override {
        ... // other code

        if(fetchFees){
            getFees(); 
        }

        // Sell
        if (to == UNISWAP_V2_PAIR && sellTotalFees > 0) {
            fees = (amount * sellTotalFees) / 100;
            taxedTokens += fees;
        }
        // Buy
        else if (from == UNISWAP_V2_PAIR && buyTotalFees > 0) {
            fees = (amount * buyTotalFees) / 100;
            taxedTokens += fees;
        }

        ... // other code
    }
    ```

    However the value of `fees` will be always zero until `buyTotalFees` or `sellTotalFees` are zero, causing `taxedTokens` and thus `totalTokensToSwap` to be zero. 
    
    This scenario is feasible since `getFees()` will set `buyTotalFees` and `sellTotalFees` to zero for every buy/sell operation conducted within the first 7 blocks after having opened trading. Unitl `setNewFees()` is not executed with non zero fee values after 7 blocks passed, `swapBack()` will be useless since it will not allow to swap back even if the contract token balance is above the `tokenSwapThreshold`.
     
- **Recommendation:**  Consider ensuring that `swapBack()` logic is executed when `contractBalance` is greater than `tokenSwapThreshold` even if `totalTokensToSwap` is still zero.



### [MEDIUM] Transferring ownership should be a two steps process
- **Location(s):** BOOMER.sol#723

- **Description:** In the `Ownable` contract from OZ, the `_transferOwnership()` function directly set the state variable `_owner` with the address specified in the input `newOwner` without ensuring that `newOwner` is actually a valid address.

    ```solidity
    function _transferOwnership(address newOwner) internal virtual {
        address oldOwner = _owner;
        _owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }
    ```
     
- **Recommendation:**  Instead of modifying the state directly consider adding a new function named `_updateOwnership()` which checks whether `_newOwner` is `msg.sender`, which means `_newOwner` signs the transaction and verifies himself as the new owner. After that, `_newOwner` could be set into `_owner`.

### [LOW] `swapBack()` timing is predictable
- **Location(s):** BOOMER.sol#1149

- **Description:** The function `swapBack` swaps the entire contract's token balance to ETH if goes above the `tokenSwapThreshold` and then sent to `devWallet`. 

However the timing of the swap operation conducted by the contract between BOOMER tokens and ETH is predictable and allows users to execute trades before/after it to make profits.
 

- **Recommendation:**  Consider ensuring that `swapBack()` logic is executed with a less predictable timing such as randomizing timing execution, using a variable threshold level or performing a gradual swap.


### [LOW] Priviliges are not revoked for old owner when ownership is transferred
He is excluded from max transaction and excluded from Fees that he can set to the maximum each time
- **Location(s):** BOOMER.sol#924

- **Description:** Contract deployer, which is set to be the contract owner, has several priviliges like being excluded from transaction amount limit or being excluded from paying fees when buying or selling to the pool.

    ```solidity
        _excludeFromMaxTransaction(msg.sender, true);
        excludeFromFees(msg.sender, true);
    ```
    However when the ownership is transferred from the initial token deployer to a new owner address, such priviliges are not granted to the new owner and revoked from the old owner.
     
- **Recommendation:** Consider granting owner priviliges to the new owner and revoking from the old owner when the ownership tranfer occurs.

### [MEDIUM] Missing input validation in setNewFees()
- **Location(s):** BOOMER.sol#987

- **Description:** The inputs `newBuyFees` and `newSellFees` in input to `setNewFees()` are not validated to be in the range [0 ; 100].

    ```solidity
    function setNewFees(uint256 newBuyFees, uint256 newSellFees) external onlyOwner {
    buyTotalFees = newBuyFees;
    sellTotalFees = newSellFees;
    } 
    ```
    Indeed if one of them is set to be greater than 100, and `fetchFees` is equal to zero then the amount of fees sent within `_transfer()`  to `address(this)` in a buy/sell operation will be more than the actual `amount` being swapped.

    Finding severity is set to MEDIUM since the owner is able to change such parameters at any time.

- **Recommendation:**  Consider validating inputs in `setNewFees()`.

### [INFO] Sandwich attack in `swapTokensForEth()`
- **Location(s):** BOOMER.sol#1134

- **Description:** The function `swapTokensForEth()` uses `swapExactTokensForETHSupportingFeeOnTransferTokens()` without setting an `amountOutMin`, leaving it as 0.

    ```solidity
     function swapTokensForEth(uint256 tokenAmount) private {
        
        address[] memory path = new address[](2);
        path[0] = address(this);
        path[1] = UNISWAP_ROUTER.WETH();

        UNISWAP_ROUTER.swapExactTokensForETHSupportingFeeOnTransferTokens(
            tokenAmount,
            0, 
            path,
            address(this),
            block.timestamp
        );
    }
    ```
  A sandwich attack is an attack where an attacker that spots such transaction in the mempool can execute one transaction before buying the same asset at lower price and another one later selling tokens at higher price making a profit on behalf of the victim.\
  Setting no restriction on the slippage through `amountOutMin` means that the transaction will accept any new price that was manipulated by attacker.

  Finding severity is set to INFORMATIONAL since on a layer 2 such Base the Sequencer responsible of transaction ordering, although centralized, is trustworthy and considered to act not maliciously.

- **Recommendation:**  Consider setting reasonable minimum output amounts, instead of 0 for `amountOutMin`.

### [INFO] Dead code
- **Location(s):** BOOMER.sol#895

- **Description:** The event `SwapAndLiquify` is never emitted in the codebase.

    ```solidity
    event SwapAndLiquify(
        uint256 tokensSwapped,
        uint256 ethReceived,
        uint256 tokensIntoLiquidity
    );
    ```
    The mapping `whitelisted` is never used in the codebase

    ```solidity
    mapping(address => bool) public whitelisted;
    ```

- **Recommendation:**  Consider removing unused code in order to enhance readability.

