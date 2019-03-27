.. role:: raw-html-m2r(raw)
   :format: html

.. _state_machine:

State machine
=============

Introduction
------------

In SpinalHDL you can define your state machine like as you would VHDL/Verilog, by using enumerations and switch case statements. But in SpinalHDL you can also use a dedicated, more readable syntax.

The following state machine is implemented in the following examples:

.. image:: /asset/picture/fsm_simple.svg
   :align: center
   :width: 300

Style A :

.. code-block:: scala

   import spinal.lib.fsm._

   class TopLevel extends Component {
     val io = new Bundle{
       val result = out Bool
     }

     val fsm = new StateMachine{
       val counter = Reg(UInt(8 bits)) init (0)
       io.result := False

       val stateA : State = new State with EntryPoint{
         whenIsActive (goto(stateB))
       }
       val stateB : State = new State{
         onEntry(counter := 0)
         whenIsActive {
           counter := counter + 1
           when(counter === 4){
             goto(stateC)
           }
         }
         onExit(io.result := True)
       }
       val stateC : State = new State{
         whenIsActive (goto(stateA))
       }
     }
   }

Style B :

.. code-block:: scala

   import spinal.lib.fsm._

   class TopLevel extends Component {
     val io = new Bundle{
       val result = out Bool
     }

     val fsm = new StateMachine{
       val stateA = new State with EntryPoint
       val stateB = new State
       val stateC = new State

       val counter = Reg(UInt(8 bits)) init (0)
       io.result := False

       stateA
         .whenIsActive (goto(stateB))

       stateB
         .onEntry(counter := 0)
         .whenIsActive {
           counter := counter + 1
           when(counter === 4){
             goto(stateC)
           }
         }
         .onExit(io.result := True)

       stateC
         .whenIsActive (goto(stateA))
     }
   }

StateMachine
------------

StateMachine is the base class that will manage the logic of your FSM.

.. code-block:: scala

   val myFsm = new StateMachine{
     // state definitions go here
   }

The StateMachine class also provide some utilities:

.. list-table::
   :header-rows: 1
   :widths: 1 1 5

   * - Name
     - Return
     - Description
   * - isActive(state)
     - Bool
     - Returns True when the state machine is in the given state
   * - isEntering(state)
     - Bool
     - Returns True when the state machine is entering the given state


States
------

There are multiple kinds of states available for you to use.


* State (the base class)
* StateDelay
* StateFsm
* StateParallelFsm

In each form, you have access to the following utilities:

.. list-table::
   :header-rows: 1
   :widths: 1 10

   * - Name
     - Description
   * - | onEntry{
       |  yourStatements
       | }
     - yourStatements is executed the cycle before entering the state
   * - | onExit{
       |  yourStatements
       | }
     - yourStatements is executed when the state machine will be in another state the next cycle
   * - | whenIsActive{
       |  yourStatements
       | }
     - yourStatements is executed when the state machine is in the current state
   * - | whenIsNext{
       |  yourStatements
       | }
     - yourStatements is executed when the state machine will be in this state on the next cycle
   * - goto(nextState)
     - Set the state of the state machine by nextState
   * - exit()
     - Set the state of the state machine to the boot one


For example, the following state machine could be defined in SpinalHDL by using the following syntax :

.. image:: /asset/picture/fsm_stateb.svg
   :align: center
   :width: 300

.. code-block:: scala

   val stateB : State = new State{
     onEntry(counter := 0)
     whenIsActive {
       counter := counter + 1
       when(counter === 4){
         goto(stateC)
       }
     }
     onExit(io.result := True)
   }

You can also define your state as the entry point of the state machine by extends the ``EntryPoint`` trait.

.. code-block:: scala

   val stateA: State = new State with EntryPoint {
     whenIsActive {
       goto(stateB)
     }
   }

StateDelay
^^^^^^^^^^

StateDelay allows you to create a state which waits a fixed number of cycles before executing statments in your ``whenCompleted{...}`` block. The standard way to write it is:

.. code-block:: scala

   val stateG : State = new StateDelay(cyclesCount=40){
     whenCompleted{
       goto(stateH)
     }
   }

But it can also be written like this:

.. code-block:: scala

   val stateG : State = new StateDelay(40){whenCompleted(goto(stateH))}

StateFsm
^^^^^^^^

StateFsm Allow you to describe a state which contains a nested state machine. When the nested state machine is done, your statments in ``whenCompleted{...}`` are executed.

There is an example of StateFsm definition :

.. code-block:: scala

   val stateC = new StateFsm(fsm=internalFsm()){
     whenCompleted{
       goto(stateD)
     }
   }

As you can see in the preceeding code, it uses a ``internalFsm`` function to create the inner state machine. An example of this kind of function is included below:

.. code-block:: scala

   def internalFsm() = new StateMachine {
     val counter = Reg(UInt(8 bits)) init (0)

     val stateA: State = new State with EntryPoint {
       whenIsActive {
         goto(stateB)
       }
     }

     val stateB: State = new State {
       onEntry (counter := 0)
       whenIsActive {
         when(counter === 4) {
           exit()
         }
         counter := counter + 1
       }
     }
   }

In the preceeding example, the ``exit()`` call will make the state machine jump to the boot state (a internal hidden state). This notifies the StateFsm that the inner state machine has been completed.

StateParallelFsm
^^^^^^^^^^^^^^^^

This state is able to handle multiple parallel state machines. When all child state machines are done, your statments in ``whenCompleted{...}`` are executed.

Here is an example of its declaration:

.. code-block:: scala

   val stateD = new StateParallelFsm (internalFsmA(), internalFsmB()){
     whenCompleted{
       goto(stateE)
     }
   }
