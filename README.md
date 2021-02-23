# soft-error-coprocessor

## How To Build Fault-Injected Binary on Ubuntu 20.04.1 LTS(for Linux on RV64GC-based processor)
```
# Dependencies
sudo apt install make cmake build-essential

# Fault-Injection LLVM
git clone https://github.com/HPCL-INHA/fault-injector.git llvm
mkdir llvm/build
cd llvm/build
cmake ../ -DCMAKE_BUILD_TYPE=Release
cmake --build . -- -j $(nproc)
# or
cmake --build tools/llc -- -j $(nproc)
cmake --build tools/opt -- -j $(nproc)
cd ../..

# Build clang & lld
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-10.0.1/lld-10.0.1.src.tar.xz
tar -Jxvf lld-10.0.1.src.tar.xz lld
wget https://github.com/llvm/llvm-project/releases/download/llvmorg-10.0.1/clang-10.0.1.src.tar.xz
tar -Jxvf clang-10.0.1.src.tar.xz clang
# directory structure
#   /Sources
#     /llvm
#     /clang
#     /lld
cd ../build
cmake ../ -DLLVM_ENABLE_PROJECTS=lld -DLLVM_ENABLE_PROJECTS=clang -DCMAKE_BUILD_TYPE=Release
cmake --build . -- -j $(nproc)

# Build riscv-gnu-toolchain for Linux
git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
cd riscv-gnu-toolchain
./configure --prefix=/opt/riscv --enable-multilib
sudo make linux -j $(nproc)
cd ..

# RISC-V Binary Build Test
./llvm/build/bin/clang test.c -o test --target=riscv64 -march=rv64gc \
-mabi=lp64d -I./riscv-gnu-toolchain/install-newlib-nano/riscv64-unknown-elf/include \
-L./riscv-gnu-toolchain/install-newlib-nano/riscv64-unknown-elf/lib -fuse-ld=lld