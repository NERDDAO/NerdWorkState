## Part Three: Looking Ahead

Ethereum is devising a strategy to reincorporate the processes currently operating outside its core protocol. The goal is to mandate that proposers obtain blocks from builders, essentially letting the protocol handle the relay's current duties. The relay system as it stands has its vulnerabilities. For instance, a relay might not properly validate a block, misverify the builder's bid in relation to the payment intended for the proposer, or even delay or fail in block delivery. On top of this, running a relay doesn't come cheap. As of now, there's a lack of a sustainable funding model for them. The Ultrasound Relay, the most utilized relay, says its operating costs are estimated to be between 70k-80k euros annually, and that's excluding other expenses like software maintenance. Relays currently operate as public utilities.

It's also worth noting that since MEV Boost is external software developed by a company (Flashbots) it isn't as rigorously tested as software within the protocol. This was evident with the Prism client post the Shapella upgrade: an integration bug with MEV Boost caused issues with the proposer's signature, leading to missed slots and slashing. The goal of integrating this process into the Ethereum protocol is to address these challenges, ensuring that even if an agreement between the proposer and builder falls apart, the proposer remains reimbursed. So, if a builder provides a faulty block, the proposer still gets the full bid, leaving the builder to bear the consequences. While the specifics of this integration, referred to as ePBS (enshrined proposer-builder separation), are still being researched, and possibly a couple of years from realization, there are already many different ideas of how it could possibly look.

### _How to Enshrine Proposer-Builder Separation_

To understand potential ePBS implementations, it's essential to first understand some basic components of Ethereum’s PoS algorithm. In Ethereum time is segmented into 12 second intervals called slots. 32 of these slots come together to form an epoch. In each slot, a validator is randomly selected to propose a block. Simultaneously, a committee is designated to attest to the validity of the block they deem compliant with Ethereum's PoS fork choice rules, ideally attesting to the most recently proposed block as the blockchain's head. If a block isn't proposed in the given slot, then 4 seconds in, the attesting validators attest to the previous block.

Now, onto ePBS designs. The most favored model spans two slots. First there’s a bidding phase, where builders send their bids to the validators. Then Slot 1 begins with the proposer picking a bid and committing to it by publishing a block that commits to that builder’s bid. A group of attestors then cast their vote in favor of this block, ensuring its place in the chain. In Slot 2, the builders see the bid that was committed in the proposer's committed block and the attestations on it. Recognizing the proposer's irreversible commitment, the builder whose bid was selected releases their block and is assured that their MEV can’t be stolen. Finally, attestors validate this new block.

