Protocol Basics
===============

Message Stream
--------------

Clients and servers communicate using **JSON RPC** over an unspecified underlying stream
transport.  Examples include TCP, SSL, WS and WSS.

Two standards `JSON RPC 1.0
<http://www.jsonrpc.org/specification_v1>`_ and `JSON RPC 2.0
<http://www.jsonrpc.org/specification>`_ are specified; use of version
2.0 is encouraged but not required.  Server support of batch requests
is encouraged for version 1.0 but not required.

.. note:: A client or server should only indicate JSON RPC 2.0 by
  setting the `jsonrpc
  <http://www.jsonrpc.org/specification#request_object>`_ member of
  its messages to ``"2.0"`` if it supports the version 2.0 protocol in
  its entirety.  ElectrumX does and will expect clients advertizing so
  to function correctly.  Those that do not will be disconnected and
  possibly blacklisted.

Clients making batch requests should limit their size depending on the
nature of their query, because servers will limit response size as an
anti-DoS mechanism.

Over TCP and SSL raw sockets each RPC call, and each response, MUST be terminated by a
single newline to delimit messages.  Websocket messages are already framed so they MUST
NOT be newline terminated.  The JSON specification does not permit control characters
within strings, so no confusion is possible there.  However it does permit newlines as
extraneous whitespace between elements; client and server MUST NOT use newlines in such a
way.

If using JSON RPC 2.0's feature of parameter passing by name, the
names shown in the description of the method or notification in
question MUST be used.

A server advertising support for a particular protocol version MUST
support each method documented for that protocol version, unless the
method is explicitly marked optional.  It may support other methods or
additional parameters with unspecified behaviour.  Use of additional
parameters is discouraged as it may conflict with future versions of
the protocol.


Notifications
-------------

Some RPC calls are subscriptions which, after the initial response,
will send a JSON RPC :dfn:`notification` each time the thing
subscribed to changes.  The `method` of the notification is the same
as the method of the subscription, and the `params` of the
notification (and their names) are given in the documentation of the
method.


Version Negotiation
-------------------

It is desirable to have a way to enhance and improve the protocol
without forcing servers and clients to upgrade at the same time.

Protocol versions are denoted by dotted number strings with at least
one dot.  Examples: "1.5", "1.4.1", "2.0".  In "a.b.c" *a* is the
major version number, *b* the minor version number, and *c* the
revision number.

A party to a connection will speak all protocol versions in a range,
say from `protocol_min` to `protocol_max`, which may be the same.

The client must send a :func:`server.version` RPC call as the first
message on the wire, in order to negotiate the precise protocol
version; see its description for more detail.
All responses received in the stream from and including the server's
response to this call will use its negotiated protocol version.


.. _script hashes:

Script Hashes
-------------

A :dfn:`script hash` is the hash of the binary bytes of the locking
script (ScriptPubKey), expressed as a hexadecimal string.  The hash
function to use is given by the "hash_function" member of
:func:`server.features` (currently :func:`sha256` only).  Like for
block and transaction hashes, when converting the big-endian binary
hash to a hexadecimal string the least-significant byte appears first,
and the most-significant byte last.

For example, the legacy Bitcoin address from the genesis block::

    1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa

has P2PKH script::

    76a91462e907b15cbf27d5425399ebf6f0fb50ebb88f1888ac

with SHA256 hash::

    6191c3b590bfcfa0475e877c302da1e323497acf3b42c08d8fa28e364edf018b

which is sent to the server reversed as::

    8b01df4e368ea28f8dc0423bcf7a4923e3a12d307c875e47a0cfbf90b5c39161

By subscribing to this hash you can find P2PKH payments to that address.

One public key, the genesis block public key, among the trillions for
that address is::

    04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb
    649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f

which has P2PK script::

    4104678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb
    649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5fac

with SHA256 hash::

    3318537dfb3135df9f3d950dbdf8a7ae68dd7c7dfef61ed17963ff80f3850474

which is sent to the server reversed as::

    740485f380ff6379d11ef6fe7d7cdd68aea7f8bd0d953d9fdf3531fb7d531833

By subscribing to this hash you can find P2PK payments to the genesis
block public key.

.. note:: The Genesis block coinbase is uniquely unspendable and
   therefore not indexed.  It will not show with the above P2PK script
   hash subscription.


