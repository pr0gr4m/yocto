# package

### 지원 가능한 패키지 형식

비트베이크는 다음과 같은 4개의 패키지 형식을 지원한다.
* RPM : 레드햇 패키지 매니저
* DEB : 데비안 패키지 매니어
* IPK : Itsy Package Management System으로 DEB을 닮은 경량화 패키지 관리 시스템
* TAR : 타르볼 파일 유형

패키지 형식들에 대한 지원은 package_rpm, package_deb, package_ipk같은 클래스를 다음과 같이 지정하여 사용한다.
```bash
PACKAGE_CLASSES ?= "package_rpm package_deb package_ipk"
```
이는 예를 들어 build/conf/local.conf 파일에 설정해 적용할 수 있다. PACKAGE_CLASSES변수를 이용하여 패키지들을 하나 이상의 형식으로 생성할 수 있다. 참고로 poky의 기본 설정은 RPM 패키지 형식으로 DNF 패키지 매니저를 이용한다.