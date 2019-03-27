.. role:: raw-html-m2r(raw)
   :format: html

.. _stream:

Stream
======

Specification
-------------

| The Stream interface is a simple handshake protocol to carry a payload.
| It can be used, for example, to push and pop elements into a FIFO, send requests to a UART controller, or something similar.

.. list-table::
   :header-rows: 1
   :widths: 1 1 1 10 1

   * - Signal
     - Type
     - Driver
     - Description
     - Don't care when
   * - valid
     - Bool
     - Master
     - When high => payload present on the interface
     - 
   * - ready
     - Bool
     - Slave
     - When low => transactions are not consumed by the slave
     - valid is low
   * - payload
     - T
     - Master
     - Content of the transaction
     - valid is low

.. wavedrom::

   { signal: [
     {name:'clk',      wave: 'p.........' },
     {name: 'valid'  , wave: '0101..01.0'},
     {name: 'ready'  , wave: 'x1x0.1x1.x'},
     {name: 'payload', wave: 'x=x=..x==x',data:['D0','D1','D2','D3']},
   ]}

Here are some examples of its usage in SpinalHDL:

.. code-block:: scala

   class StreamFifo[T <: Data](dataType: T, depth: Int) extends Component {
     val io = new Bundle {
       val push = slave Stream (dataType)
       val pop = master Stream (dataType)
     }
     ...
   }

   class StreamArbiter[T <: Data](dataType: T,portCount: Int) extends Component {
     val io = new Bundle {
       val inputs = Vec(slave Stream (dataType),portCount)
       val output = master Stream (dataType)
     }
     ...
   }

.. note::
   Each slave may or may not allow the payload to change when valid is high and ready is low. For example:


* A priority arbiter without lock logic can switch from one input to the other (which will change the payload).
* A UART controller could directly use the payload port signal to drive the UART TX pins and only consume the transaction at the end of the transmission.

Be mindful of which kind of compnent you're using.

Functions
---------

.. list-table::
   :header-rows: 1
   :widths: 5 5 1 1

   * - Syntax
     - Description
     - Return
     - Latency
   * - Stream(type : Data)
     - Create a Stream of a given type
     - Stream[T]
     - 
   * - master/slave Stream(type : Data)
     - | Create a Stream of a given type
       | Initialized with corresponding in/out setup
     - Stream[T]
     - 
   * - x.fire
     - Return True when a transaction is consumed on the bus (valid && ready)
     - Bool
     - 
   * - x.isStall
     - Return True when a transaction is stall on the bus (valid && ! ready)
     - Bool
     - 
   * - x.queue(size:Int)
     - Return a Stream connected to x through a FIFO
     - Stream[T]
     - 2
   * - | x.m2sPipe()
       | x.stage()
     - | Return a Stream driven by x
       | through a register stage that cuts the ``valid``/``payload`` paths
       | Cost = (payload width + 1) flop flop
     - Stream[T]
     - 1
   * - x.s2mPipe()
     - | Return a Stream driven by x
       | where the ``ready`` signal path is cut by a register stage
       | Cost = payload width * (mux2 + 1 flip flop)
     - Stream[T]
     - 0
   * - x.halfPipe()
     - | Return a Stream driven by x
       | valid/ready/payload paths are cut by registers
       | Cost = (payload width + 2) flip flop, bandwidth divided by two
     - Stream[T]
     - 1
   * - | x << y
       | y >> x
     - Connect y to x
     - 
     - 0
   * - | x <-< y
       | y >-> x
     - Connect y to x through a m2sPipe
     - 
     - 1
   * - | x </< y
       | y >/> x
     - Connect y to x through a s2mPipe
     - 
     - 0
   * - | x <-/< y
       | y >/-> x
     - | Connect y to x through s2mPipe().m2sPipe()
       | Which implies no combinatorial path between x and y
     - 
     - 1
   * - x.haltWhen(cond : Bool)
     - | Return a Stream connected to x that is 
       | halted when cond is True
     - Stream[T]
     - 0
   * - x.throwWhen(cond : Bool)
     - | Return a Stream connected to x
       | where transactions are dropped when cond is True
     - Stream[T]
     - 0


The following code will create this circuit:

.. image:: /asset/picture/stream_throw_m2spipe.svg
   :align: center

.. code-block:: scala

   case class RGB(channelWidth : Int) extends Bundle{
     val red   = UInt(channelWidth bit)
     val green = UInt(channelWidth bit)
     val blue  = UInt(channelWidth bit)

     def isBlack : Bool = red === 0 && green === 0 && blue === 0
   }

   val source = Stream(RGB(8))
   val sink   = Stream(RGB(8))
   sink <-< source.throwWhen(source.payload.isBlack)

Utilities
---------

There are many utilties that you can use in your design in conjunction with the Stream bus which are documented here.

StreamFifo
^^^^^^^^^^

On any stream you can call the .queue(size) to create a buffered stream. But you can also instantiate the FIFO component itself by doing the following:

.. code-block:: scala

   val streamA,streamB = Stream(Bits(8 bits))
   //...
   val myFifo = StreamFifo(
     dataType = Bits(8 bits),
     depth    = 128
   )
   myFifo.io.push << streamA
   myFifo.io.pop  >> streamB

.. list-table::
   :header-rows: 1
   :widths: 1 1 2

   * - parameter name
     - Type
     - Description
   * - dataType
     - T
     - Payload data type
   * - depth
     - Int
     - Size of the memory used to store elements


