========================
Removed Protocol Methods
========================

This documents protocol methods that are still supported in some protocol
versions, but not the most recent one.

.. _deserialized header:

Deserialized Headers
--------------------

A :dfn:`deserialized header` is a dictionary describing a block at a
given height.

A typical example would be similar to this template::

  {
    "block_height": <integer>,
    "version": <integer>,
    "prev_block_hash": <hexadecimal string>,
    "merkle_root":  <hexadecimal string>,
    "timestamp": <integer>,
    "bits": <integer>,
    "nonce": <integer>
  }

.. note:: The precise format of a deserialized block header varies by
  coin, and also potentially by height for the same coin.  Detailed
  knowledge of the meaning of a block header is neither necessary nor
  appropriate in the server.  Consequently they were removed from the
  protocol in version 1.4.


blockchain.address.get_balance
==============================

Return the confirmed and unconfirmed balances of a bitcoin address.

**Signature**

  .. function:: blockchain.address.get_balance(address)
  .. deprecated:: 1.2 removed in version 1.3

  * *address*

    The address as a Base58 string.

**Result**

  See :func:`blockchain.scripthash.get_balance`.

blockchain.address.get_history
==============================

Return the confirmed and unconfirmed history of a bitcoin address.

**Signature**

  .. function:: blockchain.address.get_history(address)
  .. deprecated:: 1.2 removed in version 1.3

  * *address*

    The address as a Base58 string.

**Result**

  As for :func:`blockchain.scripthash.get_history`.

blockchain.address.get_mempool
==============================

Return the unconfirmed transactions of a bitcoin address.

**Signature**

  .. function:: blockchain.address.get_mempool(address)
  .. deprecated:: 1.2 removed in version 1.3

  * *address*

    The address as a Base58 string.

**Result**

  As for :func:`blockchain.scripthash.get_mempool`.

blockchain.address.listunspent
==============================

Return an ordered list of UTXOs sent to a bitcoin address.

**Signature**

  .. function:: blockchain.address.listunspent(address)
  .. deprecated:: 1.2 removed in version 1.3

  * *address*

    The address as a Base58 string.

**Result**

  As for :func:`blockchain.scripthash.listunspent`.

blockchain.address.subscribe
============================

Subscribe to a bitcoin address.

**Signature**

  .. function:: blockchain.address.subscribe(address)
  .. deprecated:: 1.2 removed in version 1.3

  *address*

    The address as a Base58 string.

**Result**

  The :ref:`status <status>` of the address.

**Notifications**

  As this is a subscription, the client will receive a notification
  when the :ref:`status <status>` of the address changes.  Its
  signature is

  .. function:: blockchain.address.subscribe(address, status)
     :noindex:

blockchain.headers.subscribe
============================

Subscribe to receive block headers when a new block is found.

**Signature**

  .. blockchain.headers.subscribe()
  .. versionchanged:: 1.2
     Optional *raw* parameter added, defaulting to :const:`false`.
  .. versionchanged:: 1.3
     *raw* parameter defaults to :const:`true`.
  .. versionchanged:: 1.4
     *raw* parameter removed; responses and notifications pass raw
     headers.

  * *raw*

    This single boolean argument exists in protocol versions 1.2
    (defaulting to :const:`false`) and 1.3 (defaulting to
    :const:`true`) only.

**Result**

  The header of the current block chain tip.  If *raw* is
  :const:`true` the result is a dictionary with two members:

  * *hex*

    The binary header as a hexadecimal string.

  * *height*

    The height of the header, an integer.

  If *raw* is :const:`false` the result is the coin-specific
  :ref:`deserialized header <deserialized header>`.

