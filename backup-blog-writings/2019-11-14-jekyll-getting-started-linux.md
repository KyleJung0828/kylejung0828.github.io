---
layout: post
title:  "Jekyll로 Github Page 시작하기 (MacOS)"
date:   2019-11-14
tags: [Blog]
---

# Objectives

- Jekyll을 이용해 Github Page를 만들어봅니다.
- MacOS 사용자를 위한 가이드입니다. Windows 사용자는 [여기][2]를, Linux 사용자는 [여기][3]를 참조해주세요.

## Ruby 설치

- MacOS에서 설치할 경우 : Homebrew를 이용한 설치

설치 후 충돌 방지를 위해 PATH에 추가해야 합니다.

```console
$ brew install ruby
$ echo 'export PATH="/usr/local/opt/ruby/bin:$PATH"' >> ~/.bash_profile
$ source ~/.bash_profile
```

PATH에 제대로 추가되었다면, `which` 명령어와 버전 확인 (-v 옵션)이 제대로 동작할 것입니다.

```console
$ which ruby
/usr/bin/ruby
$ ruby -v
ruby 2.3.7p456 (2018-03-28 revision 63024) [universal.x86_64-darwin18]
```

- Windows에서 설치할 경우 : 인스톨러를 직접 다운로드 (http://rubyinstaller.org/downloads/)

- Linux에서 설치할 경우: apt로 설치
```console
$ sudo apt update
$ sudo apt install ruby-full
```

## Jekyll과 Bundler 설치

- MacOS


- Windows
(추가 예정)

- Linux
  - Jekyll 설치
```console
$ sudo gem install jekyll
```
  - Bundler 설치 & 업데이트
```console
$ sudo gem install bundle
$ bundle update --bundler
```

## 서빙 시작
- Bundler를 이용하여 `http://127.0.0.1:4000/`에 서버를 띄워봅니다.
``` console
$ bundle exec jekyll serve
Configuration file: kylejung0828.github.io/_config.yml
            Source: kylejung0828.github.io
       Destination: kylejung0828.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 1.072 seconds.
 Auto-regeneration: enabled for 'kylejung0828.github.io'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```
위와 같이 서버가 동작을 시작합니다. 이제 브라우저를 켜서 `http://127.0.0.1:4000/` 주소에 접속할 수 있습니다.





### 구글 애널리틱스 적용

### 애드센스

### Sitemap 관련 진행: (참조: http://jinyongjeong.github.io/2017/01/13/blog_make_searched/)

### Disqus 적용



### Reference

1. [Install Ruby on Your Mac: Everything You Need to Get Going][1]
2. [Jekyll로 Github Page 시작하기 (Windows)]
3.

[1]:https://stackify.com/install-ruby-on-your-mac-everything-you-need-to-get-going/
