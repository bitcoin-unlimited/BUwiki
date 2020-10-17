<div class="cwikmeta">  
{  
"title": "Blockchain Voting Techniques"  
} </div>

# Blockchain Voting Techniques

## A Blockchain as a public record
* Permanent public record of vote
* Allows permissionless independent and potentially open source vote tallies
* Orders repeat votes (allowing unambiguous discarding of one), or prevents repeats
* Provides registration & communication mechanism
* Public record of number of registrations & participants

* Cannot be the voter registration authority (Sybil attack)<sup>1</sup>

## Vote Recording
 * Data carried in a transaction
 * A token payment to a choice of destinations, each destination represents one choice.


## Merkle Tree Based Vote Tallies

* Merkle inclusion proof can prove that a particular vote was counted in O(log N) data
* But cannot prove no "ballot stuffing"

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

The merkle tree leaf nodes are a vote by participant P: 
$$
V(P, D_i) = \left\{ \begin{array}{l}
     \verb|1 if vote "yes" for decision D_i| \\
     \verb|0 otherwise|
 \end{array} \right. \tag3
$$


## Tokens
* Issued 1 to each voter by the registration authority
* Confers permission to participate in a vote, making 1 vote

## Token Shuffle

* Enables voter anonymity from the registration authority and the public
* Participant can still validate their vote it is cast

A token shuffle is CoinJoin<sup>2</sup>, CashShuffle<sup>3</sup>, or an equivalent algorithm applied to the tokens that confer permission to participate in a vote.

## Fake Vote Commitments

* Increase anonymity
* Plausible deniability (for a while) for  participants by handing out fakes
	* Hard to intimidate voters by threatening since participants can produce a fake vote.
	* This would make it hard for participants to sell their vote because participants who voted differently can provide a fake proof to get the money
* obfuscates ongoing counts 

Voting registration authority creates a number of fake votes and commits to those votes via a cryptographic hash.  The fakes are issued during the election process.  When the polls close, the fake votes are revealed and the cryptographic hash proves all the fakes.  Revealing partial fakes results in a non-matchin hash


## Voting Data Commitment

* Ensures integrity of voter choices
	* (cannot omit a choice, add extra choices, or swap choices to identifiers for a subset of voters)

Apply a cryptographic hash to the complete voting information and choice data to create an identifier that identifies the vote session.  Users and software can compare session identifiers to ensure they have the correct data.  Session identifiers should be included in each participant's vote to ensure that they are voting on the correct session with the correct data.




[1]: https://www.microsoft.com/en-us/research/publication/the-sybil-attack/
[2]: CoinJoin
[3]: CashShuffle