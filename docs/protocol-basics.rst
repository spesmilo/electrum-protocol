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
txid (in network byteorder) is used to arrive at a canonical ordering.

6. Next, with mempool transactions in the specified order, append a similar
string.

7. The :dfn:`status` of the script hash is the :func:`sha256` hash of the
full string expressed as a hexadecimal string, or :const:`null` if the
string is empty because there are no transactions.


**Status Example**

Consider the following BTC regtest address: `bcrt1qwu2d7rjn9yxalg7cjw6phqzk77lf3fmsf57wq4`,
which has script hash: `76bbdc72fdc74fbb0b049d21b0f80045f2a227cde1cd57a00925a437638984b5`.

Example RPC traffic::

    <-- ('blockchain.scripthash.subscribe', ['76bbdc72fdc74fbb0b049d21b0f80045f2a227cde1cd57a00925a437638984b5']) (id: 15)
    --> 3b3b8f1bf926bde2e8e21d80b7d23afbef8f8ef697e7e9204683d9a29801b671 (id: 15)
    <-- ('blockchain.scripthash.get_history', ['76bbdc72fdc74fbb0b049d21b0f80045f2a227cde1cd57a00925a437638984b5']) (id: 16)
    --> [
         {'tx_hash': '5c75485491ef13ca39c06a6c985de5223ac34a365fe26d461e567521f87bb378', 'height': 1860},
         {'tx_hash': '9a2808f0cebab7fd71033b2974ab476098f8a6deb972b72fed60878a44badeda', 'height': 1861},
         {'tx_hash': '07ec1e1dd23e337d55c7e21e892cca4d328cef5d6eddf29e59587e9056e891f7', 'height': 1861},
         {'tx_hash': '98edd636bf8142d9ec0b88de3c8dbd3eb3da9bcfb02f7496f21408d8c6be58f2', 'height': 0, 'fee': 200},
         {'tx_hash': '73fe76fe862413b8ff0f74b72c65c21fa1229a07d0625d994028363aa7cd58f7', 'height': 0, 'fee': 200},
         {'tx_hash': 'b870b52ab5ecc3c3133ef1b9fff54d8d49c9c3ab7aa4236bf4c8d734d4d3b231', 'height': -1, 'fee': 110}
         ] (id: 16)
    <-- ('blockchain.transaction.get', ['5c75485491ef13ca39c06a6c985de5223ac34a365fe26d461e567521f87bb378']) (id: 17)
    <-- ('blockchain.transaction.get', ['9a2808f0cebab7fd71033b2974ab476098f8a6deb972b72fed60878a44badeda']) (id: 18)
    <-- ('blockchain.transaction.get', ['07ec1e1dd23e337d55c7e21e892cca4d328cef5d6eddf29e59587e9056e891f7']) (id: 19)
    <-- ('blockchain.transaction.get', ['98edd636bf8142d9ec0b88de3c8dbd3eb3da9bcfb02f7496f21408d8c6be58f2']) (id: 20)
    <-- ('blockchain.transaction.get', ['73fe76fe862413b8ff0f74b72c65c21fa1229a07d0625d994028363aa7cd58f7']) (id: 21)
    <-- ('blockchain.transaction.get', ['b870b52ab5ecc3c3133ef1b9fff54d8d49c9c3ab7aa4236bf4c8d734d4d3b231']) (id: 22)
    --> 02000000000101887e2ea83a1fbface451aa3b2ba556757039bfe266d8b0c03383b2a6bc306a0b0000000000fdffffff0200e1f505000000001600147714df0e53290ddfa3d893b41b8056f7be98a77038b23f710000000016001496ea3197df375c6a62188cf9119aee984c0cc6150247304402206e1bbfca238613dbb5c877007273ddc708a5bdf0436e8e9df8cff4860937a64402200d86cc261234f5d448dd005c28a7bfc29d86b4d3004c76f5440aa662ba65479f01210242c23a20435cd52192581537b2017288b8cdf3fef349100c11a125d9529017e143070000 (id: 17)
    --> 0200000000010178b37bf82175561e466de25f364ac33a22e55d986c6ac039ca13ef915448755c0100000000fdffffff0200e1f505000000001600147714df0e53290ddfa3d893b41b8056f7be98a77070d0496b00000000160014e5ec5a8eff827f8ab906f94026e83a7404b8bf860247304402200a6a2c21d2829d65e5439e3153a6b157e231942d3fa9b9d90b2567b1eec11851022036e4797a67bb1bbd1e2239ab51905db07bb5a31a3ac63b0eadbb8e5e27baf63d012103a23dcf3edc18180b0d82b0be8eb1c30ae61597e99e298246e05109b7ba89763744070000 (id: 18)
    --> 02000000000101887e2ea83a1fbface451aa3b2ba556757039bfe266d8b0c03383b2a6bc306a0b0100000000fdffffff0200e1f505000000001600147714df0e53290ddfa3d893b41b8056f7be98a770c4b8004d3a000000160014692a676d16f83641a6518e9154d2b43a8465ed440247304402206578cda616dd7425b7f7cc35a67892361168275ac8a8ca0c6a3e1eefd595e9de02200466d0eb0a4ed4351ae806793151f5bfdae44904507efe4225a67f36ef2de76c0121030644479c1b8c106e2f0a495fe18dc45572b3413bda2db919c1fcf0f77f43241c44070000 (id: 19)
    --> 02000000000101dadeba448a8760ed2fb772b9dea6f8986047ab74293b0371fdb7bacef008289a0100000000fdffffff0200e1f505000000001600147714df0e53290ddfa3d893b41b8056f7be98a770a8ee536500000000160014783997038ebfd28edef06a1e088e9fd905c48d61024730440220426b0570147156e19cd4c6eff02f49e11747dc74eaad0299d884740c0f05958302200c93e46a3db4686d576943c88274ddb43afbcd4c491867642c1a9c3fba8ca3bf012103e0e2b67a3e8244900fa916decb6a9d45787ebe15b882c7a065cd0151d298fb4e45070000 (id: 20)
    --> 02000000000101f791e856907e58599ef2dd6e5def8c324dca2c891ee2c7557d333ed21d1eec070100000000fdffffff0200e1f505000000001600147714df0e53290ddfa3d893b41b8056f7be98a770fcd60a473a000000160014641837eaea0de6ed898e30ff6e093c566d790521024730440220143065381913b92b572b6fc6362f89517f0d411d19d23ba025f43885736fdf7602201176d0f5284c65023baa06d760810085e8d7e0835b9a64a8046ab3c883499357012102be2e344c2f49afa8d734a2f79a09827b038d2f8f6bfa77d5a2e077c22a3fd80a45070000 (id: 21)
    --> 02000000000101f258bec6d80814f296742fb0cf9bdab33ebd8d3cde880becd94281bf36d6ed980000000000fdffffff0192e0f505000000001600147714df0e53290ddfa3d893b41b8056f7be98a770024730440220569e38b9034c5855b866a35a16d949d16b7b544aff9d6b02b6d2ca7ebbd9c3b2022074a7d230241335436a2df1380a4d7c5442424a9a56a02124f4a0b495e9de24b50121027ba4d3ee6471a985307a37d09eb9a0a73c1a31b57616fe3e53a3d6d45400251945070000 (id: 22)
    <-- ('blockchain.transaction.get_merkle', ['5c75485491ef13ca39c06a6c985de5223ac34a365fe26d461e567521f87bb378', 1860]) (id: 23)
    <-- ('blockchain.transaction.get_merkle', ['9a2808f0cebab7fd71033b2974ab476098f8a6deb972b72fed60878a44badeda', 1861]) (id: 24)
    <-- ('blockchain.transaction.get_merkle', ['07ec1e1dd23e337d55c7e21e892cca4d328cef5d6eddf29e59587e9056e891f7', 1861]) (id: 25)
    --> {'block_height': 1860, 'merkle': ['7df6810d7ac97c43cbfc2bdc6be10e26cd54ad34ab5e563fd8bde7f86ff36975'], 'pos': 1} (id: 23)
    --> {'block_height': 1861, 'merkle': ['ac7d7a5d8932f565a2cf9cb103881bfc3bf7aa2f56ccbcdeb31e02c524ddfd0b', 'f7d333193f2e5c48c236db7c3eafd5d03f387a71c4f9b6056617faefcf3d44cf'], 'pos': 1} (id: 24)
    --> {'block_height': 1861, 'merkle': ['07ec1e1dd23e337d55c7e21e892cca4d328cef5d6eddf29e59587e9056e891f7', '5441fd37bd00b55ff5f9ea1a8956d2f3ade17c7fb7a363e8000b3e7cb40ea135'], 'pos': 2} (id: 25)

So, mined txs are first ordered by block height, but block 1861 needs a tie-breaker:
the two txs there are ordered based on position in the block.

The mempool txs are first ordered based on inverse-"height", but height 0 needs a tie-breaker:
those two txs are ordered based on network-endian txid (`f2` comes before `f7`).

Finally, the status is `3b3b8f1bf926bde2e8e21d80b7d23afbef8f8ef697e7e9204683d9a29801b671`.


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
