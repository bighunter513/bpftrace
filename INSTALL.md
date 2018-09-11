# Linux Kernel

Your kernel needs to be built with the following options:
```
CONFIG_BPF=y
CONFIG_BPF_SYSCALL=y
CONFIG_BPF_JIT=y
CONFIG_HAVE_BPF_JIT=y
CONFIG_BPF_EVENTS=y
```

To use some BPFtrace features, minimum kernel versions are required:
- 4.1+ - kprobes
- 4.3+ - uprobes
- 4.6+ - stack traces, count and hist builtins (use PERCPU maps for accuracy and efficiency)
- 4.7+ - tracepoints
- 4.9+ - timers/profiling


# Building BPFtrace

## Native build process

### Requirements

- A C++ compiler
- CMake
- Flex
- Bison
- LLVM & Clang 5.0 (or 6.0) development packages
- LibElf

### Compilation

See previous requirements, and specific targets in the sections that follow (Ubuntu, Docker).

```
git clone https://github.com/iovisor/bpftrace
mkdir -p bpftrace/build
cd bpftrace/build
cmake -DCMAKE_BUILD_TYPE=Debug ../
make
```

By default bpftrace will be built as a dynamically linked executable. If a statically linked executable would be preferred and your system has the required libraries installed, the CMake option `-DSTATIC_LINKING:BOOL=ON` can be used. Building bpftrace using the Docker method below will always result in a statically linked executable.

The latest versions of BCC and Google Test will be downloaded on each build. To speed up builds and only download their sources on the first run, use the CMake option `-DOFFLINE_BUILDS:BOOL=ON`.

To test that the build works, you can try running the test suite, and a one-liner:

```
./tests/bpftrace_test
./src/bpftrace -e 'kprobe:do_nanosleep { printf("sleep by %s\n", comm); }'
```

## Ubuntu

The llvm/clang packages that are currently available for Ubuntu have an issue, so we'll use the ones from llvm.org for now. The build instructions are:

```
cat <<EOF | sudo tee -a /etc/apt/sources.list
# from https://apt.llvm.org/:
deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial main
deb-src http://apt.llvm.org/xenial/ llvm-toolchain-xenial main
# 5.0
deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial-5.0 main
deb-src http://apt.llvm.org/xenial/ llvm-toolchain-xenial-5.0 main
# 6.0
deb http://apt.llvm.org/xenial/ llvm-toolchain-xenial-6.0 main
deb-src http://apt.llvm.org/xenial/ llvm-toolchain-xenial-6.0 main
EOF
sudo apt-get update
sudo apt-get install -y bison cmake flex g++ git libelf-dev zlib1g-dev libfl-dev
sudo apt-get install clang-5.0 libclang-5.0-dev libclang-common-5.0-dev libclang1-5.0 libllvm5.0 llvm-5.0 llvm-5.0-dev llvm-5.0-runtime
git clone https://github.com/iovisor/bpftrace
cd bpftrace
mkdir build; cd build; cmake -DCMAKE_BUILD_TYPE=DEBUG ..
make -j8
```

The bpftrace binary will be in src/bpftrace under the build directory.

## Using Docker

Building inside a Docker container will produce a statically linked bpftrace executable.

`./build.sh`

There are some more fine-grained options if you find yourself building BPFtrace a lot:
- `./build-docker-image.sh` - builds just the `bpftrace-builder` Docker image
- `./build-debug.sh` - builds BPFtrace with debugging information (requires `./build-docker-image.sh` to have already been run)
- `./build-release.sh` - builds BPFtrace in a release configuration (requires `./build-docker-image.sh` to have already been run)

`./build.sh` is equivalent to `./build-docker-image.sh && ./build-release.sh`