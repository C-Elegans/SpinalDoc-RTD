=========
Libraries
=========

.. toctree::
   :hidden:

   utils
   stream
   flow
   fragment
   fsm
   vexriscv
   bus_slave_factory
   Bus/index
   Com/index
   IO/index
   Graphics/index
   EDA/index

.. role:: raw-html-m2r(raw)
   :format: html

.. _lib_introduction:

Introduction
============

Introduction
------------
The goals of the spinal.lib package include:


* Provide functions and modules that are commonly used in hardware design (FIFO, clock crossing bridges, useful functions)
* Provide simple peripherals (UART, JTAG, VGA, ..)
* Provide some bus definitions (Avalon, AMBA, ..)
* Provide some standardized interfaces for connecting modules (Stream, Flow, Fragment)
* Provide some examples to show how Spinal can be used
* Provide some tools and facilities to make development easier (latency analyser, QSys converter, ...)

To use most of the features introduced in the following chapters you need to ``import spinal.lib._`` in your sources.

.. important::
   | This package is currently under construction. Only documented features should be considered as stable. 
   | Do not hesitate to use github for suggestions/bug/fixes/enhancements
