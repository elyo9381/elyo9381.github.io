
[Jekyll](https://jekyllrb.com/)은 코드하일라이팅의 이점을 차치하고서라도 기본적으로 Github를 통해 배포가 가능한 구조로 되어 있어   
가장 개발자 친화적인 정적 블로그 관린 Template이다.
 
사용하고자 하는 jekyll 테마(http://jekyllthemes.org/)를 선택하고   
기본적인 Jekyll 템플릿의 구조와 마크다운 문법만 익히고 있다면  
개발자들이 코딩하듯이 Github에 Commit & Push를 하는 것 만으로 쉽게(?) 내가 작성한 글들을 포스팅 할 수 있다.

물론,   
블로그 사이트 접속에 필요한 Github의 환경설정이 이미 되어 있다는 전제하에서이다.

일반적으로 개발자들이 코딩을 할때 커밋 또는 푸시전에 로컬에서 확인을 하듯이,   
Jekyll도 Github에 배포하기 전에 로컬에서 작성한 페이지를 빨리 확인해봐야 할 필요가 있다.

아무래도 Push를 해야지만 확인가능한 구조로 블로그를 운영하기에는 불편한 점이 많다.

> **이 글은 맥(Mac) 사용자들이 Jekyll 기반 블로그를 로컬에서 확인하기 위한 
아주 기본적인 환경설정에 관한 글이다.**



### 1. Jekyll 설치를 위한 brew 설치

---

본 설치는 맥에서 `brew` 를 사용하고 있다는 전제로 설명한다.

> brew는 홈브루의 일종으로 쉽게 얘기해서,      
> 리눅스의 `wget`, `yum`같은 패키지 관리툴로 패키지 설치 및 업데이트,  
> 삭제등의 관리를 쉽게 해 주는 맥용 패키지 관리툴이라고 이해하면 된다.


터미널에서 `brew`를 입력하고 엔터키를 쳤는데 `command not found`라고 뜨면 아래의 방법을 참조해서 설치해야 한다.   
설치관련해서는 구글링을 하면 쉽게 설치방법을 찾을 수 있다. [홈페이지](http://brew.sh/)

설치방법은 매우 간단하다.
위의 홈페이지에서 설치명령어를 복사해 붙여넣고 실행하면 끝.

``` bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

설치가 되었으면,   
한번 brew를 업데이트 해준다.

`$ brew update`



### 2. Jekyll 설치

---

brew 설치 및 업데이트가 되었으면, 본격적으로 jekyll설치를 진행한다.

1. **ruby 버전 확인 및 업그레이드** - Jekyll 3.x 버전부터는 루비 2.x대부터 지원하므로, 2.x로 업그레이드해 주도록 한다.  
  - 버전확인: `$ ruby -v`  
  - 업그레이드: `$ brew upgrade ruby`


2. **nodejs 설치** - Jekyll 2.x 버전부터는 nodeJs가 필요하다.
  - `$ brew install nodejs`
  
  
3. **Jekyll 설치** - gem패키지로 설치해야 하며, 루비기반으로 설치하기 위해서는 sudo로 실행해야 한다.
  - `$ sudo gem install jekyll`
  - `$ sudo gem install jekyll --no-rdoc --no-ri`   *문서설치 제외옵션, 빠른 설치가 가능하다.*
 
4. 버전을 체크해서 설치여부를 확인한다.
  - `$ jekyll -v`


### 3. Jekyll 실행

---

Jeykyll 까지 설치가 되었으면 github에서 clon한 로컬 레파지토리로 이동한다.   
Jekyll의 의존성 문제를 해결하기 위해 [bundler](http://ruby-korea.github.io/bundler-site/)를 이용한다.

bundler에 대해서 사이트에  간단히 설명된 자료를 보면,   

> 번들러는 필요한 정확한 gem과 버전을 추적하고 설치하여 루비 프로젝트를 위한 일관된 환경을 제공합니다.   
> 번들러는 의존성 지옥에서 벗어나게 하고, 필요한 gem이 개발, 스테이징, 프로덕션에 있는지 확인해 줍니다.   
> bundle install을 실행해 간단히 프로젝트에서 사용해 보세요. *( 출처: [http://ruby-korea.github.io/bundler-site/](http://ruby-korea.github.io/bundler-site/) )

 
본인의 경우, Fork를 한 테마가 budler기반으로 관리되고 있었기 때문에 일부 단점이 있음에도 불구하고,   
bundler를 그냥 쓰기로 했다.

bundler설치가 되어 있지 않으면 우선 설치부터 한다.

`$ gem install bundler`


설치가 완료되었으면 jekyll 실행을 위해 블로그 소스폴더로 이동한 다음,   
아래의 명령으로 필요한 의존성 패키지들을 체크하고 설치, 업데이트 하도록 한다.

`$ bundle install`

필요한 작업이 끝났으면 마지막으로 Jekyll을 구동한다.

`$ bundle exec jekyll serve`

``` 
Configuration file: /Users/user/Dropbox/workspace/blog/ykjang.github.io/_config.yml
            Source: /Users/user/Dropbox/workspace/blog/ykjang.github.io
       Destination: /Users/user/Dropbox/workspace/blog/ykjang.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 2.906 seconds.
 Auto-regeneration: enabled for '/Users/user/Dropbox/workspace/blog/ykjang.github.io'
Configuration file: /Users/user/Dropbox/workspace/blog/ykjang.github.io/_config.yml
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.`
```

이런 메세지가 올라오면 정상적으로 로컬에서 Jekyll이 실행된 것이다.
포스팅할 파일을 추가 또는 변경해보고 바로 로컬에서 반영되는지 확인해보자.

파일이 변경되면 변경로그가 바로 올라와서 확인이 가능하다.

이렇게 로컬에서 빠르게 본인의 글들을 확인하고 수정한 뒤,   
탈고(?)준비가 되었으면 gitbub으로 push하면 되겠다.



### [참고문서]

---

- [Jekyll 공식사이트](https://jekyllrb.com/)
- [Jekyll 테마](http://jekyllthemes.org/)
- [brew 사용법]( https://git.io/brew-docs)