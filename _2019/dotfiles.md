---
layout: lecture
title: "닷파일"
presenter: Anish
date: 2019-01-24
order: 1
video:
  aspect: 62.5
  id: YSZBWWJw3mI
translation_status: draft
---

많은 프로그램은 "닷파일"이라는 일반 텍스트 파일을 사용해 설정합니다(파일 이름이 `.`으로 시작하기 때문에 붙은 이름으로, `~/.gitconfig`가 그 예이며, 이 때문에 기본적으로 `ls`의 디렉토리 목록에 나타나지 않습니다).

여러분이 사용하는 많은 도구에는 아마 매우 세밀하게 조정할 수 있는 설정이 많이 있을 것입니다. 도구는 전용 언어로 커스터마이징하는 경우가 많습니다. 예를 들어 Vim에는 Vimscript를, 셸에는 셸 자체 언어를 사용합니다.

도구를 커스터마이징하고 선호하는 작업 방식에 맞게 조정하면 생산성이 높아집니다. GitHub에서 다른 사람의 닷파일을 복제하기보다는 직접 도구를 커스터마이징하는 데 시간을 들이기를 권합니다.

이미 설정해 둔 닷파일이 있을 것입니다. 다음 위치들을 살펴보세요.

- `~/.bashrc`
- `~/.emacs`
- `~/.vim`
- `~/.gitconfig`

일부 프로그램은 파일을 홈 폴더 바로 아래에 두지 않고 `~/.config` 아래의 폴더에 둡니다.

닷파일은 명령줄 애플리케이션만 사용하는 것이 아닙니다. 예를 들어 [MPV](https://mpv.io/) 동영상 플레이어는 `~/.config/mpv` 아래의 파일을 편집해 설정할 수 있습니다.

# 도구 커스터마이징 배우기

온라인 문서나 [매뉴얼 페이지](https://en.wikipedia.org/wiki/Man_page)를 읽으면 도구의 설정을 배울 수 있습니다. 또 다른 좋은 방법은 인터넷에서 특정 프로그램에 관한 블로그 글을 검색하는 것입니다. 글쓴이가 선호하는 설정 방법을 알려 줍니다. 다른 사람의 닷파일을 살펴보는 것도 설정 방법을 배우는 길입니다. GitHub에는 [닷파일 저장소](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories)가 아주 많습니다. 가장 인기 있는 저장소는 [여기](https://github.com/mathiasbynens/dotfiles)에서 볼 수 있습니다(다만 설정을 무작정 복사하지 않기를 권합니다).

# 정리 방법

닷파일을 어떻게 정리해야 할까요? 별도 폴더에 넣어 버전 관리를 하고, 스크립트를 사용해 원래 위치에 **심볼릭 링크**를 만들어야 합니다. 이렇게 하면 다음과 같은 이점이 있습니다.

- **쉬운 설치**: 새 컴퓨터에 로그인했을 때, 여러분의 사용자 설정을 적용하는 데 1분이면 충분합니다.
- **이식성**: 어디에서나 도구가 똑같이 동작합니다.
- **동기화**: 어디에서든 닷파일을 업데이트하고 모두 동기화된 상태로 유지할 수 있습니다.
- **변경 추적**: 프로그래머로 일하는 내내 닷파일을 관리하게 될 가능성이 크며, 오래 지속되는 프로젝트에는 버전 기록이 있으면 좋습니다.

```shell
cd ~/src
mkdir dotfiles
cd dotfiles
git init
touch bashrc
# create a bashrc with some settings, e.g.:
#     PS1='\w > '
touch install
chmod +x install
# insert the following into the install script:
#     #!/usr/bin/env bash
#     BASEDIR=$(dirname $0)
#     cd $BASEDIR
#
#     ln -s ${PWD}/bashrc ~/.bashrc
git add bashrc install
git commit -m 'Initial commit'
```

# 고급 주제

## 컴퓨터별 설정

대개는 여러 컴퓨터에서 같은 설정을 사용하고 싶지만, 특정 컴퓨터에서만 조금 다르게 하고 싶을 때도 있습니다. 이 상황을 처리하는 방법을 몇 가지 소개합니다.

### 컴퓨터별 브랜치

버전 관리로 컴퓨터마다 브랜치를 하나씩 유지하세요. 이 접근법은 논리적으로 단순하지만, 상당히 번거로울 수 있습니다.

### if 문

설정 파일에서 지원한다면 if 문에 해당하는 기능을 사용해 컴퓨터별 설정을 적용하세요. 예를 들어 셸에 다음과 같은 내용을 넣을 수 있습니다.

```shell
if [[ "$(uname)" == "Linux" ]]; then {do_something else}; fi

# Darwin is the architecture name for macOS systems
if [[ "$(uname)" == "Darwin" ]]; then {do_something}; fi

# You can also make it machine specific
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

### 파일 포함

설정 파일에서 지원한다면 다른 파일을 포함하는 기능을 사용하세요. 예를 들어 `~/.gitconfig`에 다음 설정을 넣을 수 있습니다.

```
[include]
    path = ~/.gitconfig_local
```

그러면 각 컴퓨터의 `~/.gitconfig_local`에 컴퓨터별 설정을 넣을 수 있습니다. 이러한 컴퓨터별 설정을 별도 저장소에서 관리할 수도 있습니다.

이 아이디어는 서로 다른 프로그램이 일부 설정을 공유하게 하고 싶을 때도 유용합니다. 예를 들어 `bash`와 `zsh`가 같은 별칭 모음을 공유하게 하려면 `.aliases`에 별칭을 작성하고 양쪽에 다음 블록을 넣으면 됩니다.

```bash
# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

# 참고 자료

- 강사진의 닷파일:
  [Anish](https://github.com/anishathalye/dotfiles),
  [Jon](https://github.com/jonhoo/configs),
  [Jose](https://github.com/jjgo/dotfiles)
- [GitHub does dotfiles](https://dotfiles.github.io/): 닷파일 프레임워크, 유틸리티, 예제, 튜토리얼
- [셸 시작 스크립트](https://web.archive.org/web/20260329133158/https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html): 셸에 사용하는 여러 설정 파일에 대한 설명

# 연습 문제

1. 닷파일을 넣을 폴더를 만들고 [버전 관리](/2019/version-control/)를 설정하세요.

1. 최소한 프로그램 하나에 사용자 설정을 추가하세요. 예를 들어 셸을 커스터마이징할 수 있습니다(처음에는 `$PS1`을 설정해 셸 프롬프트를 커스터마이징하는 정도로 간단해도 됩니다).

1. 새 컴퓨터에 닷파일을 빠르게 (그리고 수작업 없이) 설치하는 방법을 마련하세요. 파일마다 `ln -s`를 호출하는 셸 스크립트처럼 간단한 것이어도 되고, [전용 유틸리티](https://dotfiles.github.io/utilities/)를 사용해도 됩니다.

1. 새 가상 머신에서 설치 스크립트를 시험하세요.

1. 현재 사용하는 도구의 설정을 모두 닷파일 저장소로 옮기세요.

1. GitHub에 닷파일을 공개하세요.
