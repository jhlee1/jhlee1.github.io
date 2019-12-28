---
layout: post
title:  "Jekyll minimal mistakes 테마를 이용한 Github Pages에 블로그 만들기"
date:   2019-08-29 20:35:28 +0900
categories: jekyll
tags: ["jekyll", "ruby"]
---

## 1. Ruby 설치
- [공식 가이드 참고](https://www.ruby-lang.org/ko/documentation/installation/)
- jekyll에서만 사용할 목적이면 아무렇게나 깔아도 괜찮고 Ruby를 계속 쓸꺼면 rbenv나 rvm등 버전 관리를 사용하길 권장

## 2. 필요 gem 설치
```bash
gem install bundler
gem install jekyll
```

## 3. Github에서 blog를 올릴 Repository 생성하기
  - repository 명을 내 계정명과 일치시켜야한다
  - ex) 만약 내 계정이 johndoe123인 경우
  ```
    johndoe123.github.io
  ```
  - 해당 프로젝트 로컬에서 연결하기 (github에서 해당 프로젝트에 들어가서 clone 누르면 나오는 링크)
  ```bash
    git clone https://github.com/johndoe123/johndoe123.github.io.git
  ```

## 4. theme 적용하기
- 위에서 repository에서 Project를 가져왔으니 거기에 jekyll theme을 넣어주면 됨.
  - theme을 사용하지 않으려면 `jekyll new 프로젝트명`을 이용해서 생성해도됨 (ex. jekyll new johndoe123.github.io)
- 먼저, theme 가져오기
```bash
  git clone https://github.com/mmistakes/minimal-mistakes.git
```
- 프로젝트 폴더를 비우고 가져온 theme을 프로젝트 폴더로 복사 & 붙여넣기하면 됨
```bash
  rm -rf johndoe123.github.io/*
  mv minimal-mistakes/* johndoe123.github.io
```

## 5. 불필요한 부분 지우기
- README.md 지우기
  - theme에서 자동으로 설정된 README의 내용을 바꾸면 됨
- 네이게이션 메뉴에서 Quick Start 없애기
  - `_data/navigation.yml`에서 해당 설정 comment 처리

```
  # main links
  main:
    # - title: "Quick-Start Guide"
    #   url: https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/
    # - title: "About"
    #   url: https://mmistakes.github.io/minimal-mistakes/about/
    # - title: "Sample Posts"
    #   url: /year-archive/
    # - title: "Sample Collections"
    #   url: /collection-archive/
    # - title: "Sitemap"
    #   url: /sitemap/
```
- footer에서 피드 없애기
  - `_includes/footer.html`의 feed 관련 부분을 지워서 아래와 같이 바꾸면됨

```html
<div class="page__footer-follow">
  <ul class="social-icons">
    {% if site.footer.links %}
      {% for link in site.footer.links %}
        {% if link.label and link.url %}
          <li><a href="{{ link.url }}" rel="nofollow noopener noreferrer"><i class="{{ link.icon | default: 'fas fa-link' }}" aria-hidden="true"></i> {{ link.label }}</a></li>
        {% endif %}
      {% endfor %}
    {% endif %}
  </ul>
</div>

<div class="page__footer-copyright">&copy; {{ site.time | date: '%Y' }} {{ site.name | default: site.title }}. {{ site.data.ui-text[site.locale].powered_by | default: "Powered by" }} <a href="https://jekyllrb.com" rel="nofollow">Jekyll</a> &amp; <a href="https://mademistakes.com/work/minimal-mistakes-jekyll-theme/" rel="nofollow">Minimal Mistakes</a>.</div>
```

## 6. config 설정하기
- theme의 스킨 바꾸기

```
  minimal_mistakes_skin    : "air" # "air", "aqua", "contrast", "dark", "dirt", "neon", "mint", "plum", "sunrise"
```

- Site Settings 아래의 설정 변경

```
  # Site Settings
  locale                   : "ko-KR" #한국어 사용
  title                    : "John Doe's Blog"
  title_separator          : "-"
  name                     : "John Doe"
  description              : "I am John Doe"
  url                      : "http:johndoe123.github.io" # the base hostname & protocol for your site, e.g. http://example.com
```

- Site Author에 메인 화면에 노출될 정보들을 적으면됨
  - `bundle exec jekyll serve`로 로컬에서 켜놓고 설정을 하나씩 변경하면서 적용되는 부분을 보고 바꾸면 편함

```
  # Site Author
  author:
    name             : "Naver OGQ Tech Blog"
    avatar           : # path of avatar image, e.g. "/assets/images/bio-photo.jpg"
    bio              : "네이버 오지큐 마켓 기술 블로그"
    location         : "OGQ"
    email            :
      links:
        - label: "Email"
        icon: "fas fa-fw fa-envelope-square"
        url: mailto:joolee@ogqcorp.com
```
