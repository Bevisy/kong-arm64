FROM bevisy/debian:10_arm64 as build

# install compile dependency
RUN apt-get update && \
    apt-get install -y libpcre3-dev \
    libssl-dev perl make \
    build-essential curl wget vim jq \
    zlib1g-dev && \
    apt-get clean all

# compile and install openresty
RUN wget https://openresty.org/download/openresty-1.15.8.1.tar.gz && \
    tar zxf openresty-1.15.8.1.tar.gz && \
    wget https://github.com/Kong/openresty-patches/archive/master.tar.gz && \
    tar zxvf master.tar.gz && \
    cd openresty-1.15.8.1/bundle && \
    for i in ../../openresty-patches-master/patches/1.15.8.1/*.patch; do patch -p1 < $i; done && \
    cd .. && \
    ./configure --with-pcre-jit \
                --with-http_ssl_module \
                --with-http_realip_module \
                --with-http_stub_status_module \
                --with-http_v2_module && \
    make install && \
    cd /root && \
    rm -f openresty-1.15.8.1.tar.gz && \
    rm -rf openresty-1.15.8.1 && \
    rm -f master.tar.gz && \
    rm -rf openresty-patches-master

# install compile dependency
RUN apt-get update && \
    apt-get install -y zip unzip && \
    apt-get clean all

# compile and install luarocks
RUN wget https://github.com/luarocks/luarocks/archive/v3.1.3.tar.gz && \
    tar zxf v3.1.3.tar.gz && \
    cd luarocks-3.1.3 && \
    ./configure \
    --lua-suffix=jit \
    --with-lua=/usr/local/openresty/luajit \
    --with-lua-include=/usr/local/openresty/luajit/include/luajit-2.1 && \
    make install && \
    cd /root && \
    rm -f v3.1.3.tar.gz && \
    rm -rf luarocks-3.1.3
    rm -rf luarocks-3.1.3

ENV PATH="$PATH:/usr/local/openresty/bin:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin"

# install luarocks package dependency
RUN apt-get update && \
    apt-get install -y git m4 libyaml-dev && \
    apt-get clean all

# install kong v1.3.0
RUN luarocks install kong 1.3.0-0

RUN kong prepare

FROM bevisy/debian:10_arm64
LABEL maintainer="Binbin Zhang <binbin36520@gmail.com>"

COPY --from=builder /usr/local /usr/local

RUN adduser --system --uid 1337 --ingroup root kong && \
    chown -R kong /usr/local/kong && \
    chgrp -R 0 /usr/local/kong && \
    chmod -R g=u /usr/local/kong
RUN apt-get update && \
    apt-get install -y libpcre3-dev libssl-dev perl curl vim jq tzdata && \
    apt-get clean all

ENV PATH="$PATH:/usr/local/openresty/bin:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin"

COPY docker-entrypoint.sh /docker-entrypoint.sh

USER kong

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 8000 8443 8001 8444

STOPSIGNAL SIGTERM

CMD ["kong", "docker-start"]
