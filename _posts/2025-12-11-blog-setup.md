---
layout: post
title:  "깃허브 블로그(GitHub Pages) 개설기"
date:   2025-12-11 00:00:00 +0900
categories: [Dev, Setting]
tags: [blog, jekyll, chirpy, troubleshooting, setting, favicon, sitemap]
---

## 🚀 블로그를 시작하며

개발 공부 내용을 정리하기 위해 깃허브 블로그(GitHub Pages)를 개설했다.
과정은 단순해 보였지만, 막상 설정을 건드려보니 예상치 못한 부분에서 삽질이 필요했다.
오늘은 레포지토리 생성부터 환경 설정, 그리고 파비콘 문제 해결까지의 과정을 기록해 본다.

---

## 🛠️ 1. 기본 환경 구축과 테마 적용

### Repository 생성 및 Clone
가장 먼저 gitdhub에서 `wowosp.github.io` 이름으로 리포지토리를 생성하고 로컬로 클론(`git clone`)을 받았다.
맨땅에 헤딩하기보다는 잘 만들어진 테마를 쓰는 게 효율적이라 판단했고, 
*  [https://jamstackthemes.dev/ssg/jekyll/](https://jamstackthemes.dev/ssg/jekyll/)

테마 공유사이트를 탐험하다가 그중에 가장 맘에 들었던 **Chirpy Jekyll Theme** 테마를 선택해 적용했다.

* [https://github.com/cotes2020/jekyll-theme-chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)

### _config.yml 설정 (블로그 제어)
테마 적용 후 가장 먼저 한 일은 `_config.yml` 파일을 수정하는 것이었다.
이 파일은 블로그의 전반적인 환경을 제어하는 곳이다.

```yaml
# 1. 언어 및 시간 설정 (한국 시간 맞추기)
lang: ko
timezone: Asia/Seoul

# 2. 사이트 기본 정보
url: "[https://wowosp.github.io](https://wowosp.github.io)"
title: 재재네 Blog
tagline: 오늘의 공부일기

# 3. 댓글 기능 (Utterances)
comments:
  provider: utterances
  utterances:
    repo: wowosp/wowosp.github.io
    issue_term: pathname
```

이렇게 기본적인 언어(lang), 시간대(timezone), 그리고 댓글 시스템(Utterances)을 내 정보에 맞게 수정하여 블로그의 뼈대를 잡았다.

## 🔍 2. SEO 설정

블로그라면 당연히 검색 엔진(구글, 네이버)에 노출이 되어야 한다.
원래 Chirpy 테마의 `_config.yml`에는 SEO 인증 코드를 넣는 `webmaster_verifications` 항목이 존재한다.

하지만 무슨 이유에서인지 설정 파일에 코드를 넣어도 소유권 확인이 제대로 되지 않았다.
설정과 씨름하는 대신, 가장 확실한 방법인 **'HTML 파일 업로드'** 방식을 택했다.

* **해결:** 구글 서치콘솔과 네이버 서치어드바이저에서 제공하는 인증용 HTML 파일을 다운로드하여, 프로젝트 최상위 폴더(Root)에 그대로 넣었다.
* **결과:** 바로 소유권 확인에 성공했다. 때로는 정석보다 무식한 방법이 빠를 때가 있다.

### 사이트맵(Sitemap) 플러그인 설정
소유권 확인 후, 검색 로봇에게 길을 알려주는 `sitemap.xml`을 제출해야 했다.
지킬(Jekyll)에서 사이트맵을 자동 생성하려면 별도의 플러그인 설정이 필요했다.

먼저 `_config.yml`과 `Gemfile`을 열어 `jekyll-sitemap` 플러그인을 활성화해 주었다.

```yaml
# _config.yml
plugins:
  - jekyll-sitemap
```

```ruby
# Gemfile
gem 'jekyll-sitemap'
```

### 생성 확인과 오해
설정을 마치고 "이제 파일이 생겼겠지?" 하고 프로젝트 폴더를 아무리 뒤져봐도 `sitemap.xml` 파일이 보이지 않았다.
혹시 설정이 잘못된 건가 싶어 당황했는데, 알고 보니 이 파일은 소스 코드에 남는 게 아니라 **빌드(Build) 과정에서 자동으로 생성되어 결과물 폴더(`_site`)에만 존재하는 파일**이었다.

로컬 서버를 켠 상태로 주소창에 `localhost:4000/sitemap.xml`을 입력해 보니, 브라우저에 XML 코드가 정상적으로 출력되었다.
파일이 눈에 보이지 않아도 서버에는 존재한다는 것을 확인했고, 그대로 서치콘솔에 URL을 제출하여 등록을 마쳤다.

---

## 🐜 3. 트러블 슈팅: 파비콘(Favicon) 변경하기

설정을 마치고 나니 브라우저 탭에 떠 있는 테마 기본 아이콘(개미 모양)이 계속 눈에 밟혔다.
이미지를 바꿨다고 생각했는데, 로컬 서버를 띄워봐도 여전히 개미가 기어 다니고 있었다.

### 문제 해결 과정 (역추적)
무작정 이미지를 넣는 게 아니라, **"블로그가 빌드될 때 어떤 파일을 가져다 쓰는지"** 확인해 보기로 했다.

1.  **로컬 서버 실행:** 터미널에서 `bundle exec jekyll serve` 명령어로 서버를 띄웠다.
2.  **빌드 폴더(_site) 확인:** 지킬(Jekyll)은 빌드 결과물을 `_site`라는 폴더에 생성한다. 이곳이 실제 웹에 보여지는 부분이다.
3.  **경로 추적:** `_site/assets/img/favicons/` 경로에 들어가 보니, 테마가 사용하는 파비콘 파일들의 정확한 이름과 사이즈가 보였다. (예: `apple-touch-icon.png`, `favicon-32x32.png` 등)

### 솔루션
원인은 간단했다. **테마가 요구하는 파일명과 내가 넣은 파일명이 달랐던 것이다.**

나는 `_site` 폴더에 있는 파비콘 파일들의 이름을 그대로 복사했다.
그리고 내가 원하는 이미지를 그 이름들과 똑같이 변경하여 원본 폴더(`assets/img/favicons/`)에 덮어씌웠다.

> 이렇게 빌드 결과물을 역으로 확인하여 원본을 수정하니, 개미가 사라지고 내가 원하는 아이콘이 정상적으로 출력되었다.

---

## 🏁 마치며

생각보다 설정할 게 많았지만, 덕분에 지킬(Jekyll)의 빌드 구조와 설정 파일의 역할을 이해하게 되었다.
이제 환경 구축은 끝났으니, 앞으로 **알고리즘(PS), 오픽, 그리고 개발 일상**을 꾸준히 기록해 나갈 예정이다. (꾸준히해보자..!)