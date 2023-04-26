---
title: chirpy theme에 댓글 연동하기
author: blb
date: 2023-04-26 21:00:00 +0900
categories: [jekyll, chirpy]
tags: [blog]
render_with_liquid: false
math: true
mermaid: true
toc: true
comments: true
---

### utterences 댓글 연동하기
#### 1. https://github.com/apps/utterances 에 접속하여 install

#### 2. _config.yml 이동하여 아래와 같이 수정
    
    ```yaml
    comments:
        active: "utterances"
        utterances: 
            repo: username/username.github.io
            issue_term: "pathname"
    ```
    
#### 3. 이후 _posts 에서 메타데이터에 “comments: true” 적용
    
    ```yaml
    ---
    title: "타이틀명"
    author: "작성자명"
    date: 2023-04-26 22:00:00 +0900
    categories: [jekyll, chirpy]
    tags: [blog]
    comments: true # 댓글창 on
    ---
    ```
    
#### 4. 추가로 댓글의 theme를 변경하고 싶으면 [https://utteranc.es/](https://utteranc.es/) 에서 theme 확인 github-light, github-dark, Boxy Light 등 변경하면 하단부 Enable Utterances에서 theme 부분 확인 가능
#### 5. _includes/comments/utterances.html 이동

    ```html
    <script type="text/javascript">
    $(function() {
        const origin = "https://utteranc.es";
        const iframe = "iframe.utterances-frame";
        **const lightTheme = "boxy-light";
        const darkTheme = "photon-dark";** 
    ```

#### 6. lightTheme와 darkTheme를 수정하면 적용 완료


### 추가 이슈
#### 1. 혹시나 댓글창은 생겼는데 저장이 안되는 경우
  - 댓글이 저장될 repository - issues가 안생겨서 그러는 것
  - repository로 이동하여 settings → general → [V] issues
  - 댓글 저장 시 해당 포스트 issue 생성 후 저장되는 것 확인