**Example Result**

  With *raw* :const:`false`::

   {
     "bits": 402858285,
     "block_height": 520481,
     "merkle_root": "8e8e932eb858fd53cf09943d7efc9a8f674dc1363010ee64907a292d2fb0c25d",
     "nonce": 3288656012,
     "prev_block_hash": "000000000000000000b512b5d9fc7c5746587268547c04aa92383aaea0080289",
     "timestamp": 1520495819,
     "version": 536870912
   }

  With *raw* :const:`true`::

   {
     "height": 520481,
     "hex": "00000020890208a0ae3a3892aa047c5468725846577cfcd9b512b50000000000000000005dc2b02f2d297a9064ee103036c14d678f9afc7e3d9409cf53fd58b82e938e8ecbeca05a2d2103188ce804c4"
   }

**Notifications**

  As this is a subscription, the client will receive a notification
  when a new block is found.  The notification's signature is:

    .. blockchain.headers.subscribe(header)

    * *header*

      See **Result** above.

.. note:: should a new block arrive quickly, perhaps while the server
  is still processing prior blocks, the server may only notify of the
  most recent chain tip.  The protocol does not guarantee notification
  of all intermediate block headers.

  In a similar way the client must be prepared to handle chain
  reorganisations.  Should a re-org happen the new chain tip will not
  sit directly on top of the prior chain tip.  The client must be able
  to figure out the common ancestor block and request any missing
  block headers to acquire a consistent view of the chain state.

blockchain.numblocks.subscribe
==============================

Subscribe to receive the block height when a new block is found.

**Signature**

  .. function:: blockchain.numblocks.subscribe()
  .. deprecated:: 1.0 removed in version 1.1

**Result**

  The height of the current block, an integer.

**Notifications**

  As this is a subscription, the client will receive a notification
  when a new block is found.  The notification's signature is:

    .. function:: blockchain.numblocks.subscribe(height)
     :noindex:

blockchain.utxo.get_address
===========================

Return the address paid to by a UTXO.

**Signature**

  .. function:: blockchain.utxo.get_address(tx_hash, index)

    *Optional in version 1.0, removed in version 1.1*

  *tx_hash*

    The transaction hash as a hexadecimal string.

  *index*

    The zero-based index of the UTXO in the transaction.

**Result**

  A Base58 address string, or :const:`null`.  If the transaction
  doesn't exist, the index is out of range, or the output is not paid
  to an address, :const:`null` must be returned.  If the output is
  spent :const:`null` *may* be returned.

blockchain.block.get_header
===========================

Return the :ref:`deserialized header <deserialized header>` of the
block at the given height.

**Signature**

  .. function:: blockchain.block.get_header(height)
  .. deprecated:: 1.3 removed in version 1.4

  *height*

    The height of the block, an integer.

**Result**

  The coin-specific :ref:`deserialized header <deserialized header>`.

**Example Result**

::

  {
    "bits": 392292856,
    "block_height": 510000,
    "merkle_root": "297cfcc6a66e063692b20650d21cc0ac7a2a80f7277ebd7c5d6c7010a070d25c",
    "nonce": 3347656422,
    "prev_block_hash": "0000000000000000002292de0d9f03dfa15a04dbf09102d5d4552117b717fa86",
    "timestamp": 1519083654,
    "version": 536870912
  }

blockchain.block.get_chunk
==========================

Return a concatenated chunk of block headers from the main chain.
Typically, a chunk consists of a fixed number of block headers over
which difficulty is constant, and at the end of which difficulty is
retargeted.

In the case of Bitcoin a chunk is 2,016 headers, each of 80 bytes, so
chunk 5 consists of the block headers from height 10,080 to 12,095
inclusive.  When encoded as hexadecimal, the result string is twice as
long, so for Bitcoin it takes 322,560 bytes, making this a
bandwidth-intensive request.

**Signature**

  .. function:: blockchain.block.get_chunk(index)
  .. deprecated:: 1.2 removed in version 1.4

  *index*

    The zero-based index of the chunk, an integer.

**Result**

    The binary block headers as hexadecimal strings, in-order and
    concatenated together.  As many as headers as are available at the
    implied starting height will be returned; this may range from zero
    to the coin-specific chunk size.


blockchain.relayfee
===================

Feerates lower than this are considered zero fee and are not being
relayed to the bitcoin network by the server.
This feerate does not guarantee acceptance into the mempool of the server.

