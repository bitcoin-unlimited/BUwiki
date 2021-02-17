# Group Tokenization Consensus Specification


This document describes the blockchain and consensus changes required to implement Group Tokenization on Bitcoin Cash. It assumes that you understand the general architecture and purpose of Group tokens. If, while you are reading this document, you need greater understanding on a topic, please refer to the main Group Tokenization proposal here: [https://docs.google.com/document/d/1X-yrqBJNj6oGPku49krZqTMGNNEWnUJBRFjX7fJXvTs/edit?usp=sharing](https://docs.google.com/document/d/1X-yrqBJNj6oGPku49krZqTMGNNEWnUJBRFjX7fJXvTs/edit?usp=sharing)

**Notes on reading this document**:
 * Industry-standard conventions with respect to the use of capital MUST et. al. are followed.
 * Testable requirements are specified with an identifier (REQX.Y.Z) for use when building unit tests.
  
## 1. Blockchain Changes

Group Tokenization fits its changes within the existing transaction format by prefixing transaction output scripts with Group data. This makes the changes required to implement group tokens very small and self contained<sup>1</sup>.

### 1.2 Transaction Changes
    
Within a transaction, grouped outputs follow the existing output serialization format, except that the output script is formatted as follows. Following normal convention <> indicates a data push. Also [] indicates optional arbitrary additional script code:


\<GroupID> \<QuantityOrFlags> OP_GROUP [any normal constraint script]

  
  

## 2.1 Script Machine Changes

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

## 3.1 Transaction Validation Changes

All existing transaction validation rules are unchanged.

A UTXO is defined as "grouped" if it contains a serialized script following the format specified in section 1.2, and has the two specified attributes (GroupId and QuantityOrFlags).  An OP_GROUP instruction that appears in any other place in the serialized script is ignored.  If the "QuantityOrFlags" most significant bit is clear (the number is positive), this field should be interpreted as a quantity.  If the MSB is set, this is an "authority baton UTXO" and this field should be interpreted as Flags (described below).

TODO: Flags meanings
AUTHORITY = 1<<63  (most significant bit set)
MINT = 1ULL << 62, // Can mint tokens
MELT = 1ULL << 61, // Can melt tokens,
BATON = 1ULL << 60, // Can create authority outputs
RESCRIPT = 1ULL << 59, // Can change the redeem script
SUBGROUP = 1ULL << 58,
    
ALL_PERM_BITS = CTRL | MINT | MELT | CCHILD | RESCRIPT | SUBGROUP

### 3.1.1 Group Creation

#### GroupId flags

COVENANT = 1
HOLDS_BCH = 2


### 3.1.2 Transaction Group Enforcement

Certain properties are enforced across the inputs and outputs of a transaction.  These properties only apply to "grouped" inputs and outputs.  If a transaction has no grouped inputs or outputs an optimized validator may return VALID without further analysis.

This section enforces the following general rules: 
 * for the same group, input quantities must match output quantities (unless overridden by authorities),
 * authority baton permissions can be passed from inputs to outputs and from group to subgroup, 
 * output scripts must match previous output scripts for covenanted groups, and 
 * newly created groupIds follow the correct format.

This is not a lot of rules and the resulting algorithms are not complex.  However, the following sections specify the exact rules in great detail, covering every tiny edge condition since any differences might cause a consensus failure.  So upon the first read, these sections may make them seem daunting.  Just keep your mind on the overall picture stated in the previous paragraph and work through them slowly.  If you come upon an edge condition that seems unspecified make an in-code note that will result in a compilation error.  DO NOT GUESS OR SKIP!  Contact an author of this paper and we will clarify the requirement.

**Notes on reading this section**: 
* All operations proceed on a per-group basis and so "per GroupId" is implied in every subsequent statement.  
* References to "group inputs" and "outputs" never include the authorities -- authorities are special and are always specifically referenced.  
* Grouped UTXOs have 2 fields: a "GroupId" and "Quantity", and Grouped authority baton UTXOs have a "GroupId" and "Flags", as described in section 3.1.
* The term "GroupId" refers to both groups and subgroups (a subgroup IS a group).
* Non-compliance with any of these rules results in an immediate return of INVALID.
* The phrase "input xxx authority" (e.g "input mint authority") means "an input group authority with the xxx Flag bit set", or to be extremely pedantic, "a transaction input whose prevout UTXO scriptpubkey begins with \<GroupID> \<QuantityOrFlags> OP_GROUP, and whose GroupID field matches the one we are currently working on and whose QuantityOrFlags field has the AUTHORITY (63) and MINT (62) bits set"

So for example the words "sum of the input Quantities" should be pedantically understood as "the per-GroupId sum of Quantity fields in the non-authority-baton grouped inputs".  
The phrase "xxx authority" means "a group authority with the xxx Flag bit set" (e.g. "mint authority baton").

#### Subgroup Authority Baton Rules

Construct 3 64 bit bitmaps, perms, outPerms and subgroupPerms (per group).  Initialize them to 0.

* for every input authority perms = perms | Flags *(Figure out the permissions available for group operations)*
* For every input authority with the Baton and Subgroup bits set, subgroupPerms = subgroupPerms | Flags *(Figure out all permissions that can be granted to subgroup authorities due to group authorities)*
* For every subgroup, look up its corresponding group.  Set the subgroup's outPerms to the group's subgroupPerms. *(Record in the subgroup the permissions inherited by the parent's authorities)*
* For every subgroup, look up its corresponding group.  Set the subgroup.perms = subgroup.perms | group.perms *(Any group authority has the same powers over its subgroup)*

Note that the subgroup authority logic MUST be executed before the Authority Baton Rules and the Balance Preservation Rules, so that these permissions bitmaps are properly set up.

#### Group Genesis Rule
* If there is > 1 group that has output authorities but no input authority, return INVALID *(We only allow one group Genesis per transaction)*
* For the output authority that has no input authority:
	* If the transaction has multiple output authorities (for this group), return INVALID
	* find the SHA256 of the following data:
		* The transaction's first input "prevout" (hash and index) using standard Bitcoin serialization.
		* The scriptPubKey of the first OP_RETURN output (skip if there is no OP_RETURN output)
		* The Flags field
* If the above SHA256 matches the GroupId field, set all flags in the GroupId outPerms bitmap 

This formulation means that a transaction can therefore contain only one group genesis and the newly created group has only one output which is the genesis authority.  This is an arbitrary limitation made for simplicity.

The genesis authority Flags field may not have all permission bits set.  The group creator may want to limit authorities from genesis.

Note, if a covenanted group is created without a RESCRIPT authority, the first MINT transaction can never be valid because there is no input script from which to pull the covenant.  The genesis transaction is valid, but group is probably useless.
 
#### Authority Baton Rules

* For every input authority with the Baton bit set, outPerms = outPerms | Flags *(figure out what authorities can be passed to child authorities and apply it to the permission set)*
 * For every output authority's Flags,  Flags & ~outPerms == 0.  *(Enforce permissions when passing the baton from input to output -- no bit can be set in Flags, unless it is set in outPerms)*
 * For every non-genesis output authority's Flags,  Flags & ~ALL_PERM_BITS == 0.  *(Enforce that unused bits are zero, for later use)*

Note that the Authority Baton logic (updating outPerms) MUST be executed before the Balance Preservation Rules.
 

#### Token Quantity Rules

* If the GroupId HOLDS_BCH flag is set, Quantity MUST be 0.  Copy the BCH amount field in the UTXO into the Quantity field for the remainder of these rules (REQ3.1.2.1).

* The sum of the input Quantities MUST not exceed the maximum int64_t value (REQ3.1.2.2).
* The sum of the output Quantities MUST not exceed the maximum int64_t value (REQ3.1.2.3).
* The sum of the input Quantities MUST equal the sum of the output Quantities, unless the group's outPerms has the MINT or MELT bit set.
* If the group's perms has the MINT bit set, the sum of the output Quantities MUST be >= the sum of the input Quantities.
* If the group's perms has the MELT bit set, the sum of the output Quantities MUST be <= the sum of the output Quantities.

#### Covenant Enforcement

* For every grouped output, if the group has the COVENANT bits set and the group.perm's RESCRIPT bit is clear, the constraint (scriptpubkey) script MUST equal the first grouped non-authority input's constraint script beginning (for both scripts) after the first OP_GROUP instruction.  If this object (the first grouped non-authority) does not exist return INVALID. *(Enforce that covenanted groups use the same constraint script as the first input from that same group.  Efficient implementations should discover this constraint script while iterating through the inputs during a previous step)*


## Footnotes

1. If, upon review, you find this structure “ugly”, you are welcome within your own software to completely restructure your transaction data as you see fit. But when computing the hash of a transaction or serializing transactions or UTXOs for network transmission, place the Group data in the serialized format as described.