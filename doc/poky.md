# poky system

### 호스트 시스템 환경 설정

포키를 실행하기 위한 작업 환경은 https://www.yoctoproject.org/docs/current/mega-manual/mega-manual.html#setting-up-a-native-linux-host 링크와 같이 다음과 같은 배포판 중 하나를 사용하는 것이 좋다.
* Fedora
* openSUSE
* Debian
* Ubuntu
* RHEL
* CentOS

해당 프로젝트는 Ubuntu 16.04.6 버전을 사용한다.

### 우분투 포키 설치

다음 명령어로 필요한 패키지를 우선 설치한다.

```console
$ sudo apt install gawk wget git git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping libsdl1.2-dev xterm qemu qemu-efi qemu-system-common qemu-system-arm qemu-system-x86
```

패키지 설치 이 후 git을 이용하여 포키 소스코드를 다운로드한다.

```console
$ git clone git://git.yoctoproject.org/poky -b rocko
```

다운로드 이 후 빌드 환경 구축을 위하여 oe-init-build-env를 실행시킨다.

```console
$ source oe-init-build-env [build-directory]
You had no conf/local.conf file. This configuration file has therefore been
created for you with some default values. You may wish to edit it to, for
example, select a different MACHINE (target hardware). See conf/local.conf
for more information as common configuration options are commented.

You had no conf/bblayers.conf file. This configuration file has therefore been
created for you with some default values. To add additional metadata layers
into your configuration please add entries to conf/bblayers.conf.

The Yocto Project has extensive documentation about OE including a reference
manual which can be found at:
    http://yoctoproject.org/documentation

For more information about OpenEmbedded see their website:
    http://www.openembedded.org/


### Shell environment set up for builds. ###

You can now run 'bitbake <target>'

Common targets are:
    core-image-minimal
    core-image-sato
    meta-toolchain
    meta-ide-support

You can also run generated qemu images with a command like 'runqemu qemux86-64'
```

빌드 디렉토리를 지정하지 않으면 디폴트 디렉토리 build를 사용한다. 위와 같은 안내 문구가 생기면 성공적으로 빌드 환경을 초기화한 것이다.
초기화 이후 생성되는 build/conf/local.conf은 모든 빌드 프로세스를 조작할 수 있는 설정 파일이다. 여기서 빌드 타깃 머신을 지정하거나 크로스툴체인을 지정하거나 빌드 옵션을 지정할 수 있다.


### 타깃 이미지 빌드

포키는 이미지를 빌드하기 위해 미리 정의된 이미지 레시피를 제공한다. 다음은 가장 많이 사용하는 이미지 목록이다.
* core-image-minimal : 타깃 머신이 부팅되게 지원하며, 커널과 부트로더 테스트 및 개발에 유용한 작은 이미지
* core-image-base : 타깃 디바이스 하드웨어를 지원하는 콘솔만 제공하는 이미지
* core-image-weston : Wayland 프로토콜 라이브러리와 레퍼런스 Weston Compositor를 제공하는 이미지
* core-image-x11 : 터미널을 제공하는 x11 이미지
* core-image-sato : Sato UI를 지원하고 모바일 환경을 지원하는 x11 이미지
타깃을 위한 이미지 빌드는 다음 명령어로 수행한다.
```console
$ bitbake <recipe name>
```
우선 타깃 머신을 arm으로 설정한 후, core-image-full-cmdline을 다음 명령으로 빌드한다.
```console
$ vi build/conf/local.conf
    -> MACHINE ??= "qemuarm"
$ bitbake core-image-full-cmdline
```
일반적으로 빌드는 1-2시간정도 소요되는 작업이며, RAM 용량 부족 등으로 빌드를 실패하는 경우도 있으니 참고한다.

### QEMU에서 이미지 실행

프로젝트에 동봉된 runqemu 스크립트로 QEMU를 실행한다.
```console
$ runqemu <machine> <zImage> <filesystem>
```
여기서 machine은 qemuarm, qemux86, qemux86-64 등이 사용될 수 있는 가상화 변수가 들어간다. zImage는 커널의 경로이며 (zimage-qemuarm.bin 등), filesystem은 파일시스템(filesystem-qemuarm.ext3 등)의 경로이다. zImage와 filesystem 파라미터는 선택 사항이다.
예를 들어 다음과 같은 명령어로 qemu을 실행한다.
```console
$ runqemu qemuarm core-image-full-cmdline
```
다음과 같이 qemu이 시작하며, 부팅 성공하면 root로 비밀번호 없이 로그인 할 수 있다.
![qemu_start](https://github.com/pr0gr4m/yocto/blob/master/img/poky/1.png?raw=true)
![qemu_login](https://github.com/pr0gr4m/yocto/blob/master/img/poky/2.png?raw=true)