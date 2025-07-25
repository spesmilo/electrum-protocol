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

Messages SHOULD be :ref:`padded <padding_messages>` to bucketed lengths,
and buffered (to introduce delays) to protect against traffic analysis.

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


.. _scriptpubkeys:
.. _script hashes:

Script Hashes
-------------

A :dfn:`script hash` is the hash of the binary bytes of the locking
script (scriptPubKey), expressed as a hexadecimal string.  The hash
function to use is :func:`sha256`.  Like for
block and transaction hashes, when converting the big-endian binary
hash to a hexadecimal string the least-significant byte appears first,
and the most-significant byte last.

For example, the legacy Bitcoin address from the genesis block::

    1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa

has P2PKH script (scriptPubKey)::

    76a91462e907b15cbf27d5425399ebf6f0fb50ebb88f1888ac

with SHA256 hash::

    6191c3b590bfcfa0475e877c302da1e323497acf3b42c08d8fa28e364edf018b

the scripthash is defined as the reverse of that::

    8b01df4e368ea28f8dc0423bcf7a4923e3a12d307c875e47a0cfbf90b5c39161

By subscribing to the scriptPubKey or the scripthash,
you can find P2PKH payments to that address.

One public key, the genesis block public key, among the trillions for
that address is::

    04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb
    649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f

which has P2PK script (scriptPubKey)::

    4104678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb
    649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5fac

with SHA256 hash::

    3318537dfb3135df9f3d950dbdf8a7ae68dd7c7dfef61ed17963ff80f3850474

the scripthash is defined as the reverse of that::

    740485f380ff6379d11ef6fe7d7cdd68aea7f8bd0d953d9fdf3531fb7d531833

By subscribing to the scriptPubKey or the scripthash,
you can find P2PK payments to the genesis
block public key.

.. note:: The Genesis block coinbase is uniquely unspendable and
   therefore not indexed.  It will not show with the above P2PK script
   hash subscription.


.. _status:

Status
------

To calculate the `status` of a :ref:`scriptPubKey <scriptpubkeys>`
(or :ref:`script hash <script hashes>` or address):

1. Consider all transactions touching the scriptPubKey (both those spending
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

7. The :dfn:`status` of the scriptPubKey is the :func:`sha256` hash of the
full string expressed as a hexadecimal string, or :const:`null` if the
string is empty because there are no transactions.


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


.. _padding_messages:

Traffic analysis
----------------

The goal is to defend against a passive network Man-in-the-Middle, such as an ISP
or a Tor exit node, observing the encrypted TLS stream, and making educated guesses
of the message contents based on TCP packet flow: timing, direction, and sizes of TCP packets.

.. note:: Raw cleartext TCP as transport for the JSON-RPC payloads is clearly out-of-scope here.
  Without encryption, a passive network observer could just see all the plaintext messages anyway.

As a generic mitigation, implementations (both client and server) SHOULD

- pad messages to bucketed lengths (e.g. powers of 2, with a min size), and

- introduce small timing delays, ideally by buffering messages.

We can fully backwards-compatibly add padding to the JSON-RPC messages by adding extra
whitespaces inside the JSON objects in a way that parsers ignore.
This can be done at any protocol version.

For example, instead of sending::

  {"jsonrpc":"2.0","method":"server.version","id":0,"params":["electrum/4.5.8","1.4"]}\n


the client could send::

  {"jsonrpc":"2.0","method":"server.version","id":0,"params":["electrum/4.5.8","1.4"]           }\n


For better results, both the client and the server SHOULD implement logic to pad the messages
that they send. So that requests and responses (and notifications) SHOULD all be padded.
This does not have to be rolled out simultaneously: it is ok for only a client to pad what
they send and not the server (or the other way around),
that just limits the effectiveness of the defense against traffic analysis.

Note when the JSON-RPC messages are sent in the TLS stream, they are sometimes batched together.
That is, a single TCP packet might contain multiple small JSON-RPC messages,
e.g. if the client tries to send multiple messages in a short burst.
Also, many protocol requests are <100 bytes, so it would be wasteful to pad all to e.g. 1 kbyte.

To save bandwidth, instead of padding individual JSON-RPC messages,
participants (the client and the server) COULD implement an application-level buffer,
write the messages into that buffer, periodically empty the buffer into the TLS stream
and only add the padding into e.g. the last JSON-RPC message when emptying the buffer.

.. note:: Example implementation
  in the `Electrum client <https://github.com/spesmilo/electrum/pull/9875>`_
  and in the `electrumx server <https://github.com/spesmilo/electrumx/pull/301>`_:

  Both the client and the server writes raw JSON-RPC protocol messages into a buffer,
  which is then occasionally flushed to the wire. When it is flushed, padding is added
  to round up the total length to 1 KB, or to the next power of 2.
  The buffer is flushed if it reaches 1 KB, plus there is extra logic that periodically polls
  if the oldest message in the buffer is older than 1 second, in which case it is also flushed.

.. note:: Many protocol requests are <100 bytes. Contrast that with broadcasting a transaction,
  which could potentially be several megabytes of data.
  (max consensus-valid tx is 4 MB, times 2 for hex-encoding)
  Hence padding to a constant size is not practical.
  Instead it is recommended to pad to bucketed lengths, e.g. to powers of 2.

The specific details of the size of the buffer, how often it is flushed, and how the padding
is done is not specified by the protocol at the moment.

.. note:: Some server implementations do not deal with TLS at all,
  they only implement the raw cleartext TCP protocol, and just recommend operators
  to put a reverse proxy in front that does TLS termination.
  Such protocol implementations (both client and server) are nevertheless still
  SHOULD implement all mentioned traffic analysis protections.
  That way, if the operator tunnels the traffic over TLS externally,
  the resulting stream meaningfully receives the protections.

.. note:: Buffering the messages to introduce timing delays
  and padding to ~bucketed sizes is a good baseline.
  However even approximate timing and direction of TCP packets
  can leak too much information in some scenarios.

  To combat timing analysis, both the client and the server
  COULD send noise with a random timer, but more importantly at strategically selected events.
  For example, when the client receives a new block header notification,
  it COULD probabilistically send a random number of "server.ping" messages
  with small random sleeps in-between.

  Protocol version 1.6 extends "server.ping" so that either party can send it, and
  that it can be sent either as a JSON-RPC "Request" or as a JSON-RPC "Notification".
  If sent as a notification, the receiver is expected not to respond.

  When the server sends a block header notification to the client,
  it COULD also probabilistically send noise ("server.ping") notifications to the client,
  perhaps conditioned on whether it will also send
  :func:`blockchain.scriptpubkey.subscribe` notifications.
  (so server could send noise if there are no status notifications to be sent)