.. _status:

Status
------

To calculate the `status` of a :ref:`script hash <script hashes>` (or
address):

1. Consider all transactions touching the script hash (both those spending
from it, and those funding it), both confirmed and unconfirmed (in mempool).

2. Order confirmed transactions by increasing height (and position in the
block if there are more than one in a block).

3. form a string that is the concatenation of strings
``"tx_hash:height:"`` for each transaction in order, where:

  * ``tx_hash`` is the transaction hash in hexadecimal

  * ``height`` is the height of the block it is in.

4. For mempool transactions, we define **height** to be ``-1`` if the
transaction has at least one unconfirmed input, and ``0`` if all inputs are
confirmed.

5. Order mempool transactions by ``(-height, tx_hash)``, that is,
``0`` height txs come before ``-1`` height txs, and secondarily the
txid (endianness same as human-readable hex string used for display) is used
to arrive at a canonical ordering.

6. Next, with mempool transactions in the specified order, append a similar
string.

7. The :dfn:`status` of the script hash is the :func:`sha256` hash of the
full string expressed as a hexadecimal string, or :const:`null` if the
string is empty because there are no transactions.


**Status Example**

Consider the following BTC regtest address: `bcrt1qn7d2x7272lznt5hhk9s07q3cqnrqljnwladrd3`,
which has script hash: `f04bf26fc16a27abaa2f10540da7a24e369e6ec408f2d901202a7c0cd4a6112d`.

