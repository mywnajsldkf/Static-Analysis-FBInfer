# User Guide

## Infer workflow

자신의 프로젝트에 Infer를 적용할 수 있는 몇가지 방법들에 대해 알아보겠습니다.



> **Summary**

1. 처음 Infer 프로젝트를 시작할 때는 `make clean`, `gradle clean`명령어로 프로젝트를 정리합니다.

2. 여러번 Infer를 실행한다면, Infer를 실행할 때마다 프로젝트를 정리하거나 `--reactive` 를 infer 명령어에 추가합니다.

3. `--javac Hello.java` 와 같은 명령어를 사용하여 파일을 하나씩 분석하는 증분 빌드 시스템(Incremental Build System)을 사용하지 않는 경우에는 이 단계를 필요하지 않습니다.

   <details>
     <summary>Incremental Build란?</summary>
     <div markdown="1">
       규모가 큰 프로젝트를 빌드하는 경우, 모든 대상을 빌드하려면 오랜 시간이 걸리므로 이전에 빌드되어 이미 최신 상태인 부분은 다시 빌드하지 않습니다. 이때 증분 빌드(Incremental Build)를 시행합니다. 입력과 출력을 1:1 매핑하여 비교작업을 통해 빌드 여부를 결정하고 변경 사항이 없다면 빌드하지 않습니다.
     </div>
   </details>

4. 정적 분석에 성공하면, `infer explore` 명령어로 상세하게 실행 결과를 확인할 수 있습니다.



### The two phase of an Infer run

언어(Java, Objective-C, C)에 상관없이, Infer의 동작은 두 단계를 거칩니다.



#### 1. The capture phase

Infer가 해석할 수 있는 내부 중간 언어로 변환하는 과정입니다. 이 변환은 컴파일 과정과 비슷해서, 정보를 가져와 자체 번역을 합니다. 이러한 이유 때문에 명령어가 `infer run -- javac File.java` 또는 `infer run -- clang -c file.c`형태입니다. 컴파일에 성공한다면 Infer로 분석된 파일을 확인할 수 있지만, 컴파일 되지 않았다면 분석된 파일을 확인할 수 없습니다.

Infer를 실행하면 결과 폴더에 임시 파일이 저장되는데, 이 폴더는 `infer`를 실행한 곳에 `infer-out/`이라는 이름으로 생성됩니다. `-o`를 사용한다면 저장할 폴더를 지정할 수 있습니다.

```shell
# 결과를 저장할 폴더 생성
mkdir test_result
# Hello.java를 javac 컴파일러로 확인한 결과를 test_result라는 폴더에 저장
infer run -o test_result -- javac Hello.java
```



#### 2. The analysis phase

Infer를 실행한 결과는 `infer-out/`에 저장됩니다. Infer는 각 함수와 메서드를 따로 분석합니다. 만약 메서드 또는 함수를 분석하다가 에러를 만나면 일단 그 메서드와 함수 분석을 멈추고, 다음 메서드와 함수를 분석합니다. 이렇게 코드에서 Infer를 실행하고 발생할 수 있는 에러를 찾아 고치기를 반복하면서 작업하면 됩니다.

에러들은 표준 출력형 또는 `infer-out/report.txt`에서 확인할 수 있습니다. 



### Global (default) and differential workflows

Infer를 실행하면 `infer-out`에 저장된 이전 분석 결과들은 삭제됩니다. 이처럼 모든 프로젝트가 매번 새롭게 분석되는 것을 **default workflow**라고 합니다. `--reactive`(또는 `-r`)을 추가한다면 `infer-out/`에 있는 내용들을 사라지지 않습니다. 이러한 흐름을 `differential workflow`라고 합니다.

 `infer run -- javac Hello.java`를 사용한다면 위의 단계를 좀 더 이해할 수 있습니다.

```shell
# 컴파일된 파일인 Hello.class 파일이 생성됩니다.
infer capture -- javac Hello.java
# 컴파일한 결과를 분석하고 분석된 파일은 `infer-out` 폴더에 저장됩니다.
# 기존에 생성된 `infer-out` 폴더안의 내용은 초기화되지 않습니다.
infer analyze
```



#### Global workflow

Global workflow는 프로젝트 안에 있는 모든 파일들을 Infer로 검사할 때 적합합니다. `gradle build`를 사용하는 Gradle 기반 프로젝트가 그 예시입니다.

<details>
  <summary>gradle이란?</summary>
  <div>
    gradle은 오픈소스 빌드 도구로 개발에 있어서 자동으로 빌드를 도와주는 프로그램입니다.(ex. maven, ant)
  </div>
</details>

```shell
infer run -- gradle build
```

만약 분석전에 정리 후 시작한다면, `make clean`(make 빌드 기반 프로젝트) 또는 `gradle clean`(Gradle 프로젝트) 명령어를 사용합니다.



#### Differential workflow

모바일 앱 같은 소프트웨어 프로젝트는 incremental build system(증분 빌드 시스템)을 사용합니다. 이러한 프로젝트는 요구사항이 추가되거나 변경되어 새로운 기능이 추가되므로, 계속해서 코드가 변경된다는 특징이 있습니다. 이때는 매번 많은 양의 코드가 작성된 프로젝트를 전체 빌드하기보다는 변화된 곳만 분석하는 것이 적합합니다. Infer의 reactive mode를 사용하면 변화된 곳만 찾아 분석하는 것이 가능합니다.



> **전체 분석**

분석에 비용이 많이 들지 않지만 모든 메서드, 함수를 컴파일 해야하고 Infer 내부 포맷에 결과를 저장해야 합니다.  

```shell
gradle clean
infer capture -- gradle build
```



> **변화된 부분문 분석**

코드에서 변화된 부분만 분석하면, 프로젝트에 분석이 필요한 부분만 확인하면 되므로 분석 속도가 빨라집니다. `--reactive` 옵션을 사용합니다.

```shell
edit File.java
# File.java가 수정된 상태
infer run --reactive -- gradle build
```



`--continue` 옵션을 사용하면 Infer의 여러 변경 사항을 하나로 묶을 수 있습니다.

```bash
edit some/File1.java
# make some changes to some/File1.java
infer run --reactive -- gradle build
edit some/File2.java
# make some changes to some/File2.java
infer run --reactive --continue -- gradle build 
# File1.java와 File2.java를 모두 분석합니다. 만약 `--continue`가 없다면 두번째 변화만 분석할 수 있습니다.
```



### Exploring Infer reports

Infer를 실행하고 `infer explore` 를 입력하면 리포트로 더 많은 정보를 확인할 수 있습니다.

```shell
infer run -- gradle build
infer explore
```

