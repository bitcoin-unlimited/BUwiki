The Bitcoin P2P protocol is comprised of messages over TCP.  These messages are serialized using a custom format.  Unlike RPC protocols, messages do not necessarily have a reply and there is no way to unambiguously connect a sent message to a reply, although many communications are often request/response pairs.   High performance full node software may handle incoming messages in parallel, so it is not appropriate to assume a message reply order.  Messages that cannot be fulfilled are sometimes dropped with no reply.  

These legacy design decisions can make the protocol difficult to implement on the client side, but are generally needed when a robust implementation communicates with untrusted/uncooperative partners.  A good strategy is to wait for any message that provides the required data, with a timeout, and then separately issue the request in a retry loop to multiple peers.  If a timeout occurs, return to higher level software which should re-assess whether the data is needed.  This strategy handles situations where (for example), the client requests a block or header but a reorg has happened so that block/header is no longer on the most-difficult chain, causing peers to drop the client's request. 

## Serialization Format


## Message Envelope

## Message Types

### Announcements (unsolicited messages)

#### INV

#### XUPDATE


### Requests

#### GETDATA

#### GETBLOCKS

#### GETHEADERS

#### PING

#### VERSION

#### XVERSION
*Currently supported by Bitcoin Unlimited only*

#### Responses

#### ADDR
#### BLOCK, THINBLOCK, XTHINBLOCK, GRAPHENEBLOCK, CMPCTBLOCK

#### HEADERS

#### MERKLEBLOCK

#### PONG

#### REJECT

#### TX

#### VERACK

#### XVERACK
