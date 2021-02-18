# Project: Implement Object Property INVs for BCH Unlimited

## Rationale

Object property commitments (described [here](/object_property_commitment.md) are a technique to trustlessly provide data about objects (blocks and transactions) to connected nodes.  As blocks (and secondarily transactions) scale, problems with the bitcoin cash peer-to-peer network become evident that can be solved if meta-data about blocks is provided within block announcement (INV) messages.  In particular, transmission of a large valid block verses a fake large block in place of a small one cannot be determined until the transmission is complete.  As a secondary consideration, it is also awkward in the current code to determine whether transmission of a small block is being slowed or stalled verses transmission of a large block, because this requires messages to be parsed before they are complete, and for information to cut across abstraction levels.  

These two techniques open two DOS attack vectors where a miner pushes large fake or slow blocks to other miner nodes to keep those miners from mining on the main chain tip for as long as possible.  To mitigate these attack vectors, BCH Unlimited re-requests blocks (on different nodes) that "time out" when being downloaded.  However, the large variability of block sizes (and therefore expected download times) makes this time out value impossible to specify accurately.  Short blocks will not time out "soon enough", and large blocks risk spurious re-requests.

## Solution

A solution can occur in two phases.  Object property commitments (OPC) can be used alone to provide information about INV object sizes before being downloaded.  This information can be used to set the download timeout value on a per-block basis.  The task would be to create a new INV-type message that provided object property commitments (OPC_INV) rather than standard INV messages (and create the EXTVERSION data to communicate suppport for OPC-INV messages).

In phase 2, nodes can request partial blocks (see [here](/projects/partial_block_requests.md)).  The advantages of the partial block approach are described in the referenced link and will not be repeated here. But for a partial block technique to work, the node software needs to know to request partial blocks and to know the block size and number of transaction in it so it knows to use partial block requests, and how many transactions to ask for per request.  Therefore this object property INV technique is not wasted -- it will useful for partial block solutions (and also useful for transaction analysis if and when we start allowing larger transaction scripts).