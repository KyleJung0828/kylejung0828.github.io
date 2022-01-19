---
layout: single
title:  "[Github Pages] minimal-mistakes-jekyll이 github pages에서 배포되지 않는 이슈"
date:   2022-01-19
category: Github Pages
tags: [Github Pages]
---

### 현상

- 최근에 새 포스트를 올리려고 포스팅 마크다운 파일과 이미지를 commit하고 배포해봤는데 오류가 나는 것을 발견했다.
- 원인을 살펴보기 위해 deploy status를 살펴봤는데, UI가 많이 바뀌었다. CI 사용하여 빌드와 배포하는 것은 이전에도 동일했을테지만, Jenkins처럼 UI상으로 빌드와 배포 과정을 확인할 수 있었다.
- CI에서 웹사이트 파일을 빌드할 때 내 웹사이트 repository의 Gemfile을 참조하여 테마를 설치하는데, 최근에 정책이 강화되었는지, **github에서 정식으로 제공하는 테마만 gem으로 허용되는 것으로 바뀐 것이 문제였다.**
- 현재 Gemfile로 설치할 수 있는 dependency의 리스트는 [**여기**](https://pages.github.com/versions/)에서 확인할 수 있다. 보안의 이슈인지 3rd party theme인 `minimal-mistakes-jekyll`은 이 리스트에 존재하지 않는 것을 볼 수 있다.
- 이로 인해 Github CI builder가 나의 Gemfile을 읽다가 중간에 `gem “minimal-mistakes-jekyll”`을 만나 실행하면 찾을 수 없는 테마라는 에러 메시지가 나오는 것이다 (`Error: The minimal-mistakes-jekyll theme could not be found.`)
    
    ![1.png](/assets/images/minimal-mistakes-remote-theme/1.png)
    
- 마찬가지의 이유로 예전에 정상적으로 작동 중이던 commit으로 되돌아가 재차 push하더라도 웹사이트가 제대로 배포되지 않았다.

### 해결 방법

- 문제를 파악하기 위해 minimal mistakes 테마의 공식 홈페이지를 가서 가이드라인을 다시 살펴봤다.

![2.png](/assets/images/minimal-mistakes-remote-theme/2.png)

- Github pages와 100% 호환 가능하다고 한다. 단, **“remote theme으로 사용했을 경우에만!”**
- 다행히 해당 페이지에서 remote theme으로 사용하는 방법이 상세히 설명되어 있다.
1. Gemfile을 열어 github pages에서 지원하지 않는 gem 항목을 지운다
    
    ```ruby
    # gem "minimal-mistakes-jekyll" # 삭제
    # gem "jekyll", "~> 4.0.0" # 삭제
    ```
    
2. 대신 github-pages와 jekyll-include-cache를 추가한다.
    
    ```ruby
    gem "github-pages", group: :jekyll_plugins # 추가
    gem "jekyll-include-cache", group: :jekyll_plugins # 추가
    ```
    
3. bundle된 gem들을 다시 가져온다.
    
    ```bash
    $ bundle
    ```
    
4. `_config.yml` 파일을 열어 theme 부분을 삭제하고 remote_theme을 추가한다.
    
    ```bash
    # theme: "minimal-mistakes-jekyll" # 삭제
    remote_theme: "mmistakes/minimal-mistakes@4.24.0" # 추가
    ```
    
5. `bundle exec jekyll serve`로 로컬에서 잘 뜨는지 확인 후 `git push origin master`로 push한다.

### 마치며

- Minimal mistakes 측에서 빠르게 workaround를 업데이트한 것인지, github pages에서 미리 3rd party theme들에게 알려주고 그동안 유예기간을 준 것인지는 모르겠지만 (찾아보기가 귀찮다), remote theme을 이용하면 문제가 해결되었다.
- 역시 공식 문서를 잘 읽어야 한다.