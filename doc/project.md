# yocto project

## SDK

SDK는 개발과 디버깅을 하기 위한 파일과 도구의 모음이다. 도구는 컴파일러, 링커, 디버거, 외부 라이브러리 헤더를 포함하고 특정 유틸리티나 애플리케이션들을 포함할 수도 있다. 이러한 도구들을 툴체인(toolchain)이라고 한다.
크로스툴체인은 하나의 아키텍처에서 실행돼 다른 아키텍처에서 사용되는 바이너리를 만드는 도구다. 도구와 생성된 바이너리가 같은 아키텍처에서 돌아가면 이를 네이티브 빌드라고 한다.
커스텀 소스는 라이브러리 헤더 파일을 이용해 빌드할 수 있고, 실행할 때 바이너리는 특정 위치로 이동될 수 있다. 빌드 시점에 사용하는 파일들은 poky SDK의 일부로 sysroots 디렉토리에 있다. 이 SDK는 여러 환경 설정을 할 수 있고, 애플리케이션에 종속적을 수도 있고, 일반적인 목적으로 사용할 수도 있다.
poky SDK는 설치된 환경과 무관하게 컴퓨터에 설치할 수 있는 SDK 패키지를 만들어 실제 환경과 유사한 테스트 환경을 구성할 수 있다.

### 이미지 기반 SDK

직접 작성한 소스코드는 의존성 있는 라이브러리와 애플리케이션을 알 수 있다. 이 경우 사용자의 요구 사항을 충족하는 이미지를 직접 만들거나 poky에서 제공하는 이미지 중 가장 근접한 것을 이용할 수 있다.
이미지 기반의 SDK를 만들기 위하여 다음 명령을 실행한다.
```console
$ bitbake core-image-full-cmdline -c populate_sdk
```
위 명령으로 core-image-full-cmdline 기반 SDK가 만들어진다. 커스텀 이미지가 있으면 위 이미지 대신 이용할 수 있다. SDK는 MACHINE 변수를 사용해 설정한 머신의 아키텍처로 생성된다.
빌드된 후 바이너리 스크립트를 build/tmp/deploy/sdk/poky-glibc-x86_64-core-image-full-cmdline-armv5e-toolchain-2.4.4.sh 경로에서 찾을 수 있다.
생성된 스크립트를 이용하여 SDK를 설치해야 한다. 설치 프로그램은 다음을 제공한다.
* environment-setup-armv5te-poky-linux-gnueabi : 툴체인을 사용하기 위해 필요한 모든 환경 변수의 설정을 위해 사용
* site-config-armv5te-poky-linux-gnueabi : 툴체인 생성에 사용되는 변수들을 담고 있는 파일
* version-armv5te-poky-linux-gnueabi : 버전과 타임스탬프 정보
* sysroots : SDK 생성을 위해 사용된 이미지 rootfs 디렉토리의 복사본. 다음과 같은 하위 디렉토리는 바이너리, 헤더, 라이브러리 파일을 포함한다.
    * armv5te-poky-linux-gnueabi : ARM 머신을 위한 파일들을 포함
    * x86_64-pokysdk-linux : x86_64 머신 호환 파일들을 포함

### meta-toolchain

크로스컴파일러, 디버깅 도구, 기본 헤더와 라이브러리를 포함하는 범용적 SDK를 meta-toolchain이라고 하며, 보통 커널과 부트로더 개발과 디버깅을 위해 사용한다.
```console
$ bitbake meta-toolchain
```
위 명령의 결과는 build/tmp/deploy/sdk/poky-eglibc-x86_64-meta-toolchain-armv5te-toolchain-2.4.4.sh에 생성된다. 설치 과정은 이미지 기반 SDK와 동일하다.

### SDK 생성

다음과 같이 설치된 SDK를 사용해 빌드할 수 있다.
```console
$ source /opt/poky/2.4.4/environment-setup-armv5e-poky-linux-gnueabi
$ ${CC} hello.c -o hello
```

생성된 바이너리 포캣을 file 유틸리티를 통해 확인할 수 있다.
```console
$ file hello
```

이 외에도 리눅스 커널을 빌드하려면 다음과 같은 명령을 진행한다.
```console
$ source /opt/poky/2.4.4/environment-setup-armv5e-poky-linux-gnueabi
$ unset LDFLAGS
$ make defconfig
$ make zImage
```

### 타깃에서 애플리케이션 개발

에뮬레이팅하기 어려운 하드웨어 종속적인 부가 기능은 실제 하드웨어에서 디버깅해야 한다.
타깃 시스템에서 개발하기 위한 첫 번째 방법은 외부 툴체인을 이용해 개발 이미지를 만드는 것이다. 이 이미지는 헤더 파일과 라이브러리 링크로 구성된다. 이는 커스텀 애플리케이션을 위한 빌드 환경을 제공하고, 커스텀 툴체인이나 yocto 프로젝트 외부 툴체인과 함께 사용될 수도 있다. 다음 구문으로 이러한 기능을 추가한다.
```bash
IMAGE_FEATURES += "dev-pkgs"
```
build/conf/local.conf만 수정하고 싶을 경우엔 EXTRA_IMAGE_FEATURES 변수를 사용한다.
이 결과 생성되는 이미지는 헤더 파일들과 추가 라이브러리 링크를 포함하고, 이는 커스텀 애플리케이션 개발 주기에서 이용할 수 있다.
개발 빌드 대신 네이티브 빌드를 해야 하는 경우 poky 환경설정을 통해 SDK 이미지를 만들 수 있다. 이 이미지는 툴체인과 개발용 패키지를 포함한다. 이를 이용하면 실제 환경에서 커스텀 애플리케이션을 빌드, 실행, 테스트, 디버깅할 수 있다.
개발 도구를 이미지에 포함시키기 위해서는 tools-sdk 기능을 IMAGE_FEATURES 혹은 EXTRA_IMAGE_FEATURES 변수에 추가한다.