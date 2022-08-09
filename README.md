# Static-Analysis-FBInfer

2022년 여름 정적분석기 FBInfer에 대해 공부하는 스터디입니다.



## Topic

- (공통) [Introduction to Compiler](https://github.com/kwanghoon/Lecture_IntroToCompiler)
  - 추상구문트리(Abstract syntax tree) 이해
  - 제공된 Java 언어 외에 자신이 선호하는 언어(ex. Python, JavaScript 등)로 추상구문트리를 표현하는 프로그램 작성

- (지정) 정적 분석기 [FBInfer](https://fbinfer.com/) 
  - FBInfer 분석기 설치([ref](https://github.com/facebook/infer/blob/main/INSTALL.md))
  - FBInfer 실행 방법([ref](https://github.com/facebook/infer/tree/main/examples))
  - 예제(ex. 버퍼 오버플로우) 분석 결과 정리
    - FBInfer GitHub에 준비된 예제 / 인터넷에서 확인할 수 있는 다양한 예제 참고
    - 분석 대상: Java, C, C++, Objective C (정우: `C`, 정인: `Java`)
    - 분석 목록: [기술 문서](https://fbinfer.com/docs/all-issue-types) 참고

## Contents

- [사용자 가이드](doc/user_guide.md) 
- 설치방법
  1. [mac](doc/fbinfer_mac_install.md)
  2. [linux](https://github.com/mywnajsldkf/Static-Analysis-FBInfer/blob/master/doc/Infer_linux_install.md)
  
- 예제분석
  1. [C](https://github.com/mywnajsldkf/Static-Analysis-FBInfer/blob/master/doc/C_example_analysis.md)
  2. [JAVA](doc/java_example_analysis.md)
  3. Gradle

- Analyses
  1. [Cost Runtime Complexity Analysis](https://github.com/mywnajsldkf/Static-Analysis-FBInfer/blob/master/doc/Cost%20Runtime%20Complexity%20Analysis.md)


## Plan

스터디 진행 상황은 [프로젝트](https://github.com/mywnajsldkf/Static-Analysis-FBInfer/projects/1)로 관리하고 있습니다.

- 6월 29일 ~ 7월 2일: FBInfer 설치, Introduction to Compiler 학습
- 7월 3일 ~ 7월 9일: Infer 예제분석 결과 공유
- 7월 10일 ~ 7월 16일: git-hub 자료작성 및 업로드
- 7월 19일: 교수님 면담
- 8월 1일 ~ 8월 8일: 면담 뒤 수정사항 반영
- 8월 9일: 교수님 면담
