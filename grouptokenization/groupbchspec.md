# Group Tokenization Consensus Specification


This document describes the blockchain and consensus changes required to implement Group Tokenization on Bitcoin Cash.  It is intended for implementers, and generally does not explain why certain decisions were made.  It assumes that you understand the general architecture and purpose of Group tokens. If, while you are reading this document, you need greater understanding of a topic, please refer to the main Group Tokenization proposal document here: [https://docs.google.com/document/d/1X-yrqBJNj6oGPku49krZqTMGNNEWnUJBRFjX7fJXvTs/edit?usp=sharing](https://docs.google.com/document/d/1X-yrqBJNj6oGPku49krZqTMGNNEWnUJBRFjX7fJXvTs/edit?usp=sharing)

**Notes on reading this document**:
 * Industry-standard conventions with respect authoritative and normative statements are followed; that is, use of capital MUST, MUST NOT, et. al.
 * Testable requirements are specified with an identifier (REQX.Y.Z) for use when building unit tests.
  
## 1 Blockchain Changes

Group Tokenization fits its changes within the existing transaction format by prefixing transaction output scripts with Group data. This makes the changes required to implement group tokens very small and self contained<sup>1</sup>.

### 1.2 Transaction Changes
    
Within a transaction, grouped outputs follow the existing output serialization format, except that the output script is formatted as follows. Following normal convention <> indicates a data push. Also [] indicates optional arbitrary additional script code:

\<GroupID> \<QuantityOrFlags> OP_GROUP [any normal constraint script]

 

## 2 Script Machine Changes

OP_GROUP may appear anywhere within a script.

Execution of the OP_GROUP opcode MUST pop 2 objects from the main stack (REQ1.1.1). 

If there are fewer than 2 objects on the stack, execution MUST fail (REQ1.1.2).

Optimizing script machines may recognize the push,push,OP_GROUP pattern and skip execution.  However, such a machine MUST behave exactly as if those instructions were executed including failing the script due to bounds checking (both during the pushes and the OP_GROUP) and properly accounting for these op codes in any tallies (REQ1.1.3).

The following is an example "C" implementation of the OP_GROUP instruction:

                case OP_GROUP: // OP_GROUP just pops its args during script evaluation
                    if (stack.size() < 2)
                        return set_error(serror, SCRIPT_ERR_INVALID_STACK_OPERATION);
                    popstack(stack);
                    popstack(stack);
                    break;

## 3 Transaction Validation Changes

All existing transaction validation rules are unchanged.

A UTXO is defined as "grouped" if it contains a serialized script following the format specified in section 1.2, and has the two specified attributes (GroupId and QuantityOrFlags).  An OP_GROUP instruction that appears in any other place in the serialized script is ignored.  If the "QuantityOrFlags" most significant bit is clear (the number is positive), this field should be interpreted as a quantity.  If the MSB is set, this is an "authority baton UTXO" and this field should be interpreted as Flags (described below).

### Flags

#### Authority Permission Flags

AUTHORITY = 1<<63, *This is an authority UTXO, not a "normal" quantity holding UTXO*
MINT = 1ULL << 62, *Can mint tokens*
MELT = 1ULL << 61, *Can melt (destroy) tokens*
BATON = 1ULL << 60, *Can create authority outputs*
RESCRIPT = 1ULL << 59, *Can change the output script*
SUBGROUP = 1ULL << 58, *Can operate on subgroups*
    
ACTIVE_FLAG_BITS = AUTHORITY | MINT | MELT | BATON | RESCRIPT | SUBGROUP
ALL_FLAG_BITS = (0xffffULL << (64 - 16))
RESERVED_FLAG_BITS = ACTIVE_FLAG_BITS & ~ALL_FLAG_BITS

Reserved flag bits 57 to 48 are reserved for future hard fork group features and must be zero.  Bits 0 through 47 are used as a nonce in the Group genesis UTXO, but must be zero in other authority UTXOs.

#### Group flags

Group Identifiers are 32 byte values<sup>2</sup>.  The last 2 bytes of a group identifier<sup>4</sup> indicate properties of the group as follows:

COVENANT = bit 0,  *This group enforces the structure of output scripts*
HOLDS_BCH = bit 1,  *This Group holds BCH instead of tokens*
GROUP_RESERVED_BITS = bits 2 through 15, MUST be 0.

For example, if the "groupId"  is a byte array, the COVENANT bit is groupId[31]&1.


### 3.1 Subgroups

Subgroups are groups with a parent group.  Parent authorities can execute operations on subgroups.  Subgroup Identifiers are the byte concatenation of the 32 byte parent group identifier and arbitrary additional bytes, limited by the maximum stack element size.  Subgroups do not have their own flags; they have the same flags at the same location as their parent group.

Therefore to determine the parent group identifier from a subgroup identifier, simply take the first 32 bytes.


### 3.2 Transaction Group Enforcement

Certain properties are enforced across the inputs and outputs of a transaction.  These properties only apply to "grouped" inputs and outputs.  If a transaction has no grouped inputs or outputs an optimized validator may return VALID without further analysis.

This section enforces the following general rules: 
 * for the same group, input quantities must match output quantities (unless overridden by authorities),
 * authority baton permissions are properly passed from inputs to outputs and from group to subgroup, 
 * output scripts must match previous output scripts for covenanted groups, and 
 * newly created groups are created correctly.

This is not a lot of rules and the resulting algorithms are not complex.  However, the following sections specify the exact rules in great detail, covering every tiny edge condition since any differences might cause a consensus failure.  So upon the first read, these sections may make the rules seem daunting.  But actually, this document is much larger than a quality implementation which can fit into a single function and a few helpers.  Just keep your mind on the overall picture stated in the previous paragraph and work through the rules slowly.  A good approach is to loop through outputs then inputs collecting data on each group.  Then loop through the collected groups, verifying the rules based on data collected.  If you come upon an edge condition that seems unspecified make an in-code note that will result in a compilation error.  DO NOT GUESS OR SKIP!  Contact an author of this paper and we will clarify the requirement.

**Notes on reading these rules**: 
* All operations proceed on a per-group basis and so "per GroupId" is implied in every statement.  
* References to "group inputs" and "outputs" never include the authorities -- authorities are special and are always specifically referenced.  
* Grouped UTXOs have 2 fields: a "GroupId" and "Quantity", and Grouped authority baton UTXOs have 2 fields: "GroupId" and "Flags", as described in section 3.1.
* The term "GroupId" refers to both groups and subgroups (a subgroup IS a group).
* Non-compliance with any of these rules results in an immediate return of INVALID.

For example, the phrase "input xxx authority" (e.g "input mint authority") means "an input group authority with the xxx Flag bit set", or to be extremely pedantic, "a transaction input whose prevout UTXO scriptpubkey begins with \<GroupID> \<QuantityOrFlags> OP_GROUP, and whose GroupID field matches the one we are currently working on and whose QuantityOrFlags field has the AUTHORITY (63) and MINT (62) bits set".  

Likewise, the phrase "sum of the output Quantities" should be pedantically understood as "the sum of the Quantity fields in the non-authority-baton outputs whose GroupID field matches the one we are currently working on".  


#### Properly Constructed Group Annotations
* For every output that begins with \<push A> \<push B> OP_GROUP:
	* Length A MUST be >= 32 bytes
	* Length B MUST be 2, 4, or 8 bytes
* The QuantityOrFlags field is constructed from B as follows:
	* If Length B is 2 or 4 bytes, deserialize as a little-endian 2 or 4 byte unsigned number and zero extend to a 64 bit integer.
	* If Length is 8 bytes, deserialize as a little-endian 64 bit signed integer
* If The QuantityOrFlags field is negative (has bit 63 set) it is an Authority Baton.  *(There's no such thing as a negative quantity, even though the Quantity is signed)*

Note: These deserialization rules mean that the only way to specify an Authority Baton UTXO is when B has a length of 8 bytes.

Note: If OP_GROUP appears as a proper script prefix, both these rules and the script machine rules apply. If OP_GROUP is used in any other place within the script, only the script machine rules are applied.  This means that its valid to have an OP_GROUP in some other location within a script whose arguments have any length.  Even though this makes specification more complex, it makes validation simpler.

#### Setup

Construct 4 64 bit bitmaps (per group that appears in the transaction's inputs and outputs): perms, batonPerms, subgroupPerms and subgroupBatonPerms.  Initialize them to 0.

#### Authority Permission Accumulation

* for every input authority perms = perms | Flags *(Figure out the permissions available for group operations)*
* For every input authority with the Baton bit set, batonPerms = batonPerms | Flags *(figure out what authorities can be passed to child authorities)*
* For every input authority with the Subgroup bits set, subgroupPerms = subgroupPerms | Flags *(Figure out all permissions available on the subgroup due to group authorities)*
* For every input authority with the Baton and Subgroup bits set, subgroupBatonPerms = subgroupBatonPerms | Flags *(Figure out all permissions that can be granted to subgroup authorities due to group authorities)*

Note that the SUBGROUP authority bit means that the other permission bits in this authority apply to both this group and any of its subgroups, not exclusively to subgroups.  Authorities can be targeted exclusively to a subgroup via the "baton" system by creating a child authority with the subgroup's ID.


#### Subgroup Permission Accumulation

* For every subgroup, look up its corresponding group.  Set subgroup.batonPerms = subgroup.batonPerms | (the group's subgroupBatonPerms& ~SUBGROUP). *(Extend SUBGROUP-tagged authority permissions to the subgroup.  But do not extend subgrouping permissions itself, because there's no such thing as recursive subgroups)*
* For every subgroup, look up its corresponding group.  Set the subgroup.perms = subgroup.perms | group.subgroupPerms *(Extend SUBGROUP-tagged permissions to subgroups)*

#### 3.1.2.1 Group Genesis Rule
* If there is > 1 group (excluding subgroups) that has output authorities but batonPerms==0, return INVALID *(We only allow one group Genesis per transaction)* (REQ3.2.1.1) 
* For that output authority:
	* If the transaction has other grouped (authority or normal) outputs for this group, return INVALID *(the genesis authority is the only use of this group allowed in this transaction)* (REQ3.2.1.2) 
	* Ensure that the RESERVED_FLAG_BITS are 0 (REQ3.2.1.3) 
	* Ensure that the GROUP_RESERVED_BITS are 0 (REQ3.2.1.4) 
	* Find the SHA256 of the following data:
		* The transaction's first input "prevout" (hash and index) using standard Bitcoin serialization. *(for entropy)*
		* The scriptPubKey of the first OP_RETURN output (skip if there is no OP_RETURN output) *(commit to the human and wallet-level information and contract)*
		* The Flags field *(this contains a nonce)*
* If the above SHA256 matches the GroupId field<sup>4</sup>, this is a valid genesis UTXO.  Allow this authority UTXO even though the permission it claims are not enabled via an input authority. (REQ3.2.1.5) 
* If a subgroup (of this group) output exists, return INVALID *(do not allow subgroup operations in the same transaction as the group genesis)* (REQ3.2.1.6)

This formulation means that a transaction can contain only create one new group.  Also, no other outputs in this transaction can use this group or subgroups of this group (if implementations set all the permissions running flags to 0 this property will be enforced by other rules).  Isolating the genesis UTXO in this manner is an arbitrary limitation intended to improve implementation consistency by reducing edge cases.

It is possible for a subgroup to have no input UTXOs but have outputs because the parent group authority might operate on the subgroup.  An incorrect implementation might misinterpret this as an invalid group genesis output.   As described above, using batonPerms is one way to realize that such an output is not a genesis.  Another (normative) way is that the GroupId for any subgroup must be greater than 32 bytes.

Note that the genesis authority Flags field may not have all permission bits set.  The group creator may want to restrict certain permissions from genesis.

Note that if a covenanted group is created without a RESCRIPT authority, the first MINT transaction can never be valid because there is no input script from which to pull the covenant.  The genesis transaction is valid, but this group is probably useless.
 
#### 3.1.2.2 Authority Baton Rules

 * For every output authority it MUST be true that  Flags & ~batonPerms == 0.  *(Enforce permissions when passing the baton from input to output -- no bit can be set in Flags, unless it is set in batonPerms)* (REQ3.2.2.1)
 * For every non-genesis output authority it MUST be true that  Flags & ~ALL_FLAG_BITS == 0.  *(Enforce that unused bits are zero, for later use)* (REQ3.2.2.2)
 * For every output (genesis and non-genesis) authority it MUST be true that  Flags & ~RESERVED_FLAG_BITS == 0.  *(Enforce that reserved flag bits are zero, for later use)* (REQ3.2.2.3)

#### 3.1.2.3 Token Quantity Rules

* If the GroupId HOLDS_BCH flag is set, Quantity MUST be 0.  Copy the BCH amount field in every non-authority input and output (grouped) UTXO into the respective Quantity field (REQ3.2.3.1). *(This enables grouped BCH, rather than grouped tokens)*

* The sum of the input Quantities MUST not exceed the maximum int64_t value (REQ3.2.3.2). *(note this is a **signed** 64 bit integer or 0x7FFFF FFFF FFFF FFFF)*
* The sum of the output Quantities MUST not exceed the maximum int64_t value (REQ3.2.3.3). *(note this is a **signed** 64 bit integer or 0x7FFFF FFFF FFFF FFFF)*
* The sum of the input Quantities MUST equal the sum of the output Quantities, unless the group's perms has the MINT or MELT bit set. (REQ3.2.3.4)
* If the group's perms has the MINT bit set, the sum of the output Quantities MUST be >= the sum of the input Quantities. (REQ3.2.3.5)
* If the group's perms has the MELT bit set, the sum of the output Quantities MUST be <= the sum of the output Quantities. (REQ3.2.3.6)

Note that if the group HOLDS_BCH, the MINT and MELT operations are not actually creating or destroying BCH.  Instead these operations act as entry or exit points, allowing BCH to flow into or out of the group.  Conservation of BCH in the transaction is guaranteed because the existing logic that balances BCH in a transaction is untouched.

Note that the total ordinality of the group's tokens may exceed a signed 64 bit integer.  In this case a single transaction cannot manipulate all of them...

#### 3.1.2.4 Covenant Enforcement

For every grouped output, if the group has the COVENANT bits set and the group.perm's RESCRIPT bit is clear, all output "scriptpubkey" scripts MUST equal the first grouped non-authority input's prevout "scriptpubkey" script, beginning after the first OP_GROUP instruction (REQ3.2.4.1).  Note that it is equivalent to compare the cryptographic hashes of these script suffixes rather than the script itself.  If this object (the first grouped non-authority) does not exist return INVALID (REQ3.2.4.2). 

*(This enforces that covenanted groups use the same constraint script as the first input from that same group<sup>3</sup>.  Efficient implementations should discover this script while iterating through the inputs during a previous step)*

#### 3.1.2.5 Grouped P2SH 

The P2SH script type has hard-coded extra-script behavior which is that upon completion the second-to-top stack item (the redeem script) is subsequently executed.  The Grouped P2SH script type (described in chapter 4) MUST behave analogously<sup>5</sup>. (REQ3.2.5.1)

## 4 IsStandard Changes

Two new script types SHOULD be recognized as standard.  These are simply the P2PKH and P2SH script types with the OP_GROUP prefix:

**Grouped P2PKH** (REQ4.1)
\<groupid> \<quantityorflags> OP_GROUP OP_DUP OP_HASH160 \<pubkeyhash> OP_EQUALVERIFY OP_CHECKSIG

**Grouped P2SH**  (REQ4.2)
\<groupid> \<quantityorflags> OP_GROUP OP_HASH160 \<scripthash> OP_EQUAL


## Footnotes

1. If, upon review, you find this idea “ugly”, you are welcome within your own software to completely restructure the transaction data as you see fit. When computing the hash of a transaction or serializing transactions or UTXOs for network transmission, simply place the Group data in the serialized format as described.  And start any "grouped" script execution opcode counting as if 3 operations have already been executed.
2. 32 bytes was chosen rather than 20 to prevent a group creator from successfully executing Wagner's birthday attack on their own group identifier.  If a group creator is able to construct a group ID collision, they could break provable assurances (such as total token quantity) by using the colliding genesis transaction to create new authority batons.
3. The covenant is a contract between the group creator and the token holder.  This architecture allows the group creator to *offer* a contract upgrade to token holders.  Token holders accept the new contract by creating a transaction where the 1st grouped input UTXO is the new contract.  But token holders could reject the new contract by continuing to use old-contract UTXOs as the first grouped input.  So the group creator cannot force contract holders to upgrade to a new contract, unless the ability to do so is explicitly coded into the contract by, for example, a clause in the contract that allows a UTXO to be spent if the tx is signed by the group creator.
4. This is beyond the scope of this specification, but for clarity understand that the GroupId's flag bits are part of the hash -- Group creators need to search for a group ID with the desired flag bits in a manner similar to searching for "vanity addresses".
5. A fix to the P2SH "hack" is proposed in the full Group Tokenization document.  Since this fix can happen as a side effect of enabling robust smart contract features, it is better to consider it at that time rather than address it here.