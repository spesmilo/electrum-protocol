==================
 Protocol Methods
==================

blockchain.block.header
=======================

Return the block header at the given height.

**Signature**

  .. function:: blockchain.block.header(height, cp_height=0)
  .. versionadded:: 1.3
  .. versionchanged:: 1.4
     *cp_height* parameter added
  .. versionchanged:: 1.4.1
  .. versionchanged:: 1.6
     *cp_height* support made optional

  *height*

    The height of the block, a non-negative integer.

  *cp_height*

    Checkpoint height, a non-negative integer.  Ignored if zero,
    otherwise the following must hold:

      *height* <= *cp_height*

    The server CAN decide not to support a non-zero cp_height value, but if so,
    it MUST indicate that in its :func:`server.features` response by setting
    `method_flavours["blockchain.block.header"]["supports_cp_height"]=false`.

**Result**

  If *cp_height* is zero, the raw block header as a hexadecimal
  string.

  Otherwise a dictionary with the following keys.  This provides a
  proof that the given header is present in the blockchain; presumably
  the client has the merkle root hard-coded as a checkpoint.

  * *branch*

    The merkle branch of *header* up to *root*, deepest pairing first.

  * *header*

    The raw block header as a hexadecimal string.  Starting with version 1.4.1,
    AuxPoW data (if present in the original header) is truncated.

  * *root*

    The merkle root of all blockchain headers up to and including
    *cp_height*.


**Example Result**

With *height* 5 and *cp_height* 0 on the Bitcoin chain:

::

   "0100000085144a84488ea88d221c8bd6c059da090e88f8a2c99690ee55dbba4e00000000e11c48fecdd9e72510ca84f023370c9a38bf91ac5cae88019bee94d24528526344c36649ffff001d1d03e477"

.. _cp_height example:

With *cp_height* 8::

  {
    "branch": [
       "000000004ebadb55ee9096c9a2f8880e09da59c0d68b1c228da88e48844a1485",
       "96cbbc84783888e4cc971ae8acf86dd3c1a419370336bb3c634c97695a8c5ac9",
       "965ac94082cebbcffe458075651e9cc33ce703ab0115c72d9e8b1a9906b2b636",
       "89e5daa6950b895190716dd26054432b564ccdc2868188ba1da76de8e1dc7591"
       ],
    "header": "0100000085144a84488ea88d221c8bd6c059da090e88f8a2c99690ee55dbba4e00000000e11c48fecdd9e72510ca84f023370c9a38bf91ac5cae88019bee94d24528526344c36649ffff001d1d03e477",
    "root": "e347b1c43fd9b5415bf0d92708db8284b78daf4d0e24f9c3405f45feb85e25db"
  }

blockchain.block.headers
========================

Return a chunk of block headers from the main chain.

**Signature**

  .. function:: blockchain.block.headers(start_height, count, cp_height=0)
  .. versionadded:: 1.2
  .. versionchanged:: 1.4
     *cp_height* parameter added
  .. versionchanged:: 1.4.1
  .. versionchanged:: 1.6
     response contains *headers* field instead of *hex*
     *cp_height* support made optional

  *start_height*

    The height of the first header requested, a non-negative integer.

  *count*

    The number of headers requested, a non-negative integer.

  *cp_height*

    Checkpoint height, a non-negative integer.  Ignored if zero,
    otherwise the following must hold:

      *start_height* + (*count* - 1) <= *cp_height*

    The server CAN decide not to support a non-zero cp_height value, but if so,
    it MUST indicate that in its :func:`server.features` response by setting
    `method_flavours["blockchain.block.header"]["supports_cp_height"]=false`.
    (the flavour key `"blockchain.block.header"` is reused with the other header method).

**Result**

  A dictionary with the following members:

  * *count*
    The number of headers returned, between zero and the number
    requested.  If the chain has not extended sufficiently far, only
    the available headers will be returned.  If more headers than
    *max* were requested at most *max* will be returned.

  * *headers*

    An array containing the binary block headers in-order; each header is a
    hexadecimal string.  AuxPoW data (if present in the original header) is
    truncated if *cp_height* is nonzero.

  * *max*

    The maximum number of headers the server will return in a single
    request.  (Recommended to be at least one difficulty retarget period,
    i.e. 2016)

  The dictionary additionally has the following keys if *count* and
  *cp_height* are not zero.  This provides a proof that all the given
  headers are present in the blockchain; presumably the client has the
  merkle root hard-coded as a checkpoint.

  * *root*

    The merkle root of all blockchain headers up to and including
    *cp_height*.

  * *branch*

    The merkle branch of the last returned header up to *root*,
    deepest pairing first.


