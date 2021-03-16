---
title:  "Jekyll로 Github Page 시작하기 (MacOS)"
date:   2019-11-16
categories: Jekyll
tags: [Jekyll, MacOS]
---

이 포스팅에서는 Jekyll을 이용해 Github Page를 만들어봅니다. MacOS 사용자를 위한 가이드이며, git에 대한 기본 지식 (clone, add, commit, push, pull 등)이 없다면 설명이 어렵다고 느낄 수 있으므로, git 학습을 먼저 하는 것을 권장합니다.

## Github Pages용 repo 만들기

- Github Pages를 사용하기 위해서는 [Github](www.github.com)에 가입되어 있어야 합니다. 가입되어 있다고 가정하고 설명을 진행하겠습니다.

1. Github 사용자 페이지 (github.com/${USERNAME}를 치면 나오는 페이지)에서 repository 메뉴를 클릭합니다.
2. 오른쪽 상단의 New 버튼을 눌러 새로운 repo를 만들어 보겠습니다.
3. Repository name에는 **반드시 `${USERNAME}.github.io`로 지정해줘야 github에서 인식해줍니다** (e.g., `kylejung0828.github.io`).
4. Description에는 간단한 설명 (e.g., 내 개인 홈페이지)을 적습니다.
5. Repo의 공개 여부를 설정하는 Public/Private 라디오 버튼은 **반드시 Public으로 설정합니다.** Private repo는 사용자를 제외한 누구에게도 노출되지 않아 Github에서 웹사이트를 서빙해줄 수 없습니다.
6. 나머지는 선택적인 옵션이므로 Create repository 버튼을 눌러 repo 제작을 완료합니다.

## Github pages repo를 로컬 환경에 clone 하기

- Repo를 잘 만들었다면, 로컬 환경 (랩탑, 데스크탑)에서 작업하기 위해 git repo를 clone합니다.

- `git clone ${REPO_ADDRESS} ${DEST_ADDRESS}` 명령어를 이용하여 원하는 위치에 clone합니다 (`REPO_ADDRESS`에는 위에서 만든 `github.io` repo의 clone 주소, `DEST_ADDRESS`는 clone 받고 싶은 로컬 경로).

- 본인이 혼자 쓰는 컴퓨터에 clone 한다면 HTTP보다는 SSH로 clone 받는 것을 권장합니다.

```console
$ git clone git@github.com:KyleJung0828/kylejung0828.github.io.git ~/github/kylejung0828.github.io
```

- 완료되었다면, 이제 웹사이트를 작성하고, 로컬 서빙을 위해 Ruby와 Jekyll 설치를 진행합니다.

## Ruby 설치

- MacOS에서 설치할 경우 : Homebrew를 이용한 설치

- 설치 후 충돌 방지를 위해 PATH에 추가해야 합니다. (저는 bash_profile에 주로 하지만, 다른 여러 방법이 있을 수 있습니다)

```console
$ brew install ruby
$ echo 'export PATH=/usr/local/opt/ruby/bin:$PATH' >> ~/.bash_profile
$ source ~/.bash_profile # 적용
```

- (tip) 추가한 경로인 `/usr/local/opt/ruby/bin`에 가보면, ruby 설치를 통해 사용할 수 있는 명령어에 어떤 것이 있는지 알 수 있습니다. 이 경로에는 ruby를 실행 파일이 들어있으므로, PATH에 제대로 추가되지 않으면 ruby 명령어가 제대로 작동하지 않습니다.

- PATH에 제대로 추가되었다면, `which` 명령어와 버전 확인 (-v 옵션)이 제대로 동작할 것입니다.

```console
$ which ruby
/usr/bin/ruby
$ ruby -v
ruby 2.3.7p456 (2018-03-28 revision 63024) [universal.x86_64-darwin18]
```

## Jekyll과 Bundler 설치

- `gem install`을 이용해 Jekyll과 Bundler를 설치합니다.

```console
$ gem install --user-install bundler jekyll
```

- Jekyll과 bundler를 설치하면서 ruby 버전이 업그레이됩니다. 그럼 이제 다시 ruby 버전을 확인해봅니다. 제 경우, 2.6.5라고 뜨는 것을 확인할 수 있습니다.

```console
$ ruby -v
ruby 2.6.5p114 (2019-10-01 revision 67812) [x86_64-darwin18]
```

- 위에서 ruby를 설치했을 때와 마찬가지로, bundler와 jekyll 관련 명령어를 실행할 수 있도록 gem을 통해 설치한 bin 경로를 PATH에 추가합니다. 직전 단계와 같이, PATH에 제대로 추가되지 않으면 bundler와 jekyll이 제대로 작동하지 않습니다. **또한 ruby를 직접 버전업 하거나 bundler/jekyll 업데이트 때문에 ruby 버전이 변경되면 PATH 또한 변경해줘야 합니다.**

