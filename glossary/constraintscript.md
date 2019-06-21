<!-- TITLE: Constraint Script -->
<!-- SUBTITLE: The rules for spending -->

Bitcoins are not sent to addresses, they are actually encumbered by a program (generally called a script, which is a term for a short, simple program that typically focuses on one thing).  Spending transactions supply input parameters (in the form of an input script) to this constraint script.  If the constraint script returns success, the coins are able to be spent.  Otherwise the transaction is invalid.

The term "constraint script" refers generally to any script that adds spending constraints.  In Bitcoin, there are actually two different scripts that can do this -- the [output script](/glossary/output_script) and the [redeem script](/glossary/redeem_script).