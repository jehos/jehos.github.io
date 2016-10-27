---
layout: post
title:  Django 문서 번역을 위한 개발 환경 구축
date:   2016-10-27 20:32:00 +0900
published: true

categories: [web, translate]
---

# 어디서 부터 시작하나?

(지극히 개인적인 이유로) Django 에 기여하기로 마음먹고, 방법을 찾아보니
어디서부터 시작해야 할지 모를 정도로 복잡했다. 어딘가에 방법은 있었을텐데, 죄다
영어라서 눈에 안보였던 것일지도 모른다. 가장 먼저 찾아본건 공식 문서의
[contributing](https://docs.djangoproject.com/en/dev/internals/contributing/)
항목이었다. 

이 항목은 Django 에 기여하기를 원하는 사람들이 읽어야 할 자료들의 포털이었다.
이 곳이 일종의 진입점이었고, 여기서 내 관심사인
[translation](https://docs.djangoproject.com/en/1.10/internals/contributing/localizing/#translations)
항목을 찾아갔다. 다행히, 아주 일목 요연하게 필요한 것은 무엇이고 절차가
무엇인지 설명해 놓았다.

큰 단계를 나누자면 다음과 같다.

1. [Django internationalization and localization](https://groups.google.com/forum/#!forum/django-i18n) 에 가입할 것.
2. [transifex - django-docs](https://www.transifex.com/django/django-docs/) 에
   가입할 것.

메일링 리스트 가입은 번역시 발생하는 문제를 교류하기 위해서 이지만, 사실 글이
활발하게 올라오는 것 같지는 않다. transifex 서비스를 활용하여 번역 작업을
통합하는데, 각 언어마다 cordinator 가 존재하여 올라오는 번역들을 검수한다.
한국어 쪽의 번역은 약 2% 되어있는것으로 보아, outdated 된 자료이거나, 자동
번역의 결과물일 가능성이 있다. cordinator 가 지정되어 있는지는 확실하지 않다.

transifex 를 이용한 번역은 웹 에서 제공하는 번역 플랫폼을 이용하는 방법이고,
gettext 같은 도구를 사용하여 내 컴퓨터에서 개발환경을 직접 구축해놓고 번역하는
방법이 있다. 이를 [Specialties of Django
translation](https://docs.djangoproject.com/en/1.10/topics/i18n/translation/#specialties-of-django-i18n)
에서 언급하는데, 반드시 읽어보고 이를 연습해본다.

# 이 글에서는 무엇을 할것인가?

이 글에서 설명할 것은, gettext 를 이용한 번역에 대한 튜토리얼이다.
transifex 라는 웹 기반의 번역 도구가 있는데 왜 굳이 내 컴퓨터에서 따로
개발환경을 만들어야 하는지 궁금할 것이다. transifex 에서 작업된 번역물이
반영되는 시점은, django 의 새로운 버전이 올라오는 때에 일괄 적용된다. 
따라서 지금 당장 번역물을 올려도 한참 나중에서야 반영된다.

> Translations from Transifex are only integrated into the Django repository at the time of a new feature release. We try to update them a second time during one of the following patch releases, but that depends on the translation manager’s availability. So don’t miss the string freeze period (between the release candidate and the feature release) to take the opportunity to complete and fix the translations for your language!

*나는 번역을 한번에 완전 잘할 수 있어!* 라고 자신하지 말자, 절대 그럴리가 없다.
웹 플랫폼에서 작업한 결과와, 실제 프로그램 상에서 반영된 결과는 생각과는 많이
다르다. 따라서, 자신의 컴퓨터에서 개발환경을 구축하고, 직접 변경된 결과를 본 후
그 변경점들을 웹 플랫폼을 통해 일괄적으로 반영하는것이 좋다.

1. 번역 작업은 내 컴퓨터에서 구축된 개발 환경에서 진행하고
2. 그 번역작업의 결과물은 웹 번역 플랫폼을 통해 일괄적으로 반영한다.

따라서, (필요없어 보일수도 있지만) django 개발 환경을
구축하고, 거기에서 번역작업을 수행하는 방식에 대해 이해할 필요가 있다.


# 같이 보시오

여기에 나오는 내용은 공식 홈페이지의 메뉴얼을 참고하였다.
이 문서는 관리되지 않아 곧 녹이 슬테니, 가능하면 공식 홈페이지의 내용을 따르는
것이 좋다. 이곳의 내용은 개요(overview) 정도의 의미로만 생각해 주기 바란다.

* [Writing your first patch for Django](https://docs.djangoproject.com/en/dev/intro/contributing/)
* [Translation](https://docs.djangoproject.com/en/dev/topics/i18n/translation)
* [Sphinx -
  Internationalization](http://www.sphinx-doc.org/en/stable/intl.html)

# 절차

다음을 수행할 것이다.

1. 필요한 패키지 설치
2. virtualenvrwapper 로 Python 실행 환경 구성
3. 필요한 Python 모듈을 설치
4. django 소스 확보
5. gettext 로 번역을 위한 언어 별 골격을 추출
6. 번역
7. 번역된 결과 확인

# Prerequisite

이 튜토리얼은 다음의 환경을 기준으로 설명한다. 가능한한 특정한 환경에 묶이지
않고 일반적인 내용으로 서술하려고 하였으나 그러지 못하였다. 자신의 환경에
알맞게 번역해서 보아주시길 바란다. 버전 뒤의 `+` 는 `그 이상의 버전` 을
의미한다.

  * Ubuntu 16.04+
  * Python3.4+
  * Django 1.10+
  * virtualenvwrapper

다른 배포판에서도 상기의 내용을 모두 패키지로 제공한다. 다만, 배포판의 버전이
낮다면 보장하는 패키지의 버전 범위가 다르기 때문에, 지나치게 오래된
배포판이라면 수동으로 직접 컴파일 해서 설치해야 할 수도 있다. 다만 그 방법은 이
장에서 설명하기에는 너무 복잡해지므로, 최대한 근래에 나온 배포판을 위주로
진행해주시기 바랍니다. Fedora Core 나 Mint, Debian “stretch”, SUSE, Arch,
Gentoo 등 훌륭한 배포판은 확실하게 패키지로 제공할 것이며, FreeBSD 같은 Unix
에서도 잘 동작한다. 하고싶은 말은, **자신의 배포판 상황에 맞게 잘 해석해서
들어주세요**

# 패키지 설치

Ubuntu 16.04+ 버전이라면 다음의 명령으로 대부분의 의존성을 만족시킬 수 있을
것이다.

    sudo apt-get install virtualenvwrapper python3-pip git gettext

다음의 패키지가 설치된다.

  * **python3-pip** - Python3 를 위한 PIP 모듈 관리자
  * **virtualenvwrapper** - virtualenv 를 한단계 추상화 시켜 사용하기 편리하게 만든 유틸리티
  * **git** - django 소스를 받아오기 위한 버전관리도구
  * **gettext** - 번역 유틸리티, [Wikipedia - Gettext](https://en.wikipedia.org/wiki/Gettext)

# virtualenv 로 Python 실행 환경 생성

Python 과 같은 스크립트 언어는 버전에 따라 민감하게 반응한다. 동일한 실행
환경에서 다수의 프로젝트를 관리하다보면, 각 프로젝트마다 요구하는 모듈의 버전이 달라질
수 있는데, 이런 상황을 방지하기 위해 **독립된 것처럼 가상화된 Python 실행환경** 이 필요하다.

즉, **각 프로젝트마다 따로따로 가상화된 Python 실행 환경을 독립적으로 제공** 할
수 있게 된다. 실행 환경을 같이 쓸 때에는 A 라는 프로젝트에서 설치한 공용
라이브러리가, B 프로젝트에서는 말썽을 일으킬 수도 있지만, 실행 환경을 가상화
하여 나눠놓는다면, A 프로젝트만의 실행환경과 B 프로젝트만의 실행환경을 따로
구성할 수 있으므로 개발환경을 깨끗하게 유지할 수 있게 된다.

virtualenv 는 그러한 목적에서 태어났으며, virtualenvwrapper 는 한단계 추상화
시킨 wrapper 스크립트이다. 너무나 간단해서 몇개의 명령어만으로도 충분히 사용할
수 있다. `mkvirtualenv` 명령으로 Python 실행 환경을 생성할 수 있는데, 이때
`-p $(which python3)` 옵션을 붙임으로써, 이 Python 실행환경에서는 Python3 가
기본이 되도록 지정할 수 있다. 이 옵션을 붙이지 않으면 자칫 Python2 가 기본으로
지정될 수 있으니 유의해야 한다.

    mkvirtualenv -p $(which python3) django-devel

# `locale.Error` 가 발생할 경우

* 별도의 에러 없이 `mkvirtualenv` 가 성공한 경우, 이 부분을 건너띄기 바란다.

만약, 이 과정에서 `locale.Error: unsupported locale setting` 와 관련한 에러가
발생한다면, 이는 해당 시스템에서 사용하는 로케일의 데이터파일이 제대로 생성되지 
않았기 때문이다. 이 문제가 생기는 이유는 `ssh` 로 접속하는 경우 주로 발생하는데,
호스트 컴퓨터에서 ssh 를 통해 원격 서버로 접속할 때, 자동으로 로케일과 관련된 
`LC_*` (`LC_` 로 시작하는 모든 환경변수들을 의미)에 해당하는 쉘 환경 변수들이
ssh 클라이언트를 통해 서버측의 쉘 환경변수로 타고 넘어가는 설정이 되어있기 때문이다.

당신이 한국 로케일(`ko_KR.UTF8`) 를 사용하는 사용자이고, 
접속하는 서버에서도 이 환경을 그대로 유지하고 싶을 경우, 
이는 아주 당연하고 편리한 설정이다.
접속하는 원격 서버가 영어가 아닌 다른 국가의 로케일이라면 아주 난감하기 짝이 없을것이다.
에러메세지를 포함한 전체 메세지와 숫자, 시간, 날짜 표현까지 해당 국가의 로케일
설정으로 변경될 것이기 때문이다.

이 설정은 `sshd_config` 에서 지정되며, Ubuntu 의 경우 기본으로 이 설정이 되어 있다.
보내는 클라이언트와, 받는 서버측의 설정이 맞아야 일어나는 현상이다.

* Ubuntu 16.04 의 `/etc/ssh/ssh_config` 의 클라이언트 측 설정

      Host *:
          SendEnv LANG LC_*

* Ubuntu 16.04 의 `/etc/ssh/sshd_config` 서버측 설정

      # Allow client to pass locale environment variables
      AcceptEnv LANG LC_*

문제는, 원격 서버에서 이 로케일에 맞는 데이터를 generate 하지 않은 경우
해당 로케일을 실제 사용할 경우 에러가 발생한다는 점이다. 정리하자면 다음과
같다.

1. 내 컴퓨터에는 한글(`LC_ALL=ko_KR.UTF8`) 로 설정되어 있을 때,
2. ssh 명령으로 원격 서버에 접속할 때 이 환경이 원격 서버로 전달되어
3. 원격 서버에서는 있지도 않은 한글(`LC_ALL=ko_KR.UTF8`) 으로 쉘 환경이 설정된다.
4. 이 상태에서 Python 의 스크립트가 실행 도중 Locale 파일을 탐색하면,
이는 존재하지 않는 파일이므로 당연히 실패하게 된다.

이 경우, 선택은 다음과 같다.

* 해당 서버에 로케일 데이터를 generate 하도록 관리자에게 요청
* 해당 서버의 기본 로케일을 시스템에서 지원하는 기본 로케일로 수정
* ssh 접속할 때 아예 locale 관련 변수를 넘기지 않도록 ssh client 를 설정

이런 방법으로 문제를 해결할 수 있다. 맘 같아서는 세가지의 모든 방법을 자세하게
설명하고 싶지만, 일단 이 문제를 우회(workaround) 하는 쪽으로 방향을 잡겠다.

1. ssh 로 접속한 서버측 쉘의 로케일을 확인한다.
2. 당신이 관리자라면, 해당 로케일의 데이터를 재생성한다
3. 당신이 일반 사용자라면,
  * 서버측의 사용자 계정의 로케일 설정의 기본값을 강제로 바꾼다.
  * ssh 클라이언트에서 로케일 값을 전달하지 않거나, 기본값으로 전달하도록
    설정한다.

`locale` 명령은 현재 쉘 의 로케일을 확인한다. 다음과 같이 기본 로케일과 뒤섞여
있을 것이고, 에러메세지도 보일 것이다.

    jehos@django-devel:~$ locale
    locale: Cannot set LC_ALL to default locale: No such file or directory
    LANG=en_US.UTF-8
    LANGUAGE=
    LC_CTYPE="en_US.UTF-8"
    LC_NUMERIC=ko_KR.UTF-8
    LC_TIME=ko_KR.UTF-8
    LC_COLLATE="en_US.UTF-8"
    LC_MONETARY=ko_KR.UTF-8
    LC_MESSAGES="en_US.UTF-8"
    LC_PAPER=ko_KR.UTF-8
    LC_NAME=ko_KR.UTF-8
    LC_ADDRESS=ko_KR.UTF-8
    LC_TELEPHONE=ko_KR.UTF-8
    LC_MEASUREMENT=ko_KR.UTF-8
    LC_IDENTIFICATION=ko_KR.UTF-8
    LC_ALL=

만약 당신이 관리자일 경우, 서버상에서 간단히 `locale-gen` 으로 로케일 데이터를 
재생성 할 수 있다. 

    jehos@django-devel:~$ sudo locale-gen --lang ko_KR.UTF-8
    [sudo] password for jehos: # 암호 입력
    Generating locales (this might take a while)...
      ko_KR.UTF-8... done
    Generation complete.

만약 당신이 일반 사용자일 경우, 가장 간단한 방법은 `~/.profile` 파일에 
사용자 로케일 설정을 강제로 고정시키는 방법이다.

    echo "export LC_ALL=en_US.UTF8" >> ~/.profile

로그인 할 때마다 `~/.profile` 의 내용으로 쉘 환경이 초기화 하게 되는데, 
이때 시스템의 로케일을 초기화 시켜주는 명령어를 끼워 넣는 것이다.
따라서, 상기의 명령을 수행한 후, 반드시 로그아웃 후 다시 로그인해야 한다.
로그인 과정에서 `~/.profile` 의 내용으로 쉘을 다시 초기화 시킬 것이다.

다른 방법은, ssh 의 클라이언트로 하여금 로케일 설정을 아예 전달하지 않도록
하는 방법인데, ssh 클라이언트의 종류가 워낙 많지만, 명색이 ssh 클라이언트 라면
기본으로 제공해야 하는 기능이므로, 메뉴얼을 뒤져서 찾아보기 바란다.

로케일 문제가 해결되었다면, 다음과 같이 정상적으로 뜰 것이다.

    jehos@django-devel:~$ mkvirtualenv -p $(which python3) django-devel
    Already using interpreter /usr/bin/python3
    Using base prefix '/usr'
    New python executable in /home/jehos/.virtualenvs/django-devel/bin/python3
    Not overwriting existing python script
    /home/jehos/.virtualenvs/django-devel/bin/python (you must use
    /home/jehos/.virtualenvs/django-devel/bin/python3)
    Installing setuptools, pkg_resources, pip, wheel...done.
    (django-devel) jehos@django-devel:~$

# virtualenvwrapper 로 Python 실행 환경 운영

Python 실행 환경이 활성화(activate) 되면 프롬프트 앞에 괄호 로서 `( )` 
현재 활성화된 Python 실행 환경을 표시한다. 이를 해제하려면 `deactivate` 를 입력한다

    (django-devel) jehos@django-devel:~$ deactivate 
    jehos@django-devel:~$ 

다시 아까 생성한 Python 실행 환경을 활성화 해 보자.
`workon` 명령으로 활성화 시킬 수 있다.

    jehos@django-devel:~$ workon django-devel
    (django-devel) jehos@django-devel:~$ 

괄호`( )` 로 현재 활성화 된 Python 실행환경을 알 수 있다.

만약 존재하는 Python 실행 환경을 제거하기 위해서는, `rmvirtualenv` 로 삭제할 수
있다. 참고로, 삭제할 실행환경을 이미 사용중이라면 먼저 `deactivate` 시켜야 한다.

    (django-devel) jehos@django-devel:~$ rmvirtualenv django-devel
    Removing django-devel...
    ERROR: You cannot remove the active environment ('django-devel').
    Either switch to another environment, or run 'deactivate'.

    (django-devel) jehos@django-devel:~$ deactivate 

    jehos@django-devel:~$ rmvirtualenv django-devel
    Removing django-devel...
    jehos@django-devel:~$ 

여기 까지 따라해봤다면, 기본적인 `virtualenvwrapper` 의 사용법은 다 파악한
것이다. Python 으로 개발하는 사람이라면 반드시 알아야 할 필수 유틸리티 이니, 
조금 더 관심 있는 사람이라면
[virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/)
홈페이지를 정독해 보자. 

# 필요한 Python 모듈 설치

필요한 Python 의 모듈들을 설치해야 한다. 작업할 Python 실행 환경을 활성화 한
후, `pip` 명령으로 필요한 모듈들을 설치하면 된다. 가능하면 누락되지 않게 끔
하려고 노력했지만, 앞서 말했듯이 이 문서는 곧 녹이 슬 것이다. 따라서, 누락 된
모듈을 눈치껏 설치하는 지혜를 발휘해 주기 바란다.

일단 현 버전(django 1.10 기준) 으로 필요한 모듈은 다음과 같이 설치할 수 있다.

    pip install sphinx-intl django

* **django**
* **sphinx-intl**

# django 소스 내려받기

django 의 최신 소스를 내려받는다. django 에 commit 할 권한이 없는 여러분들은
다음의 명령으로 받아올 수 있다. 

    git clone https://github.com/django/django

정상적으로 `git clone` 이 완료되면, 다음과 같은 소스 디렉토리를 얻는다.

    django/
    ├── django
    ├── docs	# 문서
    ├── extras
    ├── js_tests
    ├── scripts
    └── tests	# 테스트 스위트

여기서, `docs/` 디렉토리로 이동하여 `gettext` 로 각 locale 의 골격을 추출한다.
해당 디렉토리 내에는 `Makefile` 스크립트가 있으므로, `make gettext` 명령으로 수행한다.

    (django-devel) jehos@django-devel:~/src/django/docs$ make gettext
    sphinx-build -b gettext   . _build/locale
    Running Sphinx v1.4.8
    making output directory...
    loading pickled environment... not yet created
    loading intersphinx inventory from https://pythonhosted.org/six/objects.inv...
    loading intersphinx inventory from http://sphinx-doc.org/objects.inv...
    intersphinx inventory has moved: http://sphinx-doc.org/objects.inv ->
    http://www.sphinx-doc.org/en/1.4.8/objects.inv
    loading intersphinx inventory from
    https://django-formtools.readthedocs.io/en/latest/objects.inv...
    loading intersphinx inventory from http://initd.org/psycopg/docs/objects.inv...
    loading intersphinx inventory from https://docs.python.org/3/objects.inv...
    building [gettext]: targets for 0 template files
    building [gettext]: targets for 374 source files that are out of date
    updating environment: 374 added, 0 changed, 0 removed
    reading sources... [100%] topics/testing/tools                                                                                              
    looking for now-outdated files... none found
    pickling environment... done
    checking consistency... done
    preparing documents... done
    writing output... [100%] topics/testing/tools                                                                                               
    writing message catalogs... [100%] contents                                                                                                 
    build succeeded.
    
    Build finished. The message catalogs are in _build/locale.

정상적으로 완료되면 `_build/locale` 디렉토리가 생성된다. 이 디렉토리는 각
언어별로 생성된 `.po` 언어 파일을 빌드할 때 사용되는 공간이다.

`sphinx-intl` 명령으로 각 언어별 골격을 생성해 보자.
[Sphinx - Internationalization](http://www.sphinx-doc.org/en/stable/intl.html)
항목을 살펴보기를 권한다.

    (django-devel) jehos@django-devel:~/src/django/docs$ sphinx-intl update -p
    _build/locale -l ko
    Create: locale/ko/LC_MESSAGES/internals.po
    Create: locale/ko/LC_MESSAGES/intro.po
    Create: locale/ko/LC_MESSAGES/howto.po
    Create: locale/ko/LC_MESSAGES/contents.po
    Create: locale/ko/LC_MESSAGES/topics.po
    Create: locale/ko/LC_MESSAGES/faq.po
    Create: locale/ko/LC_MESSAGES/releases.po
    Create: locale/ko/LC_MESSAGES/ref.po
    Create: locale/ko/LC_MESSAGES/glossary.po
    Create: locale/ko/LC_MESSAGES/index.po
    Create: locale/ko/LC_MESSAGES/misc.po

`sphinx-intl update -p _build/locale -l ko` 명령에서, `-p _build/locale` 의
`.pot` 파일을 원본으로 삼아, `-l ko` 옵션에 의해 한국어 로케일의 골격 파일인
`.po` 파일이 생성된다. 

    locale/
    └── ko
        └── LC_MESSAGES
            ├── contents.po
            ├── faq.po
            ├── glossary.po
            ├── howto.po
            ├── index.po
            ├── internals.po
            ├── intro.po
            ├── misc.po
            ├── ref.po
            ├── releases.po
            └── topics.po

이 각각의 `.po` 파일들이 바로 원문을 번역할 수 있는 틀이 된다. 내용을 열어보면
다음과 같이 구성되어 있다. `intro.po` 를 열어보면 다음과 같다.

    #: ../../intro/contributing.txt:3
    msgid "Writing your first patch for Django"
    msgstr ""
    
    #: ../../intro/contributing.txt:6
    msgid "Introduction"
    msgstr ""
    
    #: ../../intro/contributing.txt:8
    msgid ""
    "Interested in giving back to the community a little? Maybe you've found a"
    " bug in Django that you'd like to see fixed, or maybe there's a small "
    "feature you want added."
    msgstr ""

이 문법은 **reStructuredText Markup** 이며, 줄여서 **rst** 마크업 언어라고도
부른다. 다음의 두 링크를 참조할 수 있다.

* [reStructuredText](http://docutils.sourceforge.net/rst.html)
* [reStructuredText Primer](http://www.sphinx-doc.org/en/1.4.8/rest.html)

아주 민감해서, 처음에는 계속된 실수를 경험하게 될 것이다. 실수를 반복하면서
규칙을 금방 알아챌 수 있다는건 그나마 다행이다. 그만큼 문법은 아주 쉽다.

예를들어, 상기의 `intro.po` 를 번역하면 다음과 같은 형태가 된다.

    #: ../../intro/contributing.txt:3
    msgid "Writing your first patch for Django"
    msgstr "Django 를 위한 패치를 작성해보세요"
    
    #: ../../intro/contributing.txt:6
    msgid "Introduction"
    msgstr "소개"
    
    #: ../../intro/contributing.txt:8
    msgid ""
    "Interested in giving back to the community a little? Maybe you've found a"
    " bug in Django that you'd like to see fixed, or maybe there's a small "
    "feature you want added."
    msgstr ""
    "커뮤니티 기여에 관심이 있으신가요? "
    "Django 에서 고치고 싶은 버그를 발견했거나, 작은 기능을 추가하고 싶으신가요?"
    
    #: ../../intro/contributing.txt:12
    msgid ""
    "Contributing back to Django itself is the best way to see your own "
    "concerns addressed. This may seem daunting at first, but it's really "
    "pretty simple. We'll walk you through the entire process, so you can "
    "learn by example."
    msgstr ""
    "Django 를 위해 기여하는 것은 자신의 문제를 직접 해결하는데 가장 좋은
    방법입니다. "
    "처음에는 매우 어려워 보이지만 사실 매우 간단합니다. "
    "전체 과정을 하나씩 따라가보며 예제를 통해 배워보겠습니다"

한줄로 쓸 때는 `msgstr` 에 바로 붙여서 쓰면 되고, 여러줄로 쓸 때에는 `""` 를 쓴
후, 다음 줄 부터 쓰면 된다. 한번 빌드하여 제대로 출력되는지 확인해보자.
`make html` 명령이면 전체를 모두 빌드하게 되지만, 시간이 굉장히 오래 걸리므로, 
`make -e SPHINXOPTS="-D language='ko'" html` 명령으로 한국어 로케일만 빌드하도록
하자.

    (django-devel) jehos@django-devel:~/src/django/docs$ make -e SPHINXOPTS="-D language='ko'" html
    sphinx-build -b djangohtml -n -d _build/doctrees -D language=  -D language='ko' . _build/html
    Running Sphinx v1.4.8
    loading translations [ko]... done
    loading pickled environment... done
    building [mo]: targets for 11 po files that are out of date
    writing output... [100%] locale/ko/LC_MESSAGES/misc.mo                                                                                      
    building [djangohtml]: targets for 374 source files that are out of date
    updating environment: [config changed] 374 added, 0 changed, 0 removed
    reading sources... [100%] topics/testing/tools                                                                                              
    looking for now-outdated files... none found
    pickling environment... done
    checking consistency... done
    preparing documents... done
    writing output... [100%] topics/testing/tools                                                                                               
    generating indices... genindex py-modindex
    highlighting module code... [100%] django.utils.dateparse                                                                                   
    writing additional pages... search
    copying images... [100%] internals/_images/triage_process.svg                                                                               
    copying downloadable files... [100%]
    ref/contrib/gis/install/geodjango_setup.bat                                                            
    copying static files... done
    copying extra files... done
    dumping search index in English (code: en) ... done
    dumping object inventory... done
    writing templatebuiltins.js...
    build succeeded.
    
    Build finished. The HTML pages are in _build/html.

성공적으로 빌드 되었고, `_build/html` 에 원하는 결과가 있음을 알 수 있다.
이 파일은 정적 html 파일이므로, 브라우저에서 곧바로 열어보면 된다.

# 개발 환경에서의 번역 작업 흐름

![workflow-translate-sphinx](http://www.sphinx-doc.org/en/stable/_images/translation.png "Sphinx 의 번역 흐름도")

이 그림에 위에서 작업한 모든 내용이 정리되어 있다.

1. `make gettext` 명령은 사실 `sphinx-build -b gettext` 를 호출한다.
이때 생성된 `.pot` 파일은 `_build/locale` 디렉토리에 생성된다.
2.`sphinx-intl update -p _build/locale -l ko` 명령을 통해 `.pot` 파일에서 `.po`
파일을 뽑아낸다.
3. 이렇게 추출한 `.po` 파일을 번역자가 직접 작업한다.
4. 번역이 완료되면 `make -e SPHINXOPTS="-D language='ko'" html` 명령을 통해
   원하는 포맷(이 경우엔 html) 로 빌드하여 확인한다.
5. 번역한 `.po` 파일은 transifex 를 통해 업로드 하여 다음 django 릴리즈에
   반영시킨다.

`patch` 나 pull request 를 작성하여 바로 소스에 반영하는 방법도 물론 있겠지만,
아마도 번역의 일관성을 위해 반드시 cordinator 를 거치게끔 하려고 transifex 를
인터페이스로 삼는 것으로 보인다. 만약 이러한 번역 작업물이 사방팔방에서
검증없이 날아들면, 공동의 작업은 금방 누더기가 되어 버릴것이다.

때문에, 이 개발환경을 구축하여 곧바로 확인할 수 있게 한 것은 작업 도중에 미리
결과를 확인하게끔 하며, 이때 생성한 결과물을 transifex 에 제출할 수 있도록
도와주는 것이다.

# 일단 정리!

이 글에서는 **Django 의 문서 번역을 위한 개발 환경 구축하기** 를 주제로 하였다.
사실 *rst 문서 안깨지게 쓰기* 같은걸 추가로 더 써야 하는데, 여기까지 쓰는데만도
4시간이 훌쩍 넘어버려서 더이상 쓰는것도 힘들고, 보는 사람은 더 힘들 것 같아서
이만 글을 줄인다. 사실 이 글도 지금 몇개의 단위로 쪼개야 할 것처럼 보이는데..

원래는 transifex 에 번역 결과물을 적용 하는 데 까지 진행했어야 하는데, 아직
승인이 나지 않아서 대기중이다. 조만간 승인이 나면 나머지 작업 진행에 대한 글을
올리도록 하겠다.
