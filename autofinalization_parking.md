# Attacks Against Auto-finalization and Fork Parking

<center>Andrew Stone, Bitcoin Unlimited</center>
<center>Oct 23, 2020</center>

*An analysis of attack methods and costs to produce persistent forks on blockchains that use automatic trailing checkpoints (finalization) and additional proof-of-work requirements (parking) in their consensus algorithm*

## Abstract
The consensus achieved by the parking and auto-finalization technologies in Bitcoin Cash prevents intentional chain reorganization.  But they are inconsistent with consensus theory and that strongly implies that these technologies are not innovations in consensus but rather trade correctness for convenience.  This paper provides an analysis that describes specific attacks against auto-finalization and parking and shows the cost and likelihood that a malicious miner could successfully execute them.  

## Definitions

**Persistant fork** in this paper means a failure of the consensus algorithm to achieve consensus, requiring an extra-algorithmic correction.  In practice, this would mean a fork of the blockchain that first requires human intervention first to come to consensus by picking one of the two forks, and then again at every affected node to manually force a blockchain reorganization onto the chosen fork.

**Finalization** is a technique to create non-persistent blockchain checkpoints.  All forks that do not contain the finalized block are marked invalid persistently<sup>f1</sup>.   Full nodes can be directed to consider one particular block, F, "final".  F is stored in RAM and any blocks that are not on F's chain are marked as invalid.  This is not quite a checkpoint for two reasons: 
 * If the full node is restarted, F is forgotten.
 * Only one block is considered final at a time.

These differences are noted here for precision but may simply be an implementation convenience.

**Auto-finalization** is a process where the mainchain block that is at the **auto-finalization depth** (which is 10 on BCH) is marked as the "final" block.

**Parking** is a modification to the rule that determines the main chain proposed by Satoshi<sup>1</sup> "Nodes always consider the longest chain to be the correct one and will keep working on extending it."  The parking technique observes that a full node that is not in "initial block download" mode has a fork that it currently sees as the main chain (M), and a candidate fork ( C ) that it is considering switching to.  A full node that implements parking will not switch to C unless it exceeds the work in M by an amount that varies depending on the length of the fork M:
$$
forkBlock = LastCommonBlock(M,C)  \tag1
$$

$$
mainForkLength = Height(M) - Height(forkBlock))  \tag2
$$