!["Two-slot" ePBS design](https://mirror.xyz/0x218932707a30bE62Ef8559d32d954863933b412f/MhJM9li409wxuOSYaw5EcLjUQoLSx5iYZXZNgMZU4uM/_next/image?url=https%3A%2F%2Fimages.mirror-media.xyz%2Fpublication-images%2FAEwaO74mvmHhiA-jxr44e.png&w=3840&q=75)

"Two-slot" ePBS design

A newly released model is similar to the two slot approach but introduces a payload-timeliness committee. First, a builder's bid is selected and committed to by the proposer, and then the committee of validators gives its attestation. Subsequently, the builder reveals the block's payload (its transactions), and the payload-timeliness committee confirms that the payload was provided on time and its validity. The other differences between these two methods lie in the specifics of Ethereum's Proof of Stake operations, but that's beyond the scope of this post.

![ePBS design with a Payload-Timeliness Committee](https://mirror.xyz/0x218932707a30bE62Ef8559d32d954863933b412f/MhJM9li409wxuOSYaw5EcLjUQoLSx5iYZXZNgMZU4uM/_next/image?url=https%3A%2F%2Fimages.mirror-media.xyz%2Fpublication-images%2FK6ZCNivsbPFwxMxO8FzxW.png&w=3840&q=75)

ePBS design with a Payload-Timeliness Committee

Another design revolves around the concept of a slot auction. Here builders, during their bid, commit to a slot in the epoch without specifying the block. They essentially pledge to create a block during their allocated slot, offering a certain price to do so. This offers adaptability, especially for cross domain MEV which requires real time action.

So far all the ePBS designs grant the builder full control over the block's transactions. A potential workaround is the use of an inclusion list. This list, sent by the proposer to the builder, ideally all the transactions currently in the mempool or doesn’t have to be, contains transactions that must be part of the builders block if there’s space. If the builder's block is full, they must indicate so, affirming they acknowledged the list. Such a method strengthens the network censorship resistance. If a builder wants to censor a transaction it will become difficult and costly to do so over time. Due to EIP 1559, consecutively filled blocks will cause the base fee to exponentially rise. Therefore if a builder continually censors a transaction by filling up a block with dummy transactions, the escalating costs make this infeasible overtime.

There might be instances where the proposer wants to have some influence on block creation. Another ePBS feature might involve the proposer making a section of the block (either the start or end) and delegating the remainder to a builder. All of these designs and features aren't mutually exclusive, it's more about balancing their benefits and drawbacks.

### _The Optimistic Relaying Approach_

Another take on ePBS leverages our existing trusted relays. The idea is to incrementally reduce the responsibilities of the relay until it primarily serves as an optimizer, rather than a crucial component. In its first phase, we shed the relay's duty of verifying block validity. This greatly reduces the cost of running a relay as there’s no longer a need for block simulation to ensure its validity. Plus, it streamlines the relay’s role, shaving off about 100 to 200 milliseconds of latency in their communications with the proposers and builders. So, how do we ensure the proposer gets their payment if a block turns out to be invalid? The builders would be mandated to post collateral, equal to their bid, when they bid. If the block is invalid, the collateral covers the payment the proposer would've received. This concept is termed Optimistic Relaying V1.

![Optimistic Relaying V1](https://mirror.xyz/0x218932707a30bE62Ef8559d32d954863933b412f/MhJM9li409wxuOSYaw5EcLjUQoLSx5iYZXZNgMZU4uM/_next/image?url=https%3A%2F%2Fimages.mirror-media.xyz%2Fpublication-images%2F7x97rsrZiKi8rjR1SLI9j.png&w=3840&q=75)

Optimistic Relaying V1

Pushing optimistic relaying a step further to V2, we can eliminate the relay’s need to download the block, reducing another 50 to 100 milliseconds of latency. The same assurances apply: if a block never downloads, the builder's collateral pays up.

![Optimistic Relaying V2](https://mirror.xyz/0x218932707a30bE62Ef8559d32d954863933b412f/MhJM9li409wxuOSYaw5EcLjUQoLSx5iYZXZNgMZU4uM/_next/image?url=https%3A%2F%2Fimages.mirror-media.xyz%2Fpublication-images%2FlpfERnAdo_tZHTnlRmy_-.png&w=3840&q=75)

Optimistic Relaying V2

Ultimately, the end game for Optimistic Relaying starts looking a lot like the payload-timeliness committee model I touched on earlier. Here’s the sequence: Builders submit their bids on a peer to peer layer. The proposer accepts a bid and follows up with a signed header. Then, the builder rolls out the block. At this stage, the relay's sole job is overseeing the peer to peer layer mempool, basically clocking when different activities occur. The relay's role becomes super lightweight, it just needs to keep tabs on the mempool. It makes the relay operate a lot like the payload-timeliness committee. All these steps build towards a future where the relay is replaced by the payload-timeliness committee, streamlining the entire protocol.

![](https://mirror.xyz/0x218932707a30bE62Ef8559d32d954863933b412f/MhJM9li409wxuOSYaw5EcLjUQoLSx5iYZXZNgMZU4uM/_next/image?url=https%3A%2F%2Fimages.mirror-media.xyz%2Fpublication-images%2FldPK7-aS5ARf-B5I41qm-.png&w=3840&q=75)

### _Leveraging Builders for Additional Protocol Enhancements_

PBS emerged as a response by Flashbots to the centralizing effects of MEV, aiming to attempt to harness MEV for positive outcomes. Given the new role in Ethereum specializing in block building, there's an opportunity for these entities to act like supercomputers, contrasting the lightweight validators. Protocol designs are surfacing that capitalize on these powerful builders. The idea is to keep validators uncomplicated and straightforward (some might even say [cucks](https://www.youtube.com/watch?v=ymVd2Ch7wBc&t=22820s)), while builders, having no such restrictions, can function at a much higher computational level. A primary application for these enhanced builders is for scaling.

Ethereum Researcher Dankrad Feist’s Danksharding design suggests that these high resource intensive builders can construct one big block that contains all the data. That data is then segmented and committed to by multiple KZG commitments. Constructing this block requires considerable resources, but validating them is relatively inexpensive. Lightweight validators can then apply Data Availability Sampling to inspect a small section of the block and be nearly certain of the entire dataset's accessibility, yielding an added ~16x boost in data throughput from Proto-Danksharding. The intricacies of Danksharding are complicated and not covered here, but the key point is that these advanced builders can be delegated additional roles to further enhance the network.

Another idea to take advantage of the builders is the potential realization of based rollups. A few years back, Vitalik discussed rollup sequencing models, coining the term for one of them _Total Anarchy_, in which anyone can publish a rollup block and the first sequence that hits the chain is the official block. This approach had many drawbacks, like onchain spam and ambiguity over the winning sequence. However, Justin Drake's recent article on [based rollups](https://ethresear.ch/t/based-rollups-superpowers-from-l1-sequencing/15016) highlighted a more efficient strategy taking advantage of the builders. In this model, the builder on layer one functions as the rollup sequencer, cherry picking the optimal sequence to include onchain. This ensures only the optimal sequences are incorporated.

Beyond rollups, the introduction of powerful builders can spur other innovative structures, like stateless clients. They empower nodes to operate as usual but without the baggage of preserving outdated states. These allow the nodes to be more lightweight and hinge on the builder's ability to generate specific cryptographic proofs.

### _PEPC: Protocol-Enforced Proposer Commitments_

Protocol-enforced proposer commitments (PEPC, pronounced pepsi) is a concept introduced by Ethereum Foundation researcher Barnabé Monnot in October 2022. You can delve deeper into Barnabé's original post [here](https://ethresear.ch/t/unbundling-pbs-towards-protocol-enforced-proposer-commitments-pepc/13879?u=barnabe). At its core, PEPC aims to grant proposers a greater say in block building, that they’ve lost by selling the entire task to specialized builders. In PEPC, proposers can add extra conditions for a block to be deemed valid, apart from the usual Ethereum requirements. If a block fails to meet any of these extra conditions, it's considered invalid, and attestors should reject it. This is different from EigenLayer commitments where validators with extra commitments either lose some staked ETH for noncompliance or get rewarded for fulfilling them. However, the block remains valid regardless of these commitments.

Imagine Alice is a proposer in the Ethereum network. With PEPC, Alice has the flexibility to make a specific commitment for the upcoming block. She could commit that her block will contain at least three transactions pertaining to a particular smart contract, and she's willing to allocate 70% of the block's gas for these. She communicates this commitment, and it becomes part of her block's validity conditions. Now, Bob, a Builder, sees Alice's commitment. He packages together a bundle of transactions fitting Alice's criteria and sends it her way. If Alice's block, after being constructed, aligns with her commitment (i.e., contains the specified transactions consuming the designated gas), then the block is deemed valid and can be added to the blockchain. If not, Alice's block won't be accepted, ensuring that she adheres to her declared commitments.

One key difference between ePBS and PEPC lies in the commitments nature. In ePBS, proposers and Builders follow a fixed, uniform procedure. It's a kind of one size fits all approach. A proposer commits to a specific block hash, and the builder then produces a matching payload. However, PEPC introduces variety. Each proposer can set unique conditions, offering a lot more flexibility. It's crucial to note that PEPC's existence depends on ePBS, they complement each other. The exact workings of PEPC are still under discussion and research.