**Example Response**

See :ref:`here <cp_height example>` for an example of *root* and
*branch* keys.

::

  {
    "count": 2,
    "headers":
    [
      "0100000000000000000000000000000000000000000000000000000000000000000000003ba3edfd7a7b12b27ac72c3e67768f617fc81bc3888a51323a9fb8aa4b1e5e4a29ab5f49ffff001d1dac2b7c",
      "010000006fe28c0ab6f1b372c1a6a246ae63f74f931e8365e15a089c68d6190000000000982051fd1e4ba744bbbe680e1fee14677ba1a3c3540bf7b1cdb606e857233e0e61bc6649ffff001d01e36299"
    ],
    "max": 2016
  }

blockchain.estimatefee
======================

Return the estimated transaction fee per kilobyte for a transaction to
be confirmed within a certain number of blocks.

**Signature**

  .. function:: blockchain.estimatefee(number, mode=None)
  .. versionchanged:: 1.6
     *mode* argument added

  *number*

    The number of blocks to target for confirmation.

  *mode*

    A string to pass to the bitcoind *estimatesmartfee* RPC as the
    *estimate_mode* parameter. Optional. If omitted, the corresponding
    parameter to the bitcoind RPC is also omitted, i.e. the default
    value is determined by bitcoind.

**Result**

  The estimated transaction fee in whole coin units per kilobyte, as a
  floating point number.  If the daemon does not have enough
  information to make an estimate, the integer ``-1`` is returned.

**Example Result**

::

  0.00101079

.. note:: This estimate typically comes from the Bitcoin daemon, which only updates
  its estimate when new blocks are mined. The server is free to cache this internally
  for performance reasons, however it SHOULD avoid sending stale estimates
  by e.g. invalidating the cache before notifying clients of a new block header.


blockchain.headers.subscribe
============================

Subscribe to receive block headers when a new block is found.

**Signature**

  .. function:: blockchain.headers.subscribe()

**Result**

  The header of the current block chain tip.  The result is a dictionary with two members:

  * *hex*

    The binary header as a hexadecimal string.

  * *height*

    The height of the header, an integer.

**Example Result**

::

   {
     "height": 520481,
     "hex": "00000020890208a0ae3a3892aa047c5468725846577cfcd9b512b50000000000000000005dc2b02f2d297a9064ee103036c14d678f9afc7e3d9409cf53fd58b82e938e8ecbeca05a2d2103188ce804c4"
   }

**Notifications**

  As this is a subscription, the client will receive a notification
  when a new block is found.  The notification's signature is:

    .. function:: blockchain.headers.subscribe(header)
       :noindex:

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


blockchain.relayfee
===================

Return the minimum fee a low-priority transaction must pay in order to
be accepted to the daemon's memory pool.

**Signature**

  .. function:: blockchain.relayfee()

**Result**

  The fee in whole coin units (BTC, not satoshis) as a
  floating point number.

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

.. _subscribed:

blockchain.scripthash.subscribe
===============================

Subscribe to a script hash.

**Signature**

  .. function:: blockchain.scripthash.subscribe(scripthash)
  .. versionadded:: 1.1

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

  *scripthash*

    The script hash as a hexadecimal string.

**Result**

  Returns :const:`True` if the scripthash was subscribed to, otherwise :const:`False`.
  Note that :const:`False` might be returned even for something subscribed to earlier,
  because the server can drop subscriptions in rare circumstances.

blockchain.outpoint.subscribe
=============================

Subscribe to a transaction outpoint (TXO), to get notifications about its status.
A status involves up to two transactions: the funding transaction that creates
the TXO (as one of its outputs), and the spending transaction that uses it
as an input (spends it).

