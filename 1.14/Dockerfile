# This is a dockerfile for building doge

ARG VERSION=1.14.3


# CPU architecture to build binaries for
ARG ARCH

# Define default versions so that they don't have to be repeated throughout the file
ARG VER_ALPINE=3.12

# $USER name, and data $DIR to be used in the `final` image
ARG USER=dogecoind
ARG DIR=/data

# Choose where to get bitcoind sources from, options: release, git
#   NOTE: Only `SOURCE=git` can be used for RC releases
ARG SOURCE=release

# Choose where to get BerkeleyDB from, options: prebuilt, compile
#   NOTE: When compiled here total execution time exceeds allowed CI limits, so pre-built one is used by default
ARG BDB_SOURCE=prebuilt



#
## `preparer-base` installs dependencies needed by both ways of fetching the source,
#       as well as imports GPG keys needed to verify authenticity of the source.
#
FROM alpine:${VER_ALPINE} AS preparer-base

# Make sure APKs are downloaded over SSL. See: https://github.com/gliderlabs/docker-alpine/issues/184
RUN sed -i 's|http://dl-cdn.alpinelinux.org|https://alpine.global.ssl.fastly.net|g' /etc/apk/repositories

RUN apk add --no-cache gnupg
#
## Option #1: [default] Fetch bitcoind source from release tarballs
#
FROM preparer-base AS preparer-release

ARG VERSION

# Download source code (intentionally different website than checksums)
ADD https://github.com/dogecoin/dogecoin/archive/v$VERSION.tar.gz ./

RUN tar -xzf "v$VERSION.tar.gz" && \
    rm  -f   "v$VERSION.tar.gz"



#
## Option #2: Fetch bitcoind source from GitHub
#
FROM preparer-base AS preparer-git

ARG VERSION

RUN apk add --no-cache git

# Fetch the source code at a specific TAG
RUN git clone  -b "v$VERSION"  --depth=1  https://github.com/dogecoin/dogecoin.git  "/dogecoin-$VERSION/"



#
## Alias to go around `COPY` not accepting ARGs in value passed to `--from=`
#
FROM preparer-${SOURCE} AS preparer



#
## `berkeleydb-prebuilt` downloads a pre-built BerkeleyDB to make sure
#       the overall build time of this Dockerfile fits within CI limits.
#
FROM lncm/berkeleydb:v4.8.30.NC${ARCH:+-${ARCH}} AS berkeleydb-prebuilt

#
## `berkeleydb-compile` builds BerkeleyDB from source using script provided in bitcoind repo.
#

FROM berkeleydb-${BDB_SOURCE} AS berkeleydb



FROM ${ARCH:+${ARCH}/}alpine:${VER_ALPINE} AS builder

ARG VERSION
ARG SOURCE

# Use APK repos over HTTPS. See: https://github.com/gliderlabs/docker-alpine/issues/184
RUN sed -i 's|http://dl-cdn.alpinelinux.org|https://alpine.global.ssl.fastly.net|g' /etc/apk/repositories

RUN apk add --no-cache \
        autoconf \
        automake \
        boost-dev \
        build-base \
        chrpath \
        file \
        libevent-dev \
        libressl \
        libtool \
        linux-headers \
        zeromq-dev \
        openssl-dev

# Fetch pre-built berkeleydb
COPY  --from=berkeleydb /opt/  /opt/

# Change to the extracted directory
WORKDIR /dogecoin-$VERSION/

# Copy bitcoin source (downloaded & verified in previous stages)
COPY  --from=preparer /dogecoin-$VERSION/  ./

ENV DOGECOIN_PREFIX /opt/dogecoin-$VERSION

RUN ./autogen.sh

# TODO: Try to optimize on passed params
RUN ./configure \
    --prefix="$DOGE_PREFIX" \
    --disable-man \
    --disable-shared \
    --disable-ccache \
    --enable-static \
    --enable-reduce-exports \
    --without-gui \
    --without-libs \
    --with-utils \
    --with-daemon \
    --disable-wallet

RUN make -j$(( $(nproc) + 1 )) check
RUN make install

# List installed binaries pre-strip & strip them
RUN ls -lh "$PREFIX/bin/"
RUN strip -v "$PREFIX/bin/dogecoin"*

# List installed binaries post-strip & print their checksums
RUN ls -lh "$PREFIX/bin/"
RUN sha256sum "$PREFIX/bin/dogecoin"*



#
## `final` aggregates build results from previous stages into a necessary minimum
#       ready to be used, and published to Docker Hub.
#
# NOTE: this stage is emulated using QEMU
# NOTE: `${ARCH:+${ARCH}/}` - if ARCH is set, append `/` to it, leave it empty otherwise
FROM ${ARCH:+${ARCH}/}alpine:${VER_ALPINE} AS final

ARG VERSION
ARG USER
ARG DIR

LABEL maintainer="nolim1t (@nolim1t)"

# Use APK repos over HTTPS. See: https://github.com/gliderlabs/docker-alpine/issues/184
RUN sed -i 's|http://dl-cdn.alpinelinux.org|https://alpine.global.ssl.fastly.net|g' /etc/apk/repositories

RUN apk add --no-cache \
        boost-filesystem \
        boost-thread \
        boost-dev \
        libevent \
        libsodium \
        libstdc++ \
        libzmq

COPY  --from=builder $PREFIX/bin/dogecoin*  /usr/local/bin/

# NOTE: Default GID == UID == 1000
RUN adduser --disabled-password \
            --home "$DIR/" \
            --gecos "" \
            "$USER"

USER $USER

RUN mkdir -p "$DIR/.dogecoin/"

VOLUME $DIR/.dogecoin/

# REST interface
EXPOSE 8080

# P2P network (mainnet, testnet & regnet respectively)
# TODO: doge ports
EXPOSE 8333 18333 18444

# RPC interface (mainnet, testnet & regnet respectively)
# TODO: doge ports
EXPOSE 8332 18332 18443

# ZMQ ports (for transactions & blocks respectively)
EXPOSE 28332 28333

ENTRYPOINT ["dogecoind"]

CMD ["-zmqpubrawblock=tcp://0.0.0.0:28332", "-zmqpubrawtx=tcp://0.0.0.0:28333"]
