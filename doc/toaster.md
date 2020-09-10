# Toaster

Toaster는 빌드를 설정하고 실행하는 웹 인터페이스다. 빌드, 패키지, 이미지 정보를 관리하고 수집하기 위해 비트베이크와 포키 빌드 시스템과 통신한다. Toaster를 사용하는 방법은 다음과 같이 두가지가 있다.
* 로컬 : 로컬 인스턴스로 Toaster를 실행한다. 단일 사용자 개발에 적합하며, 비트베이크 커맨드라인과 일부 빌드 정보를 GUI로 제공한다.
* 호스팅 : Toaster가 호스팅된 인스턴스로 설정되면 사용자 빌드가 Toaster 빌드 서버에서 실행되도록 해당 컴포넌트가 여러 시스템에 분산된다. 다중 사용자 개발에 적합하다.

### Toaster 설치

다음과 같은 명령으로 설치한다.
```console
$ cd poky
$ pip3 install --user -r bitbake/toaster-requirements.txt
```

### Toaster 시작

poky 디렉토리에서 다음 명령어를 실행한다.
```console
$ source oe-init-build-env
$ source toaster start
```

이 후 웹 브라우저에서 http://127.0.0.1:8000으로 접속하면 다음과 같은 시작 페이지를 볼 수 있다.
![toaster_start](https://github.com/pr0gr4m/yocto/blob/master/img/toaster/1.png?raw=true)


### 빌드 모니터링

콘솔로 돌아와서 bitbake를 이용하여 빌드를 진행하면 Toaster 웹 인터페이스에서 진행 사항 등을 모니터링 할 수 있다.
![toaster_monitor](https://github.com/pr0gr4m/yocto/blob/master/img/toaster/4.png?raw=true)

이 외에도 Toaster 상에서 직접 이미지를 빌드하고 리포트를 받는 등의 관리 작업도 할 수 있다. 이와 관련된 내용은 https://www.yoctoproject.org/docs/2.1/toaster-manual/toaster-manual.html를 참고한다.