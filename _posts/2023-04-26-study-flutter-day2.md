---
title: """[ FLUTTER ] dart 문법 익히기 day2"""
author: blb
date: 2023-04-26 22:30:00 +0900
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
## DART 기본 개념
### 비동기 관련 제한된 예약어
- async, async*, sync* 로 표시된 비동기/동기 함수 내 에서는 식별자롤 사용할 수 없고, 그 외의 경우에 식별자로 사용 가능
- yield 함수명으로 지정하여 사용 가능, await를 변수명으로 지정하여 사용가능
  - but 위에 언급된 비동기/동기 함수 내에서는 사용이 제한
- test() 함수에 async를 달아서 test() async 함수로 만들고 await변수명 사용이 빌드 실패
  
|비동기 관련 제한된 예약어||
|:---:|:---:|
|await|yeild|


### 이 외의 예약어 목록
|기타 예약어|||
|:---:|:---:|:---:|
|asset|break|case|
|catch|class|const|
|continue|default|do|
|else|enum|extends|
|for|if|in|
|is|new|null|
|rethrow|return|super|
|switch|this|throw|
|true|try|var|
|void|while|with|


---
