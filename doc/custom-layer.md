# custom layer

오픈임베디드 메타데이터에 비슷한 레이어가 없다면 새로운 레이어를 추가할 수 있다. 레이어의 이름은 meta-로 시작하는 것을 권장한다.
레이어 환경설정 파일은 모든 레이어에서 필요하고 ```<layer>/conf/layer.conf``` 파일로 돼 있다. 이 파일은 텍스트 편집기를 사용해 수작업으로 만들 수도 있지만, 다음과 같이 poky 스크립트로 만들 수 있다.
```console
$ ./poky/scripts/yocto-layer create newlayer
```

해당 명령어가 파이썬 버전 문제로 에러가 난다면 다음과 같은 스크립트를 수행한다.
```bash
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.7
vi poky/scripts/yocto-layer
    -> #!/usr/bin/env python3.7 으로 변경
```
이 외에도 스크립트를 사용하다가 생기는 파이썬 버전 에러는 위와 같이 python3.7을 사용하면 해결된다.

이렇게 생성된 레이어의 예는 다음과 같다.
![layer_new]()

사용자 레이어 작업을 위해 다른 레이어를 추가할 필요가 있을 때 사용하는 변수는 다음과 같다.
* LAYERVERSION : 하나의 숫자로 레이어의 버전을 표시할 때 사용하는 선택적 변수다. 이 변수는 LAYERDEPENDS 변수에서 레이어의 특정 버전에 의존성을 걸기 위해 사용하고, LAYERVERSION_newlayer = "1"와 같이 레이어 이름에 접미사를 붙여 사용한다.
* LAYERDEPENDS : 공백으로 분리하고 레시피에 의존성을 걸기 위해 레이어 목록을 가진다. 특정 레이어 이름과 함께 LAYERDEPENDS_newlayer = "otherlayer"처럼 접미사를 붙여 사용한다.

의존성을 만족하지 않거나 버전 정보가 맞지 않으면 에러가 발생한다.

### 이미지 생성

예를 들어 core-image-sato 이미지에 애플리케이션을 포함하기 원하고 하나의 이미지 특징을 제거하기 원하면 다음 코드를 recipes-my/images/my-image-sato.bb에 추가해 이미지를 생성할 수 있다.

```bash
require recipes-sato/image/core-image-sato.bb
IMAGE_FEATURES_remove = "splash"
CORE_IMAGE_EXTRA_INSTALL += "myapp"
```

IMAGE_INSTALL 변수는 CORE_IMAGE_EXTRA_INSTALL을 추가하고 IMAGE_FEATURES에 관련된 패키지를 루트 파일시스템에 생성한다. 다음은 IMAGE_FEATURES에 추가할 수 있는 특성들이다.
* allow-empty-password : OpenSSH에서 빈 암호를 사용해 root 계정으로 로그인할 수 있게 허용한다.
* dbg-pkgs : 이미지에 설치된 모든 패키지의 디버그 심볼 패키지를 설치한다.
* debug-tweaks : 개발에 필요한 패키지를 설치해 이미지를 만든다.
* dev-pkgs : 이미지에 설치된 모든 패키지의 개발 패키지를 설치한다.
* doc-pkgs : 이미지에 설치된 모든 패키지의 문서 패키지를 설치한다.
* empty-root-password : root 암호를 빈 문자열로 설정한다.
* hwcodecs : 하드웨어 가속되는 코덱을 설치한다.
* package-management : 패키지 관리 도구를 설치하고 패키지 관리 데이터베이스를 가진다.
* nfs-server : NFS 서버를 설치한다.
* perf : perf와 같은 프로파일링 도구를 설치한다.
* read-only-rootfs : 루트 파일시스템이 읽기만 가능하게 이미지를 만든다.
* splash : 부팅하는 동안 스플래시 화면이 보이게 한다.
* ssh-server-openssh : OpenSSH 서버를 설치한다.
* staticdev-pkgs : 이미지에서 설치된 모든 패키지의 정적 개발 패키지를 설치한다.
* tools-debug : strace와 gdb 같은 디버깅 도구를 설치한다.
* tools-sdk : 디바이스에서 실행하는 전체 SDK를 설치한다.
* tools-testapps : 디바이스 테스트 도구를 설치한다.
* x11 : X 서버를 설치한다.
* x11-base : 최소 환경을 가진 X 서버를 설치한다.
* x11-sato : OpenedHand Sato 환경을 설치한다.

### 패키지 레시피 추가

패키지 레시피는 비트베이크가 애플리케이션, 커널 모듈 또는 프로젝트에서 제공하는 소프트웨어를 다운로드, 압축 풀기, 컴파일, 설치하는 방법을 제시한다. 간단한 레시피 예제는 다음과 같다.
```bash
DESCRIPTION = "hello world application"
SECTION = "examples"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://hello.c"
S = "${WORKDIR}"

do_compile() {
    ${CC} hello.c -o hello
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 hello ${D}${bindir}
}
```

do_compile과 do_install 코드 영역은 빌드와 결과 바이너리를 ${D}로 언급된 목적 디렉토리에 설치하기 위한 쉘 스크립트 명령을 사용한다.
autotools를 사용하는 경우 inherit으로 클래스를 상속받아 코드 중복을 피할 수 있다. 이는 CMake나 QMake의 경우에도 똑같이 적용된다.

### 신규 머신 추가

머신 정의에 사용되는 변수의 일반적인 집합은 다음과 같다.
* TARGET_ARCH : 타깃 아키텍처를 설정한다.
* PREFERRED_PROVIDER_virtual/kernel : 특정 커널을 사용하는 경우 기본 커널을 덮어쓴다.
* SERIAL_CONSOLES : 시리얼 콘솔과 속도를 정의한다.
* MACHINE_FEATURES : 필요한 소프트웨어 스택이 기본 이미지에 포함되게 하드웨어 특성을 정의한다.
* KERNEL_IMAGETYPE : zImage나 uImage같은 커널 이미지 종류를 선택한다.
* IMAGE_FSTYPES : tar.gz, ext4와 같은 파일시스템 이미지 종류를 선택한다.

meta-yocto-bsp/conf/machine의 소스코드에서 머신 파일들의 예제를 볼 수 있다. MACHINE_FEATURES에 이용 가능한 값은 다음과 같다.
* acpi
* alsa
* apm
* bluetooth
* efi
* ext2
* IRDA
* keyboard
* pcbios
* pci
* pcmcia
* phone
* qvga
* rtc
* screen
* serial
* touchscreen
* usbgadget
* usbhost
* vfat
* wifi