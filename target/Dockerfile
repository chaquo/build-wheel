FROM debian:stretch-20180831
SHELL ["/bin/bash", "-c"]
WORKDIR /root

RUN echo "progress=dot:giga" > .wgetrc

# NDK
RUN apt-get update && \
    apt-get install -y wget xz-utils
RUN filename=crystax-ndk-10.3.2-linux-x86_64.tar.xz && \
    wget https://www.crystax.net/download/$filename && \
    tar -xf $filename && \
    rm $filename && \
    mv crystax-ndk-* crystax

# Python 3.6 isn't included in this version of Debian, so use a third-party build.
RUN wget https://github.com/chriskuehl/python3.6-debian-stretch/releases/download/v3.6.3-1-deb9u1/{python3.6,python3.6-minimal,libpython3.6,libpython3.6-minimal,libpython3.6-stdlib}_3.6.3-1.deb9u1_amd64.deb && \
    apt install -y ./*.deb && \
    rm *.deb

# Copy Python headers and libraries into the NDK.
COPY unpackage-target.sh target/
RUN apt-get update && \
    apt-get install -y unzip && \
    target/unpackage-target.sh crystax \
        https://chaquo.com/maven/com/chaquo/python/target/3.6.5-11
