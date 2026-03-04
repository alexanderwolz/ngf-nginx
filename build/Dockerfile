ARG NGF_VERSION=2.4.2
ARG NGINX_VERSION=1.29.5
ARG ALPINE_VERSION=3.23
ARG GEOIP2_VERSION=3.4


# Stage 1 - build NGINX and additional modules
FROM alpine:${ALPINE_VERSION} AS build

ARG NGINX_VERSION
ARG GEOIP2_VERSION

RUN apk add --no-cache \
    build-base \
    wget \
    libmaxminddb-dev \
    pcre2-dev \
    openssl-dev \
    zlib-dev

WORKDIR /build

RUN wget -O nginx.tar.gz "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" \
    && mkdir nginx && tar xzf nginx.tar.gz -C nginx --strip-components 1

RUN wget -O geoip2.tar.gz "https://github.com/leev/ngx_http_geoip2_module/archive/refs/tags/${GEOIP2_VERSION}.tar.gz" \
    && mkdir geoip2 && tar xzf geoip2.tar.gz -C geoip2 --strip-components 1

WORKDIR /build/nginx

RUN ./configure \
    --with-compat \
    --add-dynamic-module=/build/geoip2 \
    && make modules


# Stage 2 - run NGINX Alpine container
FROM ghcr.io/nginx/nginx-gateway-fabric/nginx:${NGF_VERSION}
LABEL maintainer="mail@alexanderwolz.de"
ARG NGF_VERSION

USER root

RUN chmod 644 /etc/nginx-agent/nginx-agent.conf

RUN apk add --no-cache libmaxminddb

COPY --from=build /build/nginx/objs/ngx_http_geoip2_module.so /usr/lib/nginx/modules/

RUN mkdir -p /etc/nginx/main-includes \
    && echo "load_module modules/ngx_http_geoip2_module.so;" > /etc/nginx/main-includes/50-mod-http-geoip2.conf

RUN mkdir -p /etc/nginx/geoip \
    && chown -R nginx:nginx /etc/nginx/geoip

USER nginx
