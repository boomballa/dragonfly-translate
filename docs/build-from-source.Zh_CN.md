# 从源代码构建 DragonflyDB

## 运行服务

Dragonfly 在 Linux 上运行。我们建议在 Linux 5.11 或更高版本上运行它，但您也可以在较旧的内核上运行 Dragonfly。


> :warning: **Dragonfly 版本使用LTO (link time optimization)进行编译**:
  根据工作负载，这可以显着提高性能。 如果你想对Dragonfly进行基准测试或者在生产中
  使用它，你应该给予`blaze.sh` 脚本`-DCMAKE_CXX_FLAGS="-flto"` 参数来开启LTO.

## 第 1 步 - 安装依赖项

在 Debian/Ubuntu 环境:

```bash
sudo apt install ninja-build libunwind-dev libboost-fiber-dev libssl-dev \
     autoconf-archive libtool cmake g++ libzstd-dev bison libxml2-dev
```

在 Fedora 环境:

```bash
sudo dnf install -y automake boost-devel g++ git cmake libtool ninja-build libzstd-devel  \
     openssl-devel libunwind-devel autoconf-archive patch bison libxml2-devel libstdc++-static
```

在 openSUSE 环境:

```bash
sudo zypper install automake boost-devel gcc-c++ git cmake libtool ninja libzstd-devel  \
     openssl-devel libunwind-devel autoconf-archive patch bison libxml2-devel \
     libboost_context-devel libboost_system-devel
```

## 第 2 步 - 克隆项目

```bash
git clone --recursive https://github.com/dragonflydb/dragonfly && cd dragonfly
```

## 第 3 步 - 配置 & 构建

```bash
# 配置构建
./helio/blaze.sh -release

# 构建
cd build-opt && ninja dragonfly

```

## 第 4 步 - 启动实例

```bash
# 运行命令
./dragonfly --alsologtostderr

```

Dragonfly DB 可以快速响应 `http` 和 `redis` 请求！

你可以使用 `redis-cli` 客户端去连接 `localhost:6379` 或者打开浏览器去访问 `http://localhost:6379`

## 步骤 5

通过redis客户端连接服务

```bash
redis-cli
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> keys *
1) "hello"
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379>
```

## 步骤 6

继续发挥，利用 DragonflyDB 的强大功能构建您的应用程序！