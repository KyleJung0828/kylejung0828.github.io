### Pre-requisites

1. Cygwin (in which you will be building the twk and base codes) - Install basic packages:
- clang-cl
- git
- python3
2. CMake GUI for Windows
3. Visual Studio 2019 (Community Version)
4. Build Tools for Visual Studio 2019 (For MSVC Compiler and Windows SDK)

### Prepare a build directory.

```
$ cd twk-for-windows
$ mkdir build
```

### Open the root CMakeList.txt file to edit the path for Windows SDK (assuming you have already installed it)

```
$ vi CMakeLists.txt # On the project's root directory
```
After opening it, scroll down until you see something like below:
```
# Windows SDK directories
link_directories(/cygdrive/c/Program\ Files\ \(x86\)/Microsoft\ Visual\ Studio\ 12.0/VC/lib)
include_directories(/cygdrive/c/Program\ Files\ \(x86\)/Microsoft\ Visual\ Studio\ 12.0/VC/include)
```
Edit link_directories and include_directories to your Windows SDK path.

### Generating Build File using CMake GUI

1. Open CMake GUI.
2. Select `twk-for-windows/` as source path and `twk-for-windows/build/` as build path (Reminder: You must have created an empty "build" directory. Cmake will fill that directory after the configuration and generation).
3. Click on "Configure" button, and refer to the picture below for settings (Choose Visual Studio 2019 and x64 for generator and platform settings, respectively).
![](assets/cmake_generator_config.png)
![](assets/cmake_done.png)
4. Click on "Generate" button. You must check "Configuring Done" and "Generating Done" messages as you can see in the picture above.
5. Upon successful configuration and generation, you will be able to see a `.sln` file in `twk-for-windows/build/` directory. This is your build file.

### Building

1. Open the `.sln` file in Visual Studio 2019. Start building by entering `ctrl+shift+B`.
2. Deal with build errors accordingly.

### Building lib/base

1. Ignore `base/logging.h` (and any compile errors associated with it. For e.g., comment out the lines with DQ_CHECK, etc.)
2. Ignore `base/Debug/`
3. Ignore any unittest/gtest files (e.g., *unittest.cc, base/gtest_prod_util.h. Comment them out)

Do not hesitate to ask me any questions (kwanghyun_jung@tmax.co.kr).
