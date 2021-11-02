# Introduction

This repository contains the build-wheel tool, which produces Android .whl files for Chaquopy.
build-wheel itself only runs on Linux x86-64. However, it can build .whl files for all [Android
ABIs](https://developer.android.com/ndk/guides/abis) (armeabi-v7a, arm64-v8a, x86 and x86_64).


# License

Copyright (c) 2021 Chaquo Ltd

If you have purchased a license for build-wheel, then you may use the tool and distribute the
resulting .whl files however you like, including building them into an app and distributing
that app. However, redistribution of build-wheel itself is not permitted.


# Adding a package

Create a recipe directory in `packages`. Its name must be in PyPI normalized form (PEP 503).
Alternatively, you can create this directory somewhere else, and pass its path when calling
`build-wheel.py`.

Inside the recipe directory, add the following files:

* A `meta.yaml` file. This supports a subset of Conda syntax, defined in `meta-schema.yaml`.
* For non-Python packages, a `build.sh` script. See `build-wheel.py` for environment variables
  which are passed to it.
* If necessary, a `patches` subdirectory containing patch files.

The following examples are included:

* multidict: a minimal example, downloaded from PyPI.
* python-example: a pybind11-based package, downloaded from a Git repository.
* cmake-example: similar to python-example, but uses cmake. A patch is used to help cmake find
  the Android toolchain file.
* chaquopy-libzmq: a non-Python library, downloaded from a URL.
* pyzmq: a Python package which depends on a non-Python library. A patch is used to help
  `setup.py` find the library.
* scikit-learn: lists several build-time requirements in `meta.yaml`:
  * The "build" requirement (Cython) will be installed automatically.
  * The "host" requirements (NumPy etc.) should be downloaded from [the public
    repository](https://chaquo.com/pypi-7.0/) and copied into `server/pypi/dist` before running
    the build. A patch is used to allow NumPy and SciPy to be imported during the build.


# Building with Docker

Docker is the easiest way to set up all of build-wheel's dependencies.

If necessary, install Docker using the [instructions on its
website](https://docs.docker.com/install/#supported-platforms).

Build the Docker images:

    docker build -t chaquopy-base -f base.dockerfile .
    docker build -t chaquopy-target target
    docker build -t build-wheel server/pypi

If the `server/pypi` directory changes, you'll need to rerun the last of these commands to
update the build-wheel image.

Then run build-wheel from the `server/pypi` directory as follows:

    docker run -v $(pwd)/dist:/root/pypi/dist build-wheel --toolchain target/toolchains/<abi> <package>

Where:

* `<abi>` is an [Android ABI](https://developer.android.com/ndk/guides/abis).
* `<package>` is the name of the recipe directory created above.

The resulting .whl files will be generated in `server/pypi/dist`.


# Building without Docker

This is more convenient for development, but you'll have to set up all of build-wheel's
dependencies manually. Use the Dockerfiles as a guide.

Then run build-wheel from the `server/pypi` directory as follows:

    ./build-wheel.py --toolchain ../../target/toolchains/<abi> <package>

`<abi>` and `<package>` are described above. The `toolchains` directory can be copied out of
the `chaquopy-target` image.

The resulting .whl files will be generated in `server/pypi/dist`.


# Using the generated .whl files in your app

.whl files can be built into your app using the [`pip`
block](https://chaquo.com/chaquopy/doc/current/android.html#requirements) in your
`build.gradle` file. To tell pip where to find the files, use the `options` setting to pass
either [`--find-links`](https://pip.pypa.io/en/stable/reference/pip_install/#cmdoption-f) or
[`--extra-index-url`](https://pip.pypa.io/en/stable/reference/pip_install/#cmdoption-extra-index-url).
