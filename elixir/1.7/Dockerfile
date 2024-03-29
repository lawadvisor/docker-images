# https://github.com/bitwalker/alpine-erlang/blob/21.3.3/Dockerfile
FROM alpine:3.9

ENV REFRESHED_AT=2019-04-05 \
    LANG=en_US.UTF-8 \
    # Set this so that CTRL+G works properly
    TERM=xterm \
    ERLANG_VERSION=21.3.3 \
    ELIXIR_VERSION=v1.7.4

WORKDIR /tmp/erlang-build

# Install Erlang
RUN \
    mkdir -p /opt/app/ && \
    echo "@main http://dl-cdn.alpinelinux.org/alpine/v3.9/main" >> /etc/apk/repositories && \
    echo "@community http://dl-cdn.alpinelinux.org/alpine/v3.9/community" >> /etc/apk/repositories && \
    echo "@edge http://dl-cdn.alpinelinux.org/alpine/edge/main" >> /etc/apk/repositories && \
    # Upgrade Alpine and base packages
    apk --no-cache --update --available upgrade && \
    # Distillery requires bash Install bash and Erlang/OTP deps
    apk add --no-cache --update pcre@edge && \
    apk add --no-cache --update \
      bash \
      ca-certificates \
      openssl-dev \
      ncurses-dev \
      unixodbc-dev \
      zlib-dev && \
    # Install Erlang/OTP build deps
    apk add --no-cache --virtual .erlang-build \
      dpkg-dev dpkg binutils \
      git autoconf build-base perl-dev && \
    # Shallow clone Erlang/OTP
    git clone -b OTP-$ERLANG_VERSION --single-branch --depth 1 https://github.com/erlang/otp.git . && \
    # Erlang/OTP build env
    export ERL_TOP=/tmp/erlang-build && \
    export PATH=$ERL_TOP/bin:$PATH && \
    export CPPFlAGS="-D_BSD_SOURCE $CPPFLAGS" && \
    # Configure
    ./otp_build autoconf && \
    ./configure --prefix=/usr \
      --build="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
      --sysconfdir=/etc \
      --mandir=/usr/share/man \
      --infodir=/usr/share/info \
      --without-javac \
      --without-wx \
      --without-debugger \
      --without-observer \
      --without-jinterface \
      --without-cosEvent\
      --without-cosEventDomain \
      --without-cosFileTransfer \
      --without-cosNotification \
      --without-cosProperty \
      --without-cosTime \
      --without-cosTransactions \
      --without-et \
      --without-gs \
      --without-ic \
      --without-megaco \
      --without-orber \
      --without-percept \
      --without-typer \
      --enable-threads \
      --enable-shared-zlib \
      --enable-ssl=dynamic-ssl-lib \
      --enable-hipe && \
      # Build
      make -j4 && make install && \
      # Cleanup
      cd /opt/app/ && \
      rm -rf /tmp/erlang-build && \
      # Update ca certificates
      update-ca-certificates --fresh && \
      /usr/bin/erl -eval "beam_lib:strip_release('/usr/lib/erlang/lib')" -s init stop > /dev/null && \
      (/usr/bin/strip /usr/lib/erlang/erts-*/bin/* || true) && \
      rm -rf /usr/lib/erlang/usr/ && \
      rm -rf /usr/lib/erlang/misc/ && \
      for DIR in /usr/lib/erlang/erts* /usr/lib/erlang/lib/*; do \
          rm -rf ${DIR}/src/*.erl; \
          rm -rf ${DIR}/doc; \
          rm -rf ${DIR}/man; \
          rm -rf ${DIR}/examples; \
          rm -rf ${DIR}/emacs; \
          rm -rf ${DIR}/c_src; \
      done && \
      rm -rf /usr/lib/erlang/erts-*/lib/ && \
      rm /usr/lib/erlang/erts-*/bin/dialyzer && \
      rm /usr/lib/erlang/erts-*/bin/erlc && \
      rm /usr/lib/erlang/erts-*/bin/typer && \
      rm /usr/lib/erlang/erts-*/bin/ct_run && \
      apk del --force .erlang-build

WORKDIR /tmp/elixir-build

RUN \
    apk add --no-cache --update --virtual .elixir-build \
      make && \
    apk add --no-cache --update \
      git && \
    git clone https://github.com/elixir-lang/elixir --depth 1 --branch $ELIXIR_VERSION && \
    cd elixir && \
    make && make install && \
    mix local.hex --force && \
    mix local.rebar --force && \
    cd $HOME && \
    rm -rf /tmp/elixir-build && \
    apk del --force .elixir-build

WORKDIR /
