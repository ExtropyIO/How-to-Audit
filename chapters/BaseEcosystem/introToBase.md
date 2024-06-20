# BASE overview
[Base](https://www.base.org/) is an Ethereum Layer 2 created to improve Ethereum scalability and transaction throughput while leveraging its security and decentralization. Among the different types of Layer 2s, Base belongs to what are called `Rollups` in which multiple off-chain transactions are aggregated into a single on-chain data representation and submitted to the main Ethereum chain. 

Particularly, Base is an "optimistic rollup" and since it's build on the [OP stack](https://docs.optimism.io/) it inherits much of the features of Optimism mainnet. Optimism uses a "[fraud-proof](https://www.paradigm.xyz/2021/01/how-does-optimisms-rollup-really-work#fraud-proofs)" system, which assumes as valid all the aggregated rollup transactions. If then a user is able to submit a valid fraud proof within a specified time window, then the transaction is rolled back and who submitted the malicious transaction is penalized.

## Pros and cons
There are are several reasons to start building on Base. According to [Alchemy](https://www.alchemy.com/base):
1. **EVM Equivalence**. Base uses the Optimistic Virtual Machine (OVM) for smart contract execution which is built to be close as possible to the Ethereum Virtual Machine (EVM) ensuring limited efforts for developers moving to Base ecosystem, deploy contracts on it while keeping in use existing Ethereum-based tools and frameworks. Smaller differences between the two Virtual Machines will be highlighted in later sections.
2. **Low fees**. Like other optimistic rollups, Base ensure very low transaction fees. In average a transaction on Base is 10 times cheaper than on Ethereum. A transaction made on a L2 consists in two components: an execution fee payed on the L2 (in ETH or ERC-10) and a security fee payed on L1 (in ETH). More [here](https://www.alchemy.com/overviews/choose-base#:~:text=4.-,Low,-Transaction%20Costs).
3. **Scalability**. Layer 2s scalability offers high TPS and reduces bottlenecks. Base can theoretically scale the transaction throughput from 10x to 100x compared to Ethereum TPS. Indeed OP Stack based chains would ideally be able to process up to 2000 TPS.

At the same time is possible to highlight some concerns. As reported by [BinanceAcademy](https://academy.binance.com/en/articles/what-is-base-coinbase-layer-2-network):
1. **Centralization**. Base is a product of [Coinbase](https://www.coinbase.com/it), which controls the only sequencer node, giving them considerable control over L2 transactions ordering. Additionally, this centralized authority has the ability to set and modify fees associated with the Coinbase Sequencer. 
2. **Long withdrawal periods**. As stated previously there is a specified time window after a transaction is executed to submit a valid fraud proof. This time window lasts around 7 days and thus users are not able to withdraw funds until such time passed after their transaction.
3. **Security**. Base uses the fraud-proof system of Optimism, which relies on the vigilance of network participants to monitor and challenge any invalid off-chain transactions before they are finalized on the main blockchain. There is no 100% confidence that all malicious transaction will be detected.

## Differences with Ethereum
Auditing smart contracts on Base means using the auditing knowledge and expertise for EVM contracts plus some concepts to digest and some minor differences between the two stacks. Some differences could be for example that in average the time needed to produce blocks on Etherum is 12 seconds while is 2 for OP Stack based chains or that the theoritcal TPS on Base is way bigger rather than Ethereum ([Source](https://chainspect.app/compare/base-vs-ethereum)).

Also, on a contract level, there may be differences: using `msg.sender` on a Base contract for example may result in a different outcome than the same code executed on the EVM. Let's explore that a little further.

### Address Aliasing
Address Aliasing is a way to match address between L1 and L2. When an EOA sends a transaction from L1 to L2 the sender of both transaction is set to be sender of the L1 transaction. This is not true if the address who started the transaction from L1 to L2 is a Contract.
This because Contracts can have the same address on both L1 and L2 but have completely different bytecodes. There is the need to map the Contract address who generated the transaction on L1 to the corresponding contract address on the L2.

In practice the sender of the transaction on L2 will result in an "aliased" version of the L1 contract address. This aliased address is a constant offset from the actual L1 contract address such that the aliased address will never conflict with any other address on L2 and the original L1 address can easily be recovered from the aliased address.

This change in sender address is only applied to L2 transactions sent by L1 smart contracts. In all other cases, the transaction sender address is set according to the same rules used by Ethereum.

### Mempool
Another important difference to highlight is that unlike Ethereum, Base does not have a large public mempool. This because of the nature of the OP stack on which Base is built. Indeed Base is essentially a rollup with a couple of relevant components, like the **Sequencer**. It's a single and centralized entity that is the only one able to see transactions in the layer 2 mempool which is not public (Ethereum L1 mempool != Base L2 mempool). The sequencer is responsible to queue and processe the valid transactions in priority fee order (highest fee first).


### OPCODE differences
As stated previously the Optimism Virtual Machine (OVM) is built to be closer as possible to the Ethereum Virtual Machine (EVM). However having 100% match in not possible. Inded for example there are some new OPCODES in the OVM that are not in the EVM, or OPCODES that behave differently on the OVM rather than the EVM. This is important to keep in mind since the same solidity code may have slightly different results depending on the execution environment.

Some examples, like **ORIGIN** and **CALLER**, are reported in the [Optimism docs](https://docs.optimism.io/stack/differences#opcode-differences).