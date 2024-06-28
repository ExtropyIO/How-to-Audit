# Normie Memecoin hack
[Normie Token](https://basescan.org/address/0x7f12d13b34f5f4f0a9449c16bcd42f0da47af200) is a well known memecoin deployed on Base, an Etherum optimistic rollup created to improve Ethereum scalability and transaction throughput while leveraging its security and decentralization. Before the exploit, NORMIE was among the top meme coins on Base with nearly 90,000 token holders.\
Normie has no intrinsic value and its price follows only the market hype guided by the existing community around the project.\
On May 26 2024 Normie was hacked losing 99% of its price and 40 million dollars in market cap in few hours, causing a huge profit for the [attacker](https://basescan.org/address/0xf7f3a556ac21d081f6dba961b6a84e52e37a717d) which later decided to return the 90% of the stolen funds keeping only 10% for him as a bug bounty.

The [attacker](https://basescan.org/address/0xf7f3a556ac21d081f6dba961b6a84e52e37a717d) who exploited the contract leveraged a vulnerability in the [contract's code](https://vscode.blockscan.com/base/0x7f12d13b34f5f4f0a9449c16bcd42f0da47af200) that allowed him to mint unlimited supply of Normie tokens.
Indeed, inside the Normie token contract the transfer function checks whether the current transfer is a buy operation from the pair and if so it will mint new tokens to the Normie Contract if the recipient has a special role, namely it's a "premarket user". This essentially mints new Normie tokens to the Normie contract itself when users are buying from the pair since the sender and recipient balances are updated separately as you can see from the following code.
```solidity
 function _transfer(
        address sender,
        address recipient,
        uint256 amount
    ) private returns (bool) {
            ... // code
            if (
                isMarketPair[sender] &&
                !isExcludedFromFee[recipient] &&
                premarket_user[recipient] // checks that the recipient is a "premarket user"
            ) {
                _balances[address(this)] = _balances[address(this)].add(amount); // contract balance is increased
            }
            ... // code 
            _balances[sender] = _balances[sender].sub(
                amount,
                "Insufficient Balance"
            ); // sender balance decreased
            uint256 finalAmount = (isExcludedFromFee[sender] ||
                isExcludedFromFee[recipient])
                ? amount
                : takeFee(sender, recipient, amount);
            if (checkWalletLimit && !isWalletLimitExempt[recipient])
                require(balanceOf(recipient).add(finalAmount) <= _walletMax);
            _balances[recipient] = _balances[recipient].add(finalAmount); // recipient balance increased
            _get_premarket_user(recipient, amount); // VUlNERABLE POINT
            emit Transfer(sender, recipient, finalAmount);
            return true;
        
    }
```

As shown in above snippet a call to `get_premarket_user()` is made. That function is supposed to check caller permissions: if the caller is a "premarket_user" then it should exist in `premarket_user[]` mapping. 
```solidity
    function _get_premarket_user(address _address, uint256 amount) internal {
        premarket_user[_address] = !premarket_user[_address]
            ? (amount == balanceOf(teamWalletAddress))
            : premarket_user[_address];
    }
```
The problem is that if the caller is not already a "premarket user", such role is granted to him by only ensuring that the transfer amount is exactly the same of the amount of tokens in the `teamWalletAddress`. Since balances are public, all the work that an attacker done was to call the transfer function the first time to transfer the same amount of tokens of the `teamWalletAddress` in order to give to his [attacker contract](https://basescan.org/address/0xef0ba56da26b4ddfef0959c1d0fc7a73a908befc) the `premarket_user` role. 

Once gained such privilege, using again the transfer function, he was able to mint new Normie tokens to the Normie Contract and sell them on the market triggering the `swapAndLiquify()` mechanism, causing the token price to dump. Indeed, as later [stated](https://basescan.org/tx/0xac6e9db480ec98b7b627ea56c3acd5e5b379a97908370471aa4d4dc4f1dc6664) by the attacker himself, he only profited by the price change: " None of the minted tokens went to me, I simply profited from the price change due to the dump (using a flash loan to sell it short)."

Leveraging the [attacker contract](https://basescan.org/address/0xef0ba56da26b4ddfef0959c1d0fc7a73a908befc) he was able to withdraw a huge amount of ETH through several transactions using the same sequence of operations:
- transfer a small amount of ETH (usually 1 or 2) from attacker account to the malicious contract . Example txn [here](https://basescan.org/tx/0x50a8d2856c211ab076049fdb3aa2f49c1fac124d04f5a98444c340e998326a11).
- invoke a function with selector `0xbc43d77c` on the malicious contractswapping transferred ETH to Normie tokens on Sushi Swap. Example txn [here](https://basescan.org/tx/0xf757229e41d7a14a71497ce901bbe1415ea346361cac7bd67fface7e78375820).
- invoke a function with selector `0x4b293bf9` on the malicious contract swapping the received Normie tokens on the [Normie/WETH](https://basescan.org/address/0x24605e0bb933f6ec96e6bbbcea0be8cc880f6e6f) pool gaining profit. Example txn [here](https://basescan.org/tx/0x1e9f6075576c1ec59fcb4082bce8c86d3e0e2df649c03d2066de46582d300296) and a [diagram](https://app.blocksec.com/explorer/tx/base/0x1e9f6075576c1ec59fcb4082bce8c86d3e0e2df649c03d2066de46582d300296) that helps visualizing fund flows.
- call a fuction named `withdraw()` to transfer back the profited ETH from the malicious contract to the attacker address
  

After repetead the above process several times, the attacker then [moved](https://basescan.org/tx/0x8b069069e58d8428c0340ee4ef0aab8114004970dc84cfcc86aa699c3c8ef023) stolen funds to a [temporary address](https://basescan.org/address/0xbDfCaA1c260D35a57aE8C333AFff4D8dC6D90899). Then he [offered](https://basescan.org/tx/0x587f14b7ffb30b5013ab0db02e9bc94183817ef34c24a9595f33277e752f81eb) to return the 90% of the stolen funds to reimburse Normie holders and keep the other 10% for him as a bounty only if the dev team accepted to refund Normie token holders with the ETH they gained through `swapAndLiquify()` mechanism.

Normie dev team [accepted](https://x.com/NormieBase/status/1794679630408122388?t=T20ur_Q4m6q5GQn3KVx6gg&s=35) the hacker's offer and on June 6 a new version of the contract was deployed at [0x47b464edb8dc9bc67b5cd4c9310bb87b773845bd](https://basescan.org/address/0x47b464edb8dc9bc67b5cd4c9310bb87b773845bd).

Finally Normie dev team [started](https://x.com/NormieBase/status/1798797799791755701) refunding incident victims and the attacker [returned](https://basescan.org/tx/0x778d5d63ea834b87a026e68a89b9e41997712f5158bff413c9fe2582db438273) the 90% of the stolen funds (192 ETHs).

Resources: 
- https://x.com/0xGoldenDegen/status/1794621170547126698
- https://x.com/ProfoundWatcher/status/1794629010531693005
- https://www.immunebytes.com/blog/list-of-largest-crypto-hacks-in-2024
- https://neptunemutual.com/blog/taking-a-closer-look-at-normie-exploit/