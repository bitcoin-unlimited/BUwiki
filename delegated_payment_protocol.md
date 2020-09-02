# Trusted Delegated Payment Protocol
*Advanced wallet interaction with trusted applications or websites*

## Introduction 
 
The Delegated Payment Protocol is a protocol that allows an entity and a cryptocurrency wallet to establish a trust (within limits) relationship, allowing the entity to delegate cryptocurrency operations to the wallet.  Based on policies set up by the wallet owner (the "user"), the wallet may automatically execute delegated operations (typically payments), or may request user authorization.  The entity requesting the operation has no control over whether the operation is handled automatically, or promoted to user authorization.  Delegated operations can be simple payment, the funding and signing of more complex transactions, or identity operations.  Entities can be anything that can execute a cryptographic signature, or whose identity is validated by other means (such as via https).  Common entities are web sites and smart phone apps.

This service can be broadly compared to the automatic fiat payment systems available via ACH (and others) that are used to deduct monthly membership dues, etc.  However, this service may be used both for "traditional" (monthly bill) style payments, and for emerging small payment applications that are enabled by Bitcoin Cash's payment efficiency.     

And from a philosophical standpoint this service is very different than traditional automatic payment permission.  In this protocol, user control over these trust relationships is centered within the wallet, and the wallet is an active agent in the execution of every payment.  This means that this trust relationship is easily adjusted by the user.  In some traditional systems, control over the trust relationship is delegated to the entity receiving the funds.  At best, this creates problems -- a different and separate system must be learned by the user for each relationship, often involving slow and obsolete technology, for example [a physical, stamped and mailed letter with numerous required and redundant details](https://donotpay.com/learn/cancel-golds-gym-membership/).  At worst, the entity deliberately obfuscates or complexifies their system to retain payments, and/or [charges the user to stop charging the user](https://www.consumerfinance.gov/ask-cfpb/how-do-i-stop-automatic-payments-from-my-bank-account-en-2023/).  

With problems like these, it seems as if many banking systems have essentially been co-opted by bank/business relationships to extract value from the hapless depositor (who must participate because of the significant problems with holding physical bills of currency whose high denominations have been retired and is losing value to inflation).  By returning control to the wallet owner, this protocol is aligned with cryptocurrency philosophy that seeks to provide an alternative that protects depositors rather than fleecing them.  

## Related Protocols

Untrusted delegated payment protocols are available in Bitcoin Cash.

First, the [BIP21 URI scheme](https://github.com/bitcoin/bips/blob/master/bip-0021.mediawiki) provides a simple mechanism to communicate a payment to an address.

Second the [BIP70 payment protocol](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki) and its web-service-optimized equivalent [by Bitpay](https://bitpay.com/docs/payment-protocol), allow a more complex interaction that ensures that the receiver's invoice is still "live", allows the receiver to verify transaction details before it is signed, and allows a refund address to be supplied.

## Protocol Envelopes

This specification will use a URI format to describe the data contents, however, this protocol can and should be encapsulated by the envelope most appropriate for the application.  In particular it can be carried via HTTPS or an Android Intent.

Intents are a technique that allow an Android application to communicate (and access functionality) with each other. Intents can be mapped to a URI since they contain the same basic elements. In particular URI parameters can be added to an intent via the extended data key value pair dictionary. 

## Entity Wallet Trust

This protocol sets up a user-authorized trust relationship between a wallet and an entity.  However, the opposite problem also exists.  An entity cannot trust that the wallet faithfully executed payments.  If such trust existed, wallets would be created that simply respond that payments were made without actually doing so, and therefore get free service.

One solution to this problem is that the entity needs to independently verify the transaction at its desired level of assurance (i.e. unconfirmed, unconfirmed with no known doublespends, confirmed depth N), and that the transaction has the desired outputs.  From an engineering perspective this is awkward because it means that the app needs to contain blockchain libraries, and for unconfirmed assurances, access the blockchain network.

SPV proofs can be furnished to prove confirmed transactions, but 10+ minute waits is not acceptable for many anticipated uses of this technology.

Assurances of unconfirmed transactions will always be probabilistic since a miner may be secretly mining a doublespend.  However, many businesses find the risk reasonable.  Web site entities can deploy and access a trusted full node to gain this information.  At a lower level of assurance, apps can use the simpler electrum protocol to at least poll a random or trusted server or two to verify that the transaction has propagated, or propagate the transaction itself.  Maintaining a trusted full node for app payment queries increases the maintenance overhead of the app, but it is a possibility.  Accessing random servers is vulnerable to eclipse attacks, but mounting such an attack may not be economically viable for small payments.

But the emerging technology "Tailstorm" (Bobtail and Storm combined) should allow partial proof-of-work inclusion proofs.  These can be used to provide a probabilistic and economic assurance that the transaction is actually being mined, without requiring that the entity independently communicate to the blockchain or to an external server.

## Protocol Messages
### Registration

The registration phase allows an entity to set up a trust relationship with a wallet that will allow that entity to direct limited automatic (no user interaction) payments.

Registration is the process where an entity sets up a relationship with a wallet, allowing automatic payments to occur. The user must accept this relationship in the wallet. During this registration process, the wallet *SHOULD* allow the user to change the payment limits and other parameters suggested by the app. During operation, if a payment request exceeds these limits the wallet *SHOULD* fall back to a user interaction rather than rejecting the request automatically.

HTTP GET, and Intent protocols must include a pubkey and signature.

Intents MUST be issued with `startActivityForResult` so that return data can be captured.
Additional undefined fields **MUST** be ignored (but included in any signature check).

#### Format 

tdpp://**entityName**/reg?[pubkey=**pubkey**]&[maxper=**amount**]&[maxday=**amount**]&[maxweek=**amount**]&[maxmonth=**amount**]&[descper=**desc**]&[descday=**desc**]&[descweek=**desc**]&[descmonth=**desc**]&[sig=**sig**]

#### Fields  

**entityName**:   The name of the registering entity/service.  This name will be shown in the wallet during payment authorizations or trust management.  Since wallets will likely disallow 2 registrations using the same name, make your name unique (e.g. use a full DNS name if a web site).

**pubkey**: [optional] If provided, this entity will sign requests with this public key. This field MUST be provided if there is no implicit secure identity mechanism (such as https), or the wallet will reject with a NO_IDENTITY error.

**maxper**, **maxday**, **maxweek**,**maxmonth**: [optional, unsigned long integer] These fields specify the recommended automatic payment maximum in Satoshi (i.e. cryptocurrency’s finest unit).

**descper**, **descday**, **descweek**, **descmonth**: [optional, string] These fields specify short (3 lines or less) explanations of the chosen limits for optional user presentation by the wallet.

**sig**: [mandatory if pubkey] The signature of the stringified URI format of this request, alphanumerically ordered by key, less the sig key and value.

An entity should either generate a single pubkey/privkey pair when the app is installed or a new pub/private key per relationship. It is insecure to hard-code a single private key into the app. In Android, the “Room” and “KeyStore” databases are private to the application [CHECK].

  
#### Registration Response

Wallets should return data as part of the registration handling, indicating whether the registration was accepted.  A wallet's "OK" response does not indicate whether the suggested policies were accepted, or whether any automatic payment was authorized by the user.  The entity should never be provided with information pertaining to how much it can spend without "bothering" the user so that it cannot reliably quietly drain user's accounts.  A wallet's "OK" response simply indicates a willingness to act as a payment gateway for the entity.

**resultcode**: [integer] 200= OK, 300 = user reject, 302 = pubkey required
**supports**: [integer bit map]:  
	1 = Pay Address supported, 
	2 = Pay Transaction supported, 
	4 = JSON payment protocol supported. 

**error**: [optional] String response from the wallet with rejection details

  
#### Pay Address Request

This request format provides a simple payment interface.  Note that the fee is calculated by the wallet and is not included when calculating automatic payment limits.  The addresses define the cryptocurrency, so MUST contain the appropriate prefix (e.g. bitcoincash:, bitcoin: etc).

Additional undefined fields **MUST** be ignored, but included in any signature check.

tdpp://**entityName**/sendto?[amtN=**amount**]&[addrN=**address**]&[sig=**sig**]

**entityName**: Identifies the prior registration.

**amtN**: [unsigned integer, at least one] (e.g. amt0, amt1, amt2, amt3) Specify the amount to send in Satoshis (or the finest unit of the currency). Its possible to send to multiple destinations, but the wallet must use the sum of all amounts to determine if automatic or user authorization is required. If addrN exists, amtN must exist, or the wallet will fail the request.

**addrN**: [at least one] (e.g. addr1, addr2, addr3) Specify the destination address. If amtN exists, addrN must exist, or the wallet will fail the request.

The wallet will send amtN to addrN, e.g. amt1 to addr1.

**sig**: [mandatory if pubkey] The signature of the stringified URI format of this request using the pubkey provided during registration of "**entityName**", alphanumerically ordered by key, less the sig key and value.

**Example**: 

Donate 1 BCH to Bitcoin Unlimited.

tdpp://www.myapp.com/sendto?amt0=100000000&addr0=bitcoincash:pq6snv5fcx2fp6dlzg7s0m9zs8yqh74335tzvvfcmq&sig=[TODO SIG FORMAT]

#### Pay Address Response

Additional undefined fields **MUST** be ignored.

**resultcode**: [integer] 200 = OK, 300 = user reject, 301 = sig failed, 303 = unsupported request type, 304 = insufficient balance
**txid**: [hex string, required on success]  The transaction hash of the created transaction
**error**: [optional] String response from the wallet with rejection details


#### Pay Transaction Request

This request format allows the app to handle bitcoin transaction details, yet delegate funding and/or signing to the wallet.

tdpp://**appname**/tx?chain=**blockchain**&tx=**tx**&[flags=**flags**]&[sig=**sig**]

**chain**: [Mandatory, string]  The blockchain this transaction is for, as specified by the BIP21 URI prefix, for example: "bitcoincash" or "bitcoin".

**tx**: [Mandatory, hexstring]  The transaction in network serialized hex string format.  This transaction may be complete and signed, or it may be incomplete in many ways.  It may supply or be missing inputs.  In other words, it may require additional inputs to meet the output quantity.  It may require an additional outputs, directed to this wallet, for change.  It may require signatures, but some inputs may be signed.

**flags**: [optional, unsigned int bitmap]  An empty flags implies flags == 0
0: nofund: do not add any inputs
1: nopost: do not post the transaction to the network (return the finished transaction to the entity)
2: noshuffle:  do not change the order of inputs or outputs.  Change outputs must be added to the end.

#### Pay Transaction Response

**resultcode**: [integer] 200 = txcompleted, 201 = tx missing sig, 202 = tx unmodified, 203 = tx not final, 204 = temporarily cannot post, 300 = user reject, 301 = sig failed, 303 = unsupported request type, 304 = insufficient balance

Result code 201 means that the transaction has been "filled out" by this wallet, but the wallet is not capable of completing the job -- there are inputs that still must be signed.  This can legitimately occur with multisig inputs or interesting transaction forms.

Result code 203 means that the transaction creation is successful, but the transaction cannot be accepted on the blockchain at this time.  This code may or may not be returned if "nopost" is specified.

Result code 204 means that the wallet's connection to the blockchain is interrupted so this transaction cannot be posted.

**tx**: The completed transaction in network serialized hex string format.
**txid**: The completed transaction hash

**error**: [string, optional] detailed error message, if any



#### Pay JSON Payment Protocol Request

This request asks the wallet to complete a [JSON payment protocol](https://github.com/bitpay/jsonPaymentProtocol/blob/master/v2/specification.md) dialog.

tdpp://**appname**/jsonpay?uri=**url**

**url**: The payment protocol initiation URL, as specified in JSON payment protocol document

#### JSON Payment Protocol Response

**resultcode**: [integer] 200 = payment completed, 

**jppmemo**: [string, optional] The "memo" field in the payment protocol response *SHOULD* be reported back to the calling entity, if it exists and is not empty.