# bitbake

비트베이크는 Gentoo Portage의 분기 프로젝트로 시작된 파이썬과 쉘 스크립트가 섞인 코드를 파싱하는 태스크 스케줄러이다. 비트베이크에 대한 자세한 내용은 다음 링크를 참고한다.

https://www.yoctoproject.org/docs/latest/bitbake-user-manual/bitbake-user-manual.html

### 메타데이터

비트베이크가 사용하는 메타데이터는 다음 세 종류로 나뉜다.

* 환경설정 (.conf)
* 클래스 (.bbclass)
* 레시피 (.bb & .bbappend)

환경설정 파일은 전역 변수를 정의하고, 이 변수로 레시피에 정보를 전달하여 레시피가 어떻게 동작할지 결정한다.
클래스는 전체 시스템에서 사용되고 레시피에서 필요에 따라 상속할 수 있다. 클래스와 레시피는 실행할 태스크를 설명하고 비트베이크가 태스크 체인을 만드는데 필요한 정보를 제공한다.

### 메타데이터 파싱

비트베이크는 가장 먼저 .conf 확장자를 가진 환경설정 파일을 파싱한다. 이 메타데이터의 변수는 전역으로 적용된다. 우선 build/conf/bblayers.conf 라는 환경 설정 파일에서 BBLAYERS 변수에 스페이스를 구분자로 하는 레이어 디렉토리를 파싱한다. 해당 목록의 각 디렉토리의 conf/layer.conf 파일을 찾아 해당 레이어에 있는 레시피, 클래스, 설정 파일을 추가한다. LAYERDIR은 이 절차를 진행하는 동안 발견한 레이어 디렉토리이며, 레시피, 클래스, 설정 파일의 집합을 가리키게 하기 위하여 BBPATH 목록에 이 레이어를 추가한다.
사용하는 모든 레이어를 파싱한 후 메타데이터를 파싱한다. BBPATH 목록에 있는 경로에서 meta/conf/bitbake.conf를 로딩한다.
비트베이크 클래스는 기본적인 상속 메커니즘으로, inherit 명령어가 사용될 때 파싱되며, BBPATH에 정의된 디렉토리의 class 디렉토리에 있다.
하나의 비트베이크 레시피(.bb)는 실행될 태스크의 논리적 단위로, 일반적으로 이것이 빌드될 패키지이다. 레시피간의 의존성은 DEPENDS 변수를 통하여 지켜진다. 레시피 의존성 체인을 알면 비트베이크는 의존성이 없는 레시피들은 병렬로 빌드하며, 의존성이 있는 레시피들은 의존성을 만족시키는 방향으로 순차적으로 빌드한다.

### 소스코드 다운로드

비트베이크의 주요 특징 중 하나는 소스코드를 다운로드하는 것이다. 리눅스 커널부터 다양한 유틸리티의 소스코드를 git, subversion, http, ftp, ssh와 같이 다양한 다운로드 모듈을 이용하여 다운로드한다.
비트베이크가 레시피에서 do_fetch 태스크를 진행할 때 SRC_URI 변수의 내용을 살펴본다. SRC_URI의 예시는 다음과 같다. (xz 유틸리티에 대한 레시피이다.)
```bash
SRC_URI = "http://tukaani.org/xz/xz-${PV}.tar.gz"
SRC_URI[md5sum] = "5ace3264bdd00c65eeec2891346f65e6"
SRC_URI[sha256sum] = "b512f3b726d3b37b6dc4c8570e137b9311e7552e8ccbab4d39d47ce5f4177145"
```
이 경우 PV 변수에 패키지 버전을 대입하여 파일을 다운로드한 후 DL_DIR 변수가 가리키고 있는 곳에 저장한다.
다운로드가 끝난 후 다운로드한 파일의 해쉬값을 비교해 둘이 일치하면 ${DL_DIR}/xz-XX.tar.gz.done 파일을 만들어 다운로드가 완료됐다고 표시한다.

### 비트베이크 태스크

비트베이크는 순차적으로 돌아가는 명령들의 모음을 기초적인 실행 단위로 이용하는데 이를 태스크라고 한다.
다음 명령어를 실행하면 비트베이크는 레시피에서 계획되어 있는 태스크들을 실행한다.
```console
$ bitbake <recipe>
```
이 중 특정한 태스크만 실행하고 싶다면 다음 명령어를 실행한다.
```console
$ bitbake <recipe> -c <task>
```
레시피에 정의된 태스크를 알아보고 싶다면 다음 명령어를 실행한다.
```console
$ bitbake <recipe> -c listtasks
```
대표 태스크들은 다음과 같다.
* do_fetch : 필요한 소스코드를 다운로드한다.
* do_unpack : 소스코드를 압축 해제하거나 특정 리비전으로 체크아웃한다.
* do_patch : 다운로드된 파일 중 .patch 확장자를 가지고 있는 소스코드에 패치를 적용한다.
* do_configure, do_compile, do_install : 설정, 빌드, 설치를 진행한다.
* do_package : 레시피에서 복사된 파일을 디버깅 심볼, 문서, 라이브러리 같은 논리적 그룹으로 분류하고 패키징한다.

태스크의 내용이 요구 사항을 만족시키지 못할 때에는 내용을 원하는 것으로 교체하거나 덧붙일 수 있다. 이 때 _append나 _prepend가 사용된다. 예를 들어 do_install 태스크를 확장하고 싶다면 다음과 같은 코드를 작성한다.
```bash
do_install_append() {
    # todo
}
```

### rootfs 이미지 생성

poky의 가장 일반적인 사용 방식 중 하나는 실제 타깃에서 바로 이용 가능한 rootfs 이미지를 생성하는 것이다. 이미지를 만드는 과정을 4단계로 나누면 다음과 같다.
1. rootfs 디렉토리 생성
2. 필요한 파일 생성
3. 최종 파일시스템을 특정한 요구사항에 따라 감싸기
4. 가능하면 압축

위 작업은 모두 do_rootfs 서브태스크에서 진행된다.
rootfs에 설치될 패키지들의 목록은 IMAGE_INSTALL에 나열된 패키지들과 IMAGE_FEATURES에 나열된 패키지의 집합으로 정의된다. 각 IMAGE_FEATURES에 선언된 코드로 추가 패키지를 설치할 수 있는데, rootfs에 포함되는 모든 패키지의 개발용 라이브러리와 헤더를 설치하는 dev-pkgs 패키지와 같은 것을 설치할 수 있다.
선언된 패키지들 중 설치되지 말아야 할 패키지는 PACKAGE_EXCLUDE 변수로 필터링할 수 있다.
최종적으로 설치돼야 할 패키지 리스트를 이용해 do_rootfs 태스크는 각 패키지를 압축 해제하고 환경설정한다.
rootfs를 압축 해제할 때 참조된 패키지들에 대한 post-installation script가 실행된다. 이는 첫 부팅 시 생기는 문제를 피하기 위해서다.
이 후에는 rootfs의 최적화가 진행된다. prelink 프로세스가 실행 파일의 동작 시간을 줄이기 위해 동적 링크를 최적화한다. mklibs 프로세스는 사용되지 않는 심볼들을 지워 라이브러리 크기를 최적화한다.
파일시스템을 만들기 위한 디렉토리 생성이 완료되면 IMAGE_FSTYPES 변수에 나열된 파일시스템들이 나열된다.
do_rootfs가 완전히 끝난 후 만들어진 이미지는 ```build/tmp/deploy/image/<machine>/```에 위치된다.