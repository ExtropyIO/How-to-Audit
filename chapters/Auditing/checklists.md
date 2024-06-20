## Audit checklists
Used mainly [this](https://github.com/transmissions11/solcurity) and [this](https://github.com/securing/SCSVS/tree/master/1.2). 

### Variables checklist
- Is the name of the variable aligned with its logic and coherent? Avoid variables with similar names.
- Are all state variables initialized?
- Are all state variables used? Get rid of dead code.
- Is the visibility set correctly?
- If the variable is `private`, are there sensitive information stored? Anything on the blockchain is public and anyone can read the content of private variables accessing contract slots manually.
- Can it be marked as `immutable` or `constant`?
- Is the struct/array state variable updated in functions using `storage`? 
- Are duplicates possible within an array variable? Should they be?
- Mappings enforce a unique key. Does this match the logic/data model? 
- Is the variable being shadowed?
- Are magic numbers replaced by a `constant` with an intuitive name?
- Is the `delete` keyword being used instead of setting default values manually for variables?
- Is the variable datatype large enough to handle any expected value?
- Will casting lead to lack of precision?
- If ints are used, is precision loss possible? (especially for year/month/day calculation) 

### Functions checklist
- Is the name of the function aligned with its logic and coherent with other functions?
- Is the visibility set correctly? Avoid using `public` if functions are meant to be called only externally.
- Should the function be `payable` or not?
- Is `msg.value` used properly and validated?
- If `virtual` is used, is it really necessary? 
- Are access controls set for the function?
- Are the correct modifiers used?
- Are all functions used? Get rid of dead code.
- Is the function logic too complex? If yes split in smaller and simpler functions.
- Are all the inputs checked and validated to be within desired bounds?
- Are all the function preconditions checked with require statements as the first thing?
- Can edge case inputs (0, max) result of an unexpected behaviour ?
- Is reentrancy possible?
- Does the function follow the Checks-Effect-Interaction pattern?
- Is the `ReentrancyGuard` from OZ used only for single-function reentrancy? It cannot be enough for cross-contract reentrancy for example.
- If the function is frontrunned or backrunned, what will be the consequences?
- If the function is a critical, does it emit the proper event?

### Modifiers checklist
- Is the name of the modifier aligned with its logic?
- Does the modifier use the function code placeholder `_` in the correct point?
- Are external call made in the modifier?

### Calls checklist
- Is the external contract call actually needed?
- Is the callee an untrusted contract or retrieved from user input?
- How much gas is forwarded in the call?
- Can the callee return a huge return value in data size to run the caller out of gas?
- Is the called function present in the callee contract? If not, what the fallback function of the called contract do?

### Loops and control structure checklist
- Is the number of iteration limited?
- Are any unbounded loops/arrays used that can cause DoS? 
- Is the array length being updated while iterating on it?
- Are the first and the last iterations of the loop safe?
- Is the order in ternary operators correct?
- Does the loop use a valid stop condition?
- Is there a call inside the loop?

### Events checklist
- Are all the relevant informations added as event fields?
- Are only the proper event fields marked as indexed?
- Is the author of the relevant action included in the emitted data?
- Are events emitted in all the critical functions?

### Signatures checklist
- Are signatures protected against replay attacks with a `nonce` and `chainid`?
- Does the signature use `EIP-712`? 
- Offline signatures are vulnerable to phishing attacks. See [here](https://zengo.com/offline-signatures-can-drain-your-wallet-this-is-how-part-1-2/) and [here](https://zengo.com/offline-signatures-can-drain-your-wallet-including-erc-20-tokens-this-is-how-part-2-2/).


### Math calculations checklist
- Is multiplication done before division to avoid loss of precision?
- Is math within an `unchecked` block really safe from underflows/overflows?
- Is the math for calculating any fee correct?
- Is division by 0 possible? Consider user inputs or integer arithmetic that could produce that unexpectedly
- Is the math implementing the algorithm logic correctly?
- If using extreme values (e.g. maximum and minimum values of the variable type) in the formula, can they break something?

### Tokens checklist
- Are token decimals checked to be not too low or not to high?
- Are interactions with standard 18 decimals tokens safe?
- What happens in the accountings if tokens that take fees on transfers are used?
- What happens if [weird ERC-20 tokens](https://github.com/d-xo/weird-erc20) are being used in the protocol?
- What happens if rebase tokens are being used in the protocol?
- Is reentrancy possible when using `onERC721Received()` with ERC721 tokens?
- Is reentrancy possible with ERC-777 token hooks?
- Is the return value of a transfer checked?
- Is zero amount transfer allowed to be ERC-20 standard compliant? Some token can revert with zero token transfer and therefore be used to cause a DOS
- Is a transfer to address zero denied? Even if `burn()` should be used, this may break compatibility in some cases.
- Are `increaseAllowance()` and `decreaseAllowance()` used against approval race condition?
- Are safe versions functions used?
- Can tokens be locked in the contract?
- Does the contract approve tokens before using `transferFrom()` to avoid revert?
- When there is a `balanceOf(address(this))`, can manually sending tokens break something ?
- Solmate `ERC20.safeTransferLib` does not check the contract existence 
More related to tokens [here](https://gist.github.com/shayanb/cd495e23c7cf1a8b269f8ce7fd198538).
 
### Generic checklist
- Is the contract inheritance simple and correct? Avoid over-inheritance.
- Is the order of inheritance respected?
- Does the contract use self-made libraries? For example avoid creating a custom `Ownable` contract: use the one from OZ.
- Is the contract using Solidity 0.8 or above for checking math? If not use the `SafeMath` library from OZ.
- Are only needed libraries and packages imported?
- Is the contract using `abstract` keyword if it should not be deployed directly but only inherited?
- Is `block.timestamp` used to check long time intervals?
- Is true randomness being used? Does not rely on block hashes for randomness because miners can influence it.
- Is assembly code being used?
- Does the code assume a specific ETH balance? Remember that addresses can receive ETH even without `payable` functions or `receive()`
- Is the contract mixing internal accounting with actual balances?
- Are comparison operators used correctly (>, <, >=, <=), especially to prevent off-by-one errors?
- Is `tx.origin` being used for authorization?
- Is `extcodehash` compared to zero to check wehther an address is a Contract or EOA? 
- Is `selfdestruct` being used? Check also that `selfdestruct` behavior changed recently.
- Is the contract relying on pool prices instead of using oracles?
- Is the ownership change a two step process?
- Is there any front run opportunities? Be aware of sandwich attack on Vaults and DEXes
- Is a rug pull possible?
- Is there a centralisation risk? How much power is in the hands of the deployer or the contract owner?
- Are there compiler errors or warnings?
- Is latest solidity version available used?
- Specific solidity versions have specific bugs. [Here](https://github.com/ethereum/solidity/blob/develop/docs/bugs_by_version.json) a list of known bug categorized by solidity version.
- Are Ether amounts dealt in wei units?
- Can the contract be permanently paused? 
- Is key management effective? Are key leaks possible?

### DeFi checklist
- [Vaults] Can transferring ERC20 or ETH directly break something ?
- [Vaults] Are the vault balance tracked internally ?
- [Vaults] Can the 1st deposit raise a problem ?
- [Vaults] How the vault behave when locked funds are put in a strategy
- [Vaults] Is this vault taking into consideration that some ERC20 tokens are not 18 decimals ?
- [Vaults] Is the fee calculation correct ?
- [Vaults] What if only 1 wei remains in the pool ? 
- [Vaults] On a vault with strategies implemented : flash deposit-harvest-withdraw attacks are possible [reaperVaultV2](https://substack.com/redirect/d45a46d8-603e-4d28-8baf-14351ae218d5?j=eyJ1IjoiaXA3ZyJ9.bsUc99wpXzmKbwuyaCiqZeEfgGe5m-VWUAd-5x0-Fy0) handles that scenario.
- [Staking/Locktimes] Can users collaborate to reduce the lock time?
- [Chainlink/VRF] See [VRF Security Considerations](https://substack.com/redirect/51e02658-81b5-4a90-b116-3231ce6f4846?j=eyJ1IjoiaXA3ZyJ9.bsUc99wpXzmKbwuyaCiqZeEfgGe5m-VWUAd-5x0-Fy0)
- [Uniswap] Is there an expiration deadline for swapping?
- [Uniswap] Incorrect Slippage Calculation ?
See [guidelines](https://substack.com/redirect/a3b706ca-3dc0-4a0e-a79e-03d1f77817f6?j=eyJ1IjoiaXA3ZyJ9.bsUc99wpXzmKbwuyaCiqZeEfgGe5m-VWUAd-5x0-Fy0)
- [Borrowing] Liquidation should not be possible without default, for example before the first payment is due.
- [Borrowing] Liquidation must be possible and not prevented by the borrower.
- [Borrowing] Check that the borrower can only close their position after the entire principle and interest is paid back.
- [Borrowing] Pausing of repayments and liquidations is out of sync.