**Signature**

  .. function:: blockchain.relayfee()

**Result**

  The feerate in BTC/kvB, as a floating point number.

**Example Results**

::

   1e-05

::

   0.0


blockchain.scripthash.get_balance
=================================

Return the confirmed and unconfirmed balances of a :ref:`script hash
<script hashes>`.

**Signature**

  .. function:: blockchain.scripthash.get_balance(scripthash)
  .. versionadded:: 1.1
  .. deprecated:: removed in version 1.7

  *scripthash*

    The script hash as a hexadecimal string.

**Result**

  A dictionary with keys `confirmed` and `unconfirmed`.  The value of
  each is the appropriate balance in minimum coin units (satoshis).
  The `confirmed` balance is the sum of UTXO values at the current chaintip.
  The value for `unconfirmed` is the mempool delta (compared to `confirmed`),
  so note that it can also be negative.

**Result Example**

::

  {
    "confirmed": 103873966,
    "unconfirmed": 23684400
  }

blockchain.scripthash.get_history
=================================

Return the confirmed and unconfirmed history of a :ref:`script hash
<script hashes>`.

**Signature**

  .. function:: blockchain.scripthash.get_history(scripthash)
  .. versionadded:: 1.1
  .. deprecated:: removed in version 1.7

  *scripthash*

    The script hash as a hexadecimal string.

**Result**

  A list of confirmed transactions in blockchain order, with the
  output of :func:`blockchain.scripthash.get_mempool` appended to the
  list.  Each confirmed transaction is a dictionary with the following
  keys:

  * *height*

    The integer height of the block the transaction was confirmed in.

  * *tx_hash*

    The transaction hash in hexadecimal.

  See :func:`blockchain.scripthash.get_mempool` for how mempool
  transactions are returned.

**Result Examples**

::

  [
    {
      "height": 200004,
      "tx_hash": "acc3758bd2a26f869fcc67d48ff30b96464d476bca82c1cd6656e7d506816412"
    },
    {
      "height": 215008,
      "tx_hash": "f3e1bf48975b8d6060a9de8884296abb80be618dc00ae3cb2f6cee3085e09403"
    }
  ]

::

  [
    {
      "fee": 20000,
      "height": 0,
      "tx_hash": "9fbed79a1e970343fcd39f4a2d830a6bde6de0754ed2da70f489d0303ed558ec"
    }
  ]

blockchain.scripthash.get_mempool
=================================

Return the unconfirmed transactions of a :ref:`script hash <script
hashes>`.

**Signature**

  .. function:: blockchain.scripthash.get_mempool(scripthash)
  .. versionadded:: 1.1
  .. versionchanged:: 1.6
     results must be sorted (previously undefined order)
  .. deprecated:: removed in version 1.7

  *scripthash*

    The script hash as a hexadecimal string.

**Result**

  A list of mempool transactions. The order is the same as when computing the
  :ref:`status <status>` of the script hash.
  Each mempool transaction is a dictionary with the following keys:

  * *height*

    ``0`` if all inputs are confirmed, and ``-1`` otherwise.

  * *tx_hash*

    The transaction hash in hexadecimal.

  * *fee*

    The transaction fee in minimum coin units (satoshis).

**Result Example**

::

  [
    {
      "tx_hash": "45381031132c57b2ff1cbe8d8d3920cf9ed25efd9a0beb764bdb2f24c7d1c7e3",
      "height": 0,
      "fee": 24310
    }
  ]


blockchain.scripthash.listunspent
=================================

Return an ordered list of UTXOs sent to a script hash.

**Signature**

  .. function:: blockchain.scripthash.listunspent(scripthash)
  .. versionadded:: 1.1
  .. deprecated:: removed in version 1.7

  *scripthash*

    The script hash as a hexadecimal string.

