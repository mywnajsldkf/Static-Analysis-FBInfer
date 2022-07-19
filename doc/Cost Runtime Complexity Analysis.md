# Cost: Runtime Complexity Analysis

함수와 메서드의 시간 복잡성을 계산한다.

- `infer reportdiff` 를 통해 런타임 복잡성에 변화를 확인할 수 있다.
- `--cost`옵션을 활성화한다.
- 제공 언어
  - C/C++/ObjC
  - Java
  - C#/.NET

비용 분석은 프로그램의 최악의 경우에 비용의 상한(**w**orst-**c**ase **e**xecution cos**t** of progoram ⇒ WCET)을 측정한다.

**실행방법**

- `infer --cost`: 프로그램을 실행하면서 프로그램과 비용을 함께 분석한다.
- `infer --cost-only`: 비용분석만 진행한다.

ex) `infer --cost-only -- javac File.java` File.java의 비용을 분석한다.



## How the analysis works

Stefan Bydge’s PhD의 연구 논문 [Static WCET Analysis based on Abstract interpretation and Counting of Elements](http://www.es.mdh.se/pdf_publications/1789.pdf) 로부터 나왔다.

분석은 각 노드에 대해 두가지를 실행한다.

- 실행 비용(각 노드의 비용이 어느정도인지)
- 몇 번 실행되는지

두 개에 대한 곱이 노드의 최종 비용이 된다.들어오고(input) 나가는(output) 에지를 기반으로 제약조건을 해결하는 프로그램에 전달된다.

좀 더 높은 단계에서 분석은 3단계로 진행된다.

- “루프가 몇 번 반복될 수 있는지” 암시하는 제어 변수를 선택한다.

- InferBO(수치 분석)으로부터 제어 변수의 추상적인 범위를 가져온다.

- 제약 조건 해결 알고리즘을 통해 루프 및 함수에 대한 복잡도(ex. 시간 복잡도?) 구성한다.

  <details>
  <summary>Constraint solving algorithm</summary>
    변수가 충족해야하는 조건을 부과(제약 조건)하고 그 안에서 충족하는 변수에 대한 값의 집합
</details>  



## Examples

Infer 비용은 코드 실행 없이 프로그램 실행 비용을 측정할 수 있다.

```java
void loop(ArrayList<Integer> list){
	for(int i = 0; i <= list.size(); i++){
	}
}
```

이 프로그램에서 Infer는 프로그램의 실행 비용(`|.|`은 목록의 길이 참조)에 대해 정적으로 다항식(ex. 8|list| + 16)을 추론한다.

실제 상수값은 상관없이 이 코드에서 복잡도는 `O(|list|)`이다. (리스트의 크기만큼 실행되기 때문에)

이 루프는 선형임을 추론할 수 있고, 개발자는 다음처럼 코드를 수정한다.

```java
void loop(ArrayList<Integer> list){
	for (int i = 0; i <= list.size(); i++){
		foo(i);   // 새롭게 추가된 function call
	}
}
```

`foo` 는 파라미터의 선형 비용이고, infer는 자동으로 루프의 복잡도가 `O(|list|)`에서 `O(|list|^2)` 로 늘어난다. (*제곱인 이유 1. 반복문이 실행되었기 때문에 / 2. 반복문안에서 foo(i)가 실행되므로*)

→ 이게 `EXECUTION_TIME_COMPLEXITY_INCREASE`의 원인이 된다.

이 에러는 시간 복잡도가 ‘상수 → 일차’ 또는 ‘log → quad’로 증가할 때 일어난다.

다른 분석 방법과 다르게 비용 분석은 다른 분석들에 대한 비용을 보고할 때 발생한다.(원본과 수정된 파일들에 대한 결과 분석 등)

프로그램 실행 비용 분석은 `infer-out/costs-report.json` 파일에 있다. 다형식의 차수, 절차 이름, 코드 번호 등 실제 다형식을 포함한다.

**실행 방법**

1. `File.java` 의 비용 분석을 실행한다. `infer-out/costs-report.json` 을 복사해서 `previous-costs-repot.json` 으로 이름을 바꾼다. (run하면 infer-out에 있는 내용이 사라진다.)
   - 반복문의 범위를 상수로 했을 때 `costs-report.json` 에 내용을 확인할 수 없다.
2. `File.java` 파일을 바군다.
3. 바꾸고 다시 실행해서 나온 `infer-out/costs-report.json` 을 `current-costs-report.json` 으로 고친다.
4. `infer reportdiff --costs-current current-costs-report.json --costs-previous-costs-report.json` 실행한다.
5. `infer-out/differential/introduced.json` 으로 분석하면 복잡도가 오른 것을 확인할 수 있다.



## Limitations

- 완전 다항식이 아닌 affine 식에 제한된다. 제곱근을 포함하는 경계를 자동으로 추론할 수 있다.

  https://namunotebook.tistory.com/29

- 재귀를 다룰 수 없다.

- 알수 없는 요청(모델링되지 않은 라이브러리 등)에 의존한다면 실행할 수 없다.



> **실행 테스트**

```java
import java.util.ArrayList;

class Cost {
   void loop(ArrayList<Integer> list){
      for (int i = 0; i <= list.size(); i++){
      }
   }
}
```

`costs-report.json` 파일이 생성된다. 

```json
[{"hash":"22271adf7fbae6863bb02cdb1bf1b49a","loc":{"file":"Cost.java","lnum":3,"cnum":-1,"enum":-1},"procedure_name":"<init>","procedure_id":"Cost.<init>()","is_on_ui_thread":false,"exec_cost":{"polynomial_version":10,"polynomial":"hJWmvgAAAAYAAAADAAAACAAAAAiSoKBCQEA=","degree":0,"hum":{"hum_polynomial":"2","hum_degree":"0","big_o":"O(1)"},"trace":[]},"autoreleasepool_size":{"polynomial_version":10,"polynomial":"hJWmvgAAAAYAAAADAAAACAAAAAiSoKBAQEA=","degree":0,"hum":{"hum_polynomial":"0","hum_degree":"0","big_o":"O(0)"},"trace":[]}},
 
{"hash":"1ea66849271a91f1a9fccf8c3c93453a","loc":{"file":"Cost.java","lnum":4,"cnum":-1,"enum":-1},"procedure_name":"loop","procedure_id":"Cost.loop(java.util.ArrayList):void","is_on_ui_thread":false,"exec_cost":{"polynomial_version":10,"polynomial":"hJWmvgAAABMAAAAHAAAAEgAAABGRkJCQsEUA/5IpQ29zdC5qYXZh","hum":{"hum_polynomial":"⊤","hum_degree":"Top","big_o":"Top"},"trace":[{"level":0,"filename":"Cost.java","line_number":5,"column_number":-1,"description":"Unbounded loop"},{"level":0,"filename":"Cost.java","line_number":5,"column_number":-1,"description":"Loop"}]},"autoreleasepool_size":{"polynomial_version":10,"polynomial":"hJWmvgAAAAYAAAADAAAACAAAAAiSoKBAQEA=","degree":0,"hum":{"hum_polynomial":"0","hum_degree":"0","big_o":"O(0)"},"trace":[]}}]
```

앞 경로에 이름을 바꿔서 이동시킨다.

for문안에 실행할 내용을 작성해서 다시 infer로 검사한다.

```java
import java.util.ArrayList;

class Cost {
  void loop(ArrayList<Integer> list) {
    for (int i = 0; i <= list.size(); i++){
      foo(i);
    }
  }
  
  void foo(int i){
    System.out.println(i);
  }
}
```

`costs-report.json` 이 생기는데 이것도 앞 경로에 이름을 바꾸어 이동시킨다.

```json
[{"hash":"22271adf7fbae6863bb02cdb1bf1b49a","loc":{"file":"Cost.java","lnum":3,"cnum":-1,"enum":-1},"procedure_name":"<init>","procedure_id":"Cost.<init>()","is_on_ui_thread":false,"exec_cost":{"polynomial_version":10,"polynomial":"hJWmvgAAAAYAAAADAAAACAAAAAiSoKBCQEA=","degree":0,"hum":{"hum_polynomial":"2","hum_degree":"0","big_o":"O(1)"},"trace":[]},"autoreleasepool_size":{"polynomial_version":10,"polynomial":"hJWmvgAAAAYAAAADAAAACAAAAAiSoKBAQEA=","degree":0,"hum":{"hum_polynomial":"0","hum_degree":"0","big_o":"O(0)"},"trace":[]}},
 
{"hash":"fb3cbc0b1f63acb3dcf5db78559be9f6","loc":{"file":"Cost.java","lnum":10,"cnum":-1,"enum":-1},"procedure_name":"foo","procedure_id":"Cost.foo(int):void","is_on_ui_thread":false,"exec_cost":{"polynomial_version":10,"polynomial":"hJWmvgAAAAYAAAADAAAACAAAAAiSoKBDQEA=","degree":0,"hum":{"hum_polynomial":"3","hum_degree":"0","big_o":"O(1)"},"trace":[]},"autoreleasepool_size":{"polynomial_version":10,"polynomial":"hJWmvgAAAAYAAAADAAAACAAAAAiSoKBAQEA=","degree":0,"hum":{"hum_polynomial":"0","hum_degree":"0","big_o":"O(0)"},"trace":[]}},
 
{"hash":"1ea66849271a91f1a9fccf8c3c93453a","loc":{"file":"Cost.java","lnum":4,"cnum":-1,"enum":-1},"procedure_name":"loop","procedure_id":"Cost.loop(java.util.ArrayList):void","is_on_ui_thread":false,"exec_cost":{"polynomial_version":10,"polynomial":"hJWmvgAAABMAAAAHAAAAEgAAABGRkJCQsEUA/5IpQ29zdC5qYXZh","hum":{"hum_polynomial":"⊤","hum_degree":"Top","big_o":"Top"},"trace":[{"level":0,"filename":"Cost.java","line_number":5,"column_number":-1,"description":"Unbounded loop"},{"level":0,"filename":"Cost.java","line_number":5,"column_number":-1,"description":"Loop"}]},"autoreleasepool_size":{"polynomial_version":10,"polynomial":"hJWmvgAAAAYAAAADAAAACAAAAAiSoKBAQEA=","degree":0,"hum":{"hum_polynomial":"0","hum_degree":"0","big_o":"O(0)"},"trace":[]}}]
```

`costs-report.json` 을 확인하면 procudure_name: "foo" foo 함수에 대해 실행한 결과를 확인할 수 있다.



```bash
infer reportdiff --costs-current current-costs-report.json --costs-previous previous-costs-report.json
```



`infer-out/differential/indroduced.json` 에 방문해서 복잡도가 늘어난 문제를 확인한다.