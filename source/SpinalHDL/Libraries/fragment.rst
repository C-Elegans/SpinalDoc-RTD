
Fragment
========

Specification
-------------

The ``Fragment`` bundle is used to transmit a "big" thing by using multiple smaller fragments. For example:


* A picture transmitted with a width*height transaction on a ``Stream[Fragment[Pixel]]``
* An UART packet received from a controller without flow control could be transmitted on a ``Flow[Fragment[Bits]]``
* An AXI read burst could be carried by a ``Stream[Fragment[AxiReadResponse]]``

Signals defined by the ``Fragment`` bundle are:

.. list-table::
   :header-rows: 1
   :widths: 1 1 1 5

   * - Signal
     - Type
     - Driver
     - Description
   * - fragment
     - T
     - Master
     - The "payload" of the current transaction
   * - last
     - Bool
     - Master
     - High when the fragment is the last of the current packet


As you can see from this specification, the ``Fragment`` concept doesn't specify how transactions are transmitted (You can use Stream, Flow, or any other communication protocol). It only adds enough information (\ ``last``\ ) to know whether the current transaction is the first one, the last one, or one in the middle of a given packet.

.. note::
   The protocol doesn't carry a \'first\' bit because it can be calculated at any time by doing \'RegNextWhen(bus.last, bus.fire) init(True)\'

Functions
---------

For ``Stream[Fragment[T]]`` and ``Flow[Fragment[T]]``\ , the following functions are available:

.. list-table::
   :header-rows: 1
   :widths: 1 1 20

   * - Syntax
     - Return
     - Description
   * - x.first
     - Bool
     - Returns True when the next or the current transaction is/will be the first of a packet
   * - x.tail
     - Bool
     - Rseturn True when the next or the current transaction is not/will not be the first of a packet
   * - x.isFirst
     - Bool
     - Returns True when a transaction is present and is the first of a packet
   * - x.isTail
     - Bool
     - Returns True when a transaction is present and is the not the first/last of a packet
   * - x.isLast
     - Bool
     - Return True when a transaction is present and is the last of a packet


For ``Stream[Fragment[T]]``\ , following function are also accessible :

.. list-table::
   :header-rows: 1
   :widths: 1 1 1

   * - Syntax
     - Return
     - Description
   * - x.insertHeader(header : T)
     - Stream[Fragment[T]]
     - Prepend ``header`` to each packet on ``x`` and return the resulting bus

