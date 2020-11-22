

# Why Bitcoin (Cash) Script is Nearly Useless (and what to do about it)
Andrew Stone, Nov 22, 2018

*Bitcoin Cash needs transaction and/or blockchain introspection to enable interesting contracts.  This introspection can take the form of an opcode that pushes transaction data onto the stack.  It could also take the form of an opcode that validates an SPV proof-of-existence of a prior transaction.*

# Where’s the Beef?

You have been told that Bitcoin is a Turing decider, or Turing complete (within script size and stack limits), so where are the all the smart contracts? Forget about “smart” ones, where are the “dumb” ones? Where are the simplest scripts that have conditional payments based on something other than signatures? Its time to cut through the hype and lies around smart contracts, by proving what’s not possible in Bitcoin Cash today. From there we can start to have an intelligent conversation about how far we want to travel down the road of enabling smart contracts. There is a lot of space between what we have now and Ethereum’s decentralized apps that can be explored.

The first step is to look at the two major claims about computational capabilities, that of Craig Wright and Clemens Ley. So I recently skimmed Craig Wright’s work claiming to prove that Bitcoin Script (BS) is a [Turing decider](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3265146). Whether or not you accept that proof, it remains unsatisfying because the existence of a Turing subsystem does not make the system Turing complete — for an example, an automatic door is clearly not Turing Complete although there’s undoubtedly a microprocessor reading an infrared or optical sensor and deciding when to open and close the door. The Bitcoin Cash blockchain has an analogous problem, as I will clearly show in the body of this paper.

