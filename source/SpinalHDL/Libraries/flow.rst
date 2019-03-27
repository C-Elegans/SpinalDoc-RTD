
Flow
====

Specification
-------------

The Flow interface is a simple valid/payload protocol which means the slave can't halt the bus.
For example, it could be used to represent data coming from a UART controller, requests to write an on-chip memory, etc.

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
   * - payload
     - T
     - Master
     - Content of the transaction
     - valid is low


Functions
---------

.. list-table::
   :header-rows: 1
   :widths: 1 10 1 1

   * - Syntax
     - Description
     - Return
     - Latency
   * - Flow(type : Data)
     - Create a Flow of a given type
     - Flow[T]
     - 
   * - master/slave Flow(type : Data)
     - | Create a Flow of a given type
       | Initialized with the corresponding in/out setup
     - Flow[T]
     - 
   * - x.m2sPipe()
     - | Return a Flow driven by x
       | through a register stage that cuts the paths of ``valid`` and ``payload`` 
     - Flow[T]
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
   * - x.throwWhen(cond : Bool)
     - | Return a Flow connected to x 
       | that drops transactions when ``cond`` is high
     - Flow[T]
     - 0
   * - x.toReg()
     - Return a register which is loaded with ``payload`` when valid is high
     - T
     - 

