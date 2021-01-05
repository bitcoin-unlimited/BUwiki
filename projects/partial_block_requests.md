# Project: Implement Partial Block Requests in BCH Unlimited

## Rationale

Extremely large messages do not work well in the bitcoin peer-to-peer network.  Some problems are:
 1. A large message can consume significant time and bandwidth before it is discovered to be bogus (DOS attack).
 2. Transmitting most of a large message and then disconnecting can consume significant time and bandwidth (DOS attack) and cannot be distinguished from a legitimate accidental disconnect.  And intuitively, the probability of legitimate accidental disconnects increases in proportion to the message size.
 3. Intuitively, transmit bandwidth variation also increases in proportion to the message size (i.e. the longer one is demanding high bandwidth from an upstream node, the more likely that an event in the transmission path or upstream node may occur that disrupts the high bandwidth).  This means that high-bandwidth upstream nodes may become low-bandwidth nodes, causing a re-request of the block on another node.  
 4. Since TCP is stream oriented, large message transfer blocks the transmission of short messages.  For example, downloading a large block can stop all transaction notifications.  However, depending on the node's role in the network, rapid transaction or double spend proof receipt may be more important than block receipt (for 0-conf applications, or nodes that supply wallet information, for example).

## Solution

A partial message solution that cannot validate that the received message fragments are valid block data does not solve 1.  Fortunately, infrastructure to create verifiable partial block messages already exist in the protocol, with the MSG_FILTERED_BLOCK and [MERKLE_BLOCK messages](https://reference.cash/protocol/network/messages/merkleblock).  This technique provides the transactions in a block as individual messages and a message which is the merkle proof of those transactions.  It is intended to allow light clients to access only the transactions in a block that are interesting to that light client.

However, the MSG_FILTERED_BLOCK request message is awkward for the purpose of partial block access.  It accesses a bloom filter previously installed which filters ALL communications (including other transactions).  This means installing a filter for the purpose of partial block download may silence unrelated incoming transactions.  Instead, we would like to request partial transactions for just this block.  Also, specification by bloom filter is unnecessary and inefficient.  A range of transactions by index would be far simpler, and result in a smaller merkle proof.

Use of object property commitments can communicate the size and number of transactions in a block so the downstream node can choose whether to use full or partial block requests.

Additionally, there will be performance, correctness, and bandwidth optimizations.  For example for bandwidth, the MERKLE_BLOCK message references transactions by hash -- but we could save bytes by referencing them by partial hash (and on the sending side we can guarantee that there are no partial hash collisions), or by index in the block.  For correctness, responses should be matched to requests by a cookie provided by the request, and the server could pack multiple small transactions in a single message.  That single message should be a new, general-use "multi-transaction" message format, but should communicate that this is part of a partial block request (by providing the request cookie) as opposed to it being new, unconfirmed transaction arrivals (for example -- this ambiguity makes MERKLE_BLOCK messages much harder to implement in light clients).  And on the server side, once a block is read from disk it makes sense to cache it temporarily rather then release the related memory since a subsequent partial request is likely.

So the full partial block request solution will use a lot of the infrastructure already written (namely merkle proof generation) to serve the filtered block requests, but will implement new request and response messages.

