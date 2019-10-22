Provides a block header and partial merkle proof tree that selected transaction ids exist in the block.  Transactions are selected by honest servers if they match the bloom filter installed by the client.  However, note that servers can omit transactions and this cannot be detected except by receiving a MERKLEBLOCK message from an honest server.

**MERKLEBLOCK Message Format**

| ... | 4 bytes |  ... | ...|
|----|---------------|-----------|------------|
|  [block header](/protocol/p2p/block__header)| # tx in block | vector of 32 byte hashes | vector of bytes that define the merkle block traversal
