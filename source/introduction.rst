
Introduction
============

The Python Science Analysis Toolkit (pysat) is a package providing a simple and flexible interface for downloading, loading, cleaning, managing, processing, and analyzing scientific measurements. Though pysat was initially designed for in-situ satellite based measurements it aims to support all instruments in space science.

Every scientific instrument has unique properties though the general process for science data analysis is independent of platform. Find and download the data, write code to load the data, clean the data, apply custom analysis functions, and plot the results. Pysat provides a framework for this general process that builds upon these commonalities to simplify adding new instruments, reduce data management overhead, and enable instrument independent analysis routines.

This instrument independence is achieved through a pysat.Instrument object which provides a layer of separation between the user and the particulars of any given science data set. Loading data for any instrument becomes the same process with the same result, instrument.load() produces data in instrument.data. Behind the scenes different load functions will actually be called for different instruments, as appropriate, but this is now a hidden implementation detail. 

Frequently science data sets don’t have all of the parameters needed for a given analysis. If an analysis routine is to be truly instrument independent then there needs to be a mechanism to get custom data into a routine without having to modify the routine itself. Thus, a nano-kernel is attached to the pysat.Instrument object. Upon each instrument.load call a queue of user selected custom functions are applied before the data is available in instrument.data. The instrument object is ‘set and forget’, regardless of location the data available in instrument.data will be properly processed.

The final component for instrument independence requires the Python Data Analysis Library (pandas), the underlying data object, capable of handling 1D through nD data in a single structure. Pandas data structures are also indexed, thus math operations between two arrays A and B are aligned before the operation. Measurements across platforms are rarely always at the same time, thus pandas also significantly reduces the overhead while increasing the rigour of inter-platform comparisons.

This document covers installation, a tutorial on pysat including demonstration codes, an overview of adding new instruments to pysat, and an API reference.




 