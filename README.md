# Blockchain Payment Channels

Payment channels in general is a concept that has existed in Blockchains for some time. It comes from the observation that, if Alice pays Bob $10 a month for a year as phone bills, it is possible to summarise all of those payments as a single payment of $120 at the end of the year. While Bob may prefer to actually receive $10 every month, in the context of blockchains, it may be worth the trade-off as they can save a total of 11 transactions worth of transaction fees.

The issue that payment channels solve, is, how can we convince Bob that Alice will really pay $120 at the end of the year? What if Alice uses her phone for the entire year, just to run away at the end of the year without paying? If Alice was paying $10 a month, the most Bob would lose is $10, while here, he could potentially lose $120. Bob is not willing to take that risk to save transaction fees.

The most naive solution would be, let Alice sign a transaction that pays Bob $10 at the end of the first month, then sign another transaction that pays Bob $20 at the end of the second month, and so on. All of these transactions are given to Bob and can be submitted at any point. If Alice runs away by the third month, Bob can at least retrieve $20, reducing his risk greatly. However, these signed transactions are useless if Alice just spends all the money in her wallet. Bob would be holding onto signed invalid transactions, which are useless.

The solution is actually quite simple - lock $120 worth of Alice's funds in a smart contract. Then Bob has a guarantee that the signed transactions will be valid, up to $120. This is a blockchain payment channel in a nutshell.

This system works very well for two people, but can it be extended to more than two? It is actually significantly more difficult. Let's take a party of Alice, Bob and Charlie as an example. Alice could lock $120 into the smart contract, sign a transaction that pays Bob $70 and give it to Bob. Then later sign a transaction that pays Charlie $70 and give it to Charlie. Neither of them knows that Alice has committed to $140 of payments. The result of this is that, only Bob or Charlie can get their $70, and one of them has to forfeit the money.

The solution to this is also very simple - simply have all parties sign the transaction. If all parties sign the the transaction, then they are all aware of all payments that are happening. We can also switch from signing transactions to signing states. As an example, we begin with a state where everyone locks up $100 in the smart contract.

#### State 0

| Name    | Funds |
| ------- | ----- |
| Alice   | 100   |
| Bob     | 100   |
| Charlie | 100   |

Required - Alice, Bob, Charlie

Signed - Alice, Bob, Charlie

#### State 1

| Name    | Funds |
| ------- | ----- |
| Alice   | 80    |
| Bob     | 110   |
| Charlie | 110   |

Required - Alice, Bob, Charlie

Signed - Alice, Bob, Charlie

#### State 2

| Name    | Funds |
| ------- | ----- |
| Alice   | 60    |
| Bob     | 115   |
| Charlie | 125   |

Required - Alice, Bob, Charlie

Signed - Alice, Bob, Charlie

#### State 3

| Name    | Funds |
| ------- | ----- |
| Alice   | 60    |
| Bob     | 100   |
| Charlie | 140   |

Required - Alice, Bob, Charlie

Signed - Alice, Bob, Charlie

Indeed this is how Cardano's Hydra works. States MUST be signed by all parties within the Hydra head for it to be valid. Any party can submit a state and get everyone's money out at any point.

# Limitations

## Fundamental limitations (cannot be overcome)

### Trust is required

In the above example, Hydra works very well in a scenario where all parties are using their own money, and have no trust among any of them. The issue we have is that users are committing money into the Hydra head, and they are not one of the signing parties.

What does this mean for Hydra dapps like DeltaDefi? Since the smart contract takes the state signed by all parties and distributes funds according to the signed state, it is completely possible for the parties to collude to steal funds.

Put another way, any state signed by all Hydra parties is seen as truth by the smart contract. This is completely fine in the previous situation where Alice, Bob and Charlie all commit only their own money.

As an example, let's say Charlie proposes a state

#### Charlie proposed state - INVALID (insufficient signatures)

| Name    | Funds |
| ------- | ----- |
| Alice   | 60    |
| Bob     | 0     |
| Charlie | 240   |

Required - Alice, Bob, Charlie

Signed - Alice, Charlie

Without Bob's signature, this is not seen as a valid state, and of course Bob wouldn't sign it, Charlie has just attempted to steal all his money.

#### Hydra Example

Let's start a new Hydra head again with Alice, Bob and Charlie. But we're going to allow David to also commit $100 into the Hydra head.

#### State 0

| Name    | Funds |
| ------- | ----- |
| Alice   | 100   |
| Bob     | 100   |
| Charlie | 100   |
| David   | 100   |

Required - Alice, Bob, Charlie

Signed - Alice, Bob, Charlie

#### State 1

| Name    | Funds |
| ------- | ----- |
| Alice   | 134   |
| Bob     | 133   |
| Charlie | 133   |
| David   | 0     |

Required - Alice, Bob, Charlie

Signed - Alice, Bob, Charlie

David's money has been completely stolen by Alice, Bob and Charlie, and there is nothing David could do about it. The required signers are simply Alice, Bob and Charlie, so anything they all sign is truth.

This is the primary issue with Hydra heads. Anyone other than the required signers that commit funds into the head must trust at least one of the required signers.