Example RPC traffic::

    <-- ('blockchain.scripthash.subscribe', ['f04bf26fc16a27abaa2f10540da7a24e369e6ec408f2d901202a7c0cd4a6112d']) {} (id: 15)
    --> 78e96c6562cafa71c115503b9411fdfdc595a45031e2ab76ff75162fe1b0590d (id: 15)
    <-- ('blockchain.scripthash.get_history', ['f04bf26fc16a27abaa2f10540da7a24e369e6ec408f2d901202a7c0cd4a6112d']) {} (id: 16)
    --> [
    {'tx_hash': 'a6c9c361bd0bc536d6a22648efbf8f9b200e425ef6c3a7a9669dc444c532a347', 'height': 2472},
    {'tx_hash': '9c42f84b2fcdaff676ba25d9d4941741cc0d1a01cce0c23fdc4c0b2afa38431c', 'height': 2473},
    {'tx_hash': '770f2d4371b3fabb902dd9a103e2dd005fcd3971181078fca4a2a1d6ff127b30', 'height': 2473},
    {'tx_hash': '80b19848aed792565ab7c5a79b7c2a00fbf985741579396ebe0ab6098e607311', 'height': 0, 'fee': 200},
    {'tx_hash': 'e02a1dadfa83b996b24175df807b271ea5d02937ef5b35c195fac1e1bdc3198f', 'height': 0, 'fee': 300},
    {'tx_hash': 'bb4c8ab438c13b89ca80d1d5bee25b0b6b7f55673f4d801998ba97db161d9e85', 'height': -1, 'fee': 200}] (id: 16)
    <-- ('blockchain.transaction.get', ['a6c9c361bd0bc536d6a22648efbf8f9b200e425ef6c3a7a9669dc444c532a347']) {} (id: 17)
    <-- ('blockchain.transaction.get', ['9c42f84b2fcdaff676ba25d9d4941741cc0d1a01cce0c23fdc4c0b2afa38431c']) {} (id: 18)
    <-- ('blockchain.transaction.get', ['770f2d4371b3fabb902dd9a103e2dd005fcd3971181078fca4a2a1d6ff127b30']) {} (id: 19)
    <-- ('blockchain.transaction.get', ['80b19848aed792565ab7c5a79b7c2a00fbf985741579396ebe0ab6098e607311']) {} (id: 20)
    <-- ('blockchain.transaction.get', ['e02a1dadfa83b996b24175df807b271ea5d02937ef5b35c195fac1e1bdc3198f']) {} (id: 21)
    <-- ('blockchain.transaction.get', ['bb4c8ab438c13b89ca80d1d5bee25b0b6b7f55673f4d801998ba97db161d9e85']) {} (id: 22)
    --> 02000000000102f758cda73a362840995d62d0079a22a11fc2652cb7740fffb8132486fe76fe730000000000fdffffff31b2d3d434d7c8f46b23a47aabc3c9498d4df5ffb9f13e13c3c3ecb52ab570b80000000000fdffffff0280f0fa02000000001600149f9aa3795e57c535d2f7b160ff023804c60fca6ef0cff00800000000160014553a3b97c2cc7d3d4edeace3281ff038bd067629024730440220274cf69bab8d6fd25bd047ef7420274f58540c32f2cbc12944134e604d2307a7022035a8954b44cbc6b048f4457080a7bcaf903bd970ee8bd297f1ef25451a464c280121027ba4d3ee6471a985307a37d09eb9a0a73c1a31b57616fe3e53a3d6d4540025190247304402205a879e8678c331e4f00a282e629fc048ce2ec0107452ecc19773f4c144c725af02204495325f83cfc987e4e9a090c361cfe11b63173a9b7a5ce498f3924f11ec416a0121027ba4d3ee6471a985307a37d09eb9a0a73c1a31b57616fe3e53a3d6d454002519a7090000 (id: 17)
    --> 0200000000010147a332c544c49d66a9a7c3f65e420e209b8fbfef4826a2d636c50bbd61c3c9a60000000000fdffffff02784a4c00000000001600147bcf38d8a69071c21abbfe0532cb6aeacaf5c29440a5ae02000000001600149f9aa3795e57c535d2f7b160ff023804c60fca6e02473044022020fd51935d101b3bfcf169af182a2ca189407597d8f1868ca55eecae6c58fd0402205adc117093a4329dde60b9b9c579c56c1699129c6be37a581bbd03e851d36962012103c3172a9f8820681c62b8bf28961988a4642b33f4920b9da14b06965c7fff83fda8090000 (id: 18)
    --> 0200000000010147a332c544c49d66a9a7c3f65e420e209b8fbfef4826a2d636c50bbd61c3c9a60100000000fdffffff02005a6202000000001600149f9aa3795e57c535d2f7b160ff023804c60fca6e28758e0600000000160014c388b18dd1771046fecb85f44642ffb7a4efb6270247304402200967ab438e5cb1047873132a7beda3a6e2d87809f58ebcb69976e14c85c2a2700220447d761847b2ab6a239f93cc2f8d42981082e4c4fe05f43b87fcd5f1626e4ed3012103cd68ad8bddacdc82a39a82efd7a045f2d9513a9de31049286b696c87a8efbe87a8090000 (id: 19)
    --> 02000000000101307b12ffd6a1a2a4fc7810187139cd5f00dde203a1d92d90bbfab371432d0f770100000000fdffffff0280c3c901000000001600149f9aa3795e57c535d2f7b160ff023804c60fca6ee0b0c404000000001600144343eef6273442ab066a852a7d6ecc61950b2e7b024730440220704943bb017a4aec888859b667658ab85bc306625b049cc9f5ef1f832115024d022042d950df482107838db62378ad2db8c13b9b75508eff3bfd8198ecd1dc776215012103834e1e34194508e3fc7db19d3c6964149efd58f8c67548cc765c3284cad24d67a9090000 (id: 20)
    --> 02000000000102307b12ffd6a1a2a4fc7810187139cd5f00dde203a1d92d90bbfab371432d0f770000000000fdffffff1c4338fa2a0b4cdc3fc2e0cc011a0dcc411794d4d925ba76f6afcd2f4bf8429c0100000000fdffffff02c00e1602000000001600149f9aa3795e57c535d2f7b160ff023804c60fca6e54effa0200000000160014edb4b55f1f8c33ac8597fa53d08d8997131f1b930247304402204015eaf728c064682645c95c10b04c6d47741d62b5447e29fe1e3075776ceff10220013a39f59fe60965a9304eb8645333cf4480c0c570a6aa6c87abc94b2f9b39e2012103c3172a9f8820681c62b8bf28961988a4642b33f4920b9da14b06965c7fff83fd024730440220175477dcbbb54bfb19c0cc09c311dc1b789c3f2ff3d1cc8dfc50b1065612135a022020ed690530f19d60028fb5f7e760d079d3de85981ca32b7e4349c7e02d3dcf79012103c3172a9f8820681c62b8bf28961988a4642b33f4920b9da14b06965c7fff83fda9090000 (id: 21)
    --> 020000000001011173608e09b60abe6e3979157485f9fb002a7c9ba7c5b75a5692d7ae4898b1800000000000fdffffff02784a4c0000000000160014905190f141d2089497a554d5b74175929d2b2c7d40787d01000000001600149f9aa3795e57c535d2f7b160ff023804c60fca6e024730440220681b5ea1d17b311bcce35df1b9f202c15fd161c8eab0f72059d2869c2898d9460220436bf54378db5e4d9fe8c7f73fdb11db05804c8b5d6326f8cdd6e3cd906e2218012103c3172a9f8820681c62b8bf28961988a4642b33f4920b9da14b06965c7fff83fda9090000 (id: 22)
    <-- ('blockchain.transaction.get_merkle', ['a6c9c361bd0bc536d6a22648efbf8f9b200e425ef6c3a7a9669dc444c532a347', 2472]) {} (id: 23)
    <-- ('blockchain.transaction.get_merkle', ['9c42f84b2fcdaff676ba25d9d4941741cc0d1a01cce0c23fdc4c0b2afa38431c', 2473]) {} (id: 24)
    <-- ('blockchain.transaction.get_merkle', ['770f2d4371b3fabb902dd9a103e2dd005fcd3971181078fca4a2a1d6ff127b30', 2473]) {} (id: 25)
    --> {'block_height': 2472, 'merkle': ['5a4ae2a3c22bd791622edb41bd007b39e1f3bcdc8371717c71dbcf630aa8ca6f'], 'pos': 1} (id: 23)
    --> {'block_height': 2473, 'merkle': ['9b977e4cc0b2eb44cb5ea2adb30bf6eae892c9a838111031cb49c97d8ad57309', 'e7a2891bd8f96488681a596f619de4d88a22d3b08a8aee3b44bbdd913e3e3562'], 'pos': 1} (id: 24)
    --> {'block_height': 2473, 'merkle': ['770f2d4371b3fabb902dd9a103e2dd005fcd3971181078fca4a2a1d6ff127b30', 'ee43317b1b79a9a6b101df711a26d12a88b1419f63db15161a9fdb587fabae63'], 'pos': 2} (id: 25)


