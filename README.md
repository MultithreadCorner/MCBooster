MCBooster
=========

What is it?
-----------
MCBooster is an header only library designed for the fast generation of
phase space events. The library makes use of Thrust and can deploy OpenMP
threads, CUDA and Xeon Phi cores. It is focused on performance and precision.  

The Latest Version
------------------

The latest version can be found on the 
[project relases page](https://github.com/MultithreadCorner/MCBooster/releases).

Documentation
-------------

The complete and updated [Doxygen](http://www.doxygen.org/) source code documentation of this release is available in HTML format on the
[reference manual](http://multithreadcorner.github.io/MCBooster/) webpage.
Users can also browse the documentation by class, file or name using the following links:

1.[classes](http://multithreadcorner.github.io/MCBooster/classes.html)

2.[files](http://multithreadcorner.github.io/MCBooster/files.html)

3.[names](http://multithreadcorner.github.io/MCBooster/namespacemembers.html)


Examples
--------

Some example code samples demonstrating the basic usage of the library are stored in the src directory. 
These samples can be built using [CMAKE](https://cmake.org/) according the following instructions:

1. clone the git repository: __`git clone https://github.com/MultithreadCorner/MCBooster.git`__
2. go to MCBooster directory: __`cd MCBooster`__
3. create a build directory: __`mkdir build`__
4. go to build directory: __`cd build`__
4. __`cmake ../`__
5. __`make`__

Users with root privilegies can do `make install` and get the targets installed into system-install-dir/bin (usually /usr/local. __Notice that the project installation path is printed out in the setp 4__). Users without root privileges can point the installation path to a different location doing __`cmake -DCMAKE_INSTALL_PREFIX=<user-path>/ ../`__.
To run an example do ./example-name

Installation and requirements 
-----------------------------

MCBooster is a header only library, so no build process is necessary to install it.
The library run on Linux systems and requires C++11 and the [Thrust library](https://thrust.github.io/). The code samples require [ROOT](https://root.cern.ch/) and [TCLAP](http://tclap.sourceforge.net/) library. 
CUDA based projects will require a local installation of [CUDA Tookit](https://developer.nvidia.com/cuda-toolkit) with version 6.5 or higher.  
Alternatively, projects the targeting [OpenMP](http://openmp.org/wp/) backend can be compiled with nvcc or gcc directly. 

Licensing
---------

MCBooster is released under the [GNU General Public License version 3](http://www.gnu.org/licenses/gpl-3.0.en.html). Please see the file called [COPYING](https://github.com/MultithreadCorner/MCBooster/blob/master/COPYING).

Contact the developers
----------------------
Here’s what you should do if you need help or would like to contribute:

1. If you need help or would like to ask a general question, subscribe and use https://groups.google.com/d/forum/mcbooster.
2. If you found a bug, use GitHub issues.
3. If you have an idea, use GitHub issues.
4. If you want to contribute, submit a pull request.

Author
--------

MCBooster is developed and mantained by [Antonio Augusto Alves Jr](@AAAlvesJr).

Acknowledgement
---------------

MCBooster's development has been supported by the [National Science Foundation](http://nsf.gov/index.jsp) under grant number [PHY-1414736](http://nsf.gov/awardsearch/showAward?AWD_ID=1414736). Any opinions, findings, and conclusions or recommendations expressed in this material are those of the developers and do not necessarily reflect the views of the National Science Foundation.
