# Technical Activity Status
*Development status and materials related to passed technical BUIPs and other technical activity*

## BUIP Status

This is a list of interesting and active BUIPs.   Some older completed or obsolete BUIPs have been removed.  Please edit this page to add technical BUIPs that you are interested in.

| BUIP | title | status | delegate | materials | synopsis |
|-------------|--------------|-------------|----------------|------------|---------|
| BUIP033 | Parallel Validation | complete | Peter Tschipper | | deployed in BU full node
| BUIP057 | Add BIP135 Support (version bits) | complete | | | deployed in BU full node
| BUIP118/119 | CashAccounts | not started | open | [spec](https://gitlab.com/cash-accounts/specification/blob/master/SPECIFICATION.md) | waiting for a contributor
| BUIP120 | immutable stateless storage | not started | open | see Peter Rizun's CashDrive  | waiting for a contributor
| BUIP129 | Finish and Productize the BU voting system | ongoing  | Dagur  | [Beta product](https://voter.cash)  | Working web site and app
| BUIP131/151 | Bobtail Prototype | ongoing  | George Bissias |   | In development
| BUIP135 | Fund DoubleSpend Proofs | complete | Andrew Stone | | DS proofs deployed into BU full node
| BUIP121/145 | Bitcoin Cash Specification | ongoing | Josh Green (Bitcoin Verde) and Andrew Stone | [web site](http://reference.cash) | Usable as a resource.  More details being added.
| BUIP152 | Wally Wallet | beta | Andrew Stone | [here](/wally) | working SPV p2p & electrum wallet

# Technical Activity Status

General development activities by Bitcoin Unlimited members do not need a BUIP.  For those unfamiliar with the articles, the BU Lead Developer has broad powers to move development forward.  BUIPs have 2 purposes; 1. to ensure that a feature will be incorporated into BU before spending the time to develop it, and 2. to block an initiative by the Lead Developer a member disagrees with.  However, in general the Lead Developer seeks permission from the membership via BUIP for any major change (i.e. hard fork) so the membership does not need to resort to 2.

This is a list of interesting technical activities undertaken by BU developers but have not become BUIPs.

| title | status | delegate | materials | synopsis |
|-------------|--------------|-------------|----------------|------------|---------|
| Unconfirmed TX limits | ongoing | Peter Tschipper | | Limits in the thousands are supported.  Effort ongoing to scale any transaction DAG to mempool limits.
| libbitcoincash | stable | Andrew Stone | | Shared library that provides key wallet functionality.  Currently C++, Python, Java/Kotlin support
| libbitcoincashkotlin | stable | Andrew Stone | [code](https://gitlab.com/wallywallet/libbitcoincashkotlin) | Shared library that provides full SPV wallet functionality in Kotlin
| Wally Wallet | Beta | Andrew Stone | |
| Txunami | stable | Andrew Stone | | High performance testnet transaction generator
| BCH explorer | stable | Andrea Suisani | [web site](explorer.bitcoinunlimited.net) | Bitcoin Cash blockchain explorer
| ElectrsCash | stable/integrated | Dagur | [code](https://github.com/BitcoinUnlimited/ElectrsCash) | Electrum protocol server integrated into BU full node