```console
$ echo 'export PATH=~/.gem/ruby/2.6.0/bin:$PATH' >> ~/.bash_profile # 2.6.0을 현재 ruby 버전에 맞춰 입력.
$ source ~/.bash_profile # 적용
```

- 제대로 PATH에 추가되었는지 확인하기 위해서는 `printenv`를 친 후 PATH 관련 항목을 `grep` 검색하거나 `echo $PATH`를 통해 해당 경로가 제대로 추가되었는지 확인할 수 있습니다.  

```console
$ printenv | grep "PATH"
PATH=/usr/local/opt/ruby/bin:/Users/user/.gem/ruby/2.6.0/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
$ echo $PATH
/usr/local/opt/ruby/bin:/Users/user/.gem/ruby/2.6.0/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

- 위에서 `$HOME/.gem/ruby/2.6.0/bin` 경로가 잘 등록되어있는 것을 확인할 수 있습니다. 이제 `jekyll serve`와 같은 `jekyll` 명령어를 사용할 수 있습니다.

- (tip) 참고로, gem 관련 환경변수를 한꺼번에 보기 위해서는 `gem env` 명령어를 사용합니다.

```console
$ gem env
RubyGems Environment:
  - RUBYGEMS VERSION: 3.0.6
  - RUBY VERSION: 2.6.5 (2019-10-01 patchlevel 114) [x86_64-darwin18]
  - INSTALLATION DIRECTORY: /usr/local/lib/ruby/gems/2.6.0
  - USER INSTALLATION DIRECTORY: /Users/user/.gem/ruby/2.6.0
  - RUBY EXECUTABLE: /usr/local/opt/ruby/bin/ruby
# ... 이하 생략
```

## Jekyll 웹사이트 생성하기

- 아래와 같이 `jekyll new ${저장소 위치}` 명령을 통해 jekyll 웹사이트를 만들어 봅니다. 저는 이미 저의 github page repo인 kylejung0828.github.io을 clone 받아 `~/github/kylejung0828.github.io` 경로에 넣어두었기 때문에 다음과 같이 실행하겠습니다.

```console
$ jekyll new ~/github/kylejung0828.github.io
```

- 이미 github page repo를 clone했기 때문에, 아래와 같이 이미 존재하는 디렉터리라는 오류가 나옵니다.

```console
Conflict: /Users/user/github/kylejung0828.github.io exists and is not empty.
                    Ensure /Users/user/github/kylejung0828.github.io is empty or else try again with `--force` to proceed and overwrite any files.
```

- 원래 있던 repo와 충돌이 난다는 것인데요, 이 때는 어차피 최초 작업이니 `--force`를 뒤에 붙여 덮어쓰기를 실행합니다.

```console
$ jekyll new ~/github/kylejung0828.github.io --force
Running bundle install in ~/github/kylejung0828.github.io...
Bundler: Fetching gem metadata from https://rubygems.org/...........
# ... 중략
Bundler: Bundle complete! 6 Gemfile dependencies, 30 gems now installed.
Bundler: Use `bundle info [gemname]` to see where a bundled gem is installed.
New jekyll site installed in ~/github/kylejung0828.github.io.
```

- 성공했다면 `New jekyll site installed in ~` 이라는 문구가 뜹니다.

- 이제 해당 디렉토리로 이동하여, `jekyll serve` 명령어를 이용해 서버를 띄워봅시다.

```console
$ cd ~/github/kylejung0828.github.io
$ jekyll serve
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

- 출력 로그를 읽어보면 `Server running... `이라는 메시지가 보이는데, 성공적으로 서버가 동작을 시작한 것입니다.

- 또, `Server address: http://127.0.0.1:4000`이라는 메시지는 이 주소에 서버가 준비되었다는 뜻입니다.

- 이제 브라우저를 켜서 `http://127.0.0.1:4000/` (또는 `localhost:4000`) 주소에 접속하면 아래 그림과 같이 웹사이트가 뜨는 것을 확인할 수 있습니다.

![](/assets/images/2019-11-14/jekyll-minimal-start.png)

- Jekyll을 설치하면서 자동으로 깔리는 jekyll minimal 테마가 적용되어, 아주 간단한 웹사이트가 생성된 것을 볼 수 있습니다.

### Reference

1. [Jekyll on macOS][1]

[1]: https://jekyllrb.com/docs/installation/macos/
