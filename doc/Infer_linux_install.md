
# FB Infer install in linux
설치 방법은 다음 링크의 자료를 바탕으로 작성하였다.
[infer_install.md](https://github.com/facebook/infer/blob/main/INSTALL.md)



## 0. window에서 Linux 사용하기
먼저 mac을 사용하는 유저는 터미널을 이용하여 brew를 통해서 설치하면 된다.

하지만 infer은 window 운영체제를 지원하지 않기 때문에 ubuntu 프로그램을 사용하여 linux에 접근해야 한다.
ubuntu 설치 과정은 다음과 같다.

 1. 설정으로 이동
 2. 앱 이동
 3. 프로그램 및 기능 선택
 4. window 기능 켜기/끄기 
 5. Linux용 window 하위시스템 체크 이 후 재부팅  [linux 하위시스템 체크(출처: 박태준 교수님)](https://drive.google.com/file/d/1tQbNgKVL6mL-J8S2h2EJwYbB_CxMD5dL/view?usp=sharing)
 6. microsoftware store에서 ubuntu 설치
 7. ubuntu실행 후 초반 설정 [unbuntu 초반 설치(출처: 박태준 교수님)](https://drive.google.com/file/d/1zvw9UXXXEyhDsqZPSd3K719RXfa1fHKF/view?usp=sharing)


## 1. Binary file 설치
 [https://github.com/facebook/infer/releases/tag/v1.1.0](https://github.com/facebook/infer/releases/tag/v1.1.0)
 위 사이트에서 infer-linux64-v1.1.0.tar.xz을 wget으로 다운받는다.
 이 후 tar.xz 파일을 해제한 뒤에 사용하면 된다.

bin directory에 infer 실행파일이 존재한다.


## 2. Source code 설치

### 2-1. 사전 설치 프로그램 목록

 -   opam >= 2.0.0
-   pkg-config
-   Java (only needed for the Java analysis)
-   gcc >= 5.X or clang >= 3.4 (only needed for the C/Objective-C analysis)
-   autoconf >= 2.63 and automake >= 1.11.1 (if building from git)
-   CMake (only needed for the C/Objective-C analysis)
-   Ninja (optional, if you wish to use sequential linking when building the C/Objective-C analysis)
위의 프로그램을 설치하는 방법은 구글에  "< program name > install"을 검색한 뒤 설치하기 바란다.



 

### 2-2. infer 설치
#### 1) git clone

    git clone https://github.com/facebook/infer.git
git clone은 git-hub에서 링크를 통해서 소스코드를 다운받는 linux 명령어이다.
위의 코드를 입력하여 infer source code를 다운받는다.
#### 2) build and make
위의 과정을 거치면 다음과 같은 파일들이 생성이 될 것이다.
[infer directory](https://drive.google.com/file/d/1JtmyjDQwtMbqJF98UqI4p5_8WlecO42F/view?usp=sharing)

이 후 `./build-infer.sh`커맨드를 작성한다.

    ./build-infer.sh clang # C,C++ 분석만을 원할 때
    ./build-infer.sh java # java 분석만을 원할 때

위 과정을 끝나면 마지막으로 아래 커맨드를 작성한다.

    sudo make install
### 2-3. infer 실행

    infer run -- gcc -c hello.c

## 3. 주의사항
ubuntu의 경우 복사 붙여넣기를 할 때 복사는 마우스로 드래그 한 뒤 control+C 하면 되지만 붙여넣기는 control+V가 되지 않는다. 
[ubuntu 설정](https://drive.google.com/file/d/14Knj6VjLHG-UzxAetzrhzZu5iEglSLVJ/view?usp=sharing)
위의 사진처럼 unbuntu의 설정에 들어가 control+shitf+V를 체크해주면 된다.
설정에 들어가는 방법은 콘솔창 좌측상단에 ubuntu 아이콘을 우클릭 해주면 된다.
