FROM openjdk:7-jre-alpine

ARG NODE_VERSION=v6.1.0
ARG J2V8_VERSION=3a89c4d1adaa9e6dd7eec783cb2404668f725b57
ARG NODE_HASH=9e67ef0b8611e16e6e311eccf0489a50fe76ceebeea3023ef4f51be647ae4bc3
ARG J2V8_HASH=435bb3db2f8713f8d263df25af1fb4e9cc5eb11a32cec6bda81989e37fd6a424

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
    strip --strip-unneeded -R .note -R .comment libj2v8_linux_x86_64.so && \

    # copy J2V8 library to libs
    cp libj2v8_linux_x86_64.so /usr/lib && \

    # cleanup
    apk del build-dependencies && \
    rm -rf /var/cache/apk/* && \
    rm -rf /build
