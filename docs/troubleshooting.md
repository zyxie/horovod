## Troubleshooting

### Import TensorFlow failed during installation

1. Is TensorFlow installed?

If you see the error message below, it means that TensorFlow is not installed.  Please install TensorFlow before installing
Horovod.

```
error: import tensorflow failed, is it installed?

Traceback (most recent call last):
  File "/tmp/pip-OfE_YX-build/setup.py", line 29, in fully_define_extension
    import tensorflow as tf
ImportError: No module named tensorflow
```

2. Are the CUDA libraries available?

If you see the error message below, it means that TensorFlow cannot be loaded.  If you're installing Horovod into a container
on a machine without GPUs, you may use CUDA stub drivers to work around the issue.

```
error: import tensorflow failed, is it installed?

Traceback (most recent call last):
  File "/tmp/pip-41aCq9-build/setup.py", line 29, in fully_define_extension
    import tensorflow as tf
  File "/usr/local/lib/python2.7/dist-packages/tensorflow/__init__.py", line 24, in <module>
    from tensorflow.python import *
  File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/__init__.py", line 49, in <module>
    from tensorflow.python import pywrap_tensorflow
  File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/pywrap_tensorflow.py", line 52, in <module>
    raise ImportError(msg)
ImportError: Traceback (most recent call last):
  File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/pywrap_tensorflow.py", line 41, in <module>
    from tensorflow.python.pywrap_tensorflow_internal import *
  File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/pywrap_tensorflow_internal.py", line 28, in <module>
    _pywrap_tensorflow_internal = swig_import_helper()
  File "/usr/local/lib/python2.7/dist-packages/tensorflow/python/pywrap_tensorflow_internal.py", line 24, in swig_import_helper
    _mod = imp.load_module('_pywrap_tensorflow_internal', fp, pathname, description)
ImportError: libcuda.so.1: cannot open shared object file: No such file or directory
```

To use CUDA stub drivers:

```bash
# temporary add stub drivers to ld.so.cache
$ ldconfig /usr/local/cuda/lib64/stubs

# install Horovod, add other HOROVOD_* environment variables as necessary
$ pip install --no-cache-dir horovod

# revert to standard libraries
$ ldconfig
```

### MPI is not found during installation

1. Is MPI in PATH?

If you see the error message below, it means `mpicxx` was not found in PATH. Typically `mpicxx` is located in the same
directory as `mpirun`. Please add a directory containing `mpicxx` to PATH before installing Horovod.

```
error: mpicxx -show failed, is mpicxx in $PATH?

Traceback (most recent call last):
  File "/tmp/pip-dQ6A7a-build/setup.py", line 70, in get_mpi_flags
    ['mpicxx', '-show'], universal_newlines=True).strip()
  File "/usr/lib/python2.7/subprocess.py", line 566, in check_output
    process = Popen(stdout=PIPE, *popenargs, **kwargs)
  File "/usr/lib/python2.7/subprocess.py", line 710, in __init__
    errread, errwrite)
  File "/usr/lib/python2.7/subprocess.py", line 1335, in _execute_child
    raise child_exception
OSError: [Errno 2] No such file or directory
```

To use custom MPI directory:

```bash
$ export PATH=$PATH:/path/to/mpi/bin
$ pip install --no-cache-dir horovod
```

### NCCL 2 is not found during installation

If you see the error message below, it means NCCL 2 was not found in the standard libraries location. If you have a directory
where you installed NCCL 2 which has both `include` and `lib` directories containing `nccl.h` and `libnccl.so` 
respectively, you can pass it via `HOROVOD_NCCL_HOME` environment variable. Otherwise you can specify them separately
via `HOROVOD_NCCL_INCLUDE` and `HOROVOD_NCCL_LIB` environment variables.

```
build/temp.linux-x86_64-2.7/test_compile/test_nccl.cc:1:18: fatal error: nccl.h: No such file or directory
 #include <nccl.h>
                  ^
compilation terminated.
error: NCCL 2.0 library or its later version was not found (see error above).
Please specify correct NCCL location via HOROVOD_NCCL_HOME environment variable or combination of HOROVOD_NCCL_INCLUDE and HOROVOD_NCCL_LIB environment variables.

HOROVOD_NCCL_HOME - path where NCCL include and lib directories can be found
HOROVOD_NCCL_INCLUDE - path to NCCL include directory
HOROVOD_NCCL_LIB - path to NCCL lib directory
```

