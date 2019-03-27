.. role:: raw-html-m2r(raw)
   :format: html

Utils
=====

Some utilities are also present in :ref:`spinal.core <utils>`

State less utilities
--------------------

.. list-table::
   :header-rows: 1
   :widths: 2 1 5

   * - Syntax
     - Return
     - Description
   * - toGray(x : UInt)
     - Bits
     - Return the gray encoded value of x ``x`` (UInt)
   * - fromGray(x : Bits)
     - UInt
     - Return the UInt value converted value from ``x`` (gray)
   * - Reverse(x : T)
     - T
     - Reverse the order of the bits (lsb + n -> msb - n)
   * - | OHToUInt(x : Seq[Bool])
       | OHToUInt(x : BitVector)
     - UInt
     - Return the index of the single bit set (one hot) in ``x``
   * - | CountOne(x : Seq[Bool])
       | CountOne(x : BitVector)
     - UInt
     - Return the number of bits set in ``x``
   * - | MajorityVote(x : Seq[Bool])
       | MajorityVote(x : BitVector)
     - Bool
     - Return True if the number of bits set is > x.size / 2
   * - EndiannessSwap(that: T[, base:BitCount])
     - T
     - Big-Endian <-> Little-Endian
   * - OHMasking.first(x : Bits)
     - Bits
     - Apply a mask on x to keep only the first bit set
   * - OHMasking.last(x : Bits)
     - Bits
     - Apply a mask on x to keep only the last bit set
   * - | OHMasking.roundRobin(
       |  requests : Bits,
       |  ohPriority : Bits
       | )
     - Bits
     - | Apply a mask on x to only keep the bit set from ``requests``.
       | It starts looking in ``requests`` from the ``ohPriority`` position towards the most significant bit with wraparound.
       | For example if ``requests`` is "1001" and ``ohPriority`` is "0010", the the ``roundRobin`` function will start looking for a set bit in ``requests`` from its second bit moving to the left and will return "1000".


Statefull utilities
-------------------

.. list-table::
   :header-rows: 1
   :widths: 3 1 5

   * - Syntax
     - Return
     - Description
   * - Delay(that: T, cycleCount: Int)
     - T
     - Return ``that`` delayed by ``cycleCount`` cycles
   * - History(that: T, length: Int[,when : Bool])
     - List[T]
     - | Return a Vec of ``length`` elements
       | The first element is ``that``\ , the last one is ``that`` delayed by ``length``\ -1\
       | The internal shift register samples when ``when`` is asserted
   * - BufferCC(input : T)
     - T
     - Return the input signal synchronized with the current clock domain by using a 2 flip flop synchronizer


Counter
^^^^^^^

The Counter tool can be used to easly instantiate an hardware counter.

.. list-table::
   :header-rows: 1
   :widths: 1 1

   * - Instanciation syntax
     - Notes
   * - Counter(start: BigInt, end: BigInt[, inc : Bool])
     - 
   * - Counter(range : Ranget[, inc : Bool])
     - Compatible with the  ``x to y`` ``x until y`` syntaxes
   * - Counter(stateCount: BigInt[, inc : Bool])
     - Start at zero and finish at ``stateCount - 1``
   * - Counter(bitCount: BitCount[, inc : Bool])
     - Start at zero and finish at ``(1 << bitCount) - 1``


Here is an example of different syntaxes which could be used with the Counter tool:

.. code-block:: scala

   val counter = Counter(2 to 9)  //Create a counter of 10 states (2 to 9)
   counter.clear()            //When called it ask to reset the counter.
   counter.increment()        //When called it ask to increment the counter.
   counter.value              //current value
   counter.valueNext          //Next value
   counter.willOverflow       //Flag that indicate if the counter overflow this cycle
   counter.willOverflowIfInc  //Flag that indicate if the counter overflow this cycle if an increment is done
   when(counter === 5){ ... }

When a ``Counter`` overflows its end value, it restarts at its start value.

.. note::
   Currently, only up counters are supported.

Timeout
^^^^^^^

The Timeout tool can be used to easily instantiate an hardware timeout.

.. list-table::
   :header-rows: 1
   :widths: 1 1

   * - Instanciation syntax
     - Notes
   * - Timeout(cycles : BigInt)
     - Tick after ``cycles`` clocks
   * - Timeout(time : TimeNumber)
     - Tick with a duration of ``time`` between pulses
   * - Timeout(frequency : HertzNumber)
     - Tick at an ``frequency`` rate


Here is an example of different syntaxes which could be used with the Timeout tool:

.. code-block:: scala

   val timeout = Timeout(10 ms)  //Timeout who tick after 10 ms
   when(timeout){                //Check if the timeout has tick
       timeout.clear()           //Ask the timeout to clear its flag
   }

.. note::
   If you instanciate a ``Timeout`` with a time or frequancy argument, the implicit ``ClockDomain`` should have a frequency setting.

ResetCtrl
^^^^^^^^^

ResetCtrl provides some utilities to manage the behavior of resets.

asyncAssertSyncDeassert
~~~~~~~~~~~~~~~~~~~~~~~

Often times an asynchronous reset needs to be deasserted synchronously with a clock, while keeping its assertion asynchronous. To do this, you can use the ``ResetCtrl.asyncAssertSyncDeassert`` function which will the hardware necessary to synchronously release an asynchronous reset.

.. list-table::
   :header-rows: 1
   :widths: 1 1 4

   * - Argument name
     - Type
     - Description
   * - input
     - Bool
     - Signal that should be filtered
   * - clockDomain
     - ClockDomain
     - ClockDomain which will use the filtered value
   * - inputPolarity
     - Polarity
     - HIGH/LOW (default=HIGH)
   * - outputPolarity
     - Polarity
     - HIGH/LOW (default=clockDomain.config.resetActiveLevel)
   * - bufferDepth
     - Int
     - Number of register stages used to avoid metastability (default=2)


There is also another version of the function called ``ResetCtrl.asyncAssertSyncDeassertDrive``, which directly assigns the current clockDomain's reset signal to the filtred value.

Special utilities
-----------------

.. list-table::
   :header-rows: 1
   :widths: 3 1 5

   * - Syntax
     - Return
     - Description
   * - LatencyAnalysis(paths : Node*)
     - Int
     - | Returns the shortest path, in cycles, that travels through all nodes
       | between the first one and the last one

