ARG GH_REPO=ghcr.io/btbn/ffmpeg-builds
ARG BASE_TAG_SUFFIX=
FROM $GH_REPO/base:latest${BASE_TAG_SUFFIX}
# Prepare "cross" environment to heavily favour static builds
RUN apt install binutils clang -y
RUN mkdir /opt/ffbuild

ENV FFBUILD_TOOLCHAIN=x86_64-unknown-linux-gnu \
    FFBUILD_RUST_TARGET="x86_64-unknown-linux-gnu"

RUN \
    rustup default nightly && \
    echo "[unstable]\ntarget-applies-to-host = true\nhost-config = true\n" > "$CARGO_HOME"/config.toml && \
    echo "[target.$FFBUILD_RUST_TARGET]\nlinker = \"${FFBUILD_TOOLCHAIN}-gcc\"\nar = \"${FFBUILD_TOOLCHAIN}-gcc-ar\"\n" >> "$CARGO_HOME"/config.toml && \
    echo "[target.host]\nlinker = \"gcc\"\nar = \"ar\"\n" >> "$CARGO_HOME"/config.toml

ADD toolchain.cmake /toolchain.cmake
ADD cross.meson /cross.meson

ADD gen-implib.sh /usr/bin/gen-implib
RUN git clone --filter=blob:none --depth=1 https://github.com/yugr/Implib.so /opt/implib

ENV PATH="${PATH}" \
    FFBUILD_TARGET_FLAGS="--pkg-config=pkg-config --cross-prefix=x86_64-linux-gnu- --arch=x86_64 --target-os=linux" \
    FFBUILD_CROSS_PREFIX="x86_64-linux-gnu-" \
    FFBUILD_PREFIX=/opt/ffbuild \
    FFBUILD_DESTDIR=/opt/ffdest \
    FFBUILD_DESTPREFIX=/opt/ffdest/opt/ffbuild \
    PKG_CONFIG=pkg-config \
    PKG_CONFIG_LIBDIR=/opt/ffbuild/lib/pkgconfig:/opt/ffbuild/share/pkgconfig \
    CC="clang" \
    CXX="clang++" \
    LD="ld" \
    AR="ar" \
    RANLIB="ranlib" \
    NM="nm" \
    CFLAGS="-I/opt/ffbuild/include -O3 -march=skylake -pipe -fPIC -DPIC -D_FORTIFY_SOURCE=2 -fstack-protector-strong -fstack-clash-protection -pthread" \
    CXXFLAGS="-I/opt/ffbuild/include -O3 -march=skylake -pipe -fPIC -DPIC -D_FORTIFY_SOURCE=2 -fstack-protector-strong -fstack-clash-protection -pthread" \
    LDFLAGS="-L/opt/ffbuild/lib -O3 -march=skylake -pipe -fstack-protector-strong -fstack-clash-protection -Wl,-z,relro,-z,now -pthread -lm" \
    STAGE_CFLAGS="" \
    STAGE_CXXFLAGS=""
