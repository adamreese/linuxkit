FROM golang:1.7-alpine3.5 AS build

RUN \
  apk update && apk upgrade && \
  apk add --no-cache \
  ca-certificates \
  gcc \
  git \
  libc-dev \
  make \
  && true

# PR https://github.com/docker/swarmkit/pull/1965 from ijc25/containerd
ENV SWARMKIT_PR=1965
ENV SWARMKIT_COMMIT=321b9c6600a9422c3245b277a1b3ae599244d4b7

RUN mkdir -p $GOPATH/src/github.com/docker && \
  cd $GOPATH/src/github.com/docker && \
  git clone https://github.com/docker/swarmkit.git
WORKDIR $GOPATH/src/github.com/docker/swarmkit
RUN [ -z "$SWARMKIT_PR" ] || git fetch origin pull/$SWARMKIT_PR/head
RUN git checkout $SWARMKIT_COMMIT

RUN make binaries GO_GCFLAGS="-buildmode pie --ldflags '-extldflags \"-fno-PIC -static\"'"

RUN mkdir -p /build/dist/usr/bin/ /build/dist/etc
RUN cp bin/swarmd bin/swarmctl /build/dist/usr/bin/
RUN strip /build/dist/usr/bin/swarmd /build/dist/usr/bin/swarmctl

RUN cp -r /etc/ssl /build/dist/etc/ssl

FROM scratch
WORKDIR /
ENTRYPOINT []
COPY --from=build /build/dist /
CMD ["/usr/bin/swarmd", "--containerd-addr=/run/containerd/containerd.sock", "--log-level=debug", "--state-dir=/var/lib/swarmd"]
