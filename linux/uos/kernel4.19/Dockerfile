# 使用 buildpack-deps:bullseye-curl 作为基础镜像
FROM buildpack-deps:bullseye-curl

# 安装 build-essential, rpm, python, 交叉编译 arm64 gcc
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      build-essential rpm python python3 gawk bison \
      gcc-aarch64-linux-gnu g++-aarch64-linux-gnu && \
    rm -rf /var/lib/apt/lists/*

# 设置 gcc默认指向 aarch64-linux-gnu-gcc
RUN update-alternatives --install \
      /usr/bin/gcc gcc /usr/bin/aarch64-linux-gnu-gcc 10 \
      --slave /usr/bin/g++ g++ /usr/bin/aarch64-linux-gnu-g++

# 下载并编译安装 glibc 2.28 aarch64
RUN wget http://ftp.gnu.org/gnu/libc/glibc-2.28.tar.gz && \
    tar -xzf glibc-2.28.tar.gz && \
    cd glibc-2.28 && \
    mkdir build && cd build && \
    ../configure --prefix=/usr/local/glibc-2.28-aarch64 \
      --host=aarch64-linux-gnu --enable-kernel=4.19 --with-headers=/usr/aarch64-linux-gnu/include \
      CFLAGS="-Wno-error -O2" CXXFLAGS="-O2" && \
    make -j$(nproc) && \
    make install && \
    cd ../../ && \
    rm -rf glibc-2.28 glibc-2.28.tar.gz

# 安装 Node.js 20.17.0
RUN set -ex; \
    NODE_VERSION=20.17.0; \
    wget -qO- https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz | tar -xJ -C /usr/local --strip-components=1

# 删除不必要的文件
RUN unlink /usr/local/CHANGELOG.md && \
    unlink /usr/local/LICENSE && \
    unlink /usr/local/README.md

# 使用 npm 安装全局 pnpm
RUN npm install -g pnpm

# 设置 环境变量
ENV LD_LIBRARY_PATH=/usr/local/glibc-2.28-aarch64/lib
ENV LDFLAGS="-L/usr/local/glibc-2.28-aarch64/lib -Wl,-rpath,/usr/local/glibc-2.28-aarch64/lib"
ENV CPPFLAGS="-I/usr/local/glibc-2.28-aarch64/include"

# 默认命令
CMD ["bash"]