ARG DISTRO=debian
ARG SUITE=trixie
ARG BASE_IMAGE=ghcr.io/loong64/${DISTRO}:${SUITE}-slim

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
    && git clone https://github.com/loong64/live-build.git --depth 1

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update \
    && cd live-build \
    && mk-build-deps -t "apt-get -o Debug::pkgProblemResolver=yes --no-install-recommends -y" -i debian/control \
    && dpkg-buildpackage -uc -us \
    && apt install -y ../live-build_*.deb

ARG VERSION

WORKDIR /opt/live

RUN lb config --distribution sid \
        --binary-image iso \
        --apt-secure false \
        --bootloaders grub-efi \
        --debian-installer cdrom \
        --uefi-secure-boot disable \
        --debian-installer-distribution git \
        --image-name debian-testing-${VERSION} \
        --keyring-packages debian-ports-archive-keyring \
        --mirror-binary http://deb.debian.org/debian-ports \
        --mirror-bootstrap http://deb.debian.org/debian-ports \
        --debootstrap-options "--include=debian-archive-keyring,ca-certificates" \

COPY config/ config/

RUN set -ex \
    && mkdir -p config/bootloaders \
    && wget --quiet https://salsa.debian.org/installer-team/debian-installer/-/raw/master/build/boot/artwork/12-emerald/emerald.svg -O config/bootloaders/splash.svg

RUN lb build

FROM ${BASE_IMAGE}
ARG TARGETARCH

WORKDIR /opt/dist

COPY --from=builder /opt/build/*.deb ./
COPY --from=builder /opt/live/*.iso ./

VOLUME /dist

CMD cp -rf /opt/dist/* /dist/
