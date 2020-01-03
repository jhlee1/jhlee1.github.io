---
layout: post
title: ".gitignore가 안될 때"
date: 2020-01-02 12:35:00 +0900
tags: ["git"]
categories: [others]
---

I. 문제
- .gitignore 파일을 생성하기 전에 무시할 파일을 commit한 경우, .gitignore에 파일을 추가했는데 계속 tracking(추적)되는 경우가 있다.
- 특히 .intellij 가 문제

II. 해결법
- 아래의 명령어로 캐시된 데이터를 삭제
```
git rm -r --cached .
```