**Result**

  A list of unspent outputs in blockchain order.  This function takes
  the mempool into account.  Mempool transactions paying to the
  address are included at the end of the list in an undefined order.
  Any output that is spent in the mempool does not appear.  Each
  output is a dictionary with the following keys:

  * *height*

    The integer height of the block the transaction was confirmed in.
    ``0`` if the transaction is in the mempool.

  * *tx_pos*

    The zero-based index of the output in the transaction's list of
    outputs.

  * *tx_hash*

    The output's transaction hash as a hexadecimal string.

  * *value*

    The output's value in minimum coin units (satoshis).


**Warning**

  In the case of pre-segwit legacy UTXOs, the satoshi value claimed by a server should be
  verified by the client by requesting the full funding transaction and parsing it
  to look for the output amount corresponding to ``tx_hash:tx_pos``.
  This is necessary as the pre-segwit legacy sighash does not commit to the input amount, so
  the server could try to trick a client into burning their coins as fees.
  Note that it is not necessary to SPV-verify ``tx_hash``, as the sighash commits to the txid,
  and the txid commits to the raw tx, from which we read out the satoshi amount.


**Result Example**

::

  [
    {
      "tx_pos": 0,
      "value": 45318048,
      "tx_hash": "9f2c45a12db0144909b5db269415f7319179105982ac70ed80d76ea79d923ebf",
      "height": 437146
    },
    {
      "tx_pos": 0,
      "value": 919195,
      "tx_hash": "3d2290c93436a3e964cfc2f0950174d8847b1fbe3946432c4784e168da0f019f",
      "height": 441696
    }
  ]

blockchain.scripthash.subscribe
===============================

Subscribe to a script hash.

**Signature**

  .. function:: blockchain.scripthash.subscribe(scripthash)
  .. versionadded:: 1.1
  .. deprecated:: removed in version 1.7

  *scripthash*

    The script hash as a hexadecimal string.

**Result**

  The :ref:`status <status>` of the script hash.

**Notifications**

  The client will receive a notification when the :ref:`status <status>` of the script
  hash changes.  Its signature is

    .. function:: blockchain.scripthash.subscribe(scripthash, status)
       :noindex:

blockchain.scripthash.unsubscribe
=================================

Unsubscribe from a script hash, preventing future notifications if its :ref:`status
<status>` changes.

**Signature**

  .. function:: blockchain.scripthash.unsubscribe(scripthash)
  .. versionadded:: 1.4.2
  .. deprecated:: removed in version 1.7

  *scripthash*

    The script hash as a hexadecimal string.

**Result**

  Returns :const:`True` if the scripthash was subscribed to, otherwise :const:`False`.
  Note that :const:`False` might be returned even for something subscribed to earlier,
  because the server can drop subscriptions in rare circumstances.


server.version
==============

Identify the client to the server and negotiate the protocol version.

**Signature**

  .. server.version(client_name="", protocol_version="1.4")
  .. versionchanged:: 1.1
     *protocol_version* is not ignored.
  .. versionchanged:: 1.2
     Use :func:`server.ping` rather than sending version requests as a
     ping mechanism.
  .. versionchanged:: 1.4
     Only the first :func:`server.version` message is accepted.

  * *client_name*

    A string identifying the connecting client software.

  * *protocol_version*

    An array ``[protocol_min, protocol_max]``, each of which is a
    string.  If ``protocol_min`` and ``protocol_max`` are the same,
    they can be passed as a single string rather than as an array of
    two strings, as for the default value.

  The server should use the highest protocol version both support::

    version = min(client.protocol_max, server.protocol_max)

  If this is below the value::

    max(client.protocol_min, server.protocol_min)

  then there is no protocol version in common and the server must
  close the connection.  Otherwise it should send a response
  appropriate for that protocol version.

**Result**

  An array of 2 strings:

     ``[server_software_version, protocol_version]``

  identifying the server and the protocol version that will be used
  for future communication.

  *Protocol version 1.0*: A string identifying the server software.

**Examples**::

  server.version("Electrum 3.0.6", ["1.1", "1.2"])
  server.version("2.7.1", "1.0")

**Example Results**::

  ["ElectrumX 1.2.1", "1.2"]
  "ElectrumX 1.2.1"
