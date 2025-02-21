================
Protocol Changes
================

This document lists changes made by protocol version.

Version 1.0
===========

Deprecated methods
------------------

  * :func:`blockchain.utxo.get_address`
  * :func:`blockchain.numblocks.subscribe`

.. _version 1.1:

Version 1.1
===========

Changes
-------

  * improved semantics of :func:`server.version` to aid protocol
    negotiation, and a changed return value.
  * :func:`blockchain.transaction.get` no longer takes the *height*
    argument that was ignored anyway.
  * :func:`blockchain.transaction.broadcast` returns errors like any
    other JSON RPC call.  A transaction hash result is only returned on
    success.

New methods
-----------

  * :func:`blockchain.scripthash.get_balance`
  * :func:`blockchain.scripthash.get_history`
  * :func:`blockchain.scripthash.get_mempool`
  * :func:`blockchain.scripthash.listunspent`
  * :func:`blockchain.scripthash.subscribe`
  * :func:`server.features`
  * :func:`server.add_peer`

Removed methods
---------------

  * :func:`blockchain.utxo.get_address`
  * :func:`blockchain.numblocks.subscribe`

.. _version 1.2:

Version 1.2
===========

Changes
-------

  * :func:`blockchain.transaction.get` now has an optional parameter
    *verbose*.
  * :func:`blockchain.headers.subscribe` now has an optional parameter
    *raw*.
  * :func:`server.version` should not be used for "ping" functionality;
    use the new :func:`server.ping` method instead.

New methods
-----------

  * :func:`blockchain.block.headers`
  * :func:`mempool.get_fee_histogram`
  * :func:`server.ping`

Deprecated methods
------------------

  * :func:`blockchain.block.get_chunk`.  Switch to
    :func:`blockchain.block.headers`
  * :func:`blockchain.address.get_balance`.  Switch to
    :func:`blockchain.scripthash.get_balance`.
  * :func:`blockchain.address.get_history`.  Switch to
    :func:`blockchain.scripthash.get_history`.
  * :func:`blockchain.address.get_mempool`.  Switch to
    :func:`blockchain.scripthash.get_mempool`.
  * :func:`blockchain.address.listunspent`.  Switch to
    :func:`blockchain.scripthash.listunspent`.
  * :func:`blockchain.address.subscribe`.  Switch to
    :func:`blockchain.scripthash.subscribe`.
  * :func:`blockchain.headers.subscribe` with *raw* other than :const:`True`.

.. _version 1.3:

Version 1.3
===========

Changes
-------

  * :func:`blockchain.headers.subscribe` argument *raw* switches default to
    :const:`True`

New methods
-----------

  * :func:`blockchain.block.header`

Removed methods
---------------

  * :func:`blockchain.address.get_balance`
  * :func:`blockchain.address.get_history`
  * :func:`blockchain.address.get_mempool`
  * :func:`blockchain.address.listunspent`
  * :func:`blockchain.address.subscribe`

Deprecated methods
------------------

  * :func:`blockchain.block.get_header`.  Switch to
    :func:`blockchain.block.header`.

.. _version 1.4:

Version 1.4
===========

This version removes all support for :ref:`deserialized headers
<deserialized header>`.

Changes
-------

  * Deserialized headers are no longer available, so removed argument
    *raw* from :func:`blockchain.headers.subscribe`.
  * Only the first :func:`server.version` message is accepted.
  * Optional *cp_height* argument added to
    :func:`blockchain.block.header` and :func:`blockchain.block.headers`
    to return merkle proofs of the header to a given checkpoint.

New methods
-----------

  * :func:`blockchain.transaction.id_from_pos` to return a transaction
    hash, and optionally a merkle proof, given a block height and
    position in the block.

Removed methods
---------------

  * :func:`blockchain.block.get_header`
  * :func:`blockchain.block.get_chunk`

Version 1.4.1
=============

Changes
-------

  * :func:`blockchain.block.header` and :func:`blockchain.block.headers` now
    truncate AuxPoW data (if using an AuxPoW chain) when *cp_height* is
    nonzero.  AuxPoW data is still present when *cp_height* is zero.
    Non-AuxPoW chains are unaffected.


Version 1.4.2
=============

New methods
-----------

  * :func:`blockchain.scripthash.unsubscribe` to unsubscribe from a script hash.


Version 1.4.3
=============

New methods
-----------

  * :func:`blockchain.name.get_value_proof` to resolve a name (with proof).  Name index coins (e.g. Namecoin) only.


Version 1.5
===========

(this version number was skipped, no corresponding protocol is defined)


.. _version 1.6:

Version 1.6
===========

Changes
-------

  * Breaking change for the version negotiation: we now mandate that
    the :func:`server.version` message must be the first message sent.
    That is, version negotiation must happen before any other messages.
  * The status of a scripthash has its definition tightened in a
    backwards-compatible way: mempool txs now have a canonical ordering
    defined for the calculation (previously their order was undefined).
  * :func:`blockchain.scripthash.get_mempool` previously did not define
    an order for mempool transactions. We now mandate a specific ordering.
  * For :func:`blockchain.transaction.get_merkle`, the previously required
    *height* argument is now optional, and the result now includes a
    *block_hash* field.
  * Optional *mode* argument added to :func:`blockchain.estimatefee`.
  * :func:`blockchain.block.headers` now returns headers as a list,
    instead of a single concatenated hex string.
  * Support for *cp_height* in :func:`blockchain.block.header` and
    :func:`blockchain.block.headers` has been made optional.
  * Support for :const:`verbose=true` in :func:`blockchain.transaction.get`
    has been made optional.
  * :func:`server.features` now has a new mandatory field, *method_flavours*,
    which aims to provide some clarity re whether the server supports optional features.
  * :func:`server.ping` can now be sent by both the client and the server, and both
    parties must handle and respond to it. The request/response signature also changed.


New methods
-----------

  * :func:`blockchain.outpoint.subscribe` to subscribe to a transaction
    outpoint, and get a notification when it gets spent.
  * :func:`blockchain.outpoint.unsubscribe` to unsubscribe from a TXO.
  * :func:`blockchain.outpoint.get_status` to get current status of a TXO, without subscribing to changes.
  * :func:`blockchain.transaction.broadcast_package` to broadcast a package of transactions (submitpackage).
