FROM perlur/centos-ocaml-opam AS builder

ENV TEZOS_NODE_VERSION="mainnet"

RUN yum update -y && \
    yum install -y perl gmp-devel hidapi-devel libev-devel
WORKDIR /usr/src
RUN git clone https://gitlab.com/tezos/tezos.git -b ${TEZOS_NODE_VERSION}
WORKDIR tezos
RUN git checkout ${TEZOS_NODE_VERSION}
RUN eval $(opam env) && make build-deps
RUN eval $(opam env) && make
RUN eval $(opam env) && PREFIX=/usr/local make install

FROM perlur/centos-base

MAINTAINER "Mark Stopka <mark.stopka@perlur.cloud>"

ENV SERVICE_NAME "tezos-node"

RUN yum update -y && \
    yum install -y hidapi libev && \
    yum clean all && \
    dnf clean all && \
    rm -rf /var/cache/yum && \
    rm -rf /var/cache/dnf

COPY --from=builder /usr/src/tezos/tezos-accuser* /usr/local/bin/
COPY --from=builder /usr/src/tezos/tezos-admin-client /usr/local/bin/
COPY --from=builder /usr/src/tezos/tezos-baker* /usr/local/bin/
COPY --from=builder /usr/src/tezos/tezos-client /usr/local/bin/
COPY --from=builder /usr/src/tezos/tezos-codec /usr/local/bin/
COPY --from=builder /usr/src/tezos/tezos-endorser* /usr/local/bin/
COPY --from=builder /usr/src/tezos/tezos-node /usr/local/bin/
COPY --from=builder /usr/src/tezos/tezos-protocol-compiler /usr/local/bin/
COPY --from=builder /usr/src/tezos/tezos-signer /usr/local/bin/
COPY --from=builder /usr/src/tezos/tezos-validator /usr/local/bin/

RUN mkdir -p /srv/tezos/

RUN useradd -c "Tezos node user" \
            -d /srv/tezos/tezos-node/ \
            -m \
            -r \
            -s /bin/nologin \
            tezos-node

USER tezos-node

WORKDIR /srv/tezos/tezos-node/