**Signature**

  .. function:: blockchain.outpoint.subscribe(tx_hash, txout_idx, spk_hint=None)
  .. versionadded:: 1.6

  *tx_hash*

    The TXID of the funding transaction as a hexadecimal string.
    (sometimes called prevout_hash, in inputs)

  *txout_idx*

    The output index, a non-negative integer. (sometimes called prevout_n, in inputs)

  *spk_hint*

    The scriptPubKey (output script) corresponding to the outpoint, as a hexadecimal
    string. This is optional, and if provided might be used by the server to find
    the outpoint. The behaviour is undefined if an incorrect value is provided.
    The server (especially lighter ones such as EPS/BWT) might require this parameter
    to be able to serve the request, in which case the server must indicate so in its
    :func:`server.features` response, by setting
    `method_flavours["blockchain.outpoint.subscribe"]["requires_spk_hint"]=true`.

.. note::  The server MAY automatically clean up subscriptions (unsubscribe the client)
  where the spending transaction is already deeply mined at a reorg-safe height (typically
  100+ blocks deep).
  Similarly, the server MAY ignore new subscription requests if the spending tx is already
  mined at a reorg-safe height but it still MUST send at least one full response.

**Result**

  The status of the TXO, taking the mempool into consideration.
  The output is a dictionary, containing 0, 1, or 3 of the following items:

  * *height*

    The integer height of the block the funding transaction was confirmed in.
    If the funding transaction is in the mempool; the value is
    ``0`` if all its inputs are confirmed, and ``-1`` otherwise.
    This key must be present if and only if there exists a funding transaction
    (either in the best chain or in the mempool), regardless of spentness.

  * *spender_txhash*

    The TXID of the spending transaction as a hexadecimal string.
    This key is present if and only if there exists a spending transaction
    (either in the best chain or in the mempool).

  * *spender_height*

    The integer height of the block the spending transaction was confirmed in.
    If the spending transaction is in the mempool; the value is
    ``0`` if all its inputs are confirmed, and ``-1`` otherwise.
    This key is present if and only if the *spender_txhash* key is present.

**Result Examples**

::

  {}

::

  {
    "height": 1866594
  }

::

  {
    "height": 1866594,
    "spender_txhash": "4a19a360f71814c566977114c49ccfeb8a7e4719eda26cee27fa504f3f02ca09",
    "spender_height": 0
  }

**Notifications**

  The client will receive a notification when the `status` of the outpoint changes.
  That is, any event that changes any field of the `status` dictionary results in a
  notification. Some examples:

  * a funding/spending tx appearing in the mempool if there was no such tx when the client subbed
    (note: the server MUST save the subscription even if the outpoint does not exist yet)
  * funding/spending tx height changing from -1 to 0 as its inputs got mined
  * funding/spending tx height changing from 0 to a (positive) block height when it gets mined
  * note that reorgs can change any of the `status` fields and result in notifications
  * note that mempool replacement (e.g. due to RBF) or mempool eviction (and potentially other
    mempool quirks) can also change some of the `status` fields and hence result in notifications

  The client MAY receive a notification even if the status did not change
  (when e.g. there was a reorg changing the blockhash the tx is mined in but not the height).

  The signature of the notification is

    .. function:: blockchain.outpoint.subscribe([tx_hash, txout_idx], status)
       :noindex:

**Full JSON-RPC Example**

Here is an example where the client sends a request, gets an immediate response,
and then at some point later - while the connection is still open -
receives a notification.

::

  -> {
    "jsonrpc": "2.0",
    "id": 4,
    "method": "blockchain.outpoint.subscribe",
    "params": ["1872b27abc497492a775fe335abfe368af575733144a7ecd4b249d8fd885b3cf", 1]
  }
  <- {
    "jsonrpc": "2.0",
    "result": {"height": 1866594},
    "id": 4
  }

  # notification after broadcasting tx 4a19a360f71814c566977114c49ccfeb8a7e4719eda26cee27fa504f3f02ca09
  <- {
    "jsonrpc": "2.0",
    "method": "blockchain.outpoint.subscribe",
    "params": [
      ["1872b27abc497492a775fe335abfe368af575733144a7ecd4b249d8fd885b3cf", 1],
      {
        "height": 1866594,
        "spender_txhash": "4a19a360f71814c566977114c49ccfeb8a7e4719eda26cee27fa504f3f02ca09",
        "spender_height": 0
      }
    ]
  }


blockchain.outpoint.get_status
==============================

Get the status of a transaction outpoint (TXO).
Same as :func:`blockchain.outpoint.subscribe`, but without subscribing to future changes of status
(i.e. no subsequent notifications).

**Signature**

  .. function:: blockchain.outpoint.get_status(tx_hash, txout_idx, spk_hint=None)
  .. versionadded:: 1.6

  (same as :func:`blockchain.outpoint.subscribe`)