So I moved on to [Clemens Ley’s presentation at Satoshi’s Vision](https://www.youtube.com/watch?v=M6j-11H2O7c) (which I had been unable to see at the time), and found it equally unsatisfying. The proposed construction is flawed in many fundamental ways — its actual construction does not fulfill the claims its author makes at the beginning of the presentation.

# Problems with Pay-For-Computation Models

Early on in Ley’s presentation there is a key flaw that bedevils many attempts to use the blockchain — the “replay” attack. [In a very general argument](https://youtu.be/M6j-11H2O7c?t=283), Ley has “Bob” computing a result for “Alice”, and posting the result on the blockchain to receive payment. However, this construction can only give Alice an answer, not guarantee Bob payment — more likely payment goes to the miner that sees Bob’s solution, formulates his own “replay” transaction and mines it in the next block. Therefore, Alice sees the miner providing the solution, not Bob. (As an aside, this seems to be a flawed attempt to generalize the cross chain atomic swap algorithm, which only works because spending is constrained by both the function solution and Bob’s public key)

# Problems with Turing Machine Formulations

Moving deeper, it becomes very apparent that Ley’s formulation does not meet previous claims that the output script encodes logic that constrains children to particular solutions. There IS no function constraint (in this case it should be logic that verifies correct turing machine operation) whatsoever. Let’s look at Ley’s transaction formulation shown in [detail here](https://youtu.be/M6j-11H2O7c?t=984). First, we see the Turing tape data being pushed onto the stack in the first 3 instructions:

_OP_PUSH 3 OP_DROP  
OP_PUSH 1 OP_DROP  
OP_PUSH q1 OP_DROP_

But then, where is the logic that constrains the next transaction to correctly execute a Turing machine? It does not exist because it is not currently implementable in Bitcoin Script. Instead, Alice must validate and co-sign every step of Bob’s work through a multisig operation. You can see that in the script [onscreen here](https://youtu.be/M6j-11H2O7c?t=984), where the last 4 lines of the scripts are:

_OP_2  
<pubkey Alice>  
<pubkey Bob>  
OP_2  
OP_CHECKMULTISIGVERIFY_

The original premise is that Bob is doing computation work for Alice, but in reality Alice must validate and sign every step that Bob makes. So Alice must do the computation that she is supposedly paying Bob for. Additionally, Alice and Bob can conspire to produce an incorrect computation, so Ley’s early claim that this is trustless and, for example, could be used for voting is not true. Alice and Bob could conspire in their execution of the algorithm to defraud Carol, a third party who is interested in the outcome of the computation but not involved in producing it.

_The fundamental problem is that in this construction, the blockchain is neither the Turing Machine nor verifying the proper operation of some external Turing Machine._ Bob is playing the role of the Turing Machine, and Alice is the verifier. They simply use the blockchain to store the Turing Machine’s tape. To prove this, note that fundamentally a Turing Machine is defined as state definitions and transitions. Where is this definition? It does not exist in Ley’s presentation.

The purpose of the Bitcoin blockchain is to trustlessly regulate what operations can occur. A blockchain like Ethereum allows contracts to actively make payments, while Bitcoin simply decides whether proposed payments are valid. But in Ley’s construction, Alice and Bob can conspire to break the rules that a Turing Machine is supposed to follow, and the Bitcoin blockchain will accept this as valid. This is not a trustless formulation, yet the innovation of the blockchain is most fundamentally about the trustless implementation of operations that previously required trust (“triple” entry bookkeeping, for example). Once trust is introduced, Bob might as well execute the complete program off-chain, providing the result to Alice who can validate it off-chain. Blockchain unnecessary.

# Problems with Continuity

[Here](https://youtu.be/M6j-11H2O7c?t=486) Ley claims that Bob can only create a spend transaction that satisfies the constraints set by Alice, yet [here](https://www.youtube.com/watch?v=M6j-11H2O7c) we are told that Bob is creating a chain of transactions and so can implement a computation of arbitrary length. However there is currently no way in Bitcoin Cash for a transaction to constrain a chain of transactions — it can only constrain the next spend based on inputs provided by the spender. This is a known problem, with proposed solutions. This paper on [covenants](https://fc16.ifca.ai/bitcoin/papers/MES16.pdf) was the first solution, and I proposed a solution in my [GROUP tokenization](https://docs.google.com/document/d/1X-yrqBJNj6oGPku49krZqTMGNNEWnUJBRFjX7fJXvTs/edit#heading=h.xvi6rs20j4nq) proposal (p 12).

Earlier I showed that in Ley’s construction the system was not constraining correct Turing machine operation. But even if we imagine that a script exists that is able to properly constrain the next output, the next-next output could be unconstrained simply by spending the next output to a different script.

This means that Bob can execute the turing machine correctly for N transactions (even years), only to suddenly stop doing so.

_Constraint continuity over multiple spends requires, in every single step, that we trust the spender. This again defeats a key value proposition of blockchains — trustless operation._

# Problems With the Turing Tape Formulations

In both Wright’s and Ley’s constructions, the blockchain is not the Turing “tape”. In Wright’s construction, the tape is the stack that Bitcoin Script operates with, not the blockchain. This assumption is implicit throughout the paper but can be specifically seen on page 6 as he shows how the PTTM is implemented using script opcodes, or on page 9 where he says:

> _Given a Turing machine, we can build an equivalent two-stack automaton through a process of pushing the input onto one stack followed by a process of referencing forward motion on the tape as popping from the first stack and pushing onto the second stack._

In Ley’s construction, at least the blockchain does store the tape, but this is very different from it being the tape. Specifically, at least some data on the blockchain is not readable or writable. To translate this into an abstract definition, this would be equivalent to a Turing Machine that can only read and write some portion of the tape. For example, let me propose an alternate machine where every 2nd bit on the tape is 0 and cannot be changed. Producing a sequence of 1 1 1 1 is impossible on this machine but simple on a Turing machine. My alternate machine is therefore not Turing complete. Similarly a Turing “decider” cannot return True if the tape is 1 1 1 1 and false if it is not because it cannot read the values of the 2nd and 4th bits.

# Why is it Important that the “Tape” be the Blockchain?

When Turing formulated the his machine, all data was equivalent — just 1s and 0s. The result of a computation done on one Turing Machine was the same as if it was done on another — like math facts, results were universally applicable.

However, blockchains are absolutely unique in that they have created a new form of data, the cryptocurrency ledger. And computation done “outside” this ledger is not applicable “inside”. What I mean by this is that if I need to execute a mathematical algorithm over cryptocurrency data (say take an average), it is not equivalent to do so off and on-blockchain. If executed off-chain, I can discover the intended result of a function — say avg(2,6)=4. But to actually verify and allow a transaction that makes 2 payments that are the average of 2 input payments requires execution on-chain.

So even though Bitcoin Script is nearly a Turing decider (constrained by limits on stack size, number of instructions executed, etc), Bitcoin Script is not a Turing decider for Bitcoin transactions.

For example, I can easily write a program in Bitcoin Script that computes an average given 2 input values pushed onto the script’s stack: OP_ADD OP_2 OP_DIV. But I cannot write a script that applies this result to the blockchain because the “tape” — the only data that Bitcoin Script can access is the script stack, not the blockchain.

Neither Wright’s nor Ley’s formulation is capable of implementing this averaging operation on the blockchain, yet this is arguably one of the simplest mathematical operations. This is fundamentally why few non-trivial applications exist on the blockchain today.

# What We Could Do

The shortest path to trustless blockchain decidability is to leverage the existing Bitcoin Script language. We must add opcodes that allow Bitcoin Script to push blockchain information onto the tape (bitcoin script stack).

First, the simplest instruction we could implement pushes information about the current transaction onto the stack. Clearly, pushing the binary serialization of the transaction onto the stack would be possible, although that may place a large burden on the script to parse it. It would also be possible to push information about the spent UTXOs (prevouts), such as bitcoin input quantities. Since all nodes clearly must have this information to validate this transaction, this is a simple and efficient O(1) operation. I believe that this would allow a huge range of basic functionality that is currently impossible to implement on the Bitcoin Cash blockchain. For example, scripts could validate the “outputs == average of inputs” problem presented earlier by parsing out the quantity fields in the inputs and outputs. And arbitrary data could be trustlessly accessed by the current transaction by spending outputs that contain that data, and then employing an opcode that pushes the prevout onto the stack. This should allow Ley’s Turing Machine formulation to actually work.

Second, we could access blockchain history by adding opcodes that push any blockchain transaction onto the stack. The major concern here is that it is too expensive to require that all transaction validators be able to find any old (possibly spent) transaction. But this problem is easily solved: the spend script will be required to provide the transaction and the merkle proof of its inclusion in a block specified by height. All nodes and SPV wallets must store the block headers, so validation of the existence of this transaction constitutes log(N) SHA256 operations (to validate the merkle tree) and an array lookup (to validate that the merkle tree root matches that of the specified block). One caveat of this approach is that if a chain reorganization occurs, unconfirmed transactions that were calculated as valid might be now invalid since the referenced transaction may no longer be included in the blockchain. This is no worse than today where a transaction becomes invalid due to parents that are not included in the reorganized chain.

# No, This Does Not Make BCH Into Ethereum

In Ethereum, contracts are active agents — they can generate payments for example. This is a lot of complexity and invites bugs both in the script and in Ethereum consensus. If we add the proposed opcodes to Bitcoin Cash, a layer 2 (off-chain) agent must still do the work of actually making payments — that is, of generating the transactions. The Bitcoin Cash blockchain remains limited to validating transaction proposals — the only change is that it is now able to trustlessly access blockchain data as part of that validation.

# Appendix: Can the Spender Provide Transaction Information, Trustlessly?

If this information is not provided trustlessly, then there is no reason to provide it at all — whoever the system is trusting to push the “right” transaction onto the stack can simply be trusted to enforce whatever constraint that the pubkeyScript is applying to that transaction data.

This question can be broken down into 2 subquestions: Can the spender provide information about this transaction trustlessly? And can the spender provide information about prior transactions trustlessly?

Can the spender provide information about this transaction trustlessly?

Constructing a signature script that pushes this transaction onto the stack is not possible because the signature script is itself part of the transaction. But the signature script could push a portion of the transaction onto the stack. However, the signature script could lie about the contents of the transaction, so we would still require a validation function like “is this value this transaction’s hash, not including signature scripts?” But this is a new opcode, and if a new opcode is needed one that directly pushes the required data is simpler for both the script and node programmer.

Can the spender provide information about prior transactions trustlessly?

The signatureScript can push other transactions onto the stack, as suggested by nChain (here), but doing so in a trustless fashion is problematic.

To do it trustlessly, the script must push the transaction, the merkle proof of the transaction’s inclusion in a block, chain proof of that block’s inclusion in this blockchain (basically POW validation of that block and all parents to a hard-coded checkpoint or the genesis block), and POW validation of N child blocks to make transactions in orphan blocks or illegal blocks very costly. But the end result of this work is not total proof, but an economic argument that it would cost an attacker more to generate a fake blockchain satisfying these constraints than the transaction is worth. However, there has already been at least one moment where all these constraints could be satisfied with no effort on the attacker’s part, yet the transaction still not be on the blockchain. That moment happens to be right now — the contentious ABC and SV fork has created two viable chains with significant POW, and a transaction on one fork could be offered to a script on the other fork meeting all of the above critieria.

Note also that the constraint script (often called the pubkeyScript or output script) must validate all of the data previously mentioned. This would be an incredibly complex, long running script, implementing a simple operation at the level of node or wallet software.

Validating that the transaction is on the correct blockchain is particularly problematic. Doing so means validating block headers back to the genesis block or to a “checkpoint” block. This is not possible in theory because the blockchain grows in an unbounded fashion, but a script can only validate a finite number of blocks since BCH Script does not have loops. This means that an output would become unspendable after N blocks have been discovered (and note that the script length and therefore fees grow proportional to N).

For example, if N=1000, the script becomes unspendable after 6.9 days (1000/144), if the checkpointed block is the current chain tip. However, using the current chain tip would be dangerous — a chain reorganization could orphan the checkpoint block and render the script unspendable on the main chain. Instead, the script author would need to use an older block with essentially zero risk of reorganization, especally to prevent the game theoretical intentional reorganization attack which increases the attacker’s coin value by decreasing supply. Note too that if every script included code to potentially validate the proof-of-work of the last 1000 blocks, validating 1000 blocks would require “avg_transactions_in_block *(1000²)/2” double SHA256 operations. However the same functionality (validate transaction is in a block on this chain) as an opcode would not require even a single SHA256 operation because it has already been calculated during block receipt.

In practice, this means that the script will likely ignore the chain proof. This allows transactions from other blockchains to be included to satisfy a constraint in this blockchain. Since presumably a major purpose of transaction inclusion would be proof-of-payment, allowing cross-blockchain proof (without the script somehow understand that) would be extremely dangerous as token values between the two chains fluctuate.

Prior to the BCH BSV split, providing transactions from foreign blockchains would have been merely a theoretical possibility, but now it is a very real attack.
