ARG NGF_VERSION=2.4.2
ARG NGINX_VERSION=1.29.5
ARG ALPINE_VERSION=3.23
ARG GEOIP2_VERSION=3.4

# Stage 1 - build NGINX and additional modules
FROM alpine:${ALPINE_VERSION} AS build
LABEL maintainer="mail@alexanderwolz.de"

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

# Patch nginx.conf: add load_module and http-includes
RUN sed -i 's|load_module modules/ngx_http_js_module.so;|load_module modules/ngx_http_js_module.so;\nload_module modules/ngx_http_geoip2_module.so;|' /etc/nginx/nginx.conf \
    && sed -i 's|include /etc/nginx/conf.d/\*.conf;|include /etc/nginx/conf.d/*.conf;\n  include /etc/nginx-custom/http-includes/*.conf;|' /etc/nginx/nginx.conf

# GeoIP2 http context directives
RUN mkdir -p /etc/nginx-custom/http-includes \
    && chown nginx:nginx /etc/nginx-custom/http-includes
COPY --chown=nginx:nginx http-includes/geoip2.conf /etc/nginx-custom/http-includes/geoip2.conf

# GeoIP2 database directory with sample placeholder files
RUN mkdir -p /etc/nginx-custom/geoip && chown -R nginx:nginx /etc/nginx-custom/geoip
COPY --chown=nginx:nginx geoip/sample.mmdb /etc/nginx-custom/geoip/GeoLite2-Country.mmdb
COPY --chown=nginx:nginx geoip/sample.mmdb /etc/nginx-custom/geoip/GeoLite2-City.mmdb

# Defaults
RUN mkdir -p /etc/nginx-custom/geoip-defaults && chown -R nginx:nginx /etc/nginx-custom/geoip-defaults
COPY --chown=nginx:nginx geoip/sample.mmdb /etc/nginx-custom/geoip-defaults/GeoLite2-Country.mmdb
COPY --chown=nginx:nginx geoip/sample.mmdb /etc/nginx-custom/geoip-defaults/GeoLite2-City.mmdb

USER nginx
