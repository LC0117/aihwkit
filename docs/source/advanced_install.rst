Advanced installation guide
===========================

Install the aihwkit conda package
---------------------------------

At this time, the conda package is only available for the Linux environment. You can use the
following steps as an example for how to install the aihwkit conda package.

.. note::

    We are working on publishing the package in conda-forge channel.  Until then, you need to download the package for installation.

Download the aihwkit conda package tar file::

    $ wget https://aihwkit-gpu-demo.s3.us-east.cloud-object-storage.appdomain.cloud/aihwkit-condapkg.tar

Untar the file to a directory such as $HOME/aihwkit-condapkg
Create a conda environment::

    $ conda create -n aihwkit
    $ conda activate aihwkit

Install one of the conda packages.  For example:

  - CPU::

    $ conda install python=3.9 aihwkit -c conda-forge -c file://$HOME/aihwkit-condapkg

  - GPU::

    $ conda install python=3.9 aihwkit-gpu -c conda-forge -c file://$HOME/aihwkit-condapkg

Compilation
-----------

The build system for ``aihwkit`` is based on `cmake`_, making use of
scikit-build_ for generating the Python packages.

Some of the dependencies and tools are Python-based. For convenience, we
suggest creating a `virtual environment`_ as a way to isolate your
environment::

    $ python3 -m venv aihwkit_env
    $ cd aihwkit_env
    $ source bin/activate
    (aihwkit_env) $

.. note::

    The following sections assume that the command line examples are executed
    in the activated ``aihwkit_env`` environment.

Dependencies
~~~~~~~~~~~~

For compiling ``aihwkit``, the following dependencies are required:

