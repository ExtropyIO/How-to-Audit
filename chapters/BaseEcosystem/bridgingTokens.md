# Bridging ERC20 from Ethereum to Base
Developers that already have an ERC-20 token deployed on Ethereum and would like to move to Base should go through only a simple verification process in order to be listed in a special token list on Base. Indeed it exists a list that tracks all the tokens deployed on both the Base network and OP Mainnet called [Optimism Superchain token list](https://github.com/ethereum-optimism/ethereum-optimism.github.io).

For adding the token to this list you need:
1. Deploy the token on Base using a bridging framework. There are several ways to do this but for smoothing the approval process to the list it's recommended to use Base's [standard bridge](https://github.com/ethereum-optimism/specs/blob/main/specs/protocol/bridges.md) contracts and furthermore deploy your token using the [OptimismMintableERC20Factory](https://basescan.org/address/0xF10122D428B4bc8A9d050D06a2037259b4c4B83B). More [here](https://github.com/ethereum-optimism/optimism-tutorial/tree/01e4f94fa2671cfed0c6c82257345f77b3b858ef/standard-bridge-standard-token).
2. Make a sumbission for your token to be inserted in the Superchain token list following [instructions](https://github.com/ethereum-optimism/ethereum-optimism.github.io) on github.
3. Await for the approval.
