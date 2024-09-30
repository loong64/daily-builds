ARG DISTRO=debian
ARG SUITE=trixie
ARG BASE_IMAGE=${DISTRO}:${SUITE}-slim

FROM ${BASE_IMAGE} AS builder

ARG DEPENDENCIES="                 \
        devscripts                 \
        equivs                     \
        software-properties-common"

ARG TOOLS="                        \
        ca-certificates            \
        curl                       \
        git                        \
        wget"

ADD sources.list /etc/apt/sources.list

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    set -ex \
    && rm -f /etc/apt/apt.conf.d/docker-clean \
    && echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' >/etc/apt/apt.conf.d/keep-cache \
    && apt-get update \
    && apt-get install -y ${DEPENDENCIES} \
    && apt-get install -y --no-install-recommends ${TOOLS}

WORKDIR /opt/build

RUN set -ex \
    && git clone https://github.com/loong64/live-build.git --depth 1 \
    && mk-build-deps -t "apt-get -o Debug::pkgProblemResolver=yes --no-install-recommends -y" -i live-build/debian/control \
    && cd live-build \
    && dpkg-buildpackage -uc -us

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    set -ex \
    && apt update \
    && apt install -y ./live-build_*.deb

WORKDIR /opt/live

RUN set -ex \
    && lb config --distribution sid \
        --binary-image iso \
        --apt-secure false \
        --bootloaders grub-efi \
        --debian-installer cdrom \
        --uefi-secure-boot disable \
        --debian-installer-distribution git \
        --image-name debian-testing-$(date +%Y%m%d) \
        --keyring-packages debian-ports-archive-keyring \
        --mirror-binary http://deb.debian.org/debian-ports \
        --mirror-bootstrap http://deb.debian.org/debian-ports

COPY config/ config/

RUN set -ex \
    && lb build

FROM ${BASE_IMAGE}
ARG TARGETARCH

WORKDIR /opt/dist

COPY --from=builder /opt/build/live-build_*.deb .
COPY --from=builder /opt/live/debian-testing-*.iso .

VOLUME /dist

CMD cp -rf dist/* /dist/
