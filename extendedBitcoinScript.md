# Forkless inclusion of Functions, Loops, and New OpCodes in Bitcoin Script

*Andrew Stone, Oct 16, 2018*

*Bitcoin Script does not have loops, goto, or call/return instructions, so control flow (excepting “if”) is disallowed. However, it is possible to provide much of this functionality in practice. One reason for doing so is to keep the advantages of decidability while gaining the space efficiencies these language features allow. Another advantage is that function calls, finite loops, and new opcodes can be added to the language in practice without a hard fork.*

To do this, we will transform Bitcoin Script (BS) into a new language Extended Bitcoin Script (EBS). Nodes can communicate transactions containing EBS scripts to other EBS-compliant nodes and evaluate EBS scripts directly. However they can also transform EBS into BS and send the transaction to BS nodes if needed.

# Local Function Calls and Definitions

First, let us specify a function and function call.

OP_DEFFN <Name>  
… <any code>  
OP_FNEND

Functions take parameters from the stack and returns values on the stack. To execute a function, we can define a call opcode:

OP_CALL <Name>

These operations can be “compiled” into BS simply by replacement. Replace OP_CALL <name> with <any code> where ever it appears in the script.

However, actual replacement is unnecessary to execute the script. Let’s propose that we have extended full node client software to execute EBS directly. Now, when an EBS interpreter encounters OP_CALL, it can “call” into the function body, execute that, and “return” back to the next instruction. It should be obvious that the exact same sequence of BS instructions are executed as if replacement had actually occurred, and the EBS instruction (OP_CALL) does not modify program state. These observations prove that the EBS interpreter is equivalent to actual replacement and then execution by BS interpreter..

From a consensus perspective, script properties such as the length and sigop count is not that of the EBS script, but that of its BS equivalent. To calculate these, pass linearly through the EBS script (as we do today to calculate the same for BS scripts). BS instructions increment the relevant property metric as is done today. When OP_DEFFN is encountered, OP_DEFN, OP_FNEND and all code inside (the actual function definition) should not increment the property metric. Instead these property metrics should be calculated for the function alone and remembered for use when OP_CALL<Name> is executed. When an EBS interpreter encounters OP_CALL<Name> it increases the script length by the length of <any code> and decreases it by the length of “OP_CALL<Name>”. This process results in the EBS script calculating the exact same values for the property metrics as if the calculation was done for the equivalent BS script.

Functions definitions must appear in scripts before use — this prevents uncompilable possibilities like recursion or mutual calling.

# Global Function Calls And Definitions

A 256-bit cryptographic hash can be computed from every function definition by taking the double-sha256 of its contents. Without modifying the Bitcoin security model, this can be assumed to be a unique identifier since Bitcoin currently makes the same assumption for transactions and blocks, and a weaker 160 bit non-collision assumption for the redeemScript in Pay To Script Hash (P2SH) style transactions. Every defined function therefore has a global identifier.

In EBS, a script can call a function by its global identifier:

OP_CALL <Global Name>

How a node discovers global function definitions varies. Clearly the “honest” node that provided the script (that calls a global function) must be able to provide the function’s definition. If the node cannot provide the global function definition, it did not validate the script before relay, and should be banned. Nodes can cache global function definitions so they are requested rarely compared to transaction requests. This could significantly compress the effective size of transmitted transactions (for nontrivial scripts).

# New Opcodes

However, there is no need to ever transmit commonly used global functions. Instead, node client software may be shipped with a library of common global functions, and only download missing ones from other nodes.

This library of “canned” functions can be implemented in EBS and interpreted every time the function is called, but there is a more efficient alternative — implement the function in the client node’s native language. To be clear, the function must also have an EBS implementation and its global name is the hash of that implementation, and properties such as script length, instruction count and number of sigops are that of the EBS implementation. However, when the function is called, the client node could actually run the native implementation. Significant speedups should be possible due to the overhead of interpretation, the elimination of stack manipulation (unnecessary for register-based machines), access to native operations on larger integers, and parallelism.

A EBS to native “compiler” could be created to generate native implementations in a provably correct manner. However, unless that compiler contains awesome and probably novel optimization technology, it is unlikely to nearly match that of a human implementation due to problems like BS needing to construct large number addition and multiplication out of 32 bit ones complement signed arithmetic, but a native implementation could use a large number library that contains hand-optimized assembly. However, a native implementation risks correctness unless a proof-of-equivalence is made between the EBS script and the native implementation. This is a very difficult task for languages not explicitly defined to facilitate these proofs. Therefore native implementations will likely use the traditional code review and test process and should be deployed judiciously.

