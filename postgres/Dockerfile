ARG ALPINE=3.20
ARG PG_MAJOR=16
ARG PG_MINOR=4

FROM postgres:$PG_MAJOR.$PG_MINOR-alpine$ALPINE

# Make sure the below versions reflect the versions listed here:
# https://cloud.google.com/sql/docs/postgres/extensions#data-type-extensions
ARG PG_VECTOR=0.7.0
ARG PG_HLL=2.18
ARG PG_POSTGIS=3.4.0
ARG PG_REPACK=1.5.0
ARG PG_PARTMAN=5.0.1
ARG PG_CRON=1.6.2
ARG PG_SQUEEZE=1_5_0

RUN set -eux; \
    # Install build dependencies
    apk add --update --no-cache --virtual .build-deps \
      # build base
      wget \
      build-base \
      # required by pg_repack
      gawk \
      # required to build postgis extension
      autoconf \
      automake \
      cunit-dev \
      gettext-dev \
      json-c-dev \
      libtool \
      libxml2-dev \
      pcre2-dev \
      protobuf-c-dev \
      gdal-dev \
      geos-dev \
      proj-dev \
      proj-util \
      sfcgal-dev \
      # The upstream variable, '$DOCKER_PG_LLVM_DEPS' contains
      #  the correct versions of 'llvm-dev' and 'clang' for the current version of PostgreSQL.
      # This improvement has been discussed in https://github.com/docker-library/postgres/pull/1077
      $DOCKER_PG_LLVM_DEPS \
      postgresql$PG_MAJOR-dev \
    ; \
    cd /tmp; \
    # Install pgvector extension
    wget https://github.com/pgvector/pgvector/archive/refs/tags/v$PG_VECTOR.tar.gz -O pgvector.tar.gz; \
    mkdir pgvector; \
    tar xf ./pgvector.tar.gz -C pgvector --strip-components 1; \
    cd pgvector; \
    make OPTFLAGS=""; \
    make install; \
    cd ..; \
    # Install hll extension
    wget https://github.com/citusdata/postgresql-hll/archive/refs/tags/v$PG_HLL.tar.gz -O hll.tar.gz; \
    mkdir hll; \
    tar xf ./hll.tar.gz -C hll --strip-components 1; \
    cd hll; \
    make; \
    make install; \
    cd ..; \
    # Install postgis extension
    wget https://github.com/postgis/postgis/archive/refs/tags/$PG_POSTGIS.tar.gz -O postgis.tar.gz; \
    mkdir postgis; \
    tar xf ./postgis.tar.gz -C postgis --strip-components 1; \
    cd postgis; \
    ./autogen.sh; \
    ./configure; \
    make; \
    make install; \
    cd ..; \
    # Install pg_repack extension
    wget https://github.com/reorg/pg_repack/archive/refs/tags/ver_$PG_REPACK.tar.gz -O repack.tar.gz; \
    mkdir repack; \
    tar xf ./repack.tar.gz -C repack --strip-components 1; \
    cd repack; \
    make; \
    make install; \
    cd ..; \
    # Install pg_cron extension
    wget https://github.com/citusdata/pg_cron/archive/refs/tags/v$PG_CRON.tar.gz -O cron.tar.gz; \
    mkdir cron; \
    tar xf ./cron.tar.gz -C cron --strip-components 1; \
    cd cron; \
    make; \
    make install; \
    cd ..; \
    # Install pgpartman extension
    wget https://github.com/pgpartman/pg_partman/archive/refs/tags/v$PG_PARTMAN.tar.gz -O partman.tar.gz; \
    mkdir partman; \
    tar xf ./partman.tar.gz -C partman --strip-components 1; \
    cd partman; \
    make; \
    make install; \
    cd ..; \
    # Install pg_squeeze extension
    # wget https://github.com/cybertec-postgresql/pg_squeeze/archive/refs/tags/REL$PG_SQUEEZE.tar.gz -O squeeze.tar.gz; \
    # mkdir squeeze; \
    # tar xf ./squeeze.tar.gz -C squeeze --strip-components 1; \
    # cd squeeze; \
    # make; \
    # make install; \
    # cd ..; \
    # echo "wal_level = logical" >> /usr/local/share/postgresql/postgresql.conf.sample; \
    # echo "max_replication_slots = 1" >> /usr/local/share/postgresql/postgresql.conf.sample; \
    # Register shared libraries
    # echo "shared_preload_libraries = 'pg_squeeze,pg_cron,pg_partman_bgw'" >> /usr/local/share/postgresql/postgresql.conf.sample; \
    echo "shared_preload_libraries = 'pg_cron,pg_partman_bgw'" >> /usr/local/share/postgresql/postgresql.conf.sample; \
    # Delete build dependencies
    rm -r /tmp/*; \
    apk del .build-deps;

# Install runtime dependencies required by Postgis
RUN set -eux; \
    # Install build dependencies
    apk add --update --no-cache --virtual .runtime-deps \
      json-c \
      protobuf-c \
      geos \
      proj \
    ;

RUN set -eux; \
    mkdir -p /etc/postgresql; \
    cp /usr/local/share/postgresql/postgresql.conf.sample /etc/postgresql/postgresql.conf

# Make 3rd party extensions trusted so DB owners can install them without SUPERUSER privileges
RUN echo "trusted = true" >> /usr/local/share/postgresql/extension/vector.control

CMD ["postgres", "-c", "config_file=/etc/postgresql/postgresql.conf"]