$$
ExtraParkingWork = \left\{ \begin{array}{l}
     \frac{Work(M) - Work(forkBlock)}{2} \ if \ mainForkLength \ is \ 1 \\
     \frac{Work(M-1) - Work(forkBlock)}2 \ if \ mainForkLength \  is \ 2 \ or \ 3 \\
     Work(M) - Work(forkBlock) \ otherwise \\
     \\
 \end{array} \right. \tag3
$$

If we make the simplifying "steady state mining" assumption that each block requires approximately the same amount of work w, then this equation simplifies to:
$$
ExtraParkingWork = \left\{ \begin{array}{l}
     \frac{w}{2} \ if \ mainForkLength \ is \ 1 \ or \ 2 \\
     w \ if \ mainForkLength \  is \ 3 \\
     w * mainForkLength \ otherwise \\
     \\
 \end{array} \right. \tag4
 $$

## Background

The auto-finalization and parking logic was added to the Bitcoin ABC client in response to the Bitcoin Cash/Bitcoin SV fork.  At that time, there was fear that a short-term economically irrational actor would continually reorganize the Bitcoin Cash fork with empty blocks (or execute other attacks).  Doing so would block all transactions from confirming on-chain, encouraging abandonment of the fork.  By finalizing blocks 10 deep, no deeper reorganization can occur.  But it was observed that this is not good enough since an attacker could simply release its reorganizations every 8 or 9 blocks, although it does prevent doublespend attacks (since exchanges would not release funds until finalization occurred).  Parking was proposed to discourage this activity, since an attacker would have to produce twice as much work to cause a reorganization, except for the first 3 blocks.  One might expect that an attacker simply reorganize every 2 or 3 blocks, since the parking algorithm requires much less extra work for forks of those lengths.  However even with majority hash power, there is a probability that the attacker will fail to create the longest chain, and this probability increases as the depth decreases.  So the honest minority would be able to get blocks confirmed and given BCH's utilization these few blocks could confirm all existing transactions.  For example, if the honest miners have only 20% of the hash power, they will win a 3 block race 5.2% of the time but a 9 block race only 0.1% of the time<sup>1</sup>.

No such attack materialized.  

## Persistent Forking Attacks Against Auto-finalization

For completeness Eclipse/Partition Attacks and node synchronization failures are mentioned next.  However the main attack presented in this paper is the "fork matching" attack, described in section 3.

### Eclipse/Partition Attack

An eclipse or partition attack is an attack in which the target node or nodes cannot communicate outside of the eclipsed group, except via the attacker.

If an attacker can maintain an eclipse or partition for the auto-finalization depth number of blocks, the isolated nodes will persistently fork from the other nodes.

Auto-finalization therefore imports the networking architecture and source code into the forking attack surface of the cryptocurrency, whereas it was previously only a problem for doublespend attacks.

### Node Synchronization Failure

The effect of chain parking is that most nodes do not switch to the most-work chain.  A synchronizing node has "current" chain, so will choose the most work chain -- that is the one that is parked.  If this fork is greater than 10 blocks, and is maintained, it will be chosen by all new nodes in the system.  It will also be chosen by all SPV wallets.  

This would be a very expensive attack to maintain, but would force BCH developers to hard code a checkpoint into every full node and SPV wallet to rejects the attacker's chain.

### Fork Matching Attack

This attack can still cause a persistent fork in a connected network -- that is, no eclipse or partition attack is necessary.  In summary, the attacker will cause a fork and maintain it for the auto-finalization depth number of blocks, whereupon the first forked blocks are finalized in every node, resulting in a persistent fork.

Let us first examine the attack against nodes that implement finalization but not parking.

#### Preparation
To prepare for this attack, the attacker positions computing resources near full nodes associated with the services the attacker would like to fork.  These full nodes can be identified reliably by techniques discovered in [3].  The attacker must be able to reliably be the node that provides the target full node new blocks.  This is easy to verify before the attack begins, by determining whether the target node is requesting blocks from (issuing INV blocks to) an attack node.  An attacker may also be able to employ protocol shortcuts (such as forwarding a block without header or INV, or forwarding the header and then the block without waiting for a GETDATA) to further increase its block propagation rate relative to other nodes, depending on the details of a node's protocol implementation.

Note that if the attacker's objective is general mayhem rather than forking specific target nodes and services, the attack is easier since it can be targeted towards N nodes but still succeed if some nontrivial number of nodes are forked.

#### Trigger
In the first step the attacker triggers a fork F off of main chain M.  This is accomplished by mining a block $F_0$ and then waiting for another miner to mine a block $M_0$.  As soon as the attacker receives notification of the block $M_0$, it forwards $F_0$ instead, using any available protocol shortcuts.  Based on the analysis done during setup, $F_0$ will beat $M_0$ to the target nodes.  

#### Repeat

The attacker now attempts to mine a number of blocks equal to the blockchain's auto-finalization depth $F_1$ to $F_{f}$.  As main chain blocks are discovered by the rest of the network, the attacker propagates its blocks directly to its targets.  This ensures that the target nodes see the attackers block first, but that (in general) the rest of the network does not.

A node does not switch from its current chain to an equal work chain, so as long as block propagation is controlled, the fork can be maintained.

There is a risk that the target nodes will propagate the fork to untargeted nodes, converting those nodes from M to F.  There is also a risk that some of the target nodes will see the main chain block first, converting them from F to M.  These risks may be acceptable depending on the attacker's goals.

Once the fork has been maintained for the auto-finalization depth, the fork will become persistent and the attack can stop.

#### Problems

**P1 "Attack Foiled"**: The attacker must have produced as many or more blocks than the main chain miners every time the main chain miners produce a block, so that it can push the next block on F the moment a block on M is discovered.  Analysis of the exact success probability of this is included below.

**P2 "Attack Mistimed"**:  If the attacker presents $F_n$ too early or late, it can cause nodes to move between forks F and M.  Proper timing is experimentally easy to achieve on regtest.  However, the only way to determine whether this applies to the mainnet is to try it.  It turns out that it does not matter for "parking" nodes so the analysis of this problem stops here.

**P3 "Attack Spoiled"**: If the targeted nodes are miners, and they discover and propagate a block on F, the nodes mining M will switch to F, ending the attack.  Note that since F is now the main chain, F has lost no money.  Actually, this constitutes a selfish mining attack<sup>4</sup> so F's actions will increase its profitability on subsequent blocks by reducing difficulty.  To put some numbers around this problem, the likelihood that a miner with hash power 'p' as a fraction of the total hash power does not produce a block in N nodes can be calculated.  It is 1 minus likelihood that the rest of the hash power "wins" n times, or:
$$
spoil \ probability = 1 - (1-p)^n  \tag5
$$

For example, targeting 5% of the hash power will be spoiled 40% of the time with an auto-finalization depth of 10.  However, note that in cryptocurrencies like BCH, which command a small fraction of available hash power, it would not be difficult for a miner to direct a significant quantity of hash to both M and F<sup>F2</sup>, dramatically reducing the targeted hash power's percentage.

**P4 "ASERT interference"**: Since ASERT recalculates difficulty for every block, is it near impossible to have the exact same POW on two forks?  If so the fork will heal.  Or will the "absolutely scheduled" nature of ASERT generally result in the same difficulty?  It turns out that it does not matter for "parking" full nodes so the analysis of this problem stops here.

#### "Attack Foiled" Analysis

This attack differs from the traditional doublespend because the attacker must remain ahead of the main chain throughout the entire attack interval of auto-finalization depth (AFD) blocks. 

Let:

T => mining interval in minutes/block, with 100% hash power (e.g. 10 minutes)
q => the attacker's proportion of hash power 
p = 1-q => honest miner's proportion of hash power

So the honest miner's mining interval (minutes/blocks)
$$
Hmi = T/p \tag1
$$
And the attacking miner's mining interval (minutes/blocks)
$$
Ami = T/q \tag2
$$

How much time to extend the mainchain by z blocks, on average?
$$
= z * Hmi = z * T/p  \tag3
$$

How many blocks will attacker produce during honest miner's time to mine z blocks?  

Answering this question gives us the Poisson interval (lambda) of the attacker, expressed in the time expected for the honest nodes to produce a z block.  It is the attacker's block rate (inverse of 2) * the time available (3) or:

$$
lambdaAttacker = q/T * z*T/p = zq/p  \tag4
$$

Let us propose the attacker is tied at the AFD-1th block and must produce the AFDth block first.  The probability is the sum of the probabilities that the attacker will produce any of 1 to infinity blocks in the time it takes the honest miners to produce z=1 blocks.  Modelling mining as a Poisson process results in:
$$
1 - P(X= k \ successes = 0, \lambda = zq/p) = \frac{\lambda^ke^{-\lambda}}{k!} = 1- e^{-zq/p} = 1- e^{-q/p} \tag5
$$

Now let us propose that the attacker is tied at the AFD-2nd block and must win.  This is the probability that the attacker produces 2 or more blocks (directly wins), or that the attacker produces 1 block times the probability of a win from that new position.  

In general, if N blocks remain in the race, the probability of a win is the sum of the probability of producing 1,2,...N-1 blocks in one interval times the probability of winning from that new position, plus the probability that N or more blocks are discovered in this interval (which would be an automatic win):

The following equation is not quite correct, because it does not model the "head start" that the F chain gains if it mines more than 1 block within an M chain discovery interval.  But it is presented here as a stepping stone.
$$
W(N) = (\sum\limits_{i=1}^{N-1} P(X=i \ successes, q/p)W(N-i)) + \sum\limits_{i=N}^\infty P(X=i \ successes, q/p)
$$

To fully express the model, we need to introduce variables denoting the main and fork lengths (Mlen and Flen):

$$
W(N, Mlen, Flen) = \left\{ \begin{array}{l}
     0 \ if \ Flen < Mlen, \\
     1 \ if \ Flen >= N, \ otherwise \\
       (\sum\limits_{i=1}^{N-1-Flen} P(X=i \ successes, q/p)W(N-i, Mlen+1, Flen+i)) + \sum\limits_{i=N-Flen}^\infty P(X=i \ successes, q/p) \\  
 \end{array} \right.
$$

This can be calculated and plotted for a variety of auto-finalization depths and attacker hash, resulting in the following graph (see Appendix 1 for jupyter code):

<img src="/finalizationMatchingAttack.png" width=600></img>

Visually, this system is fairly robust against minority hash attackers at BCH's 10 auto-finalization depth.  Quantitatively, the likelihood of a successful attack by a 50% miner is only 25%.  However, the situation changes rapidly above about 60% hash, with an attacker with 2/3rds of the hash power resulting in a 82% chance of success and a 3/4ths miner having an 94% likelihood of forcing a persistent fork.

These majority hash attacks are very relevant to a minority hash coin like BCH since a relatively small BTC miner can easily produce this hash power and direct it temporarily onto BCH.  With an auto-finalization depth of 10, an attacker would have to redirect this hash power for about an hour to create a persistent fork.

#### Price

Let us assume that hash power is readily available to be diverted to the attack.  This is generally true for any "minor" blockchain -- that is any chain that shares its proof-of-work algorithm with another chain that consumes the majority of the available hash power, and is true for BCH which is the only chain that the author is aware of that uses auto-finalization and fork parking.  Without this assumption there is an unquantifiable cost to fabricate, deploy and manage the additional hash power required to execute this attack.

In BCH, the cost to mount the attack is the cost of production of 10 blocks.  At the time of this writing, if we assume miners are breaking even, this would be approximately $10 * 6.25 * 250$ or $15000$ USD.  This author feels that this is a very small amount compared to the profits that might be gained by leveraged short positions in BCH and executing one or multiple fork attacks.  Additionally, forcing a persistent fork, then abandoning it, can be used to doublespend, similar to a blockchain reorganization attack (in theory anyway; in practice, an observant exchange might halt operations on the forked chain).

It is unclear how BCH would "heal" a persistent fork.  If the attacker's chain is chosen as the main chain, there would be no loss, except as caused by BCH price declines and the miner's forced holding of the mined 62.5 BCH for 100 blocks.

A P1 failure results in the loss of between 1 and 9 orphan blocks or between $1000 and $14000 USD.

A P2/P4 failure may result in the loss of between $1000 and $14000 USD, or no loss if the attacker's fork is chosen as the main chain.

A majority hash attack is much more likely to fail early, costing less money, than it is to fail later, since the more blocks mined, the more the majority chain tends to pull ahead of the minority.

A P3 failure results in no loss of money.

## Persistent Forking Attacks Against Parking Auto-finalization Nodes

We can now consider how fork "parking" affects the main attack described in this paper; the "fork matching" attack.  Recall that "parking" causes nodes to resist switching forks unless the other fork's work exceeds the current one by a variable amount as described earlier.

### Attack is Possible with Lower Hash or Succeeds With Greater Probability

Note that "parking" significantly relaxes the finish criteria.  If our attacker is working on the last block of F (at the auto-finalization depth - 1), M must be over 2*(auto-finalization depth - 1) blocks ahead to trigger F nodes to move back to M.  This relationship is true for prior blocks as well, to fork depth 2.  Ignoring the first 2 blocks, the F chain can grow half as fast as the M chain and still succeed.

### P2: "Attack Mistiming", Relaxed

A fork depth of 1 is not relevant, since that is the initial fork block.  Parking does not affect the depth 2 block since the excess work is half a block.  However, once the fork depth is 3 or greater, parking requires at least one block of extra work.  This means that an arrival order problem will be ignored.   

If the next M block is presented first to F-following nodes, it will not cause a chain switch.  If the next F block is presented first to M-following nodes, there will similarly be no switch!

Parking therefore significantly reduces the likelihood of timing problems.

### P4: "ASERT Interference", Solved

Similarly to the way attack mistiming becomes more lenient due to chain parking, minor variations in the chain work due to ASERT retargeting difficulty in every block will be ignored.

### P1: "Attack Foiled", Foiled

A significant problem is that the F chain must to stay even with or be ahead of the M chain every time a M block is found.  This is a significantly more strict requirement than, for example, that the F chain eventually be even with the M chain (as is needed for the classic doublespend attack).  Parking relaxes this requirement to some degree.  By block 3, the F chain can fall 1 block behind.  And by block M > 3 the F chain can fall M blocks behind without triggering a chain switch.  The probability analysis was rerun with the BCH parking rules, yielding the following graph

<img src="/finalizationParkingMatchingAttack.png" width=600></img>

Parking allows significantly less applied hash power to result in greater attack success probability, especially at lower hash levels. To repeat the data points presented earlier, 50% hash now has a 48% success rate (double), 66% an 83% success rate and 75% has a 94% success rate (about the same).  

## Conclusion

The existence of theoretical results such as [5] and [6] that prove consensus impossible under certain constraints were deftly avoided by Satoshi because Bitcoin actually never achieves consensus.  Instead it achieves a probability of consensus that increases as the statement's depth in the chain increases.  And in practice it eventually becomes economically infeasible for the statement to be changed.

The consensus achieved by Bitcoin Cash's parking and auto-finalization technologies discourages intentional chain reorganization and completely prevents it beyond 10 blocks.  But it is inconsistent with consensus theory and that strongly implies that these technologies are not innovations in consensus but rather a tradeoff.  This tradeoff was neither identified or analyzed by the authors of these technologies, as far as I am aware.  This paper furnishes that analysis and shows how these technologies fail to achieve consensus and analyse the possibility of intentionally triggering such a failure.  

Instead of intentional chain reorganization, attackers can create intentional chain "deorganization", achieving much the same outcome in terms of doublespends or general mayhem.  The question we may now want to ask is whether the risk of persistent consensus failure is worth the price of traditional doublespend reorganization resistance?


## Footnotes
[F2] Note that mining both the main chain and fork would also allow the attacker to deliver both M and F blocks directly to targets simultaneously, reducing the chance of propagation problems.  It could also delay blocks it discovers on M.

## Appendix 1

### Probability Calculation for Fork Matching Attack Against Auto-finalization

```python
import math

def poisson(success, lam):
    return (  (lam**success)/(math.factorial(success)*(math.e**lam)) )

def poissonNabove(success, lam):
    acc = 0.0
    for i in range(0,success):
        acc += poisson(i,lam)
    return 1-acc

calced2={}
def FinParkFork(attackerHash, Mlen, Flen, finalizationDepth, park):
    # print(Flen, Mlen)
    if (Flen >= finalizationDepth):
        return 1.0

    if park:
    # cannot fall behind in fork depth 1 & 2
        if (Mlen<3 and Flen<Mlen):
            return 0.0
    # can fall behind no more than 1 block in fork depth 3
        elif (Mlen==3 and Flen<2):
            return 0.0
    # for other fork depths, attacker cannot fall behind more than double
        elif (Flen*2 < Mlen):
            return 0.0

    elif (Flen < Mlen):  # without parking, fork cannot ever fall behind
        return 0.0

    # Look in the cache for values we've already found
    if (float(attackerHash),Mlen,Flen,finalizationDepth,park) in calced2:
        return calced2[(float(attackerHash),Mlen,Flen,finalizationDepth,park)]

    honestHash = 1-attackerHash
    interval = attackerHash/honestHash

    acc = 0.0
    for i in range(0,finalizationDepth-Flen):
        # Model the F chain finding i blocks while the M chain finds 1
        acc += poisson(i,interval)*FinParkFork(attackerHash, Mlen+1, Flen+i, finalizationDepth, park)
    # Model the F chain finding more than what it needs to win in this one interval
    acc += poissonNabove(finalizationDepth-Flen, interval)

    calced2[(float(attackerHash),Mlen,Flen,finalizationDepth,park)] = acc
    return acc

import numpy as np
import matplotlib.pyplot as pyplot

def plotitP():
    x = np.linspace(start=0.0, stop=0.99, num=100)
    fig = pyplot.figure(2)
    plt = fig.add_subplot()
    plt.grid(color=(0.8,0.8,0.8))
    plt.set_xlabel('Attacker hash proportion', fontsize=15)
    plt.set_ylabel('Success probability', fontsize=15)

    y2 = [ FinParkFork(i,0,0,1,False) for i in x]
    plt.plot(x, y2, color=(.97,.95,0.9))

    y2 = [ FinParkFork(i,0,0,2,False) for i in x]
    plt.plot(x, y2, color=(.95,.88,0.8))

    y1 = [ FinParkFork(i,0,0,5,False) for i in x]
    plt.plot(x, y1, color=(.9,.8,.7))

    y3 = [ FinParkFork(i,0,0,20,False) for i in x]
    plt.plot(x, y3, color=(.8,.7,.8))
    y4 = [ FinParkFork(i,0,0,40,False) for i in x]
    plt.plot(x, y4, color=(.9,.8,.9))

    y = [ FinParkFork(i,0,0,10,False) for i in x]
    plt.plot(x, y, color=(.9,0,0))

    fig.suptitle("P: Success Probability for the Matching Fork Attack\nat 1,2,5,10(red),20,and 40 block autofinalizations", fontsize=16)
    fig.show()
    

def plotitParked():
    x = np.linspace(start=0.0, stop=0.99, num=100)
    fig = pyplot.figure(3)
    plt = fig.add_subplot()
    plt.grid(color=(0.8,0.8,0.8))
    plt.set_xlabel('Attacker hash proportion', fontsize=15)
    plt.set_ylabel('Success probability', fontsize=15)

    y2 = [ FinParkFork(i,0,0,1,True) for i in x]
    plt.plot(x, y2, color=(.97,.95,0.9))

    y2 = [ FinParkFork(i,0,0,2,True) for i in x]
    plt.plot(x, y2, color=(.95,.88,0.8))

    y1 = [ FinParkFork(i,0,0,5,True) for i in x]
    plt.plot(x, y1, color=(.9,.8,.7))

    y3 = [ FinParkFork(i,0,0,20,True) for i in x]
    plt.plot(x, y3, color=(.8,.7,.8))
    y4 = [ FinParkFork(i,0,0,40,True) for i in x]
    plt.plot(x, y4, color=(.9,.8,.9))

    y = [ FinParkFork(i,0,0,10,True) for i in x]
    plt.plot(x, y, color=(.9,0,0))

    fig.suptitle("Success Probability for the Matching Fork Attack\nat 1,2,5,10(red),20,and 40 block parked autofinalizations", fontsize=16)
    fig.show()
    
def main():
    print("F  50% at 10: ", FinParkFork(0.5,0,0,10,False))
    print("F  66% at 10: ", FinParkFork(2.0/3.0,0,0,10,False))
    print("F  75% at 10: ", FinParkFork(0.75,0,0,10,False))
    print("PF 50% at 10: ", FinParkFork(0.5,0,0,10,True))
    print("PF 66% at 10: ", FinParkFork(2.0/3.0,0,0,10,True))
    print("PF 75% at 10: ", FinParkFork(0.75,0,0,10,True))
    plotit()
    plotitParked()
```



## References

[1]: The Bitcoin white paper: https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwjk8IrmmcbsAhXQhHIEHX00BOsQFjAAegQIBBAC&url=https%3A%2F%2Fbitcoin.org%2Fbitcoin.pdf&usg=AOvVaw05-4mYD7EyyKjwcHh8i0Vw

[2]: Parking as implemented in Bitcoin Cash Node in file validation.cpp function CBlockIndex *CChainState::FindMostWorkChain()

[3]: https://read.cash/@PeterRizun/exploring-long-chains-of-unconfirmed-transactions-and-their-resistance-to-double-spend-fraud-abaecca9

[4]: The Majority is Not Enough: Bitcoin Mining is Vulnerable: https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwjW38vRs8bsAhWhl3IEHUB9BDcQFjABegQIBxAC&url=https%3A%2F%2Fwww.cs.cornell.edu%2F~ie53%2Fpublications%2FbtcProcFC.pdf&usg=AOvVaw3k8Sf3q_85cWxCUA0JcBBa

[5]: Impossibility of Distributed Consensus with One Faulty
Process: https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwiF2IHPxsbsAhX-l3IEHUKoDuAQFjADegQIARAC&url=https%3A%2F%2Fgroups.csail.mit.edu%2Ftds%2Fpapers%2FLynch%2Fjacm85.pdf&usg=AOvVaw3cwr00WJuxyxJUTcm4rELk

[6]: The Byzantine General's Problem: https://www.microsoft.com/en-us/research/publication/byzantine-generals-problem/?from=http%3A%2F%2Fresearch.microsoft.com%2Fen-us%2Fum%2Fpeople%2Flamport%2Fpubs%2Fbyz.pdf
