# C++ – toolchain, build và thư viện theo dự án

Cập nhật lần cuối: 2026-11-14

C++ không có "version manager" chuẩn như Node/Python. Cô lập bằng 3 thứ: **compiler/toolchain**, **CMake** (hệ build), **vcpkg hoặc Conan** (thư viện theo dự án, không cài vào hệ thống).

## 1. Compiler

**Windows**
```powershell
winget install Microsoft.VisualStudio.2022.BuildTools --override "--quiet --wait --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended"
winget install Kitware.CMake
winget install Ninja-build.Ninja
winget install Git.Git
```
Mở "Developer PowerShell for VS" (hoặc "x64 Native Tools Command Prompt") để có `cl.exe`. Có thể thay bằng LLVM/clang (`winget install LLVM.LLVM`) hoặc MSYS2/MinGW.

**macOS**
```bash
xcode-select --install          # clang + make
brew install cmake ninja
# Cần clang/GCC bản khác: 
brew install llvm gcc
```
Nhiều compiler cùng lúc: chọn bằng biến môi trường `CC` / `CXX` hoặc trong CMake preset.

## 2. Build với CMake + Presets (pin cấu hình trong repo)
```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.25)
project(myapp LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_executable(myapp src/main.cpp)
```
```json
// CMakePresets.json
{
  "version": 6,
  "configurePresets": [
    { "name": "default", "generator": "Ninja", "binaryDir": "${sourceDir}/build" }
  ]
}
```
```bash
cmake --preset default
cmake --build build
```
Thư mục `build/` không commit.

## 3. Thư viện theo dự án với vcpkg (manifest mode)
```bash
git clone https://github.com/microsoft/vcpkg.git ~/vcpkg      # Windows: C:\vcpkg
~/vcpkg/bootstrap-vcpkg.sh                                     # Windows: .\bootstrap-vcpkg.bat
vcpkg new --application
vcpkg add port fmt
```
`vcpkg.json` ghi dependency của dự án. Cấu hình CMake dùng toolchain:
```bash
cmake -B build -DCMAKE_TOOLCHAIN_FILE=$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake
```
Đặt biến `VCPKG_ROOT` trỏ tới thư mục vcpkg. Thay thế: **Conan** (`pip install conan` trong venv).

## Lỗi thường gặp
- Windows: `cl` không nhận ra: chưa mở Developer PowerShell.
- macOS: lỗi header sau khi cập nhật macOS: chạy lại `xcode-select --install`.
- Khác máy ra kết quả khác: chưa chốt version thư viện, hãy dùng `builtin-baseline` trong `vcpkg.json`.

Cần cô lập hoàn toàn: dùng [Dev Container](devcontainer.md).
