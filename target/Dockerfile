FROM chaquopy-base

# When moving to a new version of the NDK, carefully review the following:
# * The release notes (https://developer.android.com/ndk/downloads/revision_history)
# * https://android.googlesource.com/platform/ndk/+/ndk-release-rXX/docs/BuildSystemMaintainers.md,
#   where XX is the NDK version. Check over the `master` version of that document as well.
RUN yes | android-sdk/cmdline-tools/tools/bin/sdkmanager "ndk;18.1.5063045"

COPY build-toolchains.sh build-toolchain.sh build-common.sh target/
RUN target/build-toolchains.sh android-sdk/ndk/* "arm64-v8a armeabi-v7a x86 x86_64"

# Copy Python headers and libraries into the toolchains.
COPY unpackage-target.sh target/
RUN mkdir maven && \
    repo="https://chaquo.com/maven" && version="3.8.6-0" && \
    for abi in $(cd target/toolchains && echo *); do \
        wget -P maven $repo/com/chaquo/python/target/$version/target-$version-$abi.zip; \
    done && \
    target/unpackage-target.sh maven target/toolchains && \
    rm -r maven