So, mined txs are first ordered by block height, but block 2473 needs a tie-breaker:
the two txs there are ordered based on position in the block.

The mempool txs are first ordered based on inverse-"height", but height 0 needs a tie-breaker:
those two txs are sorted by txid (`80` comes before `e0`).

Finally, the status is `78e96c6562cafa71c115503b9411fdfdc595a45031e2ab76ff75162fe1b0590d`.


Block Headers
-------------

Originally Electrum clients would download all block headers and
verify the chain of hashes and header difficulty in order to confirm
the merkle roots with which to check transaction inclusion.

With the Bitcoin main chain now past height 500,000, the headers form
over 40MB of raw data which becomes 80MB if downloaded as text from
Electrum servers.  The situation is worse for testnet and coins with
more frequent blocks.  Downloading and verifying all this data on
initial use would take several minutes, during which Electrum was
non-responsive.

To facilitate a better experience for SPV clients, particularly on
mobile, protocol :ref:`version 1.4 <version 1.4>` introduces an
optional *cp_height* argument to the :func:`blockchain.block.header`
and :func:`blockchain.block.headers` RPC calls.

This requests the server provide a merkle proof, to a single 32-byte
checkpoint hard-coded in the client, that the header(s) provided are
valid in the same way the server proves a transaction is included in a
block.  If several consecutive headers are requested, the proof is
provided for the final header - the *prev_hash* links in the headers
are sufficient to prove the others valid.

Using this feature client software only needs to download the headers
it is interested in up to the checkpoint.  Headers after the
checkpoint must all be downloaded and validated as before.  The RPC
calls return the merkle root, so to embed a checkpoint in a client
simply make an RPC request to a couple of trusted servers for the
greatest height to which a reorganisation of the chain is infeasible,
and confirm the returned roots match.

.. note:: with 500,000 headers of 80 bytes each, a naïve server
  implementation would require hashing approximately 88MB of data to
  provide a single merkle proof.  ElectrumX implements an optimization
  such that it hashes only approximately 180KB of data per proof.
