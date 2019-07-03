# Introduction

Copyright (c) 2018 Chaquo Ltd

This repository contains the build-wheel tool, which produces Android .whl files for Chaquopy.
build-wheel itself is only supported on Linux x86-64. However, the resulting .whls can be built
into an app on any supported Android build platform, as described in the [Chaquopy
documentation](https://chaquo.com/chaquopy/doc/current/android.html#requirements).


# Adding a package

Create a new subdirectory in `packages`. Its name must be in PyPI normalized form (PEP 503).
Alternatively, you can create this subdirectory somewhere else, and use the `--extra-packages`
option when calling `build-wheel.py`.

Inside the subdirectory, add the following files:

* A `meta.yaml` file. This supports a subset of Conda syntax, defined in `meta-schema.yaml`.
* For non-Python packages, a `build.sh` script. See `build-wheel.py` for environment variables
  which are passed to it.
* If necessary, a `patches` subdirectory containing patch files.


# Building with Docker

Docker is the simplest and most reliable way to get all of build-wheel's dependencies set up.

If necessary, install Docker using the [instructions on its
website](https://docs.docker.com/install/#supported-platforms).

Build the two Docker images:

    docker build -t chaquopy-target target
    docker build -t chaquopy-pypi server/pypi

If the `server/pypi` directory changes, you'll need to rerun the second of these commands to
update the corresponding image. If the `target` directory changes, you'll need to rerun both
commands.

Then run build-wheel from the `server/pypi` directory as follows:

    docker run -v $(pwd)/dist:/root/pypi/dist chaquopy-pypi --abi <abi> <package>

Where:

* `<abi>` is an [Android ABI](https://developer.android.com/ndk/guides/abis).
* `<package>` is the name of the subdirectory created above.

The resulting .whl files will be generated in `server/pypi/dist`.


# Building without Docker

This is more convenient for development, but you'll have to set up all of build-wheel's
dependencies manually. Use the Dockerfiles as a guide.

Then run build-wheel from the `server/pypi` directory as follows:

    ./build-wheel.py --ndk <crystax> --abi <abi> <package>

Where:

* `<crystax>` is the location of the Crystax NDK.
* `<abi>` and `<package>` are as above.

The resulting .whl files will be generated in `server/pypi/dist`.
