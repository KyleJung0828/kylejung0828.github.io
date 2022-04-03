# Clang-cl과 Ninja로 Windows 개발 환경 구축하기

최근 회사에서 Linux 환경에서 개발했던 프로젝트를 Windows 환경에서 cross-platform 빌드하는 업무를 맡았습니다. Linux에서는 `Bash + CMake + Clang + Ninja`의 조합으로 개발하였는데, Windows에서 프로젝트를 빌드하려고 웹 서치를 해봐도 대부분 MSVC (Visual Studio) 환경에서 빌드하는 가이드밖에 찾을 수 없었습니다.

이는 두 가지 문제점을 발생시켰는데, 가장 큰 문제는 Visual Studio 라이선스였고 (Enterprise용의 경우, 계정 신규에만 700만원), 또 다른 문제는 Clang에서 CL로 컴파일러 프론트엔드를 변경하여 적응해야 한다는 점이었습니다. 변경해야 되는 부분이 많을수록 개발자들이 환경 구축에 많은 시간을 할애해야 한다는 점은 무시하지 못하는 단점이라고 생각했습니다. 그래서 저는 Windows에서 Linux 환경을 최대한 비슷하게 유지하여 개발 전환 비용을 낮추도록 노력해봤고, 그 과정에 대해 공유하고자 합니다.

---

### 개발 환경
- 터미널: Cygwin64
- 빌드 파일 구성: CMake (v3.15)
- 컴파일러: Clang-cl (LLVM v9.0.0) + Ninja (v1.9.0)

### Cygwin64 설치
* Linux Terminal과 비슷한 환경에서 작업하고 싶다면 Cygwin을 설치해야 합니다.
* [Cygwin64 설치 파일 다운로드][1]
* 설치 파일 실행 시, 아래 그림과 같이 중간에 패키지 매니저 창이 나오는데, 필요한 패키지를 반드시 설치해야 합니다.
* View 드롭다운에서 Full을 선택 후, Search bar에 설치하고 싶은 패키지를 입력하여 최신 버전으로 체크합니다.
* **반드시 설치해야 하는 패키지들 : git, vim**
* **Optional 패키지들 : tmux, python3-dev, perl**

![](/Images/2019-10-17-clang-ninja-on-windows/cyg64_pkgmgr.png)

**중요: clang, cmake는 설치하지 않습니다. Cygwin64 패키지 매니저를 통해 설치한 clang은 /usr/bin/clang에 위치하고, gcc처럼 동작하기 때문에 Windows에서 설치한 clang보다 우선순위가 높게 호출되어 오류를 발생시킵니다.**

### CMake for Windows 설치
* [CMake v3.15 다운로드 링크][2]를 통해 설치

### Clang-cl 설치
* Windows 컴파일러 프론트엔드인 CL.exe의 드라이버인 Clang-cl 사용을 위해 LLVM for windows의 설치가 필요.
* [LLVM v9.0.0 다운로드 링크][3]를 통해 설치
* 설치 경로를 일반적인 경로인 `C:\Program Files\LLVM`으로 하는 것을 권장.

### Ninja 설치
* [Ninja For Windows (v.1.9.0) 다운로드 링크][4]를 통해 zip 파일 다운로드.
* 압축 해제 후 ninja.exe 바이너리가 생기는데, 환경 변수 추가를 위해 임의의 디렉토리인 `C:\Program Files\ninja`를 생성하여 그 곳으로 옮깁니다. (또는 `C:\Windows\System32`에 넣을 수 있습니다)

### **(중요)** 환경변수 설정
* 아래 그림과 같이 `시스템 속성 > 환경 변수 > 시스템 변수 > Path 변수 편집` 을 선택하여 `환경 변수 편집` 창을 띄우고, LLVM과 Ninja 바이너리의 경로를 추가합니다.
* 환경변수 설정 해주지 않으면 cmake가 clang-cl compiler나 ninja를 못 찾습니다.

![](/Images/2019-10-17-clang-ninja-on-windows/syspath.png)