.. list-table::
   :header-rows: 1
   :widths: 1 4 5

   * - io name
     - Type
     - Description
   * - push
     - Stream[T]
     - Used to push elements
   * - pop
     - Stream[T]
     - Used to pop elements
   * - flush
     - Bool
     - Used to remove all elements inside the FIFO
   * - occupancy
     - UInt of log2Up(depth + 1) bits
     - Indicates the internal memory occupancy


StreamFifoCC
^^^^^^^^^^^^

You can instantiate a dual clock version of the fifo by doing the following:

.. code-block:: scala

   val clockA = ClockDomain(???)
   val clockB = ClockDomain(???)
   val streamA,streamB = Stream(Bits(8 bits))
   //...
   val myFifo = StreamFifoCC(
     dataType  = Bits(8 bits),
     depth     = 128,
     pushClock = clockA,
     popClock  = clockB
   )
   myFifo.io.push << streamA
   myFifo.io.pop  >> streamB

.. list-table::
   :header-rows: 1
   :widths: 1 1 2

   * - parameter name
     - Type
     - Description
   * - dataType
     - T
     - Payload data type
   * - depth
     - Int
     - Size of the memory used to store elements
   * - pushClock
     - ClockDomain
     - Clock domain used by the push side
   * - popClock
     - ClockDomain
     - Clock domain used by the pop side


.. list-table::
   :header-rows: 1
   :widths: 1 4 5

   * - io name
     - Type
     - Description
   * - push
     - Stream[T]
     - Used to push elements
   * - pop
     - Stream[T]
     - Used to pop elements
   * - pushOccupancy
     - UInt of log2Up(depth + 1) bits
     - Indicates the internal memory occupancy (from the push side perspective)
   * - popOccupancy
     - UInt of log2Up(depth + 1) bits
     - Indicates the internal memory occupancy  (from the pop side perspective)


StreamCCByToggle
^^^^^^^^^^^^^^^^

This component crosses a Stream across two clock domains by using synchronizers and toggling signals on either clock domain. This way of crossing a clock domain uses less area (or bram on an FPGA) than the FIFO method above, but also has a reduced bandwidth

.. code-block:: scala

   val clockA = ClockDomain(???)
   val clockB = ClockDomain(???)
   val streamA,streamB = Stream(Bits(8 bits))
   //...
   val bridge = StreamCCByToggle(
     dataType    = Bits(8 bits),
     inputClock  = clockA,
     outputClock = clockB
   )
   bridge.io.input  << streamA
   bridge.io.output >> streamB

.. list-table::
   :header-rows: 1
   :widths: 1 1 2

   * - parameter name
     - Type
     - Description
   * - dataType
     - T
     - Payload data type
   * - inputClock
     - ClockDomain
     - Clock domain used by the push side
   * - outputClock
     - ClockDomain
     - Clock domain used by the pop side


.. list-table::
   :header-rows: 1
   :widths: 1 1 2

   * - io name
     - Type
     - Description
   * - input
     - Stream[T]
     - Used to push elements
   * - output
     - Stream[T]
     - Used to pop elements


You can also use this shorter syntax which converts an existing Stream into a cross clock domain version:

.. code-block:: scala

   val clockA = ClockDomain(???)
   val clockB = ClockDomain(???)
   val streamA = Stream(Bits(8 bits))
   val streamB = StreamCCByToggle(
     input       = streamA,
     inputClock  = clockA,
     outputClock = clockB
   )

StreamArbiter
^^^^^^^^^^^^^

When you have multiple Streams and you want to arbitrate their access to a single stream, you can use the StreamArbiterFactory.

.. code-block:: scala

   val streamA, streamB, streamC = Stream(Bits(8 bits))
   val arbitredABC = StreamArbiterFactory.roundRobin.onArgs(streamA, streamB, streamC)

   val streamD, streamE, streamF = Stream(Bits(8 bits))
   val arbitredDEF = StreamArbiterFactory.lowerFirst.noLock.onArgs(streamD, streamE, streamF)

.. list-table::
   :header-rows: 1
   :widths: 1 5

   * - Arbitration functions
     - Description
   * - lowerFirst
     - Lower ports have priority over higher ports
   * - roundRobin
     - Fair round robin arbitration
   * - sequentialOrder
     - | Could be used to retrieve transactions in a sequential order
       | First transaction should come from port zero, then from port one, ...


.. list-table::
   :header-rows: 1
   :widths: 1 5

   * - Lock functions
     - Description
   * - noLock
     - The port selection could change every cycle, even if the transaction on the selected port is not consumed.
   * - transactionLock
     - The port selection is locked until the transaction on the selected port is consumed.
   * - fragmentLock
     - | Could be used to arbitrate Stream[Flow[T]].
       | In this mode, the port selection is locked until the selected port finishes its burst (last=True).


.. list-table::
   :header-rows: 1
   :widths: 2 1

   * - Generation functions
     - Return
   * - on(inputs : Seq[Stream[T]])
     - Stream[T]
   * - onArgs(inputs : Stream[T]*)
     - Stream[T]


StreamFork
^^^^^^^^^^

This utility takes its input stream and duplicates it ``outputCount`` times.

.. code-block:: scala

   val inputStream = Stream(Bits(8 bits))
   val dispatchedStreams = StreamDispatcherSequencial(
     input = inputStream,
     outputCount = 3
   )

StreamDispatcherSequencial
^^^^^^^^^^^^^^^^^^^^^^^^^^

This utility takes its input stream and routes it to ``outputCount`` streams in a sequential order.

.. code-block:: scala

   val inputStream = Stream(Bits(8 bits))
   val dispatchedStreams = StreamDispatcherSequencial(
     input = inputStream,
     outputCount = 3
   )
