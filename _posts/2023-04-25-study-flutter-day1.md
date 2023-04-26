---
title: [FLUTTER] dart 문법 익히기 day1
author: blb
date: 2023-04-25 22:30:00 +0900
categories: [flutter, dart]
tags: [flutter]
render_with_liquid: false
math: true
mermaid: true
toc: true
comments: true
---
## FLUTTER로 앱울 만들기 위한 DART 튜토리얼
* 참고자료 : 모바일 앱 개발을 위한 다트&플러터(서준수 저)

* 개발환경 
  * MacBook Air M2(Ventura 13.0.1)
  * IDE : VS Code

- 시작일 : 2023년 04월 25일
- 목표 : 5월까지 로또 번호 추천 앱 틀이라도 배포하기
- 계획 : 하루 10~30분 공부해서 올리기
---
## DART 특징
- 2011년 구글에서 프론트엔드 개발을 위해 만든언어
- 2018년 가장 배울필요 없는 언어 1위 
- 객체 지향이면서 C언어와 유사한 문법을 가지고 있음
- 코드 수정 결과를 즉시 보여주는 Hot reload 기능 내포
- AOT 컴파일로 네이티브 코드를 생성하기 때문에 모든 플랫폼에서 속도 보장
-> 여기에 FLUTTER이용시 FLUTTER의 장점까지 추가
<pre><code>
* 정리하면 DART는
  1. 객체 지향
  2. 선택적 타입(optional type)
  3. 메모리를 공유하는 thread 대신 독립 메모리를 갖는 isolate를 사용
  4. 자바스크립트와 호환
</code></pre>



## DART 기본 개념
* 모든 변수는 객체, 모든 객체는 클래스의 인스턴스
  - 숫자, 함수, null도 모두 객체이며, 모든 객체는 Object 클래스로부터 상속
* 타입 어노테이션은 타입 추론이 가능할 경우네는 필수가 아닌 선택사항
  - int number = 10 대신 var number = 10으로 선언 가능
* 타입이 예상되지 않는다는 것을 명시적으로 표현하고 싶을 때는 dynamic키워드를 사용
  - 하나의 변수가 여러 타입으로 변경이 가능하다는 의미
  - var 키워드와 비교시 string으로 지정 후 int 타입을 참조할 수 없다

```dart
var name = "Kim";
name = 100;
--> 불가 

dynamic name = "Kim";
name = 100;
--> 가능

```

* 제네릭 타입 지원
* main() 과 같은 최상위 함수 지원
* public, protected, private 키워드가 없음
  * private하려면 식별자 앞에 _ 를 사용
* 두 가지 런타임 모드 debug mode, release mode)
  * debug mode 는 dartdevc컴파일러를 통해 좀 더 쉬운 디버깅 제공
  * release mode 는 dart2js 컴파일러를 통해 앱 사이즈와 성능 최적화


- import시 python처럼 ../ 을 통해 상위 디렉토리의 파일 import 가능
- lib 디렉토리에 넣어둔 파일들을 application 명을 통해 import 가능
- 테스트 하다보니 application명에 _(언더바)가 있으면 제대로 import 되지 않는 것 같기도 하다
  - application명을 dart_tutorial로 했더니 제대로 impport되지 않아서, 디렉토리내 iml파일과 pubspec.yaml의 네임을 dartTutorial로 변경
  ```dart
  import 'package:{application명}/{file명}dart' as {단축어};
  ```

### 키워드
 
- 식별자
  - 변수, 함수 등의 이름 (변수명, 함수명)로 생각하는게 편할것 같음
```dart
int a; // 식별자 a
void foo(); // 식별자 foo
```
#### 내장 식별자
- 클래스명, 타입명, import시 prefix(단축어)로 사용 불가

|내장 식별자  ||||
|:---:|:---:|:---:|:---:|
|abstract|implements|as|import|
|convariant|interface|deferred|library|
|dynamic|mixin|export|operator|
|external|**part**|**factory**|set|
|Function|static|get|typedef|

- 일반 변수명으로 사용될 만한 식별자들이 보임
```dart
// prefix 불가
import 'package:test/testFunction.dart' as set;

// 함수명 가능
set (int a, int b){
  return a+b;
}
```


#### 문맥 키워드
- 특정 문맥에서 특별한 의미를 가지는 단어
- '특정 문맥'이 아니면 식별자로 사용 가능
  
|문맥 키워드|||||
|:---:|:---:|:---:|:---:|:---:|
|sync|async|hide|on|show|

- show 의 경우 특정 함수만 라이브러리내 import 할때 사용

---