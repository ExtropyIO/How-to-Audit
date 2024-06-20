# Normie Memecoin hack
On May 26 2024 the memecoin Normie deployed on Base was hacked losing 99% of its price and 40 million dollars in market cap in few minutes.

The attacker who exploited the contract leveraged a vulnerability in the code that allowed him to mint unlimited supply of Normie tokens. After having dumped everything on the market he offered to return the 90% of the stolen funds to reimburse Normie holders and keep the other 10% for him as a bounty.

Entities involved:
- Normie token: [0x7f12d13b34f5f4f0a9449c16bcd42f0da47af200](https://basescan.org/address/0x7f12d13b34f5f4f0a9449c16bcd42f0da47af200)
- Normie token contract code: https://vscode.blockscan.com/base/0x7f12d13b34f5f4f0a9449c16bcd42f0da47af200 
- Attacker: [0xf7f3a556ac21d081f6dba961b6a84e52e37a717d](https://basescan.org/address/0xf7f3a556ac21d081f6dba961b6a84e52e37a717d)
- Attacker's Contract: [0xef0ba56da26b4ddfef0959c1d0fc7a73a908befc](https://basescan.org/address/0xef0ba56da26b4ddfef0959c1d0fc7a73a908befc)

Relevant transactions:
- One of exploit transactions: [0xa00e72d7fe4ebb125945e00477b9f226f275c4524607ad8c847f2bb067c97ce0](https://basescan.org/tx/0xa00e72d7fe4ebb125945e00477b9f226f275c4524607ad8c847f2bb067c97ce0)
- Attacker proposal to refund Normie Holders after exploit: [0x587f14b7ffb30b5013ab0db02e9bc94183817ef34c24a9595f33277e752f81eb](https://basescan.org/tx/0x587f14b7ffb30b5013ab0db02e9bc94183817ef34c24a9595f33277e752f81eb)

Profit:
- Around 224 WETH (~$881,686)

Vulnerability:\
Inside the Normie token contract the transfer function checks whether the current transfer is a buy operation from the pair and if so it will mint new tokens to the Normie Contract if the recipient has a special role. This essentially mints new Normie tokens when devs are buying from the pair.
The vulnerability relies in a function named `get_premarket_user()`, called within each transfer, which is supposed to check caller permissions. Indeed that function checks if the caller is a "premarket_user", namely exist in `premarket_user[]` mapping. 
```solidity
    function _get_premarket_user(address _address, uint256 amount) internal {
        premarket_user[_address] = !premarket_user[_address]
            ? (amount == balanceOf(teamWalletAddress))
            : premarket_user[_address];
    }
```
The problem is that if the caller is not already a "premarket user" such permission are granted to him by only ensuring that the buying amount is exactly the same of the amount of tokens in the `teamWalletAddress`. Since balances are public, all the work that the attacker done was to buy the same amount of tokens of the `teamWalletAddress` in order to give to his attacker contract the `premarket_user` role. Once gained such privilege he was able to mint unlimited Normie tokens to the Normie Contract and sold them on the market triggering the `swapAndLiquify()` mechanism.

Project that forked Normie's code are vulnerable to the same exploit. A list of similar contracts can be found on TokenSniffer [here](https://tokensniffer.com/token/base/qzu6n2603j0sz1z5szx3u9hssh5ib6mdkjbl2dsnz4gulx29sq9f5znpxdgx).


Resources: 
- https://x.com/0xGoldenDegen/status/1794621170547126698
- https://x.com/ProfoundWatcher/status/1794629010531693005
- https://www.immunebytes.com/blog/list-of-largest-crypto-hacks-in-2024
