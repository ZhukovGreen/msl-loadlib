======================
Welcome to MSL-LoadLib
======================

|docs| |pypi|

Purpose
-------

This package is used to load a shared library in Python. It is basically just a thin wrapper
around ctypes_ (for libraries that use the ``__cdecl`` or ``__stdcall`` calling convention),
`Python for .NET`_ (for libraries that use Microsoft's .NET Framework, ``CLR``) and Py4J_
(for Java ``.jar`` or ``.class`` files). However, the primary advantage is that it is possible
to communicate with a 32-bit shared library from 64-bit Python.

Tested in Python 2.7, 3.3+. The `examples <http://msl-loadlib.readthedocs.io/en/latest/direct.html>`_
provided are currently only supported in Windows and Linux. The 32-bit server has not yet been
created for OSX.

**MSL-LoadLib** is a pure-python package, but `Python for .NET`_ depends on the .NET Common Language
Runtime (CLR) on Windows and Mono Runtime on Linux and OSX and Py4J_ depends on having a
`Java Virtual Machine`_ installed.

Install
-------

To install **MSL-LoadLib** run::

   pip install msl-loadlib

Alternatively, using the `MSL Package Manager`_ run::

   msl install loadlib

To install the dependencies on Linux, please follow the instructions on the
`prerequisites <http://msl-loadlib.readthedocs.io/en/latest/install.html#prerequisites>`_
section of the documentation.

Examples
--------

If you are loading a 64-bit library in 64-bit Python, or a 32-bit library in 32-bit Python,
then you can directly load the library using ``LoadLibrary``.

The following examples load a 64-bit library in a 64-bit Python interpreter. If you are using a 32-bit
Python interpreter then replace the **64** with **32** in the filename.

Import the ``LoadLibrary`` class and the directory where the example libraries are located

.. code:: python

   >>> from msl.loadlib import LoadLibrary
   >>> from msl.examples.loadlib import EXAMPLES_DIR

If the file extension is not included then a default extension, ``.dll`` (Windows) or ``.so`` (Linux), is used.

Load a `C++ <https://github.com/MSLNZ/msl-loadlib/blob/master/msl/examples/loadlib/cpp_lib.cpp>`_ library
and call the ``add`` function

.. code:: python

   >>> cpp = LoadLibrary(EXAMPLES_DIR + '/cpp_lib64')
   >>> cpp.lib.add(1, 2)
   3

Load a `FORTRAN <https://github.com/MSLNZ/msl-loadlib/blob/master/msl/examples/loadlib/fortran_lib.f90>`_
library and call the ``factorial`` function

.. code:: python

   >>> fortran = LoadLibrary(EXAMPLES_DIR + '/fortran_lib64')

With a FORTRAN library you must pass values by reference using ctypes_, and, since the returned value is not
of type ``int`` we must configure ctypes_ for a value of type ``double`` to be returned

.. code:: python

   >>> from ctypes import byref, c_int, c_double
   >>> fortran.lib.factorial.restype = c_double
   >>> fortran.lib.factorial(byref(c_int(37)))
   1.3763753091226343e+43

Load a `.NET <https://github.com/MSLNZ/msl-loadlib/blob/master/msl/examples/loadlib/dotnet_lib.cs>`_ library
and call the ``reverse_string`` function, we must specify that the library type is a .NET library by passing
in the ``'net'`` argument

.. code:: python

   >>> net = LoadLibrary(EXAMPLES_DIR + '/dotnet_lib64.dll', 'net')
   >>> net.lib.StringManipulation.reverse_string('abcdefghijklmnopqrstuvwxyz')
   'zyxwvutsrqponmlkjihgfedcba'

Load `Java <https://github.com/MSLNZ/msl-loadlib/blob/master/msl/examples/loadlib/Trig.java>`_ byte code
and call the ``cos`` function

.. code:: python

   >>> java = LoadLibrary(EXAMPLES_DIR + '/Trig.class')
   >>> java.lib.Trig.cos(1.234)
   0.33046510807172985

Python interacts with the `Java Virtual Machine`_ via a local network socket and therefore the connection
needs to be closed when you are done using the Java library

.. code:: python

   >>> java.gateway.shutdown()

`Inter-process communication <ipc_>`_ is used to access a 32-bit shared library from a module that is
running within a 64-bit Python interpreter. The procedure uses a client-server protocol where the client
is a subclass of ``msl.loadlib.Client64`` and the server is a subclass of ``msl.loadlib.Server32``.
See the `tutorials <http://msl-loadlib.readthedocs.io/en/latest/interprocess_communication.html>`_ for
examples on how to implement `inter-process communication <ipc_>`_.

Documentation
-------------

The documentation for **MSL-LoadLib** can be found `here <http://msl-loadlib.readthedocs.io/en/latest/index.html>`_.

Developers Guide
----------------

**MSL-LoadLib** uses pytest_ for testing the source code and sphinx_ for creating the documentation.

Run the tests (a coverage_ report is generated in the **htmlcov/index.html** file)::

   python setup.py test

Build the documentation, which can be viewed by opening the **docs/_build/html/index.html** file::

   python setup.py docs

Automatically create the API documentation from the docstrings in the source code (uses sphinx-apidoc_)::

   python setup.py apidoc

*NOTE: By default, the* **docs/_autosummary** *folder that is created by running the* **apidoc** *command is
automatically generated (it will overwrite existing files). As such, it is excluded from the repository (i.e., this
folder is specified in the* **.gitignore** *file). If you want to keep the files located in* **docs/_autosummary** *you
can rename the folder to be, for example,* **docs/_api** *and then the changes made to the files in the* **docs/_api**
*folder will be kept and will be included in the repository.*

.. |docs| image:: https://readthedocs.org/projects/msl-loadlib/badge/?version=latest
   :target: http://msl-loadlib.readthedocs.io/en/latest/?badge=latest
   :alt: Documentation Status
   :scale: 100%

.. |pypi| image:: https://badge.fury.io/py/msl-loadlib.svg
   :target: https://badge.fury.io/py/msl-loadlib

.. _ctypes: https://docs.python.org/3/library/ctypes.html
.. _Python for .NET: http://pythonnet.github.io/
.. _Py4J: https://www.py4j.org/
.. _pytest: http://doc.pytest.org/en/latest/
.. _sphinx: http://www.sphinx-doc.org/en/latest/
.. _sphinx-apidoc: http://www.sphinx-doc.org/en/latest/man/sphinx-apidoc.html
.. _coverage: http://coverage.readthedocs.io/en/latest/index.html
.. _ipc: https://en.wikipedia.org/wiki/Inter-process_communication
.. _Java Virtual Machine: https://en.wikipedia.org/wiki/Java_virtual_machine
.. _MSL Package Manager: http://msl-package-manager.readthedocs.io/en/latest/?badge=latest
