FROM chaquopy-base

# When moving to a new version of the NDK, carefully review the following:
# * The release notes (https://developer.android.com/ndk/downloads/revision_history)
# * https://android.googlesource.com/platform/ndk/+/ndk-release-rXX/docs/BuildSystemMaintainers.md,
#   where XX is the NDK version. Check over the `master` version of that document as well.
RUN yes | android-sdk/cmdline-tools/tools/bin/sdkmanager "ndk;21.4.7075529"

RUN apt-get update && \
    apt-get install -y python  # For the NDK's make_standalone_toolchain.py.
COPY build-toolchains.sh build-toolchain.sh build-common.sh target/
RUN target/build-toolchains.sh android-sdk/ndk/* "arm64-v8a armeabi-v7a x86 x86_64"
COPY for-each-abi.sh target/

RUN apt-get update && \
    apt-get -y install bison flex g++ make

# This version of Debian includes patchelf 0.9, which rearranges the file in ways that break
# some other tools, including strip (https://github.com/NixOS/patchelf/issues/10). Stripping
# before patching works on API level 26, but level 15 gives a dynamic linker error even when
# the file isn't stripped at all ("get_lib_extents[777]: 1132 - No loadable segments found").
# Build a fixed version from https://github.com/NixOS/patchelf/pull/117 .
RUN apt-get update && \
    apt-get install -y autoconf && \
    commit=6bfcafbba8d89e44f9ac9582493b4f27d9d8c369 && \
    wget -q -O - https://github.com/NixOS/patchelf/archive/$commit.tar.gz | tar -xzf - && \
    ( cd patchelf-$commit && ./bootstrap.sh && ./configure && make && make install ) && \
    rm -r patchelf-$commit

# Copy Python headers and libraries into the toolchains.
COPY unpackage-target.sh target/
RUN mkdir maven && \
    repo="https://chaquo.com/maven" && version="3.8.6-0" && \
    for abi in $(cd target/toolchains && echo *); do \
        wget -P maven $repo/com/chaquo/python/target/$version/target-$version-$abi.zip; \
    done && \
    target/unpackage-target.sh maven target/toolchains && \
    rm -r maven
