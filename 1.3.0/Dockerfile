FROM bevisy/ubuntu:16.04_arm64

ENV KONG_VERSION 1.3.0
ENV OPENRESTY_VERSION 1.15.8.1
ENV LUAROCKS_VERSION 3.1.3
ENV PCRE_VERSION=8.43
ENV OPENSSL_VERSION 1.1.1c

RUN apt-get update && apt-get install -y libreadline-dev libncurses5-dev \
  build-essential perl curl unzip m4 lua-yaml-dev wget git


RUN wget https://ftp.pcre.org/pub/pcre/pcre-${PCRE_VERSION}.tar.gz \
	&& tar -xvf pcre-${PCRE_VERSION}.tar.gz \
    && cd pcre-${PCRE_VERSION} \
    && ./configure \
    && make \
    && make install \
    && cd .. \
    && rm -rf pcre-${PCRE_VERSION}.tar.gz

RUN wget https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz \
	&& tar -zvxf openssl-${OPENSSL_VERSION}.tar.gz \
	&& wget https://raw.githubusercontent.com/openresty/openresty/master/patches/openssl-${OPENSSL_VERSION}-sess_set_get_cb_yield.patch \
	&& cd openssl-${OPENSSL_VERSION}/ \
	&& patch -p1 < ../openssl-${OPENSSL_VERSION}-sess_set_get_cb_yield.patch \
	&& ./config -fPIC \
	&& make \
	&& make install \
	&& cd .. \
    && rm -rf openssl-${OPENSSL_VERSION}-sess_set_get_cb_yield.patch \
    && rm -rf openssl-${OPENSSL_VERSION}.tar.gz

RUN apt-get install -y zlib1g-dev make && \
    apt-get clean all

RUN wget https://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz \
    && tar -xzvf openresty-${OPENRESTY_VERSION}.tar.gz \
    && wget https://github.com/Kong/openresty-patches/archive/master.tar.gz \
    && tar -zxvf master.tar.gz \
    && git clone https://github.com/Kong/lua-kong-nginx-module.git \
    && cd openresty-${OPENRESTY_VERSION} \
    && cd bundle \
	&& for i in ../../openresty-patches-master/patches/${OPENRESTY_VERSION}/*.patch; do patch -p1 < $i; done \
    && cd .. \
    && ./configure \
      --with-cc-opt="-I/usr/local/include/" \
      --with-ld-opt="-L/usr/local/lib -Wl,-u,pcre_version" \
      --add-module=../lua-kong-nginx-module \
      --with-pcre=../pcre-${PCRE_VERSION} \
      --with-openssl=../openssl-${OPENSSL_VERSION} \
      --with-pcre-jit \
      --with-http_ssl_module \
      --with-http_realip_module \
      --with-http_stub_status_module \
      --with-http_v2_module && make && make install \
    && cd .. \
    && cd /lua-kong-nginx-module \
    && install -d /usr/local/share/lua/5.1/resty/kong/ \
    && install -m 664 lualib/resty/kong/*.lua /usr/local/share/lua/5.1/resty/kong/ \
    && cd .. \
    && rm -rf openresty-${OPENRESTY_VERSION}.tar.gz \
    && rm -rf openresty-${OPENRESTY_VERSION} \
    && rm -rf openresty-patches-master \
    && rm -rf master.tar.gz \
    && rm -rf pcre-${PCRE_VERSION} \
    && rm -rf openssl-${OPENSSL_VERSION} \
    && rm -rf lua-kong-nginx-module

RUN wget http://luarocks.github.io/luarocks/releases/luarocks-${LUAROCKS_VERSION}.tar.gz \
    && tar -xzvf luarocks-${LUAROCKS_VERSION}.tar.gz \
    && cd luarocks-${LUAROCKS_VERSION} \
    && ./configure \
      --lua-suffix=jit \
      --with-lua=/usr/local/openresty/luajit \
      --with-lua-include=/usr/local/openresty/luajit/include/luajit-2.1 \
    && make build \
    && make install \
    && cd .. \
    && rm -rf luarocks-${LUAROCKS_VERSION}.tar.gz \
    && rm -rf luarocks-${LUAROCKS_VERSION}

RUN git clone https://github.com/kong/kong.git \
    && cd kong \
    && true \
    && git checkout ${KONG_VERSION} \
    && luarocks make \
      CRYPTO_LIBDIR=/usr/local/lib \
      CRYPTO_INCDIR=/usr/local/include \
      OPENSSL_LIBDIR=/usr/local/lib \
      OPENSSL_INCDIR=/usr/local/include \
    && cp -f bin/kong /usr/local/bin/kong \
    && mkdir /etc/kong \
    && cp -f kong.conf.default /etc/kong/ \
    && cp -f /etc/kong/kong.conf.default /etc/kong/kong.conf \
    && cd .. \
    && mkdir /usr/local/kong \
    && cp /usr/local/openresty/nginx/conf/* /usr/local/kong/ \
    && mkdir /usr/local/kong/logs \
    && rm -rf kong

RUN addgroup --gid 1337 kong \
    && adduser --system --uid 1337 --gid 1337 --disabled-password --no-create-home kong \
    && chown -R kong:0 /usr/local/kong \
    && chmod -R g=u /usr/local/kong

RUN  set -ex; \
     \
     curl -o /usr/local/bin/su-exec.c https://raw.githubusercontent.com/ncopa/su-exec/master/su-exec.c; \
     \
     fetch_deps='gcc libc-dev'; \
     apt-get update; \
     apt-get install -y --no-install-recommends $fetch_deps; \
     rm -rf /var/lib/apt/lists/*; \
     gcc -Wall \
         /usr/local/bin/su-exec.c -o/usr/local/bin/su-exec; \
     chown root:root /usr/local/bin/su-exec; \
     chmod 0755 /usr/local/bin/su-exec; \
     rm /usr/local/bin/su-exec.c; \
     \
     apt-get purge -y --auto-remove $fetch_deps

COPY docker-entrypoint.sh /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 8000 8443 8001 8444

STOPSIGNAL SIGQUIT

CMD ["kong", "docker-start"]

ENV PATH="$PATH:/usr/local/openresty/bin:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin"
