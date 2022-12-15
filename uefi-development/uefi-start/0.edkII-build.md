---
description: EDK II 를 이용하여 EFI 만들기
---

# 0. EDK II 빌드 환경 구성

* EDK2를 설치하고 EFI 모듈을 빌드하는 과정을 서술한다.
* 앞으로의 실습은 Ubuntu 20.04 환경에서 진행한다.

## Linux

* Linux상에서 EDK2는 GCC5와 NASM컴파일러 기반으로 빌드한다. GCC4.X대 버전도 아직 지원하기 때문에 GCC5 설치가 필수는 아니지만 여기서는 GCC5를 사용한다.

### Ubuntu 20.04 LTS

#### 의존성 패키지 설치

```shell
bash$ sudo apt install build-essential uuid-dev iasl git nasm python-is-python3 
```

* `build-essential` - 빌드 패키지 정보 목록
* `uuid-dev` - UUID(Universally Unique ID) 라이브러리
* `iasl` - Intel ASL 컴파일러/디컴파일러
* `git`
* `nasm` - 범용 x86 어셈블러
* `python-is-python3` - 우분투 20.04는 python 명령을 `python3` 로 쓰지만 edk2는 `python` 을 쓴다.

#### Git을 이용한 edk2 소스 받기

```shell
bash$ mkdir ~/src
bash$ cd ~/src
bash$ git clone https://github.com/tianocore/edk2
```

#### 서브 모듈 받기

```shell
bash$ git submodule update --init
```

#### 빌드 툴 컴파일하기

```shell
bash$ cd ~/src/edk2
bash$ make -C BaseTools
bash$ . edksetup.sh
```

위 작업까지 완료하면 edk2 디렉토리를 개발하는데 활용할 수 있다.

#### EDK2 BaseTools 빌드

```shell
bash$ make -C edk2/BaseTools
```

#### 빌드 쉘 환경 설정 <a href="#user-content-setup_build_shell_environment" id="user-content-setup_build_shell_environment"></a>

```shell
bash$ cd ~/src/edk2
bash$ export EDK_TOOLS_PATH=$HOME/src/edk2/BaseTools
bash$ . edksetup.sh BaseTools
```

#### Conf 파일 수정 <a href="#user-content-modify_conf_files" id="user-content-modify_conf_files"></a>

위의 `edksetup.sh`을 실행하면 `edk2/Conf`디렉토리가 설정 파일들로 채워진다. 빌드 플랫폼, 대상 아키텍처, 다중 스레딩 옵션 등을 설정하려면 `Conf/target.txt`파일을 편집해야한다. 아래 예제는 GCC5를 사용하여 MdeModulePkg를 빌드하는 것이다.

**빌드 대상 정보 설정**

Conf/target.txt를 텍스트 편집기로 연다.(ex: vi)

```
ACTIVE_PLATFORM       = Nt32Pkg/Nt32Pkg.dsc
TOOL_CHAIN_TAG        = MYTOOLS
```

그리고 위의 행을 찾아 아래와 같이 변경한다.

```
ACTIVE_PLATFORM       = MdeModulePkg/MdeModulePkg.dsc
TOOL_CHAIN_TAG        = GCC5
```

```
TARGET_ARCH           = IA32
```

위의 `TARGET_ARCH`를 변경하면 최종 UEFI 바이너리에 아키텍처가 반영된다.

#### Hello World빌드하기(MdeModulePkg 전체) <a href="#user-content-build_hello_world_and_the_rest_of_mdemodulepkg" id="user-content-build_hello_world_and_the_rest_of_mdemodulepkg"></a>

위의 과정을 모두 따라했다면 빌드 명령을 통해 `MdeModulePkg`를 컴파일 할 수 있다.

```
bash$ build
```

빌드가 완료되었다면 결과에 HelloWorld.efi가 포함되어 있을 것이다.

```
bash$ ls Build/MdeModule/DEBUG_*/*/HelloWorld.efi
```

### Ubuntu 16.04, 18.04

#### 의존성 패키지 설치

```bash
sudo apt-get install build-essential uuid-dev iasl git gcc-5 nasm python3-distutils
```

#### EDK2 Stable 버전 받기

<pre class="language-bash"><code class="lang-bash">bash$ mkdir ~/src
bash$ cd ~/src
<strong>git clone &#x3C;https://github.com/tianocore/edk2.git
</strong></code></pre>

#### 서브모듈 받기

```bash
bash$ git submodule update --init
```

#### 빌드 툴 컴파일

```bash
bash$ make -C BaseTools
```

#### 빌드 쉘 환경 설정

```bash
bash$ cd ~/src/edk2
bash$ export EDK_TOOLS_PATH=$HOME/src/edk2/BaseTools
bash$ . edksetup.sh
```

#### Conf 파일 설정

```bash
vim Conf/target.txt

ACTIVE_PLATFORM       = MdeModulePkg/MdeModulePkg.dsc
TOOL_CHAIN_TAG        = GCC5

TARGET_ARCH           = IA32
```

#### 빌드

```
bash$ build
bash$ ls Build/MdeModule/DEBUG_*/*/HelloWorld.efi
```

## Window

* 추후 추가 예정

## Mac OS

* 추후 추가 예정
