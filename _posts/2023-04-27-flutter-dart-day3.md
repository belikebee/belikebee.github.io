---
author: blb
title:  "[FLUTTER]Dart 문법 익히기 Day3"
date: 2023-04-27 22:30:00 +0900
categories: [flutter, dart]
tags: [flutter]
render_with_liquid: false
toc: true
comments: true
---

## FLUTTER로 앱 만들기 위한 DART 튜토리얼
* 참고자료 : 모바일 앱 개발을 위한 다트&플러터(서준수 저)

* 개발환경 
  * MacBook Air M2(Ventura 13.0.1)
  * IDE : VS Code

~~ - 시작일 : 2023년 04월 25일~~
~~ - 목표 : 5월까지 로또 번호 추천 앱 틀이라도 배포하기~~
~~ - 계획 : 하루 10~30분 공부해서 올리기 ~~

---

## DART 주석, 변수, 상수, 타입
### 주석 (comment)
- 타언어와 비슷비슷
- // 내용 : 한줄 주석
- /* 내용 */ : /*와 */ 사이의 모든 내용 주석

### 타입 (type)

  |타입|설명|
  |:---:|:---:|
  |num|int와 doubl의 suptertype|
  |int|정수|
  |double|실수|
  |string|문자|
  |bool|true or false 값을 가지는 boolean type|
  |var|type 미지정 및 type 변경 불가|
  |dynamic|type 미지정 및 type 변경 가능|
  |list|dart의 array는 list로 대체|
  |set|순서가 없고 중복이 없는 collection|
  |map|key, value 형태를 가지는 collection(like python dictonary)|

### 변수 (variable)
- 문자열은 "", ''로 감싸면 문자열로 인식
- var를 사용하여 초기 타입을 지정하지 않고 변수를 입력하고 타입 정의 가능
- var를 사용해 타입만 정의 후 변수는 나중에 정의할 수 있음
- 타입의 변경이 필요한 경우 dynamic, Object를 활용
- double, int 간의 호환 불가
  ```dart
  var balance = 1000; // var를 이용한 경우 타입은 지정되지 않지만, 초기 입력값에 맞추어 타입 정의
  balance = "test"; // 초기에 int 타입으로 정의된 상태로 string 값 변경 실패

  dynamic balance = 1000; // 타입 변경이 필요한 경우 dynamic 또는 Object 사용
  balance = "천"; // 삽가능

  // 아래와 같이 사용하는 경우 혼동 가능성 발생
  var number;
  print('number is $number.'); // number is null.

  number = 10;
  print('number is $number.'); // number is 10.

  ```

### 상수 (constant)
- 변수와 달리 값 변경 불가
- 상수로 정의하기 위해 final, const를 활용
- 상수명은 대문자로 명명하여 변수와 혼동을 줄이는 것이 좋음
- final과 const의 차이는 정의 시점의 차이
const는 컴파일 시, final은 런타임 시
따라서 const가 final보다 먼저 정의되며 컴파일 되기 전의 함수로 값을 불러오는 경우 에러 에러 발생
  ```dart
  final int PRICE = 1000;
  final NAME = 'BEE'; // 타입 생략 가능
  PRICE = 2000; // 에러 발생, 변경 불가

  const int PRICE = 1000;
  const NAME = "BEE";
  PRICE = 2000; // 에러 발생, 변경 불가
  ```

---