For example:

```bash
$ HOROVOD_GPU_ALLREDUCE=NCCL HOROVOD_NCCL_HOME=/path/to/nccl pip install --no-cache-dir horovod
```

Or:

```bash
$ HOROVOD_GPU_ALLREDUCE=NCCL HOROVOD_NCCL_INCLUDE=/path/to/nccl/include HOROVOD_NCCL_LIB=/path/to/nccl/lib pip install --no-cache-dir horovod
```

### NCCL 2 is not found during runtime

If you see the error message below, it means NCCL 2 was not found in the standard libraries location. You should add the directory
where you installed NCCL 2 libraries to the `LD_LIBRARY_PATH` environment variable.

```
Traceback (most recent call last):
  File "tf_cnn_benchmarks.py", line 46, in <module>
    import horovod.tensorflow as hvd
  File "/home/asergeev/mpi/venv-nccl/local/lib/python2.7/site-packages/horovod/tensorflow/__init__.py", line 34, in <module>
    from horovod.tensorflow.mpi_ops import size
  File "/home/asergeev/mpi/venv-nccl/local/lib/python2.7/site-packages/horovod/tensorflow/mpi_ops.py", line 74, in <module>
    ['HorovodAllgather', 'HorovodAllreduce'])
  File "/home/asergeev/mpi/venv-nccl/local/lib/python2.7/site-packages/horovod/tensorflow/mpi_ops.py", line 56, in _load_library
    library = load_library.load_op_library(filename)
  File "/home/asergeev/mpi/venv-nccl/local/lib/python2.7/site-packages/tensorflow/python/framework/load_library.py", line 64, in load_op_library
    None, None, error_msg, error_code)
tensorflow.python.framework.errors_impl.NotFoundError: libnccl.so.2: cannot open shared object file: No such file or directory
```

For example:

```bash
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/nccl-<version>/lib
$ mpirun -np 16 -x LD_LIBRARY_PATH -H server1:4,server2:4,server3:4,server4:4 python train.py
```

### Pip install: no such option: --no-cache-dir

If you see the error message below, it means that your version of pip is out of date. You can remove the `--no-cache-dir` flag
since your version of pip does not do caching. The `--no-cache-dir` flag is added to all examples to ensure that when you
change Horovod compilation flags, it will be rebuilt from source and not just reinstalled from the pip cache, which is
modern pip's [default behavior](https://pip.pypa.io/en/stable/reference/pip_install/#caching).

```
$ pip install --no-cache-dir horovod

Usage:
  pip install [options] <requirement specifier> ...
  pip install [options] -r <requirements file> ...
  pip install [options] [-e] <vcs project url> ...
  pip install [options] [-e] <local project path> ...
  pip install [options] <archive url/path> ...

no such option: --no-cache-dir
```

For example:

```bash
$ pip install horovod
```

### Running out of memory

If you notice that your program is running out of GPU memory and multiple processes
are being placed on the same GPU, it's likely that your program (or its dependencies)
create a `tf.Session` that does not use the `config` that pins specific GPU.

If possible, track down the part of program that uses these additional `tf.Session`s and pass
the same configuration.

Alternatively, you can place following snippet in the beginning of your program to ask TensorFlow
to minimize the amount of memory it will pre-allocate on each GPU:

```python
small_cfg = tf.ConfigProto()
small_cfg.gpu_options.allow_growth = True
with tf.Session(config=small_cfg):
    pass
```

As a last resort, you can **replace** setting `config.gpu_options.visible_device_list`
with different code:

```python
# Pin GPU to be used
import os
os.environ['CUDA_VISIBLE_DEVICES'] = str(hvd.local_rank())
```

**Note**: Setting `CUDA_VISIBLE_DEVICES` is incompatible with `config.gpu_options.visible_device_list`.

Setting `CUDA_VISIBLE_DEVICES` has additional disadvantage for GPU version - CUDA will not be able to use IPC, which
will likely cause NCCL and MPI to fail.  In order to disable IPC in NCCL and MPI and allow it to fallback to shared
memory, use:
* `export NCCL_P2P_DISABLE=1` for NCCL.
* `--mca btl_smcuda_use_cuda_ipc 0` flag for OpenMPI and similar flags for other vendors.
