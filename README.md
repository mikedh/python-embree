# python-embree #

This library is a thin wrapper around Embree 3.

As much as possible, it tries to emulate the C API usage. The main
point of this is to avoid creating a new API which would obfuscate the
usage of the C API. Ideally, it should be easy to read the Embree
documentation or examples, and translate things straightforwardly to
equivalent Python code.

A secondary goal is to provide easy interoperability with numpy.

**NOTE**: *very little of the library is wrapped so far, but this
library is being developed in a way that should make it as easy as
possible to wrap more functionality as necessary. If you find that a
function that you'd like to use isn't wrapped, please create an issue
or feel free to wrap it on your own and submit a pull request.*

## Installation

### Windows

If you install Embree using the MSI from Embree's website, the Embree
binaries, headers, and libraries will all be installed to `C:\Program
Files\Intel\Embree3` by default.

As an example, to compile using MSYS2 from the MinGW 64-bit console,
after `cd`ing to the root directory of this repository, it should be
possible to run:

```
$ python setup.py build_ext -I/c/Program\ Files/Intel/Embree3/include -L /c/Program\ Files/Intel/Embree3/lib
$ python setup.py install
```

to successfully compile and install python-embree.

## Tips and tricks

### Parallelism

Using
[multiprocessing](https://docs.python.org/3/library/multiprocessing.html)
for concurrency in Python requires objects that are put into queues to
be serialized using
[pickle](https://docs.python.org/3/library/pickle.html). Unfortunately,
it is not currently possible to serialize the Embree data structures
(see the Embree repository's issues
[#137](https://github.com/embree/embree/issues/137) and
[#238](https://github.com/embree/embree/issues/238)), and there do not
appear to be plans to support this feature. The rationale for not
supporting this feature is that building the Embree BVH from scratch
is usually faster than reading the equivalent amount of data from
disk.

This means that you will not be able to use any of the extensions
classes exported by
[embree.pyx](https://github.com/sampotter/python-embree/blob/master/embree.pyx)
(such as `embree.Device`, `embree.Scene`, etc.) with multiprocessing
*directly*. To get around this problem, a simple fix is to wrap a bit
of Embree functionality in a Python class with its own `__reduce__`
method. For an example, see the implementation of `TrimeshShapeModel`
[here](https://github.com/sampotter/python-flux/blob/master/flux/shape.py).
