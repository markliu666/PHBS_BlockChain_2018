# A Detailed Review of Ethereum

## Foreword

Name: Deming Liu
Student ID: 1801212889

By reading a lot of materials, I have an overall understanding of Ethereum and review Ethereum according to my own ideas. Due to the complexity of the Ethereum system, I strive to cover all the key points.

## 1. Overview of Ethereum:

If bitcoin represents blockchain 1.0, then Ethereum represents blockchain 2.0. Ethereum has been improved for many problems in Bitcoin, and many completely different mechanisms have been designed. Compared with Bitcoin, the main features are as follows:

(1) The bitcoin block generation time is about 60 minutes, and Ethereum will shorten the time to about a dozen seconds.

(2) Based on the consensus mechanism in Bitcoin, Ethereum has designed a more complex consensus mechanism based on GHOST protocol.

(3) The mining puzzle used for mining is very different. Bitcoin uses the consensus mechanism of POW. Ethereum combines POW and POS and has a memory hard mining puzzle. This is to limit the use of the ASIC, which means it is an ASIC resistance puzzle. POS is similar to a shareholding vote, and Ethereum developers have repeatedly claimed to use POS to completely replace POW.

(4) The code in Bitcoin is not turing-complete and cannot support complex smart contract operations. Ethereum added support for smart contracts and is turing-complete. Bitcoin is decentralized currency, while Ethereum is decentralized contract.

If we can decentralize currency, what else can we decentralize? So Ethereum supports smart contracts. Contracts are less efficient in real life. On the one hand, it is difficult to reach a consensus when signing a contract. On the other hand, contract execution is also very hard. With smart contracts, for the first point, contract signers can be global or even unknown. Moreover, the contract of the smart contract exists in the form of code, and the code is stored in the blockchain and cannot be tampered. For the second point, the contract will always be executed.

In Ethereum, there are a large number of ETH accounts. Bitcoin is a transaction-based ledger. The system does not clearly record how much money is in the account and that needs to be calculated by backtracking. The advantage of this design is that privacy can be protected. The downside is that it is not very convenient to use. When you spend money, you need to explain how each bitcoin is obtained, which is different from people's daily experience.

In contrast, Ethereum is an account-based ledger. Whether a transaction is legal or not, it is natural to check whether there is enough money in the account. It reflects the concept of balance.

The main risk in Bitcoin is the double spending attack, while Ethereum has a natural protective effect on double spending attack. Because in Ethereum, if there is any double spending attack, you can deduct money twice directly from the account. This is a dishonest act of payer, then what if the collector is dishonest? The attack is called replay attack and the collector replayed the transaction again. In fact, double spending attack and replay attack are symmetrical. In Bitcoin, why is there no problem with replay attack? Because in bitcoin replay attack will be resolved as a very obvious double spending attack. In the Ethereum, we can resolve this problem by adding the total number of transactions of an account, nonce. Nonce will plus one for each transaction. In the process of transferring the money, the nonce is sent together as part of the transaction. Like other transaction information, the nonce is also protected by the signature of the payer and cannot be modified by others. This way the replay attack will turn into two identical nonce transactions and will be rejected.

There are two types of accounts in Ethereum, externally owned account and smart contract account. The former account is protected by cryptography and uses a private key-public key pair to associate accounts. The one who owns the private key will own the account. The balance and nonce, the number of consumptions of the account, are two key parameters. The latter account, smart contract account, cannot initiate a transaction actively, and all transactions must be initiated by an externally owned account. Externally owned account initiates a transaction to invoke smart contract account. After that process, it is also allowed for smart contract account to send a message to call other smart contract account. The smart contract account not only has balance and nonce, but also code and storage. The former represents the code of the contract, and the latter represents the relevant state of the contract, including the value of each variable. So how is the smart contract account called? When creating a smart contract account, an address is returned, and the contract can be called by knowing the address. During the call, the code does not change while the storage changes.

As Ethereum supports smart contracts, participants are required to have a stable identity, which ensures that the person signing the contract is both responsible and profitable. The problem with this design is that privacy protection does not work well compared with Bitcoin. But you can also learn from Bitcoin and create multiple Ethereum accounts to protect your privacy.

