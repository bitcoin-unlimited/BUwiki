# Trusted Delegated Payment Protocol

  
The Delegated Payment Protocol is a protocol that allows an entity and a cryptocurrency wallet to establish a trust (within limits) relationship, allowing the entity to delegate cryptocurrency operations to the wallet.  Based on policies set up by the wallet owner (the "user"), the wallet may automatically execute delegated operations, or may request user authorization.  The entity requesting payment has no control over whether the payment request is handled automatically, or promoted to user authorization.  Delegated operations can be simple payment, or the funding and signing of more complex transactions.  Entities can be anything that can execute a cryptographic signature, or whose identity is validated by other means (such as via https).  Common entities are web sites and smart phone apps.

This facility can be broadly compared to the automatic fiat payment systems available via ACH (and others) that are used to deduct monthly membership dues, etc.  However, this facility may be used both for "traditional" (monthly bill) style payments, and for emerging small payment applications that are enabled by Bitcoin Cash's payment efficiency.     

From a philosophical standpoint this service is very different than traditional ACH automatic payment permission.  In this protocol, user control over these trust relationships is centered within the wallet, and the wallet is an active agent in the execution of every payment.  This means that this trust relationship is easily adjusted by the user.  In some traditional systems, control over the trust relationship is delegated to the entity receiving the funds.  At best, this creates problems -- a different and separate system must be learned by the user for each relationship, often involving slow and obsolete technology, for example [a physical, stamped and mailed letter with numerous required and redundant details](https://donotpay.com/learn/cancel-golds-gym-membership/).  At worst, the entity deliberately obfuscates or complexifies their system to retain payments, and/or [charges the user to stop charging the user](https://www.consumerfinance.gov/ask-cfpb/how-do-i-stop-automatic-payments-from-my-bank-account-en-2023/).  

With problems like these, it seems as if many banking systems have essentially been co-opted by bank/business relationships to extract value from the hapless depositor (who must participate because of the significant problems with holding physical bills of currency whose high denominations have been retired and is losing value to inflation).  By returning control to the wallet owner, this protocol is aligned with cryptocurrency philosophy that seeks to provide an alternative that protects depositors rather than fleecing them.  

## Related Protocols

Untrusted delegated payment protocols are available in Bitcoin Cash.

First, the [BIP21 URI scheme](https://github.com/bitcoin/bips/blob/master/bip-0021.mediawiki) provides a simple mechanism to communicate a payment to an address.

Second the [BIP70 payment protocol](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki) and its web-service-optimized equivalent [by Bitpay](https://bitpay.com/docs/payment-protocol), allow a more complex interaction that ensures that the receiver's invoice is still "live", allows the receiver to verify transaction details before it is signed, and allows a refund address to be supplied.

## Protocol Envelopes

This specification will use a URI format to describe the data contents, however, this protocol can and should be encapsulated by the envelope most appropriate for the application.  In particular it can be carried via HTTPS or an Android Intent.

Intents are a technique that allow an Android application to communicate (and access functionality) with each other. Intents can be mapped to a URI since they contain the same basic elements. In particular URI parameters can be added to an intent via the extended data key value pair dictionary. 


## Protocol Messages
### Registration

The registration phase allows an entity to set up a trust relationship with a wallet that will allow that entity to direct limited automatic (no user interaction) payments.

Registration is the process where an entity sets up a relationship with a wallet, allowing automatic payments to occur. The user must accept this relationship in the wallet. During this registration process, the wallet *SHOULD* allow the user to change the payment limits and other parameters suggested by the app. During operation, if a payment request exceeds these limits the wallet *SHOULD* fall back to a user interaction rather than rejecting the request automatically.

HTTP GET, and Intent protocols must include a pubkey and signature.

Intents MUST be issued with `startActivityForResult` so that return data can be captured.

#### Format 

tdpp://<entityName>/register?[pubkey=<pubkey>]&[maxper=<amount>]&[maxday=<amount>]&[maxweek=<amount>]&[maxmonth=<amount>]&[descper=<string>]&[descday=<string>]&[descweek=<string>]&[descmonth=<string>]&[sig=<string>]

#### Fields  

**entityName**:   The name of the registering entity/service.  This name will be shown in the wallet during payment authorizations or trust management.  Since wallets will likely disallow 2 registrations using the same name, make your name unique (e.g. use a full DNS name if a web site).

**pubkey**: [optional] If provided, this entity will sign tricklepay requests with this public key. This field MUST be provided if there is no implicit secure identity mechanism (such as https), or the wallet will reject with a NO_IDENTITY error.

**maxper**, **maxday**, **maxweek**,**maxmonth**: [optional] These fields specify the recommended automatic payment maximum in Satoshi (i.e. cryptocurrency’s finest unit).

**descper**, **descday**, **descweek**, **descmonth**: [optional] These fields specify short (3 lines or less) explanations of the chosen limits for optional user presentation by the wallet.

**sig**: [mandatory if pubkey] The signature of the stringified URI format of this request, alphanumerically ordered by key, less the sig key and value.

An entity should either generate a single pubkey/privkey pair when the app is installed or a new pub/private key per relationship. It is insecure to hard-code a single private key into the app. In Android, the “Room” and “KeyStore” databases are private to the application [CHECK].

  
#### Registration Response

Wallets should return data as part of the registration handling, indicating whether the trickle pay registration was accepted.

**resultcode**: [integer] 0 = OK, > 0 means not accepted, 1 = user reject, 2 = pubkey required
**supports**: [integer bit map]:  
	1 = Pay Address supported, 
	2 = Pay Transaction supported, 
	4 = JSON payment protocol supported. 

**error**: [optional] String response from the wallet with rejection details

  
#### Pay Address Request

This request format provides a simple payment interface.

tdpp://<appname>/sendto?[amtN=<amount>]&[addrN=<address>]&[sig=<string>]

**appname**: Identifies the prior registration.

**amtN**: [unsigned integer, at least one] (e.g. amt0, amt1, amt2, amt3) Specify the amount to send in Satoshis (or the finest unit of the currency). Its possible to send to multiple destinations, but the wallet must use the sum of all amounts to determine if automatic or user authorization is required. If addrN exists, amtN must exist, or the wallet will fail the request.

**addrN**: [at least one] (e.g. addr1, addr2, addr3) Specify the destination address. If amtN exists, addrN must exist, or the wallet will fail the request.

The wallet will send amtN to addrN, e.g. amt1 to addr1.

Note that the fee is calculated by the wallet and is not included when calculating trickle pay limits.

The addresses define the cryptocurrency.

**sig**: [mandatory if pubkey] The signature of the stringified URI format of this request using the pubkey provided during registration of "**entityName**", alphanumerically ordered by key, less the sig key and value.

**Example**: 

Donate 1 BCH to Bitcoin Unlimited.

tdpp://www.myapp.com/sendto?amt0=100000000&addr0=bitcoincash:pq6snv5fcx2fp6dlzg7s0m9zs8yqh74335tzvvfcmq&sig=

#### Pay Address Response



#### Pay Transaction Request

This request format allows the app to handle bitcoin transaction details, yet delegate funding and signing to the wallet.

tricklepay://<appname>/sign?[tx=<hexstring>]&[flags=nofund,nopost]&[sig=<string>


#### Pay JSON Payment Protocol Request

This request asks the wallet to complete a [JSON payment protocol](https://github.com/bitpay/jsonPaymentProtocol/blob/master/v2/specification.md) dialog