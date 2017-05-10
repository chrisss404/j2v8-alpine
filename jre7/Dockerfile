FROM openjdk:7-jre-alpine

ARG NODE_VERSION=v5.9.1
ARG J2V8_VERSION=9d6ab7fcbed53c1b518c9d547e1fc2480c8da0ba
ARG NODE_HASH=9bfa882ba832c526add1f939121c79d95b9c592785ec8afe043086a0c7a30c18
ARG J2V8_HASH=75bbd078aecb5c8aa755a5d502e7d4cab5f6de32b67455eff2872760c40b0c1f

RUN apk add --no-cache --virtual build-dependencies alpine-sdk python linux-headers binutils-gold && \

    # get node
    mkdir -p /build/node && \
    wget https://nodejs.org/download/release/${NODE_VERSION}/node-${NODE_VERSION}.tar.gz -O /build/node.tar.gz && \
    echo "${NODE_HASH}  /build/node.tar.gz" > /build/node.tar.gz.sha256 && \
    sha256sum -c /build/node.tar.gz.sha256 && \
    tar -xzf /build/node.tar.gz --strip 1 -C /build/node && \

    # build node
    cd /build/node && \
    ./configure --enable-static && \
    make CFLAGS="${CFLAGS} -fPIC" CXXFLAGS="${CXXFLAGS} -fPIC" -j4 && \

    # get J2V8
    mkdir -p /build/J2V8 && \
    wget https://github.com/eclipsesource/J2V8/archive/${J2V8_VERSION}.tar.gz -O /build/J2V8.tar.gz && \
    echo "${J2V8_HASH}  /build/J2V8.tar.gz" > /build/J2V8.tar.gz.sha256 && \
    sha256sum -c /build/J2V8.tar.gz.sha256 && \
    tar -xzf /build/J2V8.tar.gz --strip 1 -C /build/J2V8 && \

    # build J2V8
    cd /build/J2V8/jni && \
    g++ -I /build/node \
        -I /build/node/deps/v8 \
        -I /build/node/deps/v8/include \
        -I /build/node/src \
        -I ${JAVA_HOME} \
        -I ${JAVA_HOME}/linux \
        com_eclipsesource_v8_V8Impl.cpp -std=c++11 -fPIC -shared -o libj2v8_linux_x86_64.so \
        -Wl,--whole-archive ../../node/out/Release/libnode.a \
        -Wl,--no-whole-archive \
        -Wl,--start-group \
            /build/node/out/Release/libv8_libbase.a \
            /build/node/out/Release/libv8_libplatform.a \
            /build/node/out/Release/libv8_base.a \
            /build/node/out/Release/libv8_nosnapshot.a \
            /build/node/out/Release/libuv.a \
            /build/node/out/Release/libopenssl.a \
            /build/node/out/Release/libhttp_parser.a \
            /build/node/out/Release/libgtest.a \
            /build/node/out/Release/libzlib.a \
            /build/node/out/Release/libcares.a \
        -Wl,--end-group \
        -lrt -z noexecstack && \
    strip libj2v8_linux_x86_64.so && \

    # copy J2V8 library to libs
    cp libj2v8_linux_x86_64.so /usr/lib && \

    # cleanup
    apk del build-dependencies && \
    rm -rf /var/cache/apk/* && \
    rm -rf /build
