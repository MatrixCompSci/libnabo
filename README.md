libnabo is a fast K Nearest Neighbour library for low-dimensional spaces.
It provides a clean, legacy-free, scalar-type–agnostic API thanks to C++ templates.
Its current CPU implementation is strongly inspired by [ANN], but with more compact data types.
On the average, libnabo is 5% to 20% faster than [ANN].

libnabo depends on [Eigen], a modern C++ matrix and linear-algebra library.
libnabo works with either version 2 or 3 of Eigen.
libnabo also depends on [Boost], a C++ general library.

libnabo was developed by [Stéphane Magnenat](http://stephane.magnenat.net) as part of his work at [ASL-ETH](http://www.asl.ethz.ch) and is now maintained by [Simon Lynen](http://www.asl.ethz.ch/people/slynen).

Download
========

Ubuntu builds are available on my PPA at: https://launchpad.net/~stephane.magnenat
They provide a package with the shared library, another with the development headers and a third with the documentation.

The source code is available from github, you can clone the git tree by doing:

	git clone git://github.com/ethz-asl/libnabo.git

You can then checkout a specific branch, for instance to checkout version 1.0.2, do:

	cd libnabo
	git checkout 1.0.2

Compilation
===========

libnabo uses [CMake] as build system.
The complete compilation process depends on the system you are using (Linux, Mac OS X or Windows).
You will find a nice introductory tutorial in [this video](http://www.youtube.com/watch?v=CLvZTyji_Uw).

Prerequisites
-------------

If your operating system does not provide it, you must get [Eigen] and [Boost].
[Eigen] only needs to be downloaded and extracted.
You also need `grep`, which is available in standard on Linux or Mac OS X, you can get the window version [here](http://gnuwin32.sourceforge.net/packages/grep.htm).

Compilation options
-------------------

libnabo provides the following compilation options, available through [CMake]:

 * `SHARED_LIBS` (boolean, default: `false`): if `true`, build a shared library, otherwise build a static library

You can specify them with a command-line tool, `ccmake`, or with a graphical tool, `cmake-gui`.
Please read the [CMake documentation] for more information.

In order to utilize CUDA, please install the latest NVIDIA developer driver, and atleast CUDA 7.0:

  * For Ubuntu or Debian, follow [these instructions](http://docs.nvidia.com/cuda/cuda-getting-started-guide-for-linux/#axzz3X7AKITTy). 
  
  * For Windows, follow [these instructions](http://docs.nvidia.com/cuda/cuda-getting-started-guide-for-microsoft-windows/#axzz3X7AKITTy). Please note that this code has, as of April 12th 2015, NOT been tested on Windows.
  
  * For Mac OSX, follow [these instructions](http://docs.nvidia.com/cuda/cuda-getting-started-guide-for-mac-os-x/#axzz3X7AKITTy). Clang 6.0 is required.

Compute Model 3.2 or better is required. Compute Model 3.5 or better is recommended. To see which compute model you have, click [here](https://developer.nvidia.com/cuda-gpus).

CUDA Best Practices 
-------------------

Query points should be stored in a linearized array, and should be in sub groups of a multiple of 16. No more than 1024 groups should be queried at any given time.

Subgroups should be ordered in regard to their distance from the center of the group in least to greatest. This will insure minimalized thread divergence and greatly improve performance. 

The smaller the overall maximum distance from the center any given point within a subgroup is, the smaller the overall divergence. 

If a path proves to be too divergent, a new kernel will be launched and the query point subgroup will be split. This is suboptimal when compared to a tighly packed subgroup, so avoid it at all costs. It is also only available on Compute Model 3.5. 

Currently, in order to change the dimension could, the kernel must be recompiled. This primarily is due to the simplicity of removing point striding from the in-development version. It will be added at a later date.

In order to call CUDA from libnabo, please create a C to C++ wrapper and properly invoke it from the search function within kdtree_opencl.cpp. I have not created a full C++ wrapper yet for the CUDA code; however, I have plans to eventually do so. 

For optimal performance, restrict BucketSize to 10. Eventually a method to dynamically set BucketSize will be used, as it is highly dependent on the GPU being utilized for the computations.

Double precision is NOT available and I have no intentions of adding it. This code is single precision only. If you wish to utilize doubles, modify it to your liking.

When dynamic parralelism is finalized, tree depth will be limited to 22, as this is the standard limit for Compute Model 3.5. If NVIDIA plans to alleviate this limit for consumer GPUs in the near future, or a work around is discovered, I will remove this limit.

While dynamic parralelism will be used for divergent paths, only 1024 paths will be allowed to launch new kernels at any given point. Therefore, if the number of paths that prove to be divergent is greater than 1024, please optimize or recluster your querry set. A path is considered to be divergent if any given query point is outside of the KNN max_rad2 of the center of the cluster. 
Quick compilation and installation under Unix
---------------------------------------------

Under Unix, assuming that [Eigen] and [Boost] are installed system-wide, you can compile (with optimisation and debug information) and install libnabo in `/usr/local` with the following commands run in the top-level directory of libnabo's sources:

	SRC_DIR=`pwd`
	BUILD_DIR=${SRC_DIR}/build
	mkdir -p ${BUILD_DIR} && cd ${BUILD_DIR}
	cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo ${SRC_DIR}
	# if Eigen or Boost are not available system-wide, run at that point: 
	#   cmake-gui .
	# cmake-gui allows you to tell the location of Eigen or Boost
	make
	sudo make install

These lines will compile libnabo in a `build` sub-directory and therefore keep your source tree clean.
Note that you could compile libnabo anywhere you have write access, such as in `/tmp/libnabo`.
This out-of-source build is a nice feature of [CMake].
If [Eigen] or [Boost] are not installed system-wide, you might have to tell [CMake] where to find them (using `ccmake` or `cmake-gui`).

You can generate the documentation by typing:

	make doc

Usage
=====

libnabo is easy to use. For example, assuming that you are working with floats and that you have a point set `M` and a query point `q`, you can find the `K` nearest neighbours of `q` in `M`:

	#include "nabo/nabo.h"
	using namespace Nabo;
	using namespace Eigen;
	...
	NNSearchF* nns = NNSearchF::createKDTreeLinearHeap(M);
	
	const int K = 5;
	VectorXi indices(K);
	VectorXf dists2(K);
	
	nns->knn(q, indices, dists2, K);

In this example, `M` is an [Eigen] (refering to the software, not to the math) matrix (column major, float) and `q` is an [Eigen] vector (float).
The results `indices` and `dists2` are [Eigen] vectors of indices and squared distances refering to the columns of `M`.
See `examples/trivial.cpp` for a compilable version of this example, and `examples/usage.cpp` for a slightly more complex example involving multi-point queries.

Running `make doc` in your build directory will generate a browsable documentation in `doc/html`.
The main page `doc/html/index.html` contains a detailed overview of the usage of libnabo.

Python bindings
===============

libnabo includes python bindings that are compiled if python is available.
The resulting module is called pynabo, you can see an example in `python/test.py`.
You can find more information in the docstring-based documentation:

	python -c "import pynabo; help(pynabo.NearestNeighbourSearch)"

Unit testing
============

The distribution of libnabo integrates a unit test module, based on CTest.
Just type:

	make test
   
...in the build directory to run the tests.
Their outputs are available in the `Testing` directory.
These consist of validation and benchmarking tests.
If [ANN] or [FLANN] are detected when compiling libnabo, `make test` will also perform comparative benchmarks.

Citing libnabo
==============

If you use libnabo in the academic context, please cite this paper that evaluates its performances in the contex of ICP:

	@article{elsebergcomparison,
		title={Comparison of nearest-neighbor-search strategies and implementations for efficient shape registration},
		author={Elseberg, J. and Magnenat, S. and Siegwart, R. and N{\"u}chter, A.},
		journal={Journal of Software Engineering for Robotics (JOSER)},
		pages={2--12},
		volume={3},
		number={1},
		year={2012},
		issn={2035-3928}
	}


Bug reporting
=============

Please use [github's issue tracker](http://github.com/ethz-asl/libnabo/issues) to report bugs.


License
=======

libnabo is released under a permissive BSD license.


[ANN]: http://www.cs.umd.edu/~mount/ANN
[FLANN]: http://www.cs.ubc.ca/~mariusm/index.php/FLANN/FLANN
[CMake]: http://www.cmake.org
[CMake documentation]: http://www.cmake.org/cmake/help/cmake2.6docs.html
[Eigen]: http://eigen.tuxfamily.org
[Boost]: http://www.boost.org
