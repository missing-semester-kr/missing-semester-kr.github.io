---
layout: lecture
title: "명령줄 환경"
presenter: Jose
date: 2019-01-17
order: 1
video:
  aspect: 62.5
  id: i0rf1gpKL1E
translation_status: draft
---

## 별칭과 함수

짐작할 수 있듯이, 플래그가 많거나 옵션이 장황한 긴 명령어를 입력하는 일은 번거로워질 수 있습니다. 그래도 대부분의 셸은 **별칭**을 지원합니다. 예를 들어 bash의 별칭은 다음 구조를 가집니다(`=` 기호 양옆에 공백이 없다는 점에 주의하세요).

```bash
alias alias_name="command_to_alias"
```

<!-- `alias ll=ls -ltAh`처럼 명령어에 자주 쓰는 플래그를 별칭으로 만들 수 있습니다. 별칭은 조합할 수 있습니다.  -->

별칭에는 편리한 기능이 많습니다.

```bash
# Alias can summarize good default flags
alias ll="ls -lh"

# Save a lot of typing for common commands
alias gc="git commit"

# Alias can overwrite existing commands
alias mv="mv -i"
alias mkdir="mkdir -p"

# Alias can be composed
alias la="ls -A"
alias lla="la -l"

# To ignore an alias run it prepended with \
\ls
# Or can be disabled using unalias
unalias la

```
<!--
별칭을 없애려면 `unalias alias_name`을 실행하면 되고, 명령어를 실행할 때 별칭을 무시하려면 명령어 앞에 역슬래시를 붙여 `\alias_name`처럼 실행할 수 있습니다. 별칭이 기존 이름을 덮어쓰는 경우에 편리합니다. -->


하지만 여러 상황에서 별칭에는 한계가 있습니다. 특히 같은 인자를 받는 명령어들을 연결해 작성하려고 할 때 그렇습니다. 대안으로 **함수**가 있으며, 별칭과 직접 만든 셸 스크립트 사이의 중간 지점에 있습니다.

다음은 디렉토리를 만들고 그 안으로 이동하는 함수의 예입니다.

```bash
mcd () {
    mkdir -p $1
    cd $1
}
```

별칭과 함수는 기본적으로 셸 세션이 바뀌면 유지되지 않습니다. 별칭을 계속 유지하려면 `.bashrc`나 `.zshrc` 같은 셸 시작 스크립트 파일 중 하나에 넣어야 합니다. 저는 별칭을 `.alias`에 따로 작성하고, 여러 셸 설정 파일에서 그 파일을 `source`하는 것을 권합니다.

<!-- 마지막으로, 이 도구 중 하나에 "개선된" 버전을 별칭으로 지정하기로 했다면, 예를 들어 `alias bat=cat`처럼 했다면, bash에서 `\cat`으로 별칭을 무시하고 `command cat`으로 별칭과 함수를 모두 무시할 수 있다는 점을 알아 두면 유용합니다. -->

## 셸과 프레임워크

셸과 스크립팅 강의에서는 `bash` 셸을 다루었습니다. 단연 가장 널리 쓰이는 셸이고 대부분의 시스템에서 기본으로 제공하기 때문입니다. 하지만 유일한 선택지는 아닙니다.

예를 들어 `zsh` 셸은 `bash`의 상위 집합으로, 별도 설정 없이 다음과 같은 편리한 기능을 많이 제공합니다.

- 더 똑똑한 글로빙, `**`
- 인라인 글로빙/와일드카드 확장
- 철자 교정
- 더 나은 탭 완성/선택
- 경로 확장(`cd /u/lo/b`가 `/usr/local/bin`으로 확장됩니다.)