## 2. Ethereum’s State Tree:

We need to find a data structure and make a mapping from the account address to the account status. The address of the Ethereum account is 160 bits, which is 20 bytes and 40 hexadecimal numbers.

In Bitcoin, thousands of transactions in a block form a merkle tree to prove that a transaction belongs to the merkle tree, which is merkle proof. As Bitcoin is transaction-based, each merkle tree will not be large. However, Ethereum is account-based. If all the accounts generate a merkle tree, each state update will regenerate the merkle tree, which is very expensive.

And the merkle tree does not provide a quick way to search and update, you need to use sorted merkle tree to speed up the search. And the merkle tree of these accounts is probably not unique, even if it is a sorted merkle tree, it is very expensive to add a new account each time.

Ethereum uses an MPT (merkle patricia trie). First of all, what is a trie tree? Trie (which is cutted from the word ‘retrieval’) is a dictionary tree and a prefix tree with key-value pair. It has several characteristics: (1) the number of branches of each node in the trie depends on the value range of the key, the branching factor ranges from '0' to 'f' plus 'end', a total of 17. (2) The efficiency of the search in trie depends on the length of the key. It must be 40 in the Ethereum. (3) As long as the address is different, there will be no collision. (4) As long as the account information is entered unchanged, the structure of the trie is unique. (5) Updating the account information is not a frequent job. Locality is important, and trie directly accesses leaf nodes.

But the disadvantage of trie is that storage is a bit wasteful. That means path compression is required, so we use a patricia trie (patricia tree). Patricia trie is a path-compressed prefix tree. The height of the tree is significantly reduced and the access efficiency is improved. Especially in the case where the key is very sparse, the path compression works well. There are 2^160 addresses in Ethereum, the number of accounts compared to the number of addresses is negligible. In line with sparse condition, patricia trie works well.

The difference between the finally used MPT (merkle patricia trie) and a common patricia trie is that the content of the merkle patricia trie is a hash value, and finally the hash value of the root node is the state root of the state tree. There are three roots in Ethereum. All Ethereum nodes maintain three trees, state tree, transaction tree and receipt tree locally. Any miner who has obtained the bookkeeping right publishes some important information packaged by himself (including the roots of the three trees) to the blockchain. Based on other information, the local trees are maintained to verify the correctness of the three roots, which is to reach consensus with others. Here, three trees cannot be sent directly to the blockchain because the scale is too large. This has three advantages in my view: (1) It can guarantee that each account information is not tampered. (2) It can use merkle proof to prove how much money in the account. (3) It can prove that a certain account which has been traded not in presence.

In actual use, we use modified MPT, which has extension node, branch node and leaf node. A large MPT may contain a small MPT, for example, code is a small MPT. Each time only the changed account will have a new branch node written to the next block, and the rest will remain unchanged. This is done to preserve the history. I think there are at least two advantages to the benefits: (1) Add traceability. (2) In order to temporarily allow the fork. As Ethereum block generation rate is very fast, it has a lot of forks. The fork node needs to roll back. Moreover, some smart contracts are very complicated, and the original state cannot be restored without saving the history.

