FROM postgres:12.1-alpine
MAINTAINER Pedro G. Galaviz, hello@pggalaviz.com

#---------------------------------------------------------------------------------------
ENV ALPINE_MIRROR_BASE http://dl-cdn.alpinelinux.org
ENV ALPINE_MIRROR_VERSION v3.11

RUN echo "$ALPINE_MIRROR_BASE/alpine/$ALPINE_MIRROR_VERSION/main" > /etc/apk/repositories
RUN echo "$ALPINE_MIRROR_BASE/alpine/$ALPINE_MIRROR_VERSION/community" >> /etc/apk/repositories
#---------------------------------------------------------------------------------------

ENV POSTGIS_VERSION 3.0.0
ENV POSTGIS_SHA256 1c83fb2fc8870d36ed49859c49a12c8c4c8ae8c5c3f912a21a951c5bcc249123

# Get and extract PostGIS
RUN wget -O postgis.tar.gz "https://github.com/postgis/postgis/archive/$POSTGIS_VERSION.tar.gz" \
    && echo "$POSTGIS_SHA256 *postgis.tar.gz" | sha256sum -c - \
    && mkdir -p /usr/src/postgis \
    && tar \
      --extract \
      --file /postgis.tar.gz \
      --directory /usr/src/postgis \
      --strip-components 1 \
    && rm /postgis.tar.gz

# Install build dependencies
RUN apk update \
    && apk add --no-cache --virtual .build-llvm \
      llvm-dev clang-dev \
    && apk add --no-cache --virtual .build-deps \
      --repository "$ALPINE_MIRROR_BASE/alpine/edge/main" \
      --repository "$ALPINE_MIRROR_BASE/alpine/edge/testing" \
      --repository "$ALPINE_MIRROR_BASE/alpine/edge/community" \
      autoconf \
      automake \
      g++ \
      json-c-dev \
      libtool \
      libxml2-dev \
      make \
      perl \
      linux-headers \
      geos-dev \
      gdal-dev \
      proj-dev \
      protobuf-c-dev

# Install PostGIS
RUN cd /usr/src/postgis \
    && ./autogen.sh \
    && ./configure \
    && make -s \
    && make install \
    && apk add --no-cache --virtual .postgis-rundeps \
      json-c \
    && apk add --no-cache --virtual .postgis-rundeps-edge \
      --repository "$ALPINE_MIRROR_BASE/alpine/edge/main" \
      --repository "$ALPINE_MIRROR_BASE/alpine/edge/testing" \
      --repository "$ALPINE_MIRROR_BASE/alpine/edge/community" \
      geos \
      gdal \
      proj \
      protobuf-c \
    && cd / \
    && rm -rf /usr/src/postgis \
    && apk del .build-llvm .build-deps

COPY ./initdb-postgis.sh /docker-entrypoint-initdb.d/postgis.sh
COPY ./update-postgis.sh /usr/local/bin
