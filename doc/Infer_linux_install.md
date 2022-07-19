# infer install in linux
설치 방법은 다음 링크의 자료를 바탕으로 작성하였다.
[infer_install.md](https://github.com/facebook/infer/blob/main/INSTALL.md)


## 1. Infer 설치 전 과정

### 1-1. window에서 linux 사용하기
먼저 mac을 사용하는 유저는 터미널을 이용하여 brew를 통해서 설치하면 된다.
[intall in mac(%EC%95%84%EC%A7%81%EC%97%86%E3%85%87%E3%85%81)

하지만 infer은 window 운영체제를 지원하지 않기 때문에 ubuntu 프로그램을 사용하여 linux에 접근해야 한다.
ubuntu 설치 과정은 다음과 같다.

 1. 설정으로 이동
 2. 앱 이동
 3. 프로그램 및 기능 선택
 4. window 기능 켜기/끄기 
 5. Linux용 window 하위시스템 체크 이 후 재부팅  [linux 하위시스템 체크(출처: 박태준 교수님)](https://drive.google.com/file/d/1tQbNgKVL6mL-J8S2h2EJwYbB_CxMD5dL/view?usp=sharing)
 6. microsoftware store에서 ubuntu 설치
 7. ubuntu실행 후 초반 설정 [unbuntu 초반 설치(출처: 박태준 교수님)](https://drive.google.com/file/d/1zvw9UXXXEyhDsqZPSd3K719RXfa1fHKF/view?usp=sharing)





### 1-2. 사전 설치 프로그램 목록

 -   opam >= 2.0.0
-   pkg-config
-   Java (only needed for the Java analysis)
-   gcc >= 5.X or clang >= 3.4 (only needed for the C/Objective-C analysis)
-   autoconf >= 2.63 and automake >= 1.11.1 (if building from git)
-   CMake (only needed for the C/Objective-C analysis)
-   Ninja (optional, if you wish to use sequential linking when building the C/Objective-C analysis)
위의 프로그램을 설치하는 방법은 구글에  "< program name > install"을 검색한 뒤 설치하기 바란다.

liunx는 GUI 기반 운영체제가 아니기 때문에 처음 사용한다면 어려움을 느낄 수도 있다.
따라서, infer을 사용할 때 자주 사용할 커맨드를 몇가지 기입하겠다.

    cd #linux에서 디렉토리간 이동을 하는 커맨드이다.
    mkdir #디렉토리(폴더)를 생성하는 커맨드.
    install #파일을 다운받을 때 사용하는 커맨드.
    rm #파일을 삭제할 때 사용하는 커맨드
    rm -r #디렉토리(폴더)를 삭제하는 커맨드
   
리눅스에 대한 자세한 사용방법은 다음 링크를 참조하기 바란다.
[linux 기초](https://itholic.github.io/linux-basic-command/#:~:text=%EB%A6%AC%EB%88%85%EC%8A%A4%20%EA%B8%B0%EB%B3%B8%20%EB%AA%85%EB%A0%B9%EC%96%B4%201%20pwd%20%28print%20working%20directory%29,cat%20%28concatenate%29%2010%20head%2011%20tail%2012%20find)

 

## 2. infer 설치
### 2-1. git clone

    git clone https://github.com/facebook/infer.git
git clone은 git-hub에서 링크를 통해서 소스코드를 다운받는 linux 명령어이다.
위의 코드를 입력하여 infer source code를 다운받는다.
### 2-2. build and make
위의 과정을 거치면 다음과 같은 파일들이 생성이 될 것이다.
[infer directory](https://drive.google.com/file/d/1JtmyjDQwtMbqJF98UqI4p5_8WlecO42F/view?usp=sharing)

이 후 `./build-infer.sh`커맨드를 작성한다.

    ./build-infer.sh clang # C,C++ 분석만을 원할 때
    ./build-infer.sh java # java 분석만을 원할 때

위 과정을 끝나면 마지막으로 아래 커맨드를 작성한다.

    sudo make install
## 3. infer 실행
### 3-1. infer 커맨드 작성
위의 과정을 모두 마쳤으면 infer 디렉토리에 example 파일들을 찾아볼 수 있다.
하지만 linux를 처음하는 사람이라면 infer의 실행 커맨드도 난해 할 것이다.
기본적인 실행 커맨드는 다음과 같다.

    infer run -- gcc -c hello.c
    infer = 실행파일
    run = 실행 명령어
    gcc = 사용 컴파일러(C)
    -c = c를 분석한다.
    hello.c = 분석 code
위와 같이 해석할 수 있다.
기존 GUI는 마우스로 클릭하여 접근하였지만 리눅스는 경로를 통해서 접근을 한다.
따라서 infer의 경로와 example의 경로와 추출되는 file의 경로가 모두 다르다면

    /home/jungwoo5516/jungwoo5516/2022_summer/static_analysis/static_Analysis/infer/infer run  --gcc -c 
    /home/jungwoo5516/jungwoo5516/2022_summer/static_analysis/static_Analysis/infer/examples/hello.c
이렇게 길어진다!
### 3-2. 파일 경로찾기
이처럼 linux에서는 경로를 아는것이 중요한데

    pwd
위 커맨드를 이용하면 경로가 보여진다. pwd를 통해서 경로를 복사하고 붙여넣어서 command를 작성하면된다.
### 3-3. 주의사항
1. `pwd`를 통해서 얻은 경로는 유저가 원하는 file의 경로까지를 담지않는다.
즉, file의 디렉토리의 경로까지만을 추출하기 때문에 마지막에 /infer을 통해서 실행파일까지 추가해주어야 한다.

2. ubuntu의 경우 복사 붙여넣기를 할 때 복사는 마우스로 드래그 한 뒤 control+C 하면 되지만 붙여넣기는 control+V가 되지 않는다. 
[ubuntu 설정](https://drive.google.com/file/d/14Knj6VjLHG-UzxAetzrhzZu5iEglSLVJ/view?usp=sharing)
위의 사진처럼 unbuntu의 설정에 들어가 control+shitf+V를 체크해주면 된다.
설정에 들어가는 방법은 콘솔창 좌측상단에 ubuntu 아이콘을 우클릭 해주면 된다.
