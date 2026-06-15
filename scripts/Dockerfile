FROM alpine:3.16 AS builder
LABEL maintainer="tindy.it@gmail.com"
ARG THREADS="4"
ARG SHA=""
ARG UPDATE_RULES="0"

WORKDIR /build

# build third-party dependencies
RUN set -xe && \
    apk add --no-cache git g++ build-base linux-headers cmake python3 py3-pip && \
    apk add --no-cache curl-dev rapidjson-dev pcre2-dev yaml-cpp-dev pkgconf && \
    git clone https://github.com/ftk/quickjspp --depth=1 && \
    cd quickjspp && \
    git submodule update --init && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make quickjs -j ${THREADS} && \
    install -d /usr/lib/quickjs/ && \
    install -m644 quickjs/libquickjs.a /usr/lib/quickjs/ && \
    install -d /usr/include/quickjs/ && \
    install -m644 quickjs/quickjs.h quickjs/quickjs-libc.h /usr/include/quickjs/ && \
    install -m644 quickjspp.hpp /usr/include/ && \
    cd /build && \
    git clone https://github.com/PerMalmberg/libcron --depth=1 && \
    cd libcron && \
    git submodule update --init && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make libcron -j ${THREADS} && \
    install -m644 libcron/out/Release/liblibcron.a /usr/lib/ && \
    install -d /usr/include/libcron/ && \
    install -m644 libcron/include/libcron/* /usr/include/libcron/ && \
    install -d /usr/include/date/ && \
    install -m644 libcron/externals/date/include/date/* /usr/include/date/ && \
    cd /build && \
    git clone https://github.com/ToruNiina/toml11 --branch="v4.3.0" --depth=1 && \
    cd toml11 && \
    cmake -DCMAKE_CXX_STANDARD=11 . && \
    make install -j ${THREADS}

# use local source instead of cloning subconverter from GitHub
COPY . /build/subconverter

WORKDIR /build/subconverter

RUN set -xe && \
    if [ -n "${SHA}" ]; then \
        sed -i 's/\(v[0-9]\.[0-9]\.[0-9]\)/\1-'"${SHA}"'/' src/version.h; \
    fi && \
    if [ "${UPDATE_RULES}" = "1" ]; then \
        pip3 install gitpython && \
        python3 scripts/update_rules.py -c scripts/rules_config.conf; \
    fi && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make -j ${THREADS}

FROM alpine:3.16
LABEL maintainer="tindy.it@gmail.com"

RUN apk add --no-cache pcre2 libcurl yaml-cpp

COPY --from=builder /build/subconverter/subconverter /usr/bin/
COPY --from=builder /build/subconverter/base /base/

ENV TZ=Africa/Abidjan
RUN ln -sf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone

WORKDIR /base
CMD ["subconverter"]

EXPOSE 25500/tcp