**Result**

  (same as :func:`blockchain.outpoint.subscribe`)

blockchain.outpoint.unsubscribe
===============================

Unsubscribe from a transaction outpoint (TXO), preventing future notifications
if its `status` changes.

**Signature**

  .. function:: blockchain.outpoint.unsubscribe(tx_hash, txout_idx)
  .. versionadded:: 1.6

  *tx_hash*

    The TXID of the funding transaction as a hexadecimal string.

  *txout_idx*

    The output index, a non-negative integer.

**Result**

  Returns :const:`True` if the outpoint was subscribed to, otherwise :const:`False`.
  Note that :const:`False` might be returned even for something subscribed to earlier,
  because the server can drop subscriptions in rare circumstances.

blockchain.transaction.broadcast
================================

Broadcast a transaction to the network.

**Signature**

  .. function:: blockchain.transaction.broadcast(raw_tx)
  .. versionchanged:: 1.1
     errors returned as JSON RPC errors rather than as a result.

  *raw_tx*

    The raw transaction as a hexadecimal string.

**Result**

  The transaction hash as a hexadecimal string.

  **Note** protocol version 1.0 (only) does not respond according to
  the JSON RPC specification if an error occurs.  If the daemon
  rejects the transaction, the result is the error message string from
  the daemon, as if the call were successful.  The client needs to
  determine if an error occurred by comparing the result to the
  expected transaction hash.

**Result Examples**

::

   "a76242fce5753b4212f903ff33ac6fe66f2780f34bdb4b33b175a7815a11a98e"

Protocol version 1.0 returning an error as the result:

::

  "258: txn-mempool-conflict"

blockchain.transaction.broadcast_package
========================================

Broadcast a package of transactions to the network (submitpackage). The package must consist of a child with its parents,
and none of the parents may depend on one another. The package must be topologically sorted,
with the child being the last element in the array.

**Signature**

  .. function:: blockchain.transaction.broadcast_package(raw_txs, verbose=false)

  *raw_txs*

    An array of raw transactions, each as a hexadecimal string.

  *verbose*

    Whether the verbose bitcoind response is required.

**Result**

  If *verbose* is :const:`false`:

    A dictionary with the following keys:

    * `success`
        * Type: bool
        * Value: Indicating the result of the package submission
    * `errors`
        * Type: Optional[List[Dict]]
        * Value: Error message and txid (NOT wtxid) of transactions that were not accepted

  If *verbose* is :const:`true`:

    The bitcoind response according to its RPC API documentation.
    Note that the exact structure and semantics can depend on the bitcoind version,
    and hence the electrum protocol can make no guarantees about it.

**Result Example**

When *verbose* is :const:`false`::

    {
      "success": true
    }

    With errors:

    {
      "success": false,
      "errors":
      [
        {
          "txid": "ec6f295cd4b1b91f59cabb0ab8fdc7c76580db08be6426e465f75a69d82b9659",
          "error": "bad-txns-inputs-missingorspent"
        }
      ]
    }

When *verbose* is :const:`true`::

    {                                   (json object)
      "package_msg" : "str",            (string) The transaction package result message. "success" indicates all transactions were accepted into or are already in the mempool.
      "tx-results" : {                  (json object) transaction results keyed by wtxid
        "wtxid" : {                     (json object) transaction wtxid
          "txid" : "hex",               (string) The transaction hash in hex
          "other-wtxid" : "hex",        (string, optional) The wtxid of a different transaction with the same txid but different witness found in the mempool. This means the submitted transaction was ignored.
          "vsize" : n,                  (numeric, optional) Sigops-adjusted virtual transaction size.
          "fees" : {                    (json object, optional) Transaction fees
            "base" : n,                 (numeric) transaction fee in BTC
            "effective-feerate" : n,    (numeric, optional) if the transaction was not already in the mempool, the effective feerate in BTC per KvB. For example, the package feerate and/or feerate with modified fees from prioritisetransaction.
            "effective-includes" : [    (json array, optional) if effective-feerate is provided, the wtxids of the transactions whose fees and vsizes are included in effective-feerate.
              "hex",                    (string) transaction wtxid in hex
              ...
            ]
          },
          "error" : "str"               (string, optional) The transaction error string, if it was rejected by the mempool
        },
        ...
      },
      "replaced-transactions" : [       (json array, optional) List of txids of replaced transactions
        "hex",                          (string) The transaction id
        ...
      ]
    }

