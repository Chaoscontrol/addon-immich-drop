ARG BUILD_FROM=ghcr.io/hassio-addons/base:18.1.1
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Pin upstream source at build time (overridden by CI when syncing)
ARG UPSTREAM_REF=v0.4.0-beta

# Copy root filesystem (s6 services, scripts)
COPY rootfs /
# Ensure s6 run scripts are executable
RUN chmod +x /etc/s6-overlay/s6-rc.d/*/run || true

# Base packages and Python toolchain
# - Build deps needed for some Python wheels (e.g., Pillow)
# - Runtime libs for Pillow and friends
RUN apk add --no-cache \
        python3 \
        py3-pip \
        bash \
        coreutils \
        tini \
        # Build deps
        build-base \
        python3-dev \
        musl-dev \
        libffi-dev \
        openssl-dev \
        # Pillow runtime + headers
        zlib-dev \
        libjpeg-turbo-dev \
        tiff-dev \
        freetype-dev \
        lcms2-dev \
        openjpeg-dev \
        libwebp-dev \
    && python3 -m pip install --no-cache-dir --upgrade pip

# Fetch upstream source and install dependencies
WORKDIR /opt
# Install git only for build; smaller layer than editing previous apk block
RUN apk add --no-cache git \
    && git clone --depth 1 --branch "${UPSTREAM_REF}" https://github.com/Nasogaa/immich-drop.git immich_drop

# Install Python dependencies (omit dev-only watchers)
WORKDIR /opt/immich_drop
RUN sed -i '/^watchfiles==/d' requirements.txt \
    && sed -i '/^httptools==/d' requirements.txt \
    && python3 -m pip install --no-cache-dir -r requirements.txt \
    && python3 -m pip install --no-cache-dir python-multipart

# Ensure data dir exists and is writable (HA persists /data automatically)
RUN mkdir -p /data /data/chunks \
    && chmod -R 755 /data

# Environment defaults (overridden by bashio exports from options)
ENV HOST=0.0.0.0 \
    PORT=8080 \
    STATE_DB=/data/state.db

# Build args for OCI labels
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION="Immich Drop uploader for Home Assistant"
ARG BUILD_NAME="Immich Drop"
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.url="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}

EXPOSE 8080

# s6-overlay is the default entrypoint in hassio-addons/base; nothing to CMD here.