IPFS provides great vehicle to create distributed and fault-tolerant 
services, yet it's not obvious how convenient web or blockchain 
architectures would translate to this environment.

Global multiuser chat service would be a good testcase.

## Centralized chat service

Central node serves list of messages to client nodes.  The message list is a usual ipfs file-like object:

    {
      "links": [
      { "hash": "XLYkgq61DYaQ8NhkcqyU7rLcnSa7dSHQ16x",
      "size": 189458 },
      { "hash": "XLHBNmRQ5sJJrdMPuu48pzeyTtRo39tNDR5",
      "size": 19441 },
      { "hash": "XLWVQDqxo9Km9zLyquoC9gAP8CL1gWnHZ7z",
      "size": 5286 }
      ]
    }

First link is a link to a previous page of messages.  Last link is either a message or a link to a next page of messages.

For new clients, central node publishes the hash of that object to an ipns  [The Inter-Planetary Naming System](http://gateway.ipfs.io/ipfs/QmTkzDwWqPbnAh5YiV5VwcTLnGdwSNsNTn2aDxdXBFca7D/example#/ipfs/QmThrNbvLj7afQZhxH72m5Nn1qiVn3eMKWFYV49Zp2mv9B/ipns/readme.md)

Client nodes render the chat to user themselves.

Client node posts the message to the chat by calling central node [Making your own ipfs service](http://gateway.ipfs.io/ipfs/QmTkzDwWqPbnAh5YiV5VwcTLnGdwSNsNTn2aDxdXBFca7D/example#/ipfs/QmThrNbvLj7afQZhxH72m5Nn1qiVn3eMKWFYV49Zp2mv9B/api/service/readme.md).  The central node returns the hash of an updated chat page and updates the name in the ipns.

Clients periodically check if there was an update by resolving ipns entry - AKA poll updates.  See next section on how we may do it as "push updates".

TBD: 
1. paging

##  Decentralized chat service

Same as above, with two distinctions:
- each client node can also serve as a central node
- in case of message list fork, nodes resolve it by consensus 
  merging

To achieve this, client maintains a peers list.

When client posts a message, it

1. Updates his local message list and ipns entry
1. Calls his peers to update them
1. If a peer returns a list, that has different entries, than local - merge and repeat