blockchain.transaction.get
==========================

Return a raw transaction.

**Signature**

  .. function:: blockchain.transaction.get(tx_hash, verbose=false)
  .. versionchanged:: 1.1
     ignored argument *height* removed
  .. versionchanged:: 1.2
     *verbose* argument added
  .. versionchanged:: 1.6
     support of *verbose=true* made optional

  *tx_hash*

    The transaction hash as a hexadecimal string.

  *verbose*

    Whether the verbose bitcoind response is required.
    The server MUST support the verbose=false option (which is the default).
    The server CAN decide not to support the verbose=true option, but if so,
    it MUST indicate that in its :func:`server.features` response by setting
    `method_flavours["blockchain.transaction.get"]["supports_verbose_true"]=false`.

**Result**

    If *verbose* is :const:`false`:

       The raw transaction as a hexadecimal string.

    If *verbose* is :const:`true`:

       The result is a bitcoind-specific dictionary -- whatever bitcoind
       returns when asked for a verbose form of the raw transaction.

**Example Results**

When *verbose* is :const:`false`::

  "01000000015bb9142c960a838329694d3fe9ba08c2a6421c5158d8f7044cb7c48006c1b48"
  "4000000006a4730440220229ea5359a63c2b83a713fcc20d8c41b20d48fe639a639d2a824"
  "6a137f29d0fc02201de12de9c056912a4e581a62d12fb5f43ee6c08ed0238c32a1ee76921"
  "3ca8b8b412103bcf9a004f1f7a9a8d8acce7b51c983233d107329ff7c4fb53e44c855dbe1"
  "f6a4feffffff02c6b68200000000001976a9141041fb024bd7a1338ef1959026bbba86006"
  "4fe5f88ac50a8cf00000000001976a91445dac110239a7a3814535c15858b939211f85298"
  "88ac61ee0700"

When *verbose* is :const:`true`::

 {
   "blockhash": "0000000000000000015a4f37ece911e5e3549f988e855548ce7494a0a08b2ad6",
   "blocktime": 1520074861,
   "confirmations": 679,
   "hash": "36a3692a41a8ac60b73f7f41ee23f5c917413e5b2fad9e44b34865bd0d601a3d",
   "hex": "01000000015bb9142c960a838329694d3fe9ba08c2a6421c5158d8f7044cb7c48006c1b484000000006a4730440220229ea5359a63c2b83a713fcc20d8c41b20d48fe639a639d2a8246a137f29d0fc02201de12de9c056912a4e581a62d12fb5f43ee6c08ed0238c32a1ee769213ca8b8b412103bcf9a004f1f7a9a8d8acce7b51c983233d107329ff7c4fb53e44c855dbe1f6a4feffffff02c6b68200000000001976a9141041fb024bd7a1338ef1959026bbba860064fe5f88ac50a8cf00000000001976a91445dac110239a7a3814535c15858b939211f8529888ac61ee0700",
   "locktime": 519777,
   "size": 225,
   "time": 1520074861,
   "txid": "36a3692a41a8ac60b73f7f41ee23f5c917413e5b2fad9e44b34865bd0d601a3d",
   "version": 1,
   "vin": [ {
     "scriptSig": {
       "asm": "30440220229ea5359a63c2b83a713fcc20d8c41b20d48fe639a639d2a8246a137f29d0fc02201de12de9c056912a4e581a62d12fb5f43ee6c08ed0238c32a1ee769213ca8b8b[ALL|FORKID] 03bcf9a004f1f7a9a8d8acce7b51c983233d107329ff7c4fb53e44c855dbe1f6a4",
       "hex": "4730440220229ea5359a63c2b83a713fcc20d8c41b20d48fe639a639d2a8246a137f29d0fc02201de12de9c056912a4e581a62d12fb5f43ee6c08ed0238c32a1ee769213ca8b8b412103bcf9a004f1f7a9a8d8acce7b51c983233d107329ff7c4fb53e44c855dbe1f6a4"
     },
     "sequence": 4294967294,
     "txid": "84b4c10680c4b74c04f7d858511c42a6c208bae93f4d692983830a962c14b95b",
     "vout": 0}],
   "vout": [ { "n": 0,
              "scriptPubKey": { "addresses": [ "12UxrUZ6tyTLoR1rT1N4nuCgS9DDURTJgP"],
                                "asm": "OP_DUP OP_HASH160 1041fb024bd7a1338ef1959026bbba860064fe5f OP_EQUALVERIFY OP_CHECKSIG",
                                "hex": "76a9141041fb024bd7a1338ef1959026bbba860064fe5f88ac",
                                "reqSigs": 1,
                                "type": "pubkeyhash"},
              "value": 0.0856647},
            { "n": 1,
              "scriptPubKey": { "addresses": [ "17NMgYPrguizvpJmB1Sz62ZHeeFydBYbZJ"],
                                "asm": "OP_DUP OP_HASH160 45dac110239a7a3814535c15858b939211f85298 OP_EQUALVERIFY OP_CHECKSIG",
                                "hex": "76a91445dac110239a7a3814535c15858b939211f8529888ac",
                                "reqSigs": 1,
                                "type": "pubkeyhash"},
              "value": 0.1360904}]}

