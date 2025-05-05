FROM ubuntu:20.04 AS builder

LABEL maintainer="Ahmed Agiza"

ENV GO111MODULE=on \
    CGO_ENABLED=1 \
    PATH=/usr/local/go/bin:$PATH

RUN apt update && apt upgrade -y && apt install wget -y
RUN DEBIAN_FRONTEND=noninteractive TZ=Etc/UTC apt-get -y install tzdata


# Install CMake for OpenDB
RUN wget https://cmake.org/files/v3.14/cmake-3.14.0-Linux-x86_64.sh && \
    chmod +x cmake-3.14.0-Linux-x86_64.sh  && \
    ./cmake-3.14.0-Linux-x86_64.sh --skip-license --prefix=/usr/local && rm -rf cmake-3.14.0-Linux-x86_64.sh \
    && rm -rf /var/lib/apt/lists/*


# Install Tcl, automake, pcre, flex, and bison for OpenDB
#RUN apt-get update && apt-get install -y automake autotools-dev bison flex git libpcre3-dev tcl tcl-dev tcl8.6 tcl8.6-dev tk tk-dev && rm -rf /var/lib/apt/lists/*
RUN apt-get update && apt-get install -y automake autotools-dev bison flex git libpcre3-dev tcl tcl-dev && rm -rf /var/lib/apt/lists/*

# Install SWIG for OpenDB
RUN  wget https://github.com/swig/swig/archive/rel-4.0.1.tar.gz \
    && tar xfz rel-4.0.1.tar.gz \
    && rm -rf rel-4.0.1.tar.gz \
    && cd swig-rel-4.0.1 \
    && ./autogen.sh && ./configure --prefix=/usr && make -j $(nproc) && make install \
    && cd .. \
    && rm -rf swig-rel-4.0.1

# Install Go
RUN wget https://go.dev/dl/go1.17.linux-amd64.tar.gz \
    && rm -rf /usr/local/go/ \
    && tar -C /usr/local/ -xzf go1.17.linux-amd64.tar.gz

WORKDIR /build

COPY server/go.mod .
COPY server/go.sum .
RUN go mod download

COPY Makefile .
COPY config.mk .
COPY server ./server

ENV CPLUS_INCLUDE_PATH=/usr/include/tcl:$CPLUS_INCLUDE_PATH
ENV C_INCLUDE_PATH=/usr/include/tcl:$C_INCLUDE_PATH

RUN make build

WORKDIR /dist
RUN cp /build/server/bin/edav-server ./server

RUN ldd server | tr -s '[:blank:]' '\n' | grep '^/' | \
    xargs -I % sh -c 'mkdir -p $(dirname ./%); cp % ./%;'
RUN mkdir -p lib64 && cp /lib64/ld-linux-x86-64.so.2 lib64/

RUN mkdir /data

FROM scratch

COPY --chown=0:0 --from=builder /dist /

COPY --chown=65534:0 --from=builder /data /data
COPY --chown=65534:0 --from=builder /tmp /tmp

USER 65534
WORKDIR /data

ENTRYPOINT ["/server"]
