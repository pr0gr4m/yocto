# bitbake metadata

비트베이크 메타데이터는 다음과 같은 주요 3개 영역으로 분류한다.
* 설정(.conf 파일) : 환경설정 파일은 전역으로 영향을 미치는 파일로, 클래스와 레시피의 동작을 위한 정보를 제공한다.
* 클래스(.bbclass 파일) : 전체 시스템에서 이용할 수 있고 쉬운 유지 보수와 코드 중복을 피하기 위해 레시피에서 상속한다.
* 레시피(.bb or .bbappend 파일) : 태스크가 실행되도록 정의하고 비트베이크가 필요한 태스크 체인을 생성할 수 있게 필요한 정보를 제공한다.

비트베이크가 레시피를 실행하면 로컬에서 생성돼 상속된 클래스와 레시피 관련 메타데이터가 다른 곳에서 side effect를 발생시키지 않는다.

## 메타데이터 사용

비트베이크의 각 변수 값을 확인하는 명령어는 다음과 같다.
```console
$ bitbake -e <script> | grep <variable>
```

### 변수 할당

```bash
FOO = "bar"
```

### 변수 확장 및 참조

```bash
A = "aval"
B = "pre${A}post"
A = "nval"
```
위와 같은 참조는 실행 시점에 확장된다. 위와 같은 경우 실행 시점에 A 값이 nval이기 때문에 B는 prenvalpost로 확장된다.

### ?= 연산자

```bash
A ?= "aval"
```
?= 연산자는 변수의 디폴트 값을 설정한다. 위 경우 A가 이전 값 설정을 갖고 있지 않다면 aval로 설정되며, 이전 값 설정을 갖고 있다면 해당 값을 보유한다. 하나의 변수에 여러 ?= 할당이 있다면 첫 번째 할당을 사용한다.

### ??= 연산자

```bash
A ??= "aval"
```
변수의 디폴트 값을 설정한다. ??=는 파싱 절차가 끝날 때까지 적용되지 않는다. 따라서 하나의 변수에 여러 ??= 할당이 있다면 마지막 할당을 사용한다.

### := 연산자

```bash
A := "aval"
```
변수에 즉시 값을 적용할 경우에는 := 연산자를 사용한다.

### +=, =+, .=, =. 연산자

```bash
B = "bval"
B += "add"
C = "cval"
C =+ "add"
```
앞뒤 추가 연산자이다. 위 예제에서 B는 bval add가 되고, C는 add cval이 된다. += 연산의 경우 호출 사이에 공백을 갖는데, 이를 피하고자 할 때 .=를 사용한다.

### append, prepend 연산자

```bash
A ?= "aval"
A .= "more"
B ?= "bval"
B_prepend = "more"
```
.=, =.와 비슷한 연산자이다. 위 예제의 경우 .= 연산은 A를 확장하기 전에 실행하여 A는 more이 된다. B의 경우 prepend 연산자가 실행되기 전에 B가 확장돼 B는 morebval이 된다.
append, prepend 연산자 외에 변수의 값을 제거하는 remove 연산도 있다.

### OVERRIDES

```bash
OVERRIDES = "arch:os:machine"
TEST = "defaultval"
TEST_os = "osval"
TEST_other = "otherval"
```
비트베이크는 OVERRIDES 변수에 콜론(:)으로 선택 조건을 나열한다. 위 예에서 TEST는 OVERRIDES에 os가 있기 때문에 osval이 된다.
또한 다음과 같이 OVERRIDES에 있는 어떤 값을 기반으로 조건적 추가도 가능하다.
```bash
DEPENDS = "glibc ncurses"
OVERRIDES = "machine:local"
DEPENDS_append_machine = " libmad"
```
위 예에서 OVERRIDES에 machine이 있으므로 DEPENDS는 glibc ncurses libmad로 설정된다.

### include

비트베이크는 include를 통해 그 위치에 있는 파일을 추가한다. 추가 라인에서 특정 경로가 상대 경로면 비트베이크는 BBPATH 내에서 찾을 수 있는 첫 번째 것을 포함시킨다. 참조된 파일을 찾을 수 없어도 파싱이 실패하지는 않는다.

### require

include와 비슷하게 파일을 추가하지만, require된 파일을 찾을 수 없으면 ParseError를 보여준다. 특별한 경우를 제외하면 require보단 include를 사용하는 것이 좋다.

### 파이썬 변수 설정

```bash
VARIABLE = "${@<python-command>}"
```
위와 같이 파이썬 코드를 사용하여 변수를 설정할 수 있다.

### 전역 네임스페이스 파이썬 함수 정의

```bash
def get_depends(d):
    if d.getVar('CONDITION'):
        return "depwithcond"
    else:
        return "dep"

CONDITION = "1"
DEPENDS = "${@get_depends(d)}"
```
보통 파이썬 함수를 작성할 때 비트베이크 데이터베이스에 접근하는데, 데이터베이스는 d 변수로 표현한다.
위 예에서 데이터베이스는 CONDITION 값을 요청하고 그것에 의존성 있는 값을 리턴한다.

### inherit

inherit 지시어를 통해 레시피(.bb)에서 필요한 기능의 클래스를 사용할 수 있다. 예를 들어 autoconf나 automake에 관련된 작업을 추상화하여 레시피에 사용하기 위해 .bbclass에 넣을 수 있다.
주어진 .bbclass는 BBPATH에서 상속받은 파일 이름인 classes/filename.bbclass로 찾을 수 있다. autoconf 또는 automake를 사용하는 레시피에서 다음과 같이 사용할 수 있다.
```bash
inherit autotools
```
흔히 사용되는 autoconf 또는 automake 기반의 프로젝트들을 쉽게 작업하기 위해 기본적으로 제공하도록 비트베이크는 autotools.bbclass를 상속받게 할 수 있다.