blockchain.transaction.get_merkle
=================================

Return the merkle branch to a confirmed transaction given its hash
and height.

**Signature**

  .. function:: blockchain.transaction.get_merkle(tx_hash, height=None)
  .. versionchanged:: 1.6
     *height* argument made optional (previously mandatory)

  *tx_hash*

    The transaction hash as a hexadecimal string.

  *height*

    Optionally, the height at which it was confirmed, an integer.
    Clients are encouraged to provide this field when they can, to reduce server load.

**Result**

  A dictionary with the following keys:

  * *block_height*

    The height of the block the transaction was confirmed in.

  * *merkle*

    A list of transaction hashes the current hash is paired with,
    recursively, in order to trace up to obtain merkle root of the
    block, deepest pairing first.

  * *pos*

    The 0-based index of the position of the transaction in the
    ordered list of transactions in the block.

**Result Example**

::

  {
    "merkle":
    [
      "713d6c7e6ce7bbea708d61162231eaa8ecb31c4c5dd84f81c20409a90069cb24",
      "03dbaec78d4a52fbaf3c7aa5d3fccd9d8654f323940716ddf5ee2e4bda458fde",
      "e670224b23f156c27993ac3071940c0ff865b812e21e0a162fe7a005d6e57851",
      "369a1619a67c3108a8850118602e3669455c70cdcdb89248b64cc6325575b885",
      "4756688678644dcb27d62931f04013254a62aeee5dec139d1aac9f7b1f318112",
      "7b97e73abc043836fd890555bfce54757d387943a6860e5450525e8e9ab46be5",
      "61505055e8b639b7c64fd58bce6fc5c2378b92e025a02583303f69930091b1c3",
      "27a654ff1895385ac14a574a0415d3bbba9ec23a8774f22ec20d53dd0b5386ff",
      "5312ed87933075e60a9511857d23d460a085f3b6e9e5e565ad2443d223cfccdc",
      "94f60b14a9f106440a197054936e6fb92abbd69d6059b38fdf79b33fc864fca0",
      "2d64851151550e8c4d337f335ee28874401d55b358a66f1bafab2c3e9f48773d"
    ],
    "block_height": 450538,
    "pos": 710
  }

blockchain.transaction.id_from_pos
==================================

Return a transaction hash and optionally a merkle proof,
given a block height and a position in the block.

**Signature**

  .. function:: blockchain.transaction.id_from_pos(height, tx_pos, merkle=false)
  .. versionadded:: 1.4

  *height*

    The main chain block height, a non-negative integer.

  *tx_pos*

    A zero-based index of the transaction in the given block, an integer.

  *merkle*

    Whether a merkle proof should also be returned, a boolean.

**Result**

  If *merkle* is :const:`false`, the transaction hash as a hexadecimal string.
  If :const:`true`, a dictionary with the following keys:

  * *tx_hash*

    The transaction hash as a hexadecimal string.

  * *merkle*

    A list of transaction hashes the current hash is paired with,
    recursively, in order to trace up to obtain merkle root of the
    block, deepest pairing first.

**Example Results**

When *merkle* is :const:`false`::

  "fc12dfcb4723715a456c6984e298e00c479706067da81be969e8085544b0ba08"