However, you'll notice that the trust barrier is actually relatively low. David does not need to trust more than half of the required signers, he only needs to trust ONE. No matter how many required signers there are, as long as there is ONE honest party, the others cannot collude to steal anyone's money.

But, the trust required by users like David should be acknowledged.

### Unclosable States

Let's take a very interesting example of a Payment channel state. We start by having Alice, Bob, and Charlie lock $100 each into the Payment channel contract. Then progress the state to a very interesting state.

#### State 0

| Name    | Funds |
| ------- | ----- |
| Alice   | 100   |
| Bob     | 100   |
| Charlie | 100   |

Required - Alice, Bob, Charlie

Signed - Alice, Bob, Charlie

#### State 1 (UNCLOSABLE)

| Name    | Funds |
| ------- | ----- |
| Alice   | 200   |
| Bob     | 200   |
| Charlie | 200   |

Required - Alice, Bob, Charlie

Signed - Alice, Bob, Charlie

Realistically, this is completely possible, the states are not really validated by anything, and as long as the participants sign it, it is perfectly valid.

But, since they each only locked $100 into the Payment channel contract, this is what we would call an unclosable state. It is impossible for this to form a valid transaction, the total funds available in this Payment channel is $300, so there cannot be $600 of output. Which means the 3 participants were kind of silly to sign it, it is simply an unclosable state.

This actually extends to tokens also. In fact, currently within Hydra, BOTH minting AND burning tokens results in unclosable states. For example

#### State 0

| Name    | Funds |
| ------- | ----- |
| Alice   | 100   |
| Bob     | 100   |
| Charlie | 100   |

Required - Alice, Bob, Charlie

Signed - Alice, Bob, Charlie

#### State 1 (UNCLOSABLE)

| Name    | Funds         |
| ------- | ------------- |
| Alice   | 100 + 1 DELTA |
| Bob     | 100           |
| Charlie | 100           |

Required - Alice, Bob, Charlie

Signed - Alice, Bob, Charlie

## Practical limitations (Can be overcome with better software)

You'll notice that previously, we have not mentioned any transactions in relation to Hydra. Indeed, the need to build Cardano transactions to interact with Hydra is completely an illusion, and enforced only by honest parties. It is perfectly possible to directly sign a State without any transactions to mutate the State.

However, since we value the trust of our users, we will definitely be running Hydra as it was intended. Hydra was made in a way that it runs the Cardano Ledger. What this means, is that, Hydra networks are not really too different from a Cardano fork.

If you started a new Cardano network, with a new Genesis block, and ran it, it would behave quite similarly to a Hydra network. But with some crucial differences. First of all, Hydra networks don't have a Consensus mechanism. What this means is that Staking keys are essentially pointless in Hydra heads, they cannot be staked to anything to obtain staking rewards.

### Staking keys

On first glance, this is true, Staking keys do appear to be useless. But Staking keys have more uses in Cardano than just to obtain staking rewards. Staking keys can be scripts, which allows certain advanced smart contract tricks. The first version of DeltaDefi for example, used this trick quite a lot, I would say a lot of serious dapps abuse this trick, including USDA.

Unfortunately, the Hydra team has overlooked this use case of Staking keys. It is not currently possible to use such tricks, because Staking keys cannot be registered.

Without a consensus mechanism, how are states resolved? Simple, all required signers must sign a state for it to be valid, as explained previously.

### Signing states

What you'll notice is that, since every Hydra-node is running the Cardano Ledger, they are actually storing two separate states, the signed state and the Ledger state. The Ledger state is needed in order to validate transactions, while the signed state is needed to close Hydra heads (submitting the final state to the Cardano mainnet). The signed state is called a `Snapshot`

The issue is that these two states can deviate from each other. How does this happen? Since every single participant (required signers) must sign each `Snapshot`, it is completely possible for one participant to miss and `Snapshot` to sign, for various reasons, packet loss, the server was down for a little bit, etc.

But the issue is, the Ledger state has progressed, while the `Snapshot` has not progressed. In any other payment channel system, we would simply abandon the payment channel and close it with an old `Snapshot` at this point.

A problem that's specific to dapps like DeltaDefi, is that the DeltaDefi Hydra network is formed with protocol parameters that are very different to the mainnet.

The biggest change is obviously the fact that transaction fees are 0. A more subtle, but more important for our discussion, is that we have reduced the `minUtxoValue` to 0. You'll notice on the mainnet, it is impossible to have UTxOs with very small ADA values. This is to reduce what's called dust. UTxOs contain a fair amount of data regardless of how much ADA is in them, `txHash`, `txIndex`, `address`, and they also have to be packed into blocks. The cost of each existing UTxO for the blockchain is very relevant, therefore, you are not allowed to simply create a large amount of UTxOs that contain very little ADA.

But DeltaDefi allows this, because we want to allow users to create an infinite amount of orders as they wish. The problem this raises, is that any state that contains UTxOs with 0 ADA makes it impossible to close. Our DEX will be in such an unclosable state for 99% of its lifetime, and only at the last moment, we will reconcile the state to a closable state, and close the Hydra head.

This means that any time anything goes wrong, the head is doomed and cannot be closed.
