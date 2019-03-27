.. role:: raw-html-m2r(raw)
   :format: html

.. _bus_slave_factory:

Bus Slave Factory
=================

Introduction
------------

In many situations a bank of registers needs to be connected to a bus. The ``BusSlaveFactory`` is a tool that provides an abstract and easy way to generate a bus interface.

To see the capabilities of the tool, an simple example uses the Apb3SlaveFactory variation to implement a :ref:`memory mapped UART <memory_mapped_uart>`. There is also another example of a :ref:`Timer <timer>` which contains a memory mapped function.

You can find more documentation about the internal implementation of the ``BusSlaveFactory`` tool :ref:`here <bus_slave_factory_implementation>`

Functionality
-------------

Currently there are three implementations of the ``BusSlaveFactory`` tool : APB3, AXI-lite 3, and Avalon. 
Each implementation of the tool takes one instance of the corresponding bus as an argument and offers the following functions to map your hardware onto the memory map:

.. list-table::
   :header-rows: 1
   :widths: 2 1 10

   * - Name
     - Return
     - Description
   * - busDataWidth
     - Int
     - Return the data width of the bus
   * - read(that,address,bitOffset)
     - 
     - When the bus reads from ``address``\ , fill the response with ``that`` at ``bitOffset``
   * - write(that,address,bitOffset)
     - 
     - When the bus writes to ``address``\ , assign ``that`` data from the bus at ``bitOffset``
   * - onWrite(address)(doThat)
     - 
     - Call ``doThat`` when a write transaction occurs at ``address``
   * - onRead(address)(doThat)
     - 
     - Call ``doThat`` when a read transaction occurs at ``address``
   * - nonStopWrite(that,bitOffset)
     - 
     - Permanently assign ``that`` to the bus's data at ``bitOffset``
   * - readAndWrite(that,address,bitOffset)
     - 
     - Make ``that`` readable and writeable at ``address`` and placed at ``bitOffset`` in the word
   * - readMultiWord(that,address)
     - 
     - | Create a memory mapping to read ``that`` from ``address``. 
       | If ``that`` is bigger than one word it extends the register to subsequent addresses
   * - writeMultiWord(that,address)
     - 
     - | Create a memory mapping to write ``that`` at 'address'. 
       | If ``that`` is bigger than one word it extends the register to subsequent addresses
   * - createWriteOnly(dataType,address,bitOffset)
     - T
     - Create a write only register of type ``dataType`` at ``address`` and placed at ``bitOffset`` in the word
   * - createReadWrite(dataType,address,bitOffset)
     - T
     - Create a read write register of type ``dataType`` at ``address`` and placed at ``bitOffset`` in the word
   * - createAndDriveFlow(dataType,address,bitOffset)
     - Flow[T]
     - Create a writeable Flow register of type ``dataType`` at ``address`` and placed at ``bitOffset`` in the word
   * - drive(that,address,bitOffset)
     - 
     - Drive ``that`` with a register writeable at ``address`` placed at ``bitOffset`` in the word
   * - driveAndRead(that,address,bitOffset)
     - 
     - Drive ``that`` with a register readable and writeable at ``address`` placed at ``bitOffset`` in the word
   * - driveFlow(that,address,bitOffset)
     - 
     - Emit a transaction on ``that`` when a write happens at ``address`` by using data placed at ``bitOffset`` in the word
   * - | readStreamNonBlocking(that,
       |                       address,
       |                       validBitOffset,
       |                       payloadBitOffset)
     - 
     - | Read ``that`` and consume the transaction when a read happens at ``address``. 
       | valid <= validBitOffset bit
       | payload <= payloadBitOffset+widthOf(payload) downto ``payloadBitOffset``
   * - | doBitsAccumulationAndClearOnRead(that,
       |                                  address,
       |                                  bitOffset)
     - 
     - | Instantiate an internal register which at each cycle does
       | ``reg := reg | that``
       | Then when a read occurs, the register is cleared. This register is readable at ``address`` and placed at ``bitOffset`` in the word