When *merkle* is :const:`true`::

  {
    "tx_hash": "fc12dfcb4723715a456c6984e298e00c479706067da81be969e8085544b0ba08",
    "merkle":
    [
      "928c4275dfd6270349e76aa5a49b355eefeb9e31ffbe95dd75fed81d219a23f8",
      "5f35bfb3d5ef2ba19e105dcd976928e675945b9b82d98a93d71cbad0e714d04e",
      "f136bcffeeed8844d54f90fc3ce79ce827cd8f019cf1d18470f72e4680f99207",
      "6539b8ab33cedf98c31d4e5addfe40995ff96c4ea5257620dfbf86b34ce005ab",
      "7ecc598708186b0b5bd10404f5aeb8a1a35fd91d1febbb2aac2d018954885b1e",
      "a263aae6c470b9cde03b90675998ff6116f3132163911fafbeeb7843095d3b41",
      "c203983baffe527edb4da836bc46e3607b9a36fa2c6cb60c1027f0964d971b29",
      "306d89790df94c4632d652d142207f53746729a7809caa1c294b895a76ce34a9",
      "c0b4eff21eea5e7974fe93c62b5aab51ed8f8d3adad4583c7a84a98f9e428f04",
      "f0bd9d2d4c4cf00a1dd7ab3b48bbbb4218477313591284dcc2d7ca0aaa444e8d",
      "503d3349648b985c1b571f59059e4da55a57b0163b08cc50379d73be80c4c8f3"
    ]
  }

mempool.get_fee_histogram
=========================

Return a histogram of the fee rates paid by transactions in the memory
pool, weighted by transaction size.

**Signature**

  .. function:: mempool.get_fee_histogram()
  .. versionadded:: 1.2

**Result**

  The histogram is an array of [*fee*, *vsize*] pairs, where |vsize_n|
  is the cumulative virtual size of mempool transactions with a fee rate
  in the interval [|fee_n1|, |fee_n|], and |fee_n1| > |fee_n|.

  .. |vsize_n| replace:: vsize\ :sub:`n`
  .. |fee_n| replace:: fee\ :sub:`n`
  .. |fee_n1| replace:: fee\ :sub:`n-1`

  Fee intervals may have variable size.  The choice of appropriate
  intervals is currently not part of the protocol.

  *fee* uses sat/vbyte as unit, and must be a non-negative integer or float.

  *vsize* uses vbyte as unit, and must be a non-negative integer.

**Example Results**

::

    [[12, 128812], [4, 92524], [2, 6478638], [1, 22890421]]

::

   [[59.5, 30324], [40.1, 34305], [35.0, 38459], [29.3, 41270], [27.0, 45167], [24.3, 53512], [22.9, 53488], [21.8, 70279], [20.0, 65328], [18.2, 72180], [18.1, 5254], [18.0, 191579], [16.5, 103640], [15.7, 106715], [15.1, 141776], [14.0, 183261], [13.5, 166496], [11.8, 166050], [11.1, 242436], [9.2, 184043], [7.1, 202137], [5.2, 222011], [4.8, 344788], [4.6, 17101], [4.5, 1696864], [4.1, 598001], [4.0, 32688687], [3.9, 505192], [3.8, 38417], [3.7, 2944970], [3.3, 693364], [3.2, 726373], [3.1, 308878], [3.0, 11884957], [2.6, 996967], [2.3, 822802], [2.2, 9075547], [2.1, 12149801], [2.0, 16387874], [1.4, 873120], [1.3, 3493364], [1.1, 2302460], [1.0, 23204633]]


server.add_peer
===============

A newly-started server uses this call to get itself into other servers'
peers lists.  It should not be used by wallet clients.

**Signature**

  .. function:: server.add_peer(features)

  .. versionadded:: 1.1

  * *features*

    The same information that a call to the sender's
    :func:`server.features` RPC call would return.

**Result**

  A boolean indicating whether the request was tentatively accepted.
  The requesting server will appear in :func:`server.peers.subscribe`
  when further sanity checks complete successfully.


server.banner
=============

Return a banner to be shown in the Electrum console.

**Signature**

  .. function:: server.banner()

**Result**

  A string.

**Example Result**

  ::

     "Welcome to Electrum!"


server.donation_address
=======================

Return a server donation address.

**Signature**

  .. function:: server.donation_address()

**Result**

  A string.

**Example Result**

  ::

     "1BWwXJH3q6PRsizBkSGm2Uw4Sz1urZ5sCj"


server.features
===============