One advantage is the existence of the implementation written in EBS that effectively becomes a “reference implementation” that could be used in comparison to the native implementation both during testing, with fuzzing tools for example, and even during normal operation.

Finally, very commonly used functions can be given a well-known short identifier. These could be defined for a particular software client version (which is discovered via XVERSION upon node connection) or as an industry-wide standard. Other short identifiers can be assigned dynamically via mutual agreement between 2 nodes based on function call frequency analysis of transactions in prior blocks.

We have therefore created native implementations of procedures that are identified by short identifiers. This is functionally equivalent to adding new opcodes.

# Looping

Looping a known number of times can be implemented via a technique normally used in optimization called “loop unrolling”. Quite simply:

OP_REPEAT<N>  
  <code>  
OP_REPEATEND

is expanded to:

<code> <code> … (N times)… <code>

<code> can be BS code, or EBS code containing OP_CALL or OP_REPEAT constructions. Of course, the human language will use more modern syntax, and EBS “compilers” will convert. For example:

repeat(5)  
{  
    <code>  
}

Conditional looping of N times or less is also possible.

OP_REPEAT_IF<N, {condition check code}> {looping code}

Or with modern syntax:

Nwhile(20, {condition check code})  
{  
    <looping code>  
}

is expanded to:

{conditional check code}  
OP_IF  
  {looping code}  
  {conditional check code }  
  OP_IF  
    {looping code}  
    {conditional check code}  
    … (n times) …  
  OP_ENDIF  
OP_ENDIF

The if clauses are nested so that the first failure falls out of the entire clause. It is possible to to the same without nesting, but this requires pushing items onto the stack.

As with function calls, actual expansion (translation into BS) is not necessary. An EBS implementing node can execute the OP_REPEAT or OP_REPEAT_IF instructions directly and the result will be the exact same instruction execution sequence as executing the compiled BS code.

# Fees and Consensus

We have shown how function calls and looping can be added to Bitcoin Script in practice without using a hard fork.

This technique uses macro expansion and loop unrolling to create what is in essence one long sequential script, even though this sequential script may never actually exist in memory. The purpose of this virtual script is to report properties such as script length and number of sigops that are used for consensus checking and fee calculation. For these techniques to have practical use, it is necessary to increase the maximum number of opcodes and perhaps the maximum script length — at a maximum BS size of 100 instructions, there is very little room to produce interesting scripts.

If we imagine that this script length is dramatically increased, these techniques begin to separate the reported cost of operations from the actual cost. For example, the “script length” is no longer the real number of bytes needed to store the script on disk. Nor is it the number of instructions executed (due to untaken if statements). Also, the execution time of a native implementation of a function will be much lower than its execution as a BS script. Therefore the instruction count metric loses its meaning as a measure of how long it will take the script to be executed.

This is the conceptual error that Ryan Charles makes when he claims “OP_DATASIGVERIFY is million-fold subsidy” ([https://www.yours.org/content/how-to-implement-ecdsa-signature-verification-in-script-and-why-datasi-9f113344542f](https://www.yours.org/content/how-to-implement-ecdsa-signature-verification-in-script-and-why-datasi-9f113344542f)). First, the run time of a native implementation is going to be significantly shorter than that of the equivalent BS script. Second, CHECKSIG already has this problem, so singling out DATASIGVERIFY smacks of politics. Third, we have solved the problem for CHECKSIG, by implementing a “sigops” limit, and DATASIGVERIFY is using that solution. Fourth, we are trying to build a real, practical money-for-the-world solution, not a research project, and so if given the opportunity to replace a million lines of BS with a pre-existing native solution (already used in CHECKSIG) the choice is obvious.

# Conclusion

Arbitrary opcodes implementing not-quite Turing Machine expressible algorithms (constrained by limited number of operations, limited tape (stack), and limited size loops) are possible to implement in BCH nodes in a permissionless manner. We cannot stop them. So, when an algorithm proves popular, we must consider a long-term strategy to implement popular functions natively as opcodes via a hard fork.

Admittedly, adding them also does not provide many advantages to an EBS user or EBS compliant node with a native implementation of the function. However, it does have two effects:

1.  Brings the script property metrics back into line with actual costs.
2.  Standardizes a native implementation of an algorithm which reduces risk of forks due to a divergence between a BS and native implementation.

Function calls and constrained loops still guarantee termination, because termination for all BS scripts is already proven, and we transformed EBS into BS. There is therefore no reason to prevent their inclusion via hard fork into BS. However, as above, there is also little benefit to EBS compliant nodes.