Modified MPT is shown in this figure.
![Image text](https://github.com/markliu666/PHBS_BlockChain_2018/blob/master/Modified%20MPT.png)

The structure of the block header is shown below

```go
type Header struct {
 
    ParentHash  common.Hash    `json:"parentHash"       gencodec:"required"`
 
    UncleHash   common.Hash    `json:"sha3Uncles"       gencodec:"required"`
 
    Coinbase    common.Address `json:"miner"            gencodec:"required"`
 
    Root        common.Hash    `json:"stateRoot"        gencodec:"required"`
 
    TxHash      common.Hash    `json:"transactionsRoot" gencodec:"required"`
 
    ReceiptHash common.Hash    `json:"receiptsRoot"     gencodec:"required"`
 
    Bloom       Bloom          `json:"logsBloom"        gencodec:"required"`
 
    Difficulty  *big.Int       `json:"difficulty"       gencodec:"required"`
 
    Number      *big.Int       `json:"number"           gencodec:"required"`
 
    GasLimit    uint64         `json:"gasLimit"         gencodec:"required"`
 
    GasUsed     uint64         `json:"gasUsed"          gencodec:"required"`
 
    Time        *big.Int       `json:"timestamp"        gencodec:"required"`
 
    Extra       []byte         `json:"extraData"        gencodec:"required"`
 
    MixDigest   common.Hash    `json:"mixHash"          gencodec:"required"`
 
    Nonce       BlockNonce     `json:"nonce"            gencodec:"required"`
 
}
```
The structure of the block is shown below.

```go
type Block struct {
 
    header       *Header
 
    uncles       []*Header
 
    transactions Transactions
 
    // caches
 
    hash atomic.Value
 
    size atomic.Value
 
    // Td is used by package core to store the total difficulty
 
    // of the chain up to and including the block.
 
    td *big.Int
 
    // These fields are used by package eth to track
 
    // inter-peer block relay.
 
    ReceivedAt   time.Time
 
    ReceivedFrom interface{}
 
}
```

In the key-value pair in the MPT, the key is the account address, and the value is the account status. The storage of value needs to be serialized, using RLP (Recursive Length Prefix), and finally becomes nested array of bytes. The content here is more complicated and will not be described in detail.

## 3. Ethereum's Trading Tree and Receipt Tree:

We have just introduced the most complicated state tree in the three trees of Ethereum. What about the other two trees? Ethereum's trading tree is similar to the tree in Bitcoin, and each transaction has a receipt, and the nodes of the transaction tree and the receipt tree are one-to-one. The receipt tree is designed for quick results from smart contracts. Both the transaction tree and the receipt tree use MPT. There is no special reason for this. It may be to unify the data structure of the three trees and to search them more quickly (again by merkle proof).

So what is the difference between the three trees? The status tree represents the global situation, and the transaction tree and receipt tree represent the current changes. The state tree is a shared node, and the following nodes inherit the previous node. However, both the transaction tree and the receipt tree are internally independent.

More complex query operations need to introduce a data structure, bloom filter, to quickly find out if an element is in a collection. The main principle is that each element in the set takes the hash to find the corresponding position in the vector, and uses 0-1 to mark whether it exists in the set at that position. The bloom filter has a total length of 128 bits. The main feature is that the bloom filter will have a false positive, but there will be no false negative. That is to say, it will not miss any correct result, but it may report true when there is actually no existence in this collection. The bloom filter is available at every level, and the bloom filter can be used to quickly filter out blocks that do not have information that satisfies certain conditions.

In this way, Ethereum is a transaction-driven state machine. In fact, Bitcoin is also a transaction-driven state machine, the state is UTXO, moving from the current state to the next state. It should be noted that the state transitions of Ethereum and Bitcoin are deterministic. So everyone must reach a consensus, each node must maintain the same data locally.

## 4. Ethereum Consensus Mechanism GHOST:

As I mentioned earlier, like Bitcoin, Ethereum must also reach a consensus. What is the difference between Ethereum's consensus mechanism and Bitcoin? Ethereum's block generation rate is very fast, and it is too late to broadcast new blocks. Some miners may dig up other blocks attached to the front blocks. In this way, Ethereum's forks are much more than bitcoin. If the miners' rewards on the forked chain are invalidated, the miners' enthusiasm will be much lower.

Then we can design a mechanism to allow some of the failed miners to still have rewards. Uncle block means that the first block of the forked chain is the uncle block of the new block of the longest legal chain. If it is recorded as uncle block by the new block, you can get 7/8*bonus reward, and the new block can also get 1/32*bonus reward. The new block can record up to two uncle blocks.

Let us discuss it now. If the uncle block must be separated by only one generation, then there are two problems: (1) The uncle block is deliberately not included by one of the later miners (2) More than two uncle blocks appear. Then if the uncle block can be defined by many generations, then (1) The burden of the system is too heavy. (2) And there will be more and more forks behind, and there is no timely reward, which is not conducive to the maintenance of the longest chain. GHOST's solution is uncle blocks are separated by up to 7 generations. The farther the block, the less reward you get, from 7/8 to 2/8, a total of six kinds of uncle blocks. The advantage is that the four problems of the above two opposites are solved in a balanced manner.

Miners in Bitcoin have two rewards, static reward is block reward, and dynamic reward is transaction fee. Ethereum is similar, static reward is block reward, dynamic reward is gas fee. Professor mentioned gas fee in the class, I will not go into details here, and it will be mentioned in the smart contract. However, uncle blocks will not get the gas fee, because the transactions in uncle blocks are invalid and will not be verified by the full node, but the difficulty of mining is still checked.

There is still a question, can the block behind the uncle blocks be rewarded? No, otherwise it is easy to cause a forking attack, in that case the cost of a fork will be lower.

## 5. The Mining Algorithm of Ethereum:

There is a famous saying that the blockchain is secured by mining. Bitcoin's mining algorithm is bug bounty, and no shortcuts have been found so far, so it is a good algorithm. However, Bitcoin's algorithm consumes a lot of energy on the one hand, and on the other hand, the trend of Bitcoin mining is ASIC mining and the formation of a mining pool. This is unfair to individual miners, and mining centralization is very beneficial. Moreover, there are many connections in the mining pool, and the released blocks are accepted by other nodes more quickly. The advantages of the mining pool are getting bigger and bigger, which is also known as centralization bias.

Nakamoto said that one CPU, one vote. This idea is not implemented in Bitcoin. Ethereum designed the memory hard mining puzzle for ASIC resistance. Drawing on historical experience, Litecoin has also used this kind of puzzle, which is characterized by the need to save arrays in memory, otherwise the computational complexity of mining is greatly increased. However, it is very difficult for each SPV to verify, which violates one of the principles of puzzle, difficult to solve, but easy to verify.

Ethereum improved the puzzle of Litecoin, using two data sets. The small data set is a 16M cache, and the large data set is a 1G dataset, which is a DAG. SPVs save the small one, and full nodes save the large one.

How is the large dataset generated? The first element is generated by reading 256 times through the pseudo-random iteration of the cache, and the latter elements are like this. So how to mine? The large dataset reads 128 numbers, calculates the hash according to the block header and nonce, maps to a certain position of the large dataset, reads the element and the next adjacent element. This is done by reading 128 numbers within 64 times. Finally, calculate the hash value and compare the difficulty of mining. If it is not satisfied, change the nonce.

Ethereum mining is now based on GPU, which is relatively successful, so that universal equipment can participate. Not only because of the use of memory, but also because the developers of Ethereum continue to claim to move from POW to POS (the ASIC chip development cycle is very long), Ethereum avoid the use of ASIC chips. Whether such intimidation can continue to be effective is also very interesting.

Another difference between Ethereum's mining and Bitcoin’s mining is that Ethereum uses pre-mining to distribute to Ethereum designers, and pre-sale pre-mining Ethers for developing Ethereum. At present, the supply of Ethereum in Ethereum is about 100 million, with a market value of about 50 billion US dollars.

The mining situation of Ethereum is shown in these two figures.
![Image text](https://github.com/markliu666/PHBS_BlockChain_2018/blob/master/Ether_supply_growth.png)
![Image text](https://github.com/markliu666/PHBS_BlockChain_2018/blob/master/market_capitalization.png)

## 6. Ethereum Difficulty Adjustment Algorithm:

Ethereum difficulty adjustment algorithm is shown in these four figures.
![Image text](https://github.com/markliu666/PHBS_BlockChain_2018/blob/master/difficulty01.png)
![Image text](https://github.com/markliu666/PHBS_BlockChain_2018/blob/master/diificulty02.png)
![Image text](https://github.com/markliu666/PHBS_BlockChain_2018/blob/master/difficulty03.png)
![Image text](https://github.com/markliu666/PHBS_BlockChain_2018/blob/master/difficulty04.png)

The difficulty adjustment of Ethereum is similar to Bitcoin, but there are many differences. The first one is that uncle blocks will affect the difficulty. The second is to deal with hacking or other black swan events, the difficulty is reduced by up to 99 at a time. The third is to have a difficulty bomb. It increases exponentially as the block number increases. The design of the difficulty bomb was originally designed to force the miners to switch during the transition from POW to POS, because it is difficult to insist on mining when the bomb is large enough. But now POS has not been developed yet, the difficulty bomb is already very large, so it has retreated 3 million blocks and adjusted the bonus from 5 to 3. Note that the bonus is not decremented like Bitcoin, but is only adjusted for the difficulty bomb. After the difficulty is reduced, if it is still 5 rewards, it is unfair to the miners who solve the difficult problem ahead.

Ethereum difficulty adjustment situation is shown in this figure.
![Image text](https://github.com/markliu666/PHBS_BlockChain_2018/blob/master/block_difficulty_growth.png)

## 7. Proof of Stake:

There are many people who say Bitcoin mining is very resource-intensive. These resources all need to consume funds. Whoever has more funds can use more resources for mining. Since it is the amount of money for the competition, why don’t you directly compare funds? That is virtual mining. The advantages are: (1) eliminating the mining process, saving resources (2) forming an internal ecological closed loop, reducing external influences (internal use of digital currency voting, rather than external funds)

But the miners with the most amount of money in the POS have the greatest possibility to win every time, so the security deposit needs to be locked for a while. This is Proof of Deposit. At the same time, another challenge for POS is the problem of betting on both sides, and another is the nothing at stake problem. Mining on both sides, if one side mines successfully, the other side of the deposit will not be locked.

The POS consensus protocol that Ethereum is preparing to adopt is Casper, the Friendly Finality Gadget (FFG). In the transition phase, Casper and POW are mixed to provide finality for POW. Finality is the final state. Transactions included in finality will not be canceled, while transactions based solely on POW may be rolled back.

In Ethereum, verification work is done by the validator. To be a validator, you need to lock some deposit. The role of validator is to help the system reach a consensus and vote to decide which chain is the longest legal chain. The weight of the vote depends on the amount of the deposit.

Ethereum uses 100 blocks as an epoch. Whether this epoch can become a finality requires voting. Voting requires two phase commit: (1) prepare message (2) commit message. Each round of voting requires more than two-thirds of votes. In the actual system, there is no longer a distinction between two rounds of voting, and the epoch is reduced from 100 blocks to 50. And only one round of voting. Each round of voting is a commit message for the current epoch and a prepare message for the next epoch. After two consecutive rounds of voting have been certified, epoch becomes finality.

Validator can be rewarded for participating in the verification, and there will be penalties for bad behavior, such as deposit deduction. If a validator does not do anything, a partial deposit will be deducted. If a validator votes indiscriminately, or bets on both sides, system will deduct all the deposit. The deducted deposit is destroyed, which is equivalent to reducing supply. Each validator has a term, and there will be a waiting period when the term expires. During the waiting period, other nodes can report the behaviours of this validator. This is done to ensure that each validator is a good node.

Now POS is still not very mature, and like Bitcoin, 51% of attacks still exist. So now Ethereum is still in the transition phase. There is now a digital currency called EOS that uses a full POS. This POS is called DPOS (Delegated Proof of Stake). Vote for 21 super nodes first, and then the super nodes generate consensus. Maybe in the future Ethereum will use this approach, DPOS needs the test of bug bounty.

## 8. Smart Contract:

Supporting for smart contracts an important change from Bitcoin for Ethereum. A smart contract is a piece of code that runs on a blockchain, and the logic of the code defines the content of the contract. The smart contract's account holds the current running status of the contract. Balance represents the current balance. Nonce represents the number of transactions. Code stands for contract code. Storage represents the storage of the state, and the data structure is an MPT. Solidity is the most commonly used language for smart contracts and is syntactically similar to Javascript.

A transfers money to B. If B is an externally owned account, then it is a normal transfer. If B is a smart contract account, then the contract is called. The specific function of the contract is called in the data field. The value is 0, because it is not a normal transfer, but a contract. But it takes a certain amount of gas. A contract can also call another contract, there are at least three ways. The first method is to call it directly. The second method is to use the call function of the address type. The third method is delegatecall. For example, the smart contract in the auction has a bid function, and each bidder calls the bid function to bid, and simultaneously sends the corresponding amount of Ethers. Use the withdraw function to retrieve the locked Ethers that failed to bid. All functions that accept external transfers need to add the payable keyword. By default, the fallback function is called.

So how are smart contracts created? First, a certain transaction is initiated by an externally owned account and sent to the 0x0 address (the amount can be 0, but the gas fee cannot be 0, otherwise the miner will not package the transaction into the block). Put the code into the data field, and compile the code into bytecode, then run it on EVM (Ethereum Virtual Machine) (for enhanced portability, and EVM addressing space is 256bits)

Let's take a look at the gas fee. The smart contract is a turing-complete programming model, and there may be an infinite loop. However, in computer science, the halting problem is simply unsolvable. So charging the gas fee to the person who initiated the contract is a solution. Different orders in the EVM consume different gas fees. Simple instructions are cheap, and complicated instructions are expensive. When the full node receives the call of the smart contract, first charge all the gas fees (deducted from the payer’s account) according to GasLimit. If the charge is more, it will be refunded. If the charge is less, it will roll back (but it has already been consumed). The gas fee will not be refunded, or there will be a malicious attack).

The resources consumed by transactions in Bitcoin can be seen from the size of the bytes (1M upper limit), but Ethereum cannot, because some smart contracts have very short transaction bytes, but the logic is very complicated. Therefore, Ethereum must be limited by the use of the global GasLimit. Each transaction has its own GasLimit, but instead of adding all the transactions’ GasLimit together, Ethereum defines a global GasLimit that limits the total resource consumption.

The 1M upper limit of Bitcoin cannot be adjusted. If it must be adjusted, it may be forked. But Ethereum's GasLimit can be fine-tuned. Each miner can raise or lower 1/1024 on the basis of the previous GasLimit. Due to the rapid speed of block generation, the adjustment can be very fast. Eventually GasLimit will tend to average the opinions of all miners.

Now there is a question, is it to mine first or to execute a smart contract first? It should be the first to execute the smart contract and then mine. Before trying a nonce, you need to generate the block header first, so the state of the three trees should have been changed already. There is no gas fee if a miner fails to mine, and there is no compensation. And the miner also needs to verify the correctness of the transaction in the released block, update the three trees locally, and then verify the comparison with the header in the blockchain. Therefore, slow miners in the Ethereum are particularly at disadvantage.

So will there be miners who don’t want to verify because of no reward? (Not verifying that means a major threat to the safety of Ethereum) No. Because if a miner skips the verification process, he cannot update the local three trees, and he will not be able to mine later. So any miner cannot skip the verification process.

## Conclusion

There are many problems in Ethereum, and there have been many crises. But overall, it is still the second largest digital currency in the blockchain. The Ethereum blockchain ecosystem is diverse, decentralized, robust, vibrant and creative. Since its release, this rapidly adopted technology is expected to solve the problems of almost all existing industries. Developers, thinkers, innovators, and leaders around the world are gathering to create a new way of communicating, trading, and organizing. We are here for web3.0. We are here for a decentralized future. We are here for Ethereum.

## Reference

[1] 李赫, 孙继飞, 杨泳, 汪松. (2017). 基于区块链2.0的以太坊初探. 中国金融电脑(6), 57-60.

[2] 张舒, 杨宇光. (2018). 区块链技术基础及应用. 信息安全研究, v.4；No.33(06), 89-94.

[3] 高航, 俞学劢, 王毛路. (2016). 区块链与新经济:数字货币2.0时代. 电子工业出版社.

[4] Mavridou, A. , Laszka, A. (2017). Designing Secure Ethereum Smart Contracts: A Finite State Machine Based Approach.


