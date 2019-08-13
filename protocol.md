The Bitcoin P2P protocol is comprised of messages over TCP.  These messages are serialized using a custom format.  Unlike RPC protocols, messages do not necessarily have a reply and there is no way to unambiguously connect a sent message to a reply, although many communications are often request/response pairs.   High performance full node software may handle incoming messages in parallel, so it is not appropriate to assume a message reply order.  Messages that cannot be fulfilled are sometimes dropped with no reply.  

These legacy design decisions can make the protocol difficult to implement on the client side, but are generally needed when a robust implementation communicates with untrusted/uncooperative partners.  A good strategy is to wait for any message that provides the required data, with a timeout, and then separately issue the request in a retry loop to multiple peers.  If a timeout occurs, return to higher level software which should re-assess whether the data is needed.  This strategy handles situations where (for example), the client requests a block or header but a reorg has happened so that block/header is no longer on the most-difficult chain, causing peers to drop the client's request. 

## Serialization Format

Bitcoin uses a custom serialization format that is generally little-endian.


## Message Envelope

## Message Types

### Announcements (unsolicited messages)

#### INV
*Notifies peers about the existence of some information (block or transaction)*

#### XUPDATE
*Communicates a change in peer capabilities*
#### FILTERLOAD
*Inserts a transaction filter into the peer*
#### FILTERADD
*Add a single item into an existing filter*
#### FILTERCLEAR
*Remove an existing filter*

### Requests

#### GETDATA
*Requests information (previously announced via an INV) from a peer*

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
#### MERKLEBLOCK
*Provides a provable subset of a block's transactions, as filtered by FILTERADD*

#### PONG

#### REJECT
*General response by well-behaved clients if a message cannot be handled*


#### TX

#### VERACK

#### XVERACK
