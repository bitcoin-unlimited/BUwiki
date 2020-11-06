<div class="cwikmeta">  
{  
"title": "Blockchain Voting Techniques",
"related":["/voting_impossible_triangle"]
} </div>

# Blockchain Voting Techniques
<center>Andrew Stone, Bitcoin Unlimited.  Oct 2020</center>

<center><em>A toolbox of technologies that can be used to solve problems around elections.</em></center>

## A Blockchain as a public record
* Permanent public record of vote
* Allows permissionless, independent and potentially open source vote tallies
* Orders repeat votes (allowing unambiguous discarding of one), or prevents repeats
* Provides registration & communication mechanism
* Public record of number of registrations & participants
* Blockchains cannot stand alone as the voter registration authority (Sybil attack)<sup>1</sup>

A blockchain represents a public commitment of ordered data.  This data repository can be used to record votes in a variety of ways.  Any interested party can access the blockchain to discover all votes, so long as they know how to differentiate votes from other blockchain activity.

Note that blockchain reorganizations or mining denial-of-service attacks can be used to remove or ignore transactions and therefore votes, preventing their inclusion in the blockchain's main chain.  But these attacks cannot be used to change a participant's vote.

These attacks could affect an election that specifies a moment in time when polls close.  However, reorganizations leave strong historical evidence of misbehavior in terms of proof-of-work, and denial-of-service attacks leave strong cotemporal evidence (e.g. any participant actively monitoring the network during the voting will see vote records posted to the network but not included in the blockchain).

## Blockchain Vote Recording Techniques
 * Data Carried: Encoded as data in an arbitrary, otherwise unrelated transaction
 * Token: A token payment to a choice of destinations, each destination represents one choice.
 * A hybrid where a token represents permission-to-vote, but the actual vote is data carried in a transaction the melts or returns the token to the voting authority.

### Vote Via Data Carrying 
One construction<sup>6</sup> of a data carrying vote allows any entity to construct an arbitrary transaction that sends to a specific output contract.  Spending this output in a new transaction is the act of voting.  To satisfy the contract, the voter must push a public key P registered by the voting authority, a signature of the transaction using P, vote identifier and choice data, and the signature of that data using P.

