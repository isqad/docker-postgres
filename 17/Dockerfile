FROM postgres:17

RUN echo "Asia/Yekaterinburg" > /etc/timezone \
  && dpkg-reconfigure -f noninteractive tzdata \
  && sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
  && echo 'LANG="en_US.UTF-8"' > /etc/default/locale \
  && dpkg-reconfigure --frontend=noninteractive locales \
  && update-locale LANG=en_US.UTF-8

RUN DEBIAN_FRONTEND=noninteractive apt-get update -qq \
  && apt-get install -yq --no-install-recommends \
    postgresql-plperl-17 \
    postgresql-17-pgq3 \
    libpq-dev \
    postgresql-server-dev-17 \
    postgresql-common \
    wget \
    make \
    build-essential \
    libssl-dev \
    libkrb5-dev \
    ca-certificates \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
  && truncate -s 0 /var/log/*log

ARG PG_RUM_VER=1.3.15
RUN  wget https://github.com/postgrespro/rum/archive/${PG_RUM_VER}.tar.gz \
    && tar -xzvf ${PG_RUM_VER}.tar.gz\
    && make -C rum-${PG_RUM_VER} install USE_PGXS=1 \
    && rm ${PG_RUM_VER}.tar.gz \
    && rm -rf rum-${PG_RUM_VER}

ARG PG_JOBMON_VER=1.4.1
ARG PG_PARTMAN_VER=5.4.3

RUN wget https://github.com/omniti-labs/pg_jobmon/archive/v${PG_JOBMON_VER}.tar.gz \
    && tar -xzvf v${PG_JOBMON_VER}.tar.gz \
    && make -C pg_jobmon-${PG_JOBMON_VER} \
    && make NO_BGW=1 install -C pg_jobmon-${PG_JOBMON_VER} \
    && rm v${PG_JOBMON_VER}.tar.gz \
    && rm -rf pg_jobmon-${PG_JOBMON_VER} && \

  wget https://github.com/pgpartman/pg_partman/archive/v${PG_PARTMAN_VER}.tar.gz \
    && tar -xzvf v${PG_PARTMAN_VER}.tar.gz \
    && make -C pg_partman-${PG_PARTMAN_VER} \
    && make install -C pg_partman-${PG_PARTMAN_VER} \
    && rm v${PG_PARTMAN_VER}.tar.gz \
    && rm -rf pg_partman-${PG_PARTMAN_VER}

# Configure
RUN echo "listen_addresses = '*'" >> /usr/share/postgresql/17/postgresql.conf.sample && \
  echo "plperl.on_init = 'use utf8; use re; package utf8;'" >> /usr/share/postgresql/17/postgresql.conf.sample

# # Cleanup
RUN apt-get purge -y build-essential \
  make \
  wget

ENV PATH /usr/bin:$PATH
ENV POSTGRES_HOST_AUTH_METHOD trust
COPY initdb.sh /docker-entrypoint-initdb.d/
