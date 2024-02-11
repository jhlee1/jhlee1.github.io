---
layout: post
title: "Jenkins에 html파일이 깨질 때"
date: 2020-01-03 18:50:00 +0900
tags: ["jenkins", "devops", "serenity_report"]
categories: [devops]
---
I. 문제
- 젠킨스 보안정책으로 html 파일이 제대로 안보일 때가 있다.
- 나의 경우엔 Serenity Test Report 이미지가 잘 안보임
- (참조 링크)[https://wiki.jenkins.io/display/JENKINS/Configuring+Content+Security+Policy]

II. 해결 방법
1. Manage jenkins로 들어가서
2. Script Console을 클릭
3. 아래의 명령어를 입력

```
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "")

```