---
title: github blog에 chirpy theme 적용기
author: blb
date: 2023-04-25 22:00:00 +0900
categories: [jekyll, chirpy]
tags: [blog]
render_with_liquid: false
math: true
mermaid: true
toc: true
comments: true
---

- 윈도우11에서 작업하다가 wsl 이슈가 계속 발생하여 mac으로 이동하여 작업진행함
- 추후 윈도우에서도 테스트 도전..!
  
---
- 기존 repo fork 후 clone 하여 작업 push 했을때 아래처럼나옴 ㅡ.ㅡ
```
	-- layout: home # Index page -—
```

- 위에서 진행한 작업은 다음과 같다.

	- bash tools/init
	- bundle
	- jekyll serve —> 실패

  
## Theme 적용을 향한 끝없는 여정
### 1. theme 내 ruby 버전(.github/workflows/pages-deploy.yml.hook)과 mac에 설치된 버전이 다른것 확인 ruby 버전 3.0.2로 업데이트
- brew install rbenv ruby-build
- rbenv install 3.0.2	
- rbenv global 3.0.2
- rbenv init
	- 나온 문구 vim ~/.zshrc 아래줄에 추가
- source ~/.zshrc 로 적용


### 2. 버전 업데이트 후 jekyll serve에서 에러를 뱉길래 bundle 부터 입력했더니 아래와 같은 문구 출력
**An error occurred while installing eventmachine (1.2.7), and Bundler cannot continue.**
- gem install ruby_dev 실행
- Gemfile.lock 에 있는 bundler version설치
- gem install bundler -v (해당 버전)	
	**—> 실패**
- Gemfile.lock 삭제
- bundle 재실행 실패, 동일한 이슈 발생
		- 위의 멘트와 다른 이슈가 나와서 되는 줄 알고 살짝 기대
**—> 실패**

### 3.  ruby version 3.2.2 로 업데이트 진행
- 에러 발생
**..../ruby-3.2.0/lib/yaml.rb:3: warning: It seems your ruby installation is missing psych (for YAML output).
To eliminate this warning, please install libyaml and reinstall your ruby.
uh-oh! RDoc had a problem:
cannot load such file -- psych**
- libyaml이 설치되어 있지 않아서 발생하는 이슈
- ruby 3.2.0 이상 버전에서는 libyaml 필요
—> brew install libyaml 설치
- ruby 3.2.2 설치완료
- 업데이트 버전 적용 시 자꾸 이전 버전으로 나와서 헤맸는데
- 디렉토리 내 .ruby-version에서 버전 교체 필요..
		- **여러 번 업데이트 하는 경우 침착하게 둘러볼 필요**
—> bundle 성공...ㅎ
- bash tools/init 재실행 이후 push 완료
**—> 테마 적용 성공**



## 이후 deploy가 실패하는 경우 기록
- _config.yml 파일 수정 중 빈번하게 발생

	```yaml
	1. 줄바꿈은 띄어쓰기 2번이라는 걸 어디선가 보고 했는데 안되길래 띄어쓰기를 많이 넣었더니 왜 인지 명확하지 않지만 죽음

	title: 테스트       중
	
	# 줄바꿈은 2번 처럼 해결

	2. indent가 안맞아서 죽음

	title: |-
	테스트
	입니다 
	
	# indent 맞추기로 오류 해결 및 줄바꿈 성공
	# yaml 문법 : | 줄바꿈 포함, > 줄바꿈 미포함, - 마지막 줄 줄바꿈 미포함
	title: |-
		테스트
		입니다

	3. 경로 틀려서 죽음

	avatar: assets/img/avatar.png

	# 시작은 무조건 / 로 시작할것
	avatar: /assets/img/avatar.png

	4. 웹이미지 URL 입력시 잘못입력하면 죽음 
	
	
	# 이미지는 다운받아 경로를 입력하는게 안정적 보임
	```
      



---
해당 블로그 구축에 도움되었던 블로그들의 링크

https://www.irgroup.org/posts/jekyll-chirpy/

https://velog.io/@hashnsalt/Github-Blog-%EB%A7%8C%EB%93%A4%EA%B8%B0-2