===============================  ========  ======
Dependency                       Version   Notes
===============================  ========  ======
C++11 compatible compiler
`cmake`_                         3.18+
`pybind11`_                      2.6.2+    Versions 2.6.0+ can be installed using ``pip`` (recommended)
`scikit-build`_                  0.11.0+
`Python 3 development headers`_  3.7+
BLAS implementation                        `OpenBLAS`_ or `Intel MKL`_
`PyTorch`_                       1.7+      The libtorch library and headers are needed [#f1]_
`OpenMP`_                        11.0.0+   Optional, OpenMP library and headers [#f2]_
CUDA                             9.0+      Optional, for GPU-enabled simulator
`Nvidia CUB`_                    1.8.0     Optional, for GPU-enabled simulator [#f4]_
`googletest`_                    1.10.0    Optional, for building the C++ tests [#f4]_
===============================  ========  ======

Please refer to your operative system documentation for instructions on how
to install the different dependencies. The following section contains quick
instructions for several operative systems:

Debian-based
""""""""""""

On a Debian-based operative system, the following commands can be used for
installing the minimal dependencies::

    $ sudo apt-get install python3-dev libopenblas-dev
    $ pip install cmake scikit-build torch pybind11

OSX
"""

On an OSX-based system, the following commands can be used for installing the
minimal dependencies (note that ``Xcode`` needs to be installed)::

    $ brew install openblas
    $ pip install cmake scikit-build torch pybind11

miniconda (e.g. linux)
""""""""""""""""""""""

On a miniconda-based system, the following commands can be used for installing
the minimal dependencies [#f3]_::

    $ conda install cmake openblas pybind11
    $ conda install -c conda-forge scikit-build
    $ conda install -c pytorch pytorch

.. note::
    You can also install all the requirements by::

        $ pip install -r requirements.txt
        $ pip install -r requirements-dev.txt
        $ pip install -r requirements-examples.txt

.. note::
    If you are using CUDA (see below) then you need to have a
    CUDA-enabled pytorch installed. Please refer to the torch website
    how to install that


Windows using conda (Experimental)
""""""""""""""""""""""""""""""""""

On a Windows-based system, the following instructions can be used for
installing the dependencies:

1. Install (regular) `Miniconda`_, install newest `Cuda`_ driver (if available)
   and the `MS Visual Studio 2019`_ community edition with ``Desktop development
   with C++`` workload.

2. Start ``anaconda powershell`` (miniconda) and install the following
   packages::

    $ conda install pybind11 scikit-build
    $ conda install pytorch -c pytorch
    $ conda install -c intel mkl mkl-devel mkl-static mkl-include

Using this method, please make sure that the flags ``-DRPU_BLAS=MKL`` and
``-G "Visual Studio 16 2019"`` are passed to the installation and compilation
commands. In particular, use the following command instead of the default one
in the `Installing and compiling` sub-section::

    $ pip install -v aihwkit --install-option="-DUSE_CUDA=ON" --install-option="-DRPU_BLAS=MKL" --install-option="-GVisual Studio 16 2019"

Windows with OpenBLAS (Experimental)
""""""""""""""""""""""""""""""""""""

As an alternative on Windows-based system, compilation using OpenBLAS is also
possible. We recommend installing OpenBLAS following this
`OpenBLAS - Visual Studio`_ installation and usage guide. It requires
installing `MS Visual Studio 2019`_ and `Miniconda`_.

After compiling and installing OpenBLAS, in the same Miniconda terminal, the
following commands can be used for installing the minimal dependencies::

    $ conda install pybind11 scikit-build
    $ conda install pytorch -c pytorch

For compiling ``aihwkit``, it is recommended to use the x64 Native Tools Command
Prompt for VS 2019.

.. note::

    If you want to use ``pip`` instead of ``conda``, the following commands can
    be used::

        $ pip install cmake scikit-build pybind11
        $ pip install torch -f https://download.pytorch.org/whl/torch_stable.html


Installing and compiling
~~~~~~~~~~~~~~~~~~~~~~~~

Once the dependencies are in place, the following command can be used for
compiling. Here we assume that you have already cloned the directory
and changed into it::

    $ git clone https://github.com/IBM/aihwkit.git
    $ cd aihwkit

You can typically install requirements by (but see above for more
specific details)::

    $ pip install -r requirements.txt
    $ pip install -r requirements-dev.txt
    $ pip install -r requirements-examples.txt


Without GPU support (with OpenBLAS):
  This uses the OpenBLAS library for fast numerical computations::

    $ make build

  .. note::

    Note that openblas needs to be installed, e.g. with::
        $ conda install openblas

Without GPU support (with MKL):
  This uses the Intel MKL library instead of the OpenBlas library::

    $ make build_mkl

  .. note::
     Note that MKL needs to be installed and environment variable
     ``MKLROOT`` set if not in standard folders. E.g. with::

         $ conda install -c intel mkl mkl-devel mkl-static mkl-include

With GPU support:
  The CUDA library needs to be set up properly so that the compiler
  can find it (you may need to set ``CUDA_HOME``). Please refer to the
  installation instructions. This also uses MKL
  as default, whihc thus needs to be installed (see above). Then::

      $ make build_cuda

  If you know your CUDA architecture, then you can give it directly
  (which will result typically in a much quicker initially loading time)::

      $ make build_cuda flags="-DRPU_CUDA_ARCHITECTURES='60'"


If there are any issue with the dependencies or the compilation, the output
of the command will help diagnosing the issue.

In-place installation
~~~~~~~~~~~~~~~~~~~~~

If you want install the library inside the cloned directory (see also
:doc:`developer_install`), it is more convenient for developers. For
that simply replace the above make commands with ``build_inplace``,
e.g.::

    $ make build_inplace_cuda

Here, you need to make sure that the ``PYTHONPATH`` is set to the ``src``
sub-directory of the ahwkit base directory, e.g. by (when being in the base directory)::

    $ export PYTHONPATH=`pwd`/src:$PYTHONPATH

CUDA-enabled docker image
~~~~~~~~~~~~~~~~~~~~~~~~~
As an alternative to a regular install, a CUDA-enabled docker image can also be
built using the ``CUDA.Dockerfile`` included in the repository.

In order to build the image, first identify the ``CUDA_ARCH`` for your GPU
using ` `nvidia-smi`` in your local machine::

    export CUDA_ARCH=$(nvidia-smi --query-gpu=compute_cap --format=csv | sed -n '2 p' | tr -d '.')
    echo $CUDA_ARCH

The image can be built via::

    docker build \
    --tag aihwkit:cuda \
    --build-arg USERNAME=${USER} \
    --build-arg USERID=(id -u $USER) \
    --build-arg GROUPID=(id -g $USER) \
    --build-arg CUDA_ARCH=${CUDA_ARCH} \
    --file CUDA.Dockerfile .

If building your image against a different CUDA version, please make sure to
update the ``CUDA_ARCH`` build argument accordingly.

.. note::

    Please note that the instructions on this page refer to installing as an
    end user. If you are planning to contribute to the project, an alternative
    setup and tips can be found at the :doc:`developer_install` section that
    is more tuned towards the needs of a development cycle.

.. [#f1] This library uses PyTorch as both a build dependency and a runtime
   dependency. Please ensure that your torch installation includes ``libtorch``
   and the development headers - they are included by default if installing
   torch from ``pip``.

.. [#f2] Support for the parts of the OpenMP 4.0+. Some compilers like LLVM or
   Clang do not support OpenMP. In case of you want to add shared memory
   processing support to the library using one of these compilers, you will
   need to install OpenMP library in your system.

.. [#f3] Please note that currently support for conda-based distributions is
   experimental, and further commands might be needed.

.. [#f4] Both ``Nvidia CUB`` and ``googletest`` are downloaded and compiled
   automatically during the build process. As a result, they do not need to be
   installed manually.

.. _virtual environment: https://docs.python.org/3/library/venv.html

.. _cmake: https://cmake.org/
.. _Nvidia CUB: https://github.com/NVlabs/cub
.. _pybind11: https://github.com/pybind/pybind11
.. _Python 3 development headers: https://www.python.org/downloads/
.. _OpenBLAS: https://www.openblas.net
.. _Intel MKL: https://software.intel.com/content/www/us/en/develop/tools/math-kernel-library.html
.. _scikit-build: https://github.com/scikit-build/scikit-build
.. _googletest: https://github.com/google/googletest
.. _PyTorch: https://pytorch.org
.. _OpenMP: https://openmp.llvm.org
.. _OpenBLAS - Visual Studio: https://github.com/xianyi/OpenBLAS/wiki/How-to-use-OpenBLAS-in-Microsoft-Visual-Studio
.. _MS Visual Studio 2019: https://visualstudio.microsoft.com/vs/
.. _Miniconda: https://docs.conda.io/en/latest/miniconda.html
.. _Cuda: https://developer.nvidia.com/cuda-toolkit
