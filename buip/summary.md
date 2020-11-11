# Technical Activity Status
*Development status and materials related to passed technical BUIPs and other technical activity*

## BUIP Status

This is a list of interesting and active BUIPs.   Some older completed or obsolete BUIPs have been removed.  Please edit this page to add technical BUIPs that you are interested in.

| BUIP | title | status | delegate | materials | notes |
|-------------|--------------|-------------|----------------|------------|---------|
| [BUIP033](/buipref/033.md) | Parallel Validation | complete | Peter Tschipper | | deployed in BU full node
| [BUIP057](/buipref/057.md) | Add BIP135 Support (version bits) | complete | | | deployed in BU full node
| [BUIP077](/buipref/077.md) | Group Tokenization | ongoing | Andrew Stone | [spec](https://docs.google.com/document/d/1X-yrqBJNj6oGPku49krZqTMGNNEWnUJBRFjX7fJXvTs/edit?usp=sharing)  | deployed in [nextchain](http://nextchain.cash)
| [BUIP098](/buipref/098.md) | BCH/BSV compromise | complete | Andrew Stone | | miners and community chose to split rather than vote
| [BUIP118](/buipref/118.md)/[119](/buipref/119.md) | CashAccounts | complete | open | [spec](https://gitlab.com/cash-accounts/specification/blob/master/SPECIFICATION.md), [test](https://github.com/BitcoinUnlimited/BitcoinUnlimited/blob/dev/qa/rpc-tests/electrum_cashaccount.py), [improved spec (not implemented)](https://github.com/BitcoinUnlimited/ElectrsCash/pull/23) | It was decided to deploy this in ElectrsCash rather than the full node wallet
| [BUIP120](/buipref/120.md) | immutable stateless storage | not started | open | see Peter Rizun's CashDrive  | waiting for a contributor
| [BUIP129](/buipref/129.md) | Finish and Productize the BU voting system | ongoing  | Dagur  | [Beta product](https://voter.cash)  | Working web site and app
| [BUIP131](/buipref/131.md)/[151](/buipref/151.md) | Bobtail Prototype | ongoing  | George Bissias |   | In development
| [BUIP135](/buipref/135.md) | Fund DoubleSpend Proofs | complete | Andrew Stone | | DS proofs deployed into BU full node
| [BUIP121](/buipref/121.md)/[145](/buipref/145.md) | Bitcoin Cash Specification | ongoing | Josh Green (Bitcoin Verde) and Andrew Stone | [web site](http://reference.cash) | Usable as a resource.  More details being added.
| [BUIP152](/buipref/152.md) | Wally Wallet | beta | Andrew Stone | [here](/wally) | working SPV p2p & electrum wallet

## Other Technical Activity

General development activities by Bitcoin Unlimited members do not need a BUIP.  For those unfamiliar with the articles, the BU Lead Developer has broad powers to move development forward.  BUIPs have 2 purposes; 1. to ensure that a feature will be incorporated into BU before spending the time to develop it, and 2. to block an initiative by the Lead Developer a member disagrees with.  However, in general the Lead Developer seeks permission from the membership via BUIP for any major change (i.e. hard fork) so the membership does not need to resort to 2.

This is a list of interesting technical activities undertaken by BU developers but have not become BUIPs.

| title | status | delegate | materials | notes |
|-------------|--------------|-------------|----------------|------------|---------|
| BCH explorer | stable | Andrea Suisani | [web site](explorer.bitcoinunlimited.net) | Bitcoin Cash blockchain explorer |
| ElectrsCash | stable/integrated | Dagur | [code](https://github.com/BitcoinUnlimited/ElectrsCash) | Electrum protocol server integrated into BU full node
| libbitcoincash | stable | Andrew Stone | | Shared library, built from the BU full node source code, that provides key wallet functionality.  Currently C++, Python, Java/Kotlin support
| libbitcoincashkotlin | stable | Andrew Stone | [code](https://gitlab.com/wallywallet/libbitcoincashkotlin) | Shared library that provides full SPV wallet functionality in Kotlin
| NextChain testnet | development | Andrew Stone | [web site](http:://www.nextchain.cash) | Group tokens, OP_EXEC, OP_PUSH_TX_DATA, OP_PLACE, Big Integers, covenants, MAST
| Txunami | stable | Andrew Stone | [code](https://github.com/gandrewstone/txunami)| High performance testnet transaction generator |
| Unconfirmed TX limits | ongoing | Peter Tschipper | | Limits in the thousands are supported.  Effort ongoing to scale any transaction DAG to mempool limits. 
| Wally Wallet | Beta | Andrew Stone | [info](/wally) | Android BCH SPV wallet with a focus on ecosystem development and adoption.