The use of data signatures (BCH's OP_CHECKDATASIGVERIFY) allows the vote to be safely placed in the transaction's satisfier script (also called scriptSig) even though the satisfier script is not signed as part of the transaction's signature.


## Merkle Tally Tree
*tallies votes in a manner that admits succinct inclusion proofs*

* Merkle inclusion proof can prove that a particular vote was counted in O(log N) data
* If the total participation is known, a merkle inclusion proof also proves no "ballot stuffing" 

Extend a normal merkle tree at each node N with vote tallies of the sum of all the votes in the subtree rooted at N.  The node hashes must concatenate the child hashes as in a normal Merkle tree and the vote tallies to ensure that the tallies cannot be modified without forcing the higher Merkle node hashes to change.  The merkle tree root therefore contains the election results.  

Providing a more formal construction:

Assume a vote with different yes/no decisions $D_n$, functions LeftChild and RightChild that return Merkle tree children, H is a cryptographic hash function, and the operator "|" is byte string concatenation:

$$
Tally(N, D_i)= Tally(LeftChild(N).D_i + RightChild(N).D_i)  \tag1
$$

Merkle Node hash calculation:
$$
H(N) = H(Tally(N, D_x)) | H(LeftChild(N)) | H(RightChild(N))  \tag2
$$

The merkle tree leaf nodes are the hash of a vote, and participant information (such as the blockchain transaction or transaction hash, public key & signature) P, that proves the validity of the vote: 
$$
Tally(P, D_i) = \left\{ \begin{array}{l}
     \verb|1 if vote "yes" for decision D_i| \\
     \verb|0 otherwise|
 \end{array} \right. \tag3
$$

$$
V_n = H(Tally(P,D_x) | P) \tag4
$$

The following diagram illustrates the merkle tree of votes.  Vote tallies appear in brackets.  For brevity, tallies have been omitted from many nodes, except those that are part of the merkle proof of vote $V_2$, which is indicated in light blue.  

```mermaid  
graph TD
R("Root[5,3]") -- 0 --> B["B[3,1]"]
R -- 1 --> C["C[2,2]"]
B -- 0 --> D["D[2,0]"]
B -- 1 --> E["E[1,1]"]
C -- 0 --> F
C -- 1 --> G
D -- 0 --> H
D -- 1 --> I
E -- 0 --> J["J[1,0]"]
E -- 1 --> K["K[0,1]"]
F -- 0 --> L
F -- 1 --> M
G -- 0 --> N
G -- 1 --> O
H --> V0{"V0"}
I --> V1{"V1"}
J --> V2{"V2[1,0]"}
K --> V3{"V3[0,1]"}
L --> V4{"V4"}
M --> V5{"V5"}
N --> V6{"V6"}
O --> V7{"V7"}
classDef proof fill:#cff,stroke:#333,stroke-width:3px;
class V2,K,D,C proof;
```

So the Merkle proof is "$V_2[1,0]$, K[0,1], D[2,0], and C[2,2] at index 2".

As with normal Merkle trees, specifying index 2 is required to communicate the data concatenation order at each level.  To explain, note that the Merkle proof execution for the provided example begins as follows:

$$
 x \leftarrow H(V_2[1,0])  \tag{a1}
$$

$$
x \leftarrow H(x | K[0,1])  \tag{a2}
$$

$$
x \leftarrow H(D[2,0] | x), ... \tag{a3}
$$
But how did the algorithm "know" to switch the order of concatenation in the last operation?  The answer is by using the element index.

To explain, note that the path from the root to child nodes are labelled with a 0 or 1.  Traversing any path and interpreting these a bits in a number results in the zero based element index of the vote in the tree, in this case its index 2 or 010 binary.  Starting with the least significant bit, use each bit to communicate to the prover the concatenation order (e.g. H(current val, next) or H(next, current val)) of the hash at each tree level, ignoring the bottom-most level which does not concatenate any elements.  For example the 2nd hash operation (a2, combining J and K) executes H(current value J | K[0,1]) because the first bit is 0, but the 3rd hash operation (a3, combining D and E) executes H( D[2,0] | current value E)  -- note the different order -- because the next bit in the index is a 1.  Therefore the index number of the merkle leaf communicates the prover in what order the proof elements must be combined.  

An incorrectly specified index would result in the prover hashing data in the wrong order, yielding a different merkle root and therefore a failed merkle proof.   So a correct merkle proof also proves a vote's position in the tree.

Actually, this additional index data can be eliminated if the hash function is commutative<sup>5</sup>, if there is no other use for it.  A simple commutative hash function CH based on cryptographic hash H is CH(x,y) = H(sort(x,y))

### Merkle Tally Tree Inclusion Proof properties

Since the total vote count is committed in the merkle tally tree, the merkle inclusion proof also commits to the total vote count.  This prevents extra-depth attacks (although this is already prevented if merkle trees are correctly formulated -- each vote's hash MUST be included as the tree leaf, rather than the vote's raw bytes.  

The commitment to the total vote count also prevents ballot stuffing if participants independently know the vote count.  

However, its is possible to create a dishonest merkle tally tree (or partial tree), and derive a correct-looking merkle inclusion proof that claims false tallies (by replacing some voters' votes) in branches that the proof does not provide.  But this merkle inclusion proof CANNOT have the same merkle root as the honest tree, and repeated queries for various merkle inclusion proofs on the dishonest tree will probabilistically reveal the dishonesty, since the creator will not be able to provide the full merkle path to non-existent votes.

As a vote progresses, a "merkle mountain range" style approach can be used to show that a voter's vote is part of an ongoing tally.  In the merkle mountain range, multiple merkle power-of-two trees are constructed, and whenever 2 trees contain the same # of elements, those two trees are joined together by making them the children of a new tree of 1 greater depth.

### Sorted Merkle Tally Trees<sup>7</sup> 

If vote hashes are sorted prior to inclusion in a merkle tally tree, the "merkle mountain range" approach cannot be used.  However, there are several interesting properties.

* Vote non-inclusion proofs are possible by constructing a proof to two adjacent leaves that bracket the missing vote in sorted order.
* It is "suspicious" to dishonestly claim that a tree T0 is actually a subtree of the real tree T1, because all the votes in T1's other children would need to be sorted greater than or less than the votes in T0 (which, as the honest tree, presumably covers approximately the full range of hash values).  With a large voting pool, this would be so statistically anomalous that it would never happen in practice.
* Similarly, a tree that does not cover approximately the full range of hash values can be identified as a subtree of the full vote, rather than the full tree.


## Tokens
* Issued 1 to each voter by the registration authority
* Confers permission to participate in a vote.  Each token allows exactly 1 vote, enforced by blockchain token semantics

Note that either the "data carrier" or "destination" voting architectures are possible using tokens.

## Token Shuffle

* Enables voter anonymity from the registration authority and the public
* Participant can still validate their vote it is cast

A token shuffle is CoinJoin<sup>2</sup>, CashShuffle<sup>3</sup>, or an equivalent algorithm applied to the tokens that confer permission to participate in a vote, rather than currency.

A coin join is a very simple concept so is described briefly here:  

A transaction may consist of many inputs and many outputs.  Let us propose that many individuals contribute a single input of one token and receive a single output of one token (to a different address from their input) into a single transaction.  If the order of the inputs does not correspond to the order of the outputs, and the input and output quantities are all the same, it is impossible to determine which input corresponds to which output.  This process can be repeated with different partners to increase the pool of possible entities that may control a particular output.

The traditional coin shuffle suffers from a problem with varying input quantities:  Its possible to connect inputs to outputs if participants include different quantities because each participants' quantity must be preserved.  However, this is not a problem for voting tokens since each participant has exactly 1.

Also note that actual construction of the multi-participant transaction may leak identity information or suffer denial-of-service attacks, so implementations are significantly more complex than this conceptual description.



## Voting Topic and Data Commitments<sup>6</sup>

* Ensures integrity of voter choices
	* Cannot omit a choice, add extra choices, or swap choices to identifiers for a subset of voters
* Prevents vote replay

Apply a cryptographic hash to the complete voting information and choice data to create an identifier that probabilistically uniquely identifies the vote session.  Participants and network software can compare identifiers to ensure they have the correct data.  The larger network can reject unknown identifiers which might notify participants of misbehaving or malicious software.  Identifiers should be included in each participant's vote so that the vote itself affirms its vote on a particular topic with the particular topic data.  This prevents both malicious topic data and replay attacks.

## Fake Vote Commitments

* Increase anonymity
* Plausible deniability (for a while) for  participants by handing out fakes
	* Hard to intimidate voters by threatening since participants can produce a fake vote.
	* This would make it hard for participants to sell their vote because participants who voted differently can provide a fake proof to get the money
* obfuscates ongoing counts 

The voting registration authority creates a number of fake votes and commits to those votes via a cryptographic hash.  The fakes are issued during the election process.  When the polls close, the fake votes are revealed and the cryptographic hash proves all the fakes.  Revealing partial fakes results in a non-matching hash so it is not possible to affect the election outcome via a partial reveal.

## Vote Encryption

 * Votes are encrypted using a public/private keypair, which is revealed when all votes are cast
 * Key could be an aggregate
 * Problem: Entity controlling key reveal or last reveal of an aggregate key could choose not to reveal if they do not like the election results.

Vote encryption would prevent miner blockchain attacks from denying votes for one candidate from entering the blockchain since the attacker cannot determine the contents of a vote until after the polls close and presumably the blockchain has advanced beyond the point which re-organization is affordable.

## ZK-SNARKs

TBD

## References

[1]:  The Sybil Attack: https://www.microsoft.com/en-us/research/publication/the-sybil-attack/
[2]: CoinJoin: https://bitcointalk.org/?topic=279249
[3]: CoinShuffle:  https://www.darrentapp.com/pdfs/coinshuffle.pdf,  https://github.com/cashshuffle/spec
[5]: Commutative Merkle Trees: https://medium.com/@g.andrew.stone/tree-signature-variations-using-commutative-hash-trees-4a8a47d4f8ce
[6]: TBD: Dagur & Jorgen voting project & paper
[7]: In conversation with Dagur Valberg Johannsson