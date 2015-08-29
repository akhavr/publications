IPFS provides great vehicle to create distributed and fault-tolerant 
services, yet it's not obvious how convenient web or blockchain 
architectures would translate to this environment.

Global multiuser chat service would be a good testcase.

## Centralized chat service

Each message is an ipfs object like this:

    $ echo '{"data": "first message", "author": "akhavr@khavr.com"}' | \
      ipfs object put 
    added QmTHy71953CfqmKYKzptSjCt7Mh2D9b6P2GbeoREdzDaJE    

Central node serves list of messages to client nodes.  The message list is a usual ipfs file-like object with links: 


    $ echo '{"data": "Root chat", "owner": "QmaWWPUhsM1MPuV9Y9dWR6zF73eY1rAEiBsB6DxRvkaA9z", "links": [{"hash": "QmTHy71953CfqmKYKzptSjCt7Mh2D9b6P2GbeoREdzDaJE"}]}' | ipfs object put 
    added QmaYZ55GfWwMhBj6agHzdztyZNJF3xjp96rNPwhTAqjtZm

    $ ipfs object get QmaYZ55GfWwMhBj6agHzdztyZNJF3xjp96rNPwhTAqjtZm
    {
      "Links": [
        {
          "Name": "",
          "Hash": "QmTHy71953CfqmKYKzptSjCt7Mh2D9b6P2GbeoREdzDaJE",
          "Size": 0
        }
      ],
      "Data": "Root chat"
    }

First link may be a link to a previous page of messages.  Last link is either a message or a link to a next page of messages.

To serve updates to clients, central node publishes the hash of the last page to an ipns:

    $ ipfs name publish QmaYZ55GfWwMhBj6agHzdztyZNJF3xjp96rNPwhTAqjtZm
    Published to QmaWWPUhsM1MPuV9Y9dWR6zF73eY1rAEiBsB6DxRvkaA9z: QmaYZ55GfWwMhBj6agHzdztyZNJF3xjp96rNPwhTAqjtZm

Clients have the central node hardcoded and resolve its name to a chat list:

    $ ipfs name resolve QmaWWPUhsM1MPuV9Y9dWR6zF73eY1rAEiBsB6DxRvkaA9z 
    /ipfs/QmaYZ55GfWwMhBj6agHzdztyZNJF3xjp96rNPwhTAqjtZm
    $ ipfs object get QmaYZ55GfWwMhBj6agHzdztyZNJF3xjp96rNPwhTAqjtZm
    {
      "Links": [
        {
          "Name": "",
          "Hash": "QmTHy71953CfqmKYKzptSjCt7Mh2D9b6P2GbeoREdzDaJE",
          "Size": 0
        }
       ],
       "Data": "Root chat"
    }
    $ ipfs object get QmTHy71953CfqmKYKzptSjCt7Mh2D9b6P2GbeoREdzDaJE
    {
      "Links": [],
      "Data": "first message"
    }

Client nodes render the chat to user themselves.

Client nodes post the message to the chat by calling the central node. [Making your own ipfs service](http://gateway.ipfs.io/ipfs/QmTkzDwWqPbnAh5YiV5VwcTLnGdwSNsNTn2aDxdXBFca7D/example#/ipfs/QmThrNbvLj7afQZhxH72m5Nn1qiVn3eMKWFYV49Zp2mv9B/api/service/readme.md).  The central node returns the hash of an updated chat page and updates the name in the ipns.

Clients periodically check if there was an update by resolving ipns entry - AKA poll updates.  See next section on how we may do it as "push updates".

##  Decentralized chat service

Same as above, with two distinctions:
- each client node can also serve as a central node
- in case of a message list fork, nodes resolve it by consensus 
  merging

To achieve this, each client maintains a peers list.

When a client posts a message, it

1. Updates his local message list and the ipns entry
1. Calls his peers to update them
1. If a peer returns a list, that has different messages than the local node got, then merge the list and repeat