또한 여러 셸은 **프레임워크**로 개선할 수 있습니다. [prezto](https://github.com/sorin-ionescu/prezto)나 [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)처럼 인기 있는 범용 프레임워크도 있고, [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)이나 [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)처럼 특정 기능에 집중한 작은 프레임워크도 있습니다. [fish](https://fishshell.com/) 같은 다른 셸은 이런 사용자 친화적 기능을 많이 기본으로 제공합니다. 이런 기능의 예는 다음과 같습니다.

- 오른쪽 프롬프트
- 명령어 구문 강조
- 기록 부분 문자열 검색
- man 페이지를 바탕으로 한 플래그 완성
- 더 똑똑한 자동 완성
- 프롬프트 테마

이런 프레임워크를 사용할 때 주의할 점은 실행하는 코드가 제대로 최적화되지 않았거나 코드가 너무 많으면 셸이 느려지기 시작할 수 있다는 것입니다. 언제든 프로파일링을 하고, 자주 사용하지 않거나 속도보다 중요하게 여기지 않는 기능을 끌 수 있습니다.

## 터미널 에뮬레이터와 멀티플렉서

셸을 원하는 대로 설정하는 것과 함께, 사용할 **터미널 에뮬레이터**와 그 설정을 알아보는 데 시간을 들일 가치가 있습니다. 터미널 에뮬레이터는 많이 있습니다([비교 자료](https://anarc.at/blog/2018-04-12-terminal-emulators-1/)를 보세요).

터미널에서 수백 시간에서 수천 시간을 보낼 수도 있으므로 설정을 살펴보면 보람이 있습니다. 터미널에서 바꾸고 싶을 만한 항목은 다음과 같습니다.

- 글꼴 선택
- 색 구성표
- 키보드 단축키
- 탭/분할 창 지원
- 스크롤백 설정
- 성능([Alacritty](https://github.com/jwilm/alacritty) 같은 일부 최신 터미널은 GPU 가속을 제공합니다.)

[tmux](https://github.com/tmux/tmux) 같은 **터미널 멀티플렉서**도 언급할 만합니다. `tmux`를 사용하면 여러 셸 세션을 분할 창과 탭으로 나눌 수 있습니다. 연결과 분리도 지원하는데, 이는 원격 서버에서 작업하면서 현재 프로세스를 disown할 걱정 없이 셸을 계속 실행해 두고 싶을 때 매우 흔히 쓰이는 기능입니다(기본적으로 로그아웃하면 프로세스가 종료됩니다). 이렇게 `tmux`를 사용하면 복잡한 터미널 배치에 들어갔다 나올 수 있습니다. 터미널 에뮬레이터와 마찬가지로 `tmux`도 `~/.tmux.conf` 파일을 편집하여 폭넓게 설정할 수 있습니다.


## 명령줄 유틸리티

대부분의 UNIX 기반 운영체제가 기본으로 제공하는 명령줄 유틸리티만으로도 보통 해야 하는 일의 99%를 하기에 충분하고도 남습니다.


다음 몇 개의 절에서는 아주 흔한 셸 작업에 사용할 수 있는, 사용하기 더 편리한 대체 도구를 다루겠습니다. 이 도구 중 일부는 명령어에 새롭고 개선된 기능을 더하고, 다른 도구는 더 나은 기본 설정과 함께 더 간단하고 직관적인 인터페이스를 제공하는 데만 집중합니다.

### `fasd`와 `cd`

개선된 경로 확장과 탭 자동 완성이 있어도 디렉토리 변경은 꽤 반복적인 일이 될 수 있습니다. [Fasd](https://github.com/clvv/fasd)(또는 [autojump](https://github.com/wting/autojump))는 최근에 방문했거나 자주 방문하는 폴더를 추적하고 퍼지 매칭을 수행하여 이 문제를 해결합니다.

따라서 제가 `/home/user/awesome_project/code` 경로에 방문한 적이 있다면 `z code`를 실행하면 그곳으로 `cd`합니다. code라는 폴더가 여러 개라면 `z awe code`를 실행해서 구별할 수 있으며, 이렇게 하면 더 가까운 일치 결과가 됩니다. autojump와 달리 fasd는 `cd`를 수행하는 대신 자주 사용하거나 최근에 사용한 파일, 폴더 또는 둘 다를 확장해 주기만 하는 명령어도 제공합니다.


### `bat`과 `cat`

`cat`도 제 역할을 완벽하게 하지만, [bat](https://github.com/sharkdp/bat)은 구문 강조, 페이징, 줄 번호, git 연동을 제공하여 이를 개선합니다.


### `exa`/`ranger`와 `ls`

`ls`는 훌륭한 명령어지만, 크기를 바이트 그대로 표시하는 것처럼 일부 기본 설정은 불편할 수 있습니다. [exa](https://github.com/ogham/exa)는 더 나은 기본 설정을 제공합니다.

많은 폴더를 탐색하거나 많은 파일을 미리 봐야 한다면, [ranger](https://github.com/ranger/ranger)는 훌륭한 인터페이스 덕분에 `cd`와 `cat`보다 훨씬 효율적일 수 있습니다. 상당히 자유롭게 설정할 수 있으며, 제대로 설정하면 터미널에서 [이미지를 미리 볼](https://github.com/ranger/ranger/wiki/Image-Previews) 수도 있습니다.

### `fd`와 `find`

[fd](https://github.com/sharkdp/fd)는 `find`를 대체하는 간단하고 빠르며 사용자 친화적인 도구입니다. `--name` 플래그를 사용해야 한다는 것(99%의 경우에 하고 싶은 일이 이것입니다) 같은 `find`의 기본 동작은 일상적으로 사용하기 더 쉽게 해 줍니다. 또한 `git`을 인식하여 기본적으로 `.gitignore`에 있는 파일과 `.git` 폴더를 건너뜁니다. 기본적으로 보기 좋은 색상 구분도 제공합니다.

### `rg/fzf`와 `grep`

`grep`은 훌륭한 도구지만, 여러 파일을 한 번에 grep하고 싶다면 그 목적에 더 나은 도구가 있습니다. [ack](https://github.com/beyondgrep/ack3), [ag](https://github.com/ggreer/the_silver_searcher), [rg](https://github.com/BurntSushi/ripgrep)는 gitignore 규칙을 지키면서 현재 디렉토리에서 정규 표현식 패턴을 재귀적으로 검색합니다. 모두 비슷하게 동작하지만, 저는 홈 디렉토리 전체를 매우 빠르게 검색할 수 있는 `rg`를 선호합니다.

마찬가지로 `CMD | grep PATTERN`을 계속해서 반복하고 있는 자신을 쉽게 발견할 수 있습니다. [fzf](https://github.com/junegunn/fzf)는 거의 모든 명령어의 출력을 대화형으로 필터링할 수 있게 해 주는 명령줄 퍼지 검색 도구입니다.

### `rsync`와 `cp/scp`

`mv`와 `scp`는 대부분의 상황에서 완벽하지만, 많은 파일이나 큰 파일을 복사/이동할 때, 또는 데이터 일부가 목적지에 이미 있을 때는 `rsync`가 훨씬 낫습니다. `rsync`는 이미 전송한 파일을 건너뛰며, `--partial` 플래그를 사용하면 이전에 중단된 복사를 이어서 할 수 있습니다.

### `trash`와 `rm`

`rm`은 파일을 한 번 지우면 되돌릴 수 없다는 점에서 위험한 명령어입니다. 하지만 현대 운영체제는 파일 탐색기에서 무언가를 지울 때 그렇게 동작하지 않습니다. 주기적으로 비워지는 휴지통 폴더로 옮길 뿐입니다.

휴지통을 관리하는 방식은 운영체제마다 다르므로 하나의 공통 CLI 유틸리티는 없습니다. 여러 도구 중 macOS에는 [trash](https://hasseg.org/trash/)가 있고, linux에는 [trash-cli](https://github.com/andreafrancia/trash-cli/)가 있습니다.

### `mosh`와 `ssh`

`ssh `는 매우 편리한 도구지만, 연결이 느리면 지연이 거슬릴 수 있고 연결이 끊어지면 다시 연결해야 합니다. [mosh](https://mosh.org/)는 로밍을 허용하고, 간헐적인 연결을 지원하며, 지능적인 로컬 에코를 제공하는 편리한 도구입니다.

### `tldr`와 `man`

대부분의 경우 `man`과 `-h`/'--help' 플래그로 명령어가 무슨 일을 하고 어떤 옵션이 있는지 알아낼 수 있습니다. 하지만 설명이 자세하다면 이를 살펴보는 일이 다소 벅찰 수도 있습니다.

[tldr](https://github.com/tldr-pages/tldr) 명령어는 명령줄에서 사용할 수 있는 커뮤니티 주도의 문서 시스템으로, 명령어가 무슨 일을 하는지와 가장 흔한 인자 옵션을 보여 주는 간단한 예제를 몇 개 제공합니다.


### `aunpack`과 `tar/unzip/unrar`

[이 xkcd](https://xkcd.com/1168/)에서 언급하듯이 `tar`의 옵션을 기억하는 일은 꽤 까다로울 수 있고, .rar 파일의 `unrar`처럼 완전히 다른 도구가 필요할 때도 있습니다.
[atool](https://www.nongnu.org/atool/) 패키지가 제공하는 `aunpack` 명령어는 올바른 옵션을 알아내고, 압축을 푼 내용은 항상 새 폴더에 넣습니다.


## 연습 문제

1. `cat .bash_history | sort | uniq -c | sort -rn | head -n 10`(zsh라면 `cat .zhistory | sort | uniq -c | sort -rn | head -n 10`)을 실행해 가장 많이 사용한 명령어 10개를 구하고, 이 명령어에 짧은 별칭을 만드는 것을 고려해 보세요.
1. 터미널 에뮬레이터를 하나 고르고 다음 속성을 바꾸는 방법을 알아보세요.
    - 글꼴 선택
    - 색 구성표. 표준 색 구성표에는 색이 몇 개 있나요? 왜 그럴까요?
    - 스크롤백 기록 크기

1. `fasd`나 비슷한 소프트웨어를 설치하고, 전달된 인자에 퍼지 매칭을 수행해 가장 높은 순위의 결과를 여러분이 선택한 편집기에서 여는 `v`라는 bash/zsh 함수를 작성하세요. 그런 다음 일치하는 결과가 여러 개일 때 `fzf`로 선택할 수 있도록 수정하세요.
1. `fzf`는 퍼지 검색을 수행하기에 꽤 편리하고 셸 기록은 이런 종류의 검색을 자주 하게 되는 대상이므로, `fzf`를 `^R`에 연결하는 방법을 알아보세요. [여기](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings)에서 정보를 찾을 수 있습니다.
1. `ack`의 `--bar` 옵션은 무슨 일을 하나요?