Return a list of features and services supported by the server.

**Signature**

  .. function:: server.features()
  .. versionchanged:: 1.6
     added *method_flavours* field to result

**Result**

  A dictionary of keys and values.  Each key represents a feature or
  service of the server, and the value gives additional information.

  The following features MUST be reported by the server.  Additional
  key-value pairs may be returned.

  * *hosts*

    A dictionary, keyed by host name, that this server can be reached
    at.  Normally this will only have a single entry; other entries
    can be used in case there are other connection routes (e.g. Tor).

    The value for a host is itself a dictionary, with the following
    optional keys:

    * *ssl_port*

      An integer.  Omit or set to :const:`null` if SSL connectivity
      is not provided.

    * *tcp_port*

      An integer.  Omit or set to :const:`null` if TCP connectivity is
      not provided.

    A server should ignore information provided about any host other
    than the one it connected to.

  * *genesis_hash*

    The hash of the genesis block.  This is used to detect if a peer
    is connected to one serving a different network.

  * *hash_function*

    The hash function the server uses for :ref:`script hashing
    <script hashes>`.  The client must use this function to hash
    pay-to-scripts to produce script hashes to send to the server.
    The default is "sha256".  "sha256" is currently the only
    acceptable value.

  * *server_version*

    A string that identifies the server software.  Should be the same
    as the first element of the result to the :func:`server.version` RPC call.

  * *protocol_max*
  * *protocol_min*

    Strings that are the minimum and maximum Electrum protocol
    versions this server speaks.  Example: "1.1".

  * *pruning*

    An integer, the pruning limit.  Omit or set to :const:`null` if
    there is no pruning limit.  Should be the same as what would
    suffix the letter ``p`` in the IRC real name.

  * *method_flavours*

    A dictionary that describes whether optional features of certain protocol methods
    are supported by the server. The server might also require an otherwise optional
    argument to be set by the client, that too should be clearly advertised here.
    The keys are protocol method name strings, and the values are dictionaries
    that are specific to the given protocol method.

    If a server supports all functionality defined for the negotiated protocol version,
    it can just set this to the empty dict (but the `method_flavours` key itself
    must always be present).

**Example Result**

::

  {
      "genesis_hash": "000000000933ea01ad0ee984209779baaec3ced90fa3f408719526f8d77f4943",
      "hosts": {"14.3.140.101": {"tcp_port": 51001, "ssl_port": 51002}},
      "protocol_max": "1.0",
      "protocol_min": "1.0",
      "pruning": null,
      "server_version": "ElectrumX 1.0.17",
      "hash_function": "sha256",
      "method_flavours": {
          "blockchain.outpoint.subscribe": {"requires_spk_hint": true},
          "blockchain.transaction.get": {"supports_verbose_true": false}
      }
  }


server.peers.subscribe
======================

Return a list of peer servers.  Despite the name this is not a
subscription and the server must send no notifications.

**Signature**

  .. function:: server.peers.subscribe()

**Result**

  An array of peer servers, each returned as a 3-element array.  For
  example::

    ["107.150.45.210",
     "e.anonyhost.org",
     ["v1.0", "p10000", "t", "s995"]]

  The first element is the IP address, the second is the host name
  (which might also be an IP address), and the third is a list of
  server features.  Each feature and starts with a letter.  'v'
  indicates the server maximum protocol version, 'p' its pruning limit
  and is omitted if it does not prune, 't' is the TCP port number, and
  's' is the SSL port number.  If a port is not given for 's' or 't'
  the default port for the coin network is implied.  If 's' or 't' is
  missing then the server does not support that transport.

server.ping
===========

Ping the server to ensure it is responding, and to keep the session
alive.  The server may disconnect clients that have sent no requests
for roughly 10 minutes.

**Signature**

  .. function:: server.ping()
  .. versionadded:: 1.2

**Result**

  Returns :const:`null`.

server.version
==============

Identify the client to the server and negotiate the protocol version.
This must be the first message sent on the wire.
Only the first :func:`server.version` message is accepted.

**Signature**

  .. function:: server.version(client_name="", protocol_version="1.4")

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

**Example**::

  server.version("Electrum 3.0.6", ["1.1", "1.2"])

**Example Result**::

  ["ElectrumX 1.2.1", "1.2"]


Some more stuff for altcoins
============================

:ref:`Protocol Methods for altcoins`