### Windows MSVC SDK 설치 및 로딩
* Windows MSVC SDK를 설치하지 않거나, 설치했더라도 로딩하지 않으면 다음과 같이 rc 관련 에러가 납니다.

```
[2/2] Linking C executable cmTC_56f76.exe
    FAILED: cmTC_56f76.exe
    cmd.exe /C "cd . && "C:\Program Files\CMake\bin\cmake.exe" -E vs_link_exe --intdir=CMakeFiles\cmTC_56f76.dir --rc=rc --mt=CMAKE_MT-NOTFOUND --manifests  -- CMAKE_LINKER-NOTFOUND /nologo CMakeFiles\cmTC_56f76.dir\testCCompiler.c.obj  /out:cmTC_56f76.exe /implib:cmTC_56f76.lib /pdb:cmTC_56f76.pdb /version:0.0  /machine:x64  /debug /INCREMENTAL /subsystem:console  kernel32.lib user32.lib gdi32.lib winspool.lib shell32.lib ole32.lib oleaut32.lib uuid.lib comdlg32.lib advapi32.lib && cd ."
    RC Pass 1: command "rc /fo CMakeFiles\cmTC_56f76.dir/manifest.res CMakeFiles\cmTC_56f76.dir/manifest.rc" failed (exit code 0) with the following output:
    지정된 파일을 찾을 수 없습니다
    ninja: build stopped: subcommand failed.
```

* SDK 설치: [MS 홈페이지][5] 방문하여 최신 SDK 설치.
* SDK 로딩: SDK 경로에서 64비트 아키텍쳐에 맞는 batch file 실행 (일반적으로 `C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\VC\Auxiliary\Build`에 위치)

```
$ cd "C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\VC\Auxiliary\Build"
$ vcvars64.bat (`vcvarsall.bat x64`와 같음)
```

### 빌드 디렉토리 생성
* 프로젝트 build 디렉토리에 Windows 빌드를 위한 `win` 디렉토리 생성

```
$ cd ${PROJECT_PATH}/build (e.g., cd ~/myproj/build)
$ mkdir win
$ cd win
```

* 만약 `win` 디렉토리가 이미 존재한다면 `rd /s /q win`을 호출하여 삭제 후 `mkdir win`을 실행.

### 7. CMake를 이용하여 ninja 빌드 파일 생성

```
$ pwd
${PROJECT_PATH}/build/win (e.g., /home/kyle/myproj/build/win)
$ cmake -H. -GNinja -Bbuild -DCMAKE_SYSTEM_NAME=Windows  -DCMAKE_BUILD_TYPE=Debug -DCMAKE_EXPORT_COMPILE_COMMANDS=ON ../../src (../../src 경로는 프로젝트 소스 경로로 지정)
```

* Configuring done, Generating done 이라는 메시지가 뜨면 성공.
* `${PROJECT_PATH}/build/win/build` 디렉토리로 이동하여 `ninja` 실행하면 빌드가 진행됩니다.

### 배치 파일 실행
* SDK 로딩 > 빌드 디렉토리 생성 > cmake의 3단계를 간소화시킨 batch file을 실행하면 편리합니다 (다만, step by step으로 직접 쳐보면서 개발 환경을 구성하는 것을 권장합니다).
* Batch file 예제 (`~/myproj/build/wininit.bat`)
```
echo "========== Loading Windows SDK... ==============="
cd "C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\VC\Auxiliary\Build"
call vcvars64.bat
IF %ERRORLEVEL% neq 0 (
  EXIT /B %ERRORLEVEL%
)
echo "========== Windows SDK Loaded. ================"
cd "C:\cygwin\home\kyle\myproj/build"
rd /s /q win
mkdir win
cd win
echo "========== Running CMake... ==================="
cmake -H. -GNinja -Bbuild -DCMAKE_SYSTEM_NAME=Windows -DCMAKE_BUILD_TYPE=Debug -DCMAKE_EXPORT_COMPILE_COMMANDS=ON ../../src
IF %ERRORLEVEL% neq 0 (
  EXIT /B %ERRORLEVEL%
)
cd build
echo "========== Init for Windows Done. Do ninja ========="
```

