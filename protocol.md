The Bitcoin P2P protocol is comprised of messages over TCP.  These messages are serialized using a custom format.  Unlike RPC protocols, messages do not necessarily have a reply and there is no way to unambiguously connect a sent message to a reply, although many communications are often request/response pairs.   High performance full node software may handle incoming messages in parallel, so it is not appropriate to assume a message reply order.  Messages that cannot be fulfilled are sometimes dropped with no reply.  

These legacy design decisions can make the protocol difficult to implement on the client side, but are generally needed when a robust implementation communicates with untrusted/uncooperative partners.  A good strategy is to wait for any message that provides the required data, with a timeout, and then separately issue the request in a retry loop to multiple peers.  If a timeout occurs, return to higher level software which should re-assess whether the data is needed.  This strategy handles situations where (for example), the client requests a block or header but a reorg has happened so that block/header is no longer on the most-difficult chain, causing peers to drop the client's request. 

## Serialization Format

Bitcoin uses a custom serialization format that is generally little-endian.


## Message Envelope

The following table describes the message header:

| network identifier | command | size | checksum | contents |
|-------------|--------------|-------------|----------------|------------|
| 0: 4 bytes | 4: 12 bytes | 16: 4 bytes | 20: 4 bytes | 24: size bytes |
|*BCH*:E3,E1,F3,E8<br>*BTC*:F9,BE,B4,D9<br>*tBCH*:F4,E5,F3,F4<BR>*tBTC*:0B,11,09,07 | ascii null extended | little endian uint32 | little endian uint32 | depends on command

### Network Identifier
The network identifier is used to separate blockchains and test networks.  This reduces unnecessary load on peers, allowing them to rapidly ban nodes rather then forcing the peer to do a blockchain analysis before banning.  

Unfortunately, the BCH and BSV blockchains use the same network identifier.

### Command
The command is the exact lowercase bytes in the titles of each subsection in the Message Types section below followed by zeros -- e.g. the INV message's command is literally the 12 bytes: 'i','n',v',0,0,0,0,0,0,0,0,0.  This is not a "C" string.  If a command was exactly 12 bytes, there would be no null terminator!

### Size
Size is the size of the contents field in bytes, not the size of the entire message.

### Checksum
Checksum is a message checksum.  Since TCP has message integrity checksums, and a peer can cause another node to waste processing power validating bad checksums, it is not recommended that nodes verify this checksum.  

Senders should calculate this checksum to be compatible with all software.  However, the XVERSION message can be used to tell XVERSION compatible clients that checksums will not be calculated.  In this case the field must be set to 0 (but not enforced as 0 on the receiver's side).  This may allow a future reuse of this field.

### Contents
The contents of messages are described in the next section.

## Message Contents

### Announcements (unsolicited messages)

#### INV
*Notifies peers about the existence of some information (block or transaction)*

| 4 bytes | 32 bytes |
|---------|----------|
|   type  |   hash   |

NOTE: Since a block header is a relatively small data structure, and block propagation speed is an important network metric, a peer may send HEADER messages in place of INV messages when a block arrives.

##### Type
The type of the object offered
##### Hash
The [hash identifier](glossary/hash__identifier) of the object that is available at the peer.

#### XUPDATE
*Communicates a change in peer capabilities*
#### FILTERLOAD
*Inserts a transaction filter into the peer*

This message installs a bloom filter into the peer.  Subsequent INV notifications and MERKLEBLOCK messages only provide transactions that in match this bloom filter in some manner.  The following items in a transaction are checked against the bloom filter:

 - The transaction hash
 - Each data field in every [output script](glossary/output__script) in the transaction
	 - Most importantly, this allows public keys and public key hashes (essentially bitcoin addresses) to be added to the bloom filter, allowing a wallet to detect an incoming transfer.
 - Each [previous output](glossary/previous__output) in the transaction
	 - This allows a wallet to detect that a different wallet has spent funds that are co-controlled
 - Each data field in every [input script](glossary/input__script) in the transaction.

See [C:{CBloomFilter::MatchAndInsertOutputs, CBloomFilter::MatchInputs}]

| up to 36000 bytes |
|-------------------|
|   [bloom filter](objects/bloom_filter)   |

#### FILTERADD
*Add a single item into an existing filter*
#### FILTERCLEAR
*Remove an existing filter*

### Requests

#### GETDATA
*Requests information (previously announced via an INV) from a peer*

A GETDATA request is formatted as a vector of INV data:

| compact int |  4 bytes | 32 bytes | ... |
|---------------|-----------|------------|--|
| number of elements | elem N type | elem N hash | elem N+1 type and hash...

#### GETBLOCKS

#### GETHEADERS
*Requests block headers from a peer*

#### PING
*Keep-alive*

#### VERSION
*Describes peer capabilities*

#### XVERSION
*Describes peer capabilities in an extensible manner*
*Currently supported by Bitcoin Unlimited only*

#### Responses
Note that some of these "response" messages can also be sent without solicitation (i.e. without a request).

#### ADDR
*Provides a peer with the addresses of other peers*

#### BLOCK, THINBLOCK, XTHINBLOCK, GRAPHENEBLOCK, CMPCTBLOCK
*Provides a block*

#### HEADERS
*Provides a set of block headers (unsolicited or GETHEADERS response)*


#### [MERKLEBLOCK](protocol/p2p/merkleblock)
*Provides a provable subset of a block's transactions, as filtered by FILTERADD*

#### PONG

#### REJECT
*General response by well-behaved clients if a message cannot be handled*


#### TX

#### VERACK

#### XVERACK