Batch file을 만들었다면, 다음과 같이 실행하면 됩니다.
```
$ cd ~/myproj/build
$ ./wininit.bat
$ ninja
```

[1]:"https://cygwin.com/setup-x86_64.exe"
[2]:"https://github.com/Kitware/CMake/releases/download/v3.15.4/cmake-3.15.4-win64-x64.msi"
[3]:"http://releases.llvm.org/9.0.0/LLVM-9.0.0-win64.exe"
[4]:"https://github.com/ninja-build/ninja/releases/download/v1.9.0/ninja-win.zip"
[5]:"https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools&rel=16"





















# ToOffice For Windows 개발 환경 구축 가이드라인

>기존 Linux 환경에서 개발 환경인 `Bash + CMake + Clang + Ninja` 를 Windows에서 최대한 비슷하게 유지하여 개발 전환 비용을 낮추기 위한 개발 환경 구축 가이드라인입니다.

---

### 개발 환경
- 터미널: cmd.exe
- 빌드 파일 구성: CMake (v3.15)
- 컴파일러: Clang-cl (LLVM v9.0.0) + Ninja (v1.9.0)

### 0. Windows에 Office Repository Clone 하기
* Windows용 SSH Key 추가합니다.
* Windows에서 Office Repository를 clone하기 위해 Windows용 Git Bash (https://github.com/git-for-windows/git/releases/download/v2.23.0.windows.1/Git-2.23.0-64-bit.exe) 나 Cygwin64 (문서 밑에 Cygwin64 설정 매뉴얼 참조)를 설치합니다. (Cygwin64 추천)
* Git Bash / Cygwin64를 켜고, 원하는 경로에 Office를 Clone 받습니다. ***(Cygwin64 사용자들은 home directory (~/)에 office를 clone 받는 것을 권장합니다.)***

```
$ git clone git@192.168.105.92:tmaxoffice/office.git
```

* Office clone 후 Windows 작업 Branch로 checkout 합니다.

```
$ cd office
$ git checkout f/pk1-2/For_Windows_Ninja/kwanghyun_jung
```

***중요: git checkout 후 Git Bash/Cygwin64를 종료합니다. 이후 작업은 모두 cmd.exe를 켜고 진행합니다.***

### 1. CMake for Windows 설치
* [CMake v3.15 다운로드 링크][2] (https://github.com/Kitware/CMake/releases/download/v3.15.4/cmake-3.15.4-win64-x64.msi) 를 통해 설치

### 2. Clang-cl 설치
* Windows 컴파일러 프론트엔드인 CL.exe의 드라이버인 Clang-cl 사용을 위해 LLVM for windows의 설치가 필요합니다.
* [LLVM v9.0.0 다운로드 링크][3] (http://releases.llvm.org/9.0.0/LLVM-9.0.0-win64.exe) 를 통해 설치합니다.
* 설치 경로를 일반적인 경로인 `C:\Program Files\LLVM`으로 하는 것을 권장합니다.

### 3. Ninja 설치
* [Ninja For Windows (v.1.9.0) 다운로드 링크][4] (https://github.com/ninja-build/ninja/releases/download/v1.9.0/ninja-win.zip) 를 통해 zip 파일 다운로드.
* 압축 해제 후 ninja.exe 바이너리가 생기는데, 환경 변수 추가를 위해 `C:\Program Files\ninja` 라는 디렉토리를 생성하여 그 곳으로 옮깁니다. (또는 `C:\Windows\System32`에 넣으면 OS가 자동으로 인식하니, 두 가지 방법 중 편한 것을 선택하면 됩니다)

### 4. **(중요)** 환경변수 설정
* 아래 그림과 같이 `시스템 속성 > 환경 변수 > 시스템 변수 > Path 변수 편집` 을 선택하여 `환경 변수 편집` 창을 띄우고, LLVM, Ninja, CMake 바이너리의 경로를 추가합니다.
* 환경변수 설정 해주지 않으면 cmake가 clang-cl compiler나 ninja를 못 찾습니다.

![](syspath.png)

### 5. Windows MSVC SDK 설치 및 로딩
* Windows MSVC SDK를 설치하지 않거나, 설치했더라도 로딩하지 않으면 CMake 명령 입력 시 다음과 같이 rc 관련 에러가 납니다.

```
[2/2] Linking C executable cmTC_56f76.exe
    FAILED: cmTC_56f76.exe
    cmd.exe /C "cd . && "C:\Program Files\CMake\bin\cmake.exe" -E vs_link_exe --intdir=CMakeFiles\cmTC_56f76.dir --rc=rc --mt=CMAKE_MT-NOTFOUND --manifests  -- CMAKE_LINKER-NOTFOUND /nologo CMakeFiles\cmTC_56f76.dir\testCCompiler.c.obj  /out:cmTC_56f76.exe /implib:cmTC_56f76.lib /pdb:cmTC_56f76.pdb /version:0.0  /machine:x64  /debug /INCREMENTAL /subsystem:console  kernel32.lib user32.lib gdi32.lib winspool.lib shell32.lib ole32.lib oleaut32.lib uuid.lib comdlg32.lib advapi32.lib && cd ."
    RC Pass 1: command "rc /fo CMakeFiles\cmTC_56f76.dir/manifest.res CMakeFiles\cmTC_56f76.dir/manifest.rc" failed (exit code 0) with the following output:
    지정된 파일을 찾을 수 없습니다
    ninja: build stopped: subcommand failed.
```

* SDK 설치: [MS 홈페이지][5] (https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools&rel=16) 방문하여 최신 SDK를 설치합니다.
* 설치 창에서 아래 그림과 같이 `Visual C++ 빌드 도구`를 선택한 후 설치합니다.

![](winsdk.png)

* SDK 로딩: SDK 경로에서 64비트 아키텍쳐에 맞는 batch file 실행 (일반적으로 `C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\VC\Auxiliary\Build`에 위치)

```
$ cd "C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\VC\Auxiliary\Build"
$ vcvars64.bat (`vcvarsall.bat x64`와 같음)
```
### 6. CMake를 이용하여 ninja 빌드 파일 생성
* 아래 명령어를 통해 ninja 빌드 파일을 생성합니다 (`office/build/win`에 빌드 파일이 생성)

```
$ cd {OFFICE_PATH}/build (e.g., cd "C:\cygwin64\home\Tmax\office\build")
$ cmake -H. -GNinja -Bwin -DCMAKE_SYSTEM_NAME=Windows  -DCMAKE_BUILD_TYPE=Debug -DCMAKE_EXPORT_COMPILE_COMMANDS=ON -S../src
```

* 아래 그림과 같이 Configuring done, Generating done 이라는 메시지가 뜨면 CMake가 성공적으로 완료된 것입니다.
* 이제 `office/build/win`으로 이동하여 `ninja` 실행하면 ninja 빌드가 가능합니다.

![](cmake_done.png)

### (Optional) - Cygwin64 설치
* Linux Terminal과 비슷한 환경에서 작업하고 싶다면 Cygwin64 Terminal을 사용할 수 있습니다.
* ***주의: Cygwin64로 ninja를 빌드하는 것은 cmd로 ninja 빌드 환경을 모두 구축한 후 넘어가는 것을 권장합니다. 그렇지 않을 경우 예상치 못한 에러가 발생합니다.***
* [Cygwin64 설치 파일 다운로드][6] (https://cygwin.com/setup-x86_64.exe)
* 설치 파일 실행 시, 아래 그림과 같이 중간에 패키지 매니저 창이 나오는데, 필요한 패키지를 반드시 설치해야 합니다.
* View 드롭다운에서 Full을 선택 후, Search bar에 설치하고 싶은 패키지를 입력하여 최신 버전으로 체크합니다.
* **반드시 설치해야 하는 패키지들 : git, vim**
* **Optional 패키지들 : tmux, python3-dev, perl**

![](cyg64_pkgmgr.png)

***중요: clang, cmake는 설치하지 않습니다. Cygwin64 패키지 매니저를 통해 설치한 clang은 /usr/bin/clang에 위치하고, gcc처럼 동작하기 때문에 Windows에서 설치한 clang보다 우선순위가 높게 호출되어 오류를 발생시킵니다.***

### (Optional) - Batch File 실행
* 5-6번 과정을 단축시킨 `${OFFICE_PATH}/build/wininit.bat` batch file을 실행하면 편리합니다 (다만, step by step으로 직접 쳐보면서 개발 환경을 구성하는 것을 권장합니다).
* `wininit.bat` 파일을 열어 `vcvars64.bat` 파일 경로와 자신의 office repository 경로를 수정한 후 사용하도록 합니다.

```
$ cd ${OFFICE_PATH}/build
$ wininit.bat (./wininit.bat on cygwin64)
```

## 멀티플랫폼 코딩 가이드
1. OS Flag
* TOS에서 동작하는 로직은 `OS_TOS` flag로, Windows에서 동작하는 로직은 `OS_WIN` flag를 사용하여 분기를 구분하도록 합니다.
* 예시
```cpp
uint16 OfFont::bigEndien16ToHostByteOrder(uint16 bigEndien)
{                                                                                                   
#if OS_TOS                                                                                         
   return be16toh(bigEndien);                                                              
#elif OS_WIN                                                                              
   return _byteswap_ushort(bigEndien);                                                      
#endif                                                                               
}
```

2. CMake
* 각 담당 Subdirectory 중 의존성이 낮은 순서대로 빌드합니다. 각 Subdirectory의 `CMakeLists.txt`를 열고 원하지 않는 부분을 주석 처리하여 조금씩 빌드를 해나갑니다.
* 기존 Office Devel의 `CMakeLists.txt`에서의 주석과 구분하기 위해 3개의 hash sign (`###`)을 사용합니다.
* 예시
```cmake
...
### add_subdirectory(tw) // 기존 Devel 주석과 구분하기 위해 3개의 hash sign (`###`)으로 주석 처리
### add_subdirectory(tp) // 기존 Devel 주석과 구분하기 위해 3개의 hash sign (`###`)으로 주석 처리
add_subdirectory(tc) // tc만 빌드 시도
# add_subdirectory(debugger) // 기존 Devel에서 주석 처리된 부분은 1개의 hash sign (`#`)
...

```

3. String
* POSIX/Windows에서 사용되는 string 객체 타입이 달라 빌드 에러나 동작 오류가 나타나는 경우가 많습니다. 이럴 때는 `base::FilePath::StringType`을 사용하면 OS에 따라 자동으로 `std::string`이나 `std::wstring` 중 알맞은 string 객체를 사용하게 됩니다. `base::FilePath::StringType`은 다음과 같이 정의되어 있습니다.

```cpp
#if defined(OS_POSIX)
  typedef std::string StringType;
#elif defined(OS_WIN)
  typedef std::string StringType;
#endif
```

* 따라서 `std::string` 사용처 중 Windows에서 `std::wstring`으로 변경해야 하는 구문은 `base::FilePath::StringType`으로 대체해야 정상 동작합니다 (`base/files/file_path.h`를 include 해야 합니다).
* 마찬가지로, `char`이나 `wchar`도 OS에 따라 `base::FilePath::CharType`으로 자동 정의되기 때문에, 알맞게 대체해야 정상 동작합니다.
```cpp
typedef StringType::value_type CharType;
```

[1]:"https://github.com/git-for-windows/git/releases/download/v2.23.0.windows.1/Git-2.23.0-64-bit.exe"
[2]:"https://github.com/Kitware/CMake/releases/download/v3.15.4/cmake-3.15.4-win64-x64.msi"
[3]:"http://releases.llvm.org/9.0.0/LLVM-9.0.0-win64.exe"
[4]:"https://github.com/ninja-build/ninja/releases/download/v1.9.0/ninja-win.zip"
[5]:"https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools&rel=16"
[6]:"https://cygwin.com/setup-x86_64.exe"
