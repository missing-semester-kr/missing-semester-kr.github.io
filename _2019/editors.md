---
layout: lecture
title: "편집기"
presenter: Anish
date: 2019-01-22
order: 1
video:
  aspect: 62.5
  id: 1vLcusYSrI4
translation_status: draft
---

# 편집기의 중요성

프로그래머인 우리는 대부분의 시간을 일반 텍스트 파일을 편집하며 보냅니다. 자신의 필요에 맞는 편집기를 배우는 데 시간을 투자할 만합니다.

새 편집기는 어떻게 배울까요? 일시적으로 생산성이 떨어지더라도 한동안 그 편집기를 반드시 사용하도록 스스로 강제하면 됩니다. 머지않아 보상을 받을 것입니다(기본을 배우기에는 2주면 충분합니다).

우리는 Vim을 가르칠 예정이지만, 다른 편집기도 시험해 보기를 권합니다. 편집기는 매우 개인적인 선택이며, 사람들은 [강한 의견](https://en.wikipedia.org/wiki/Editor_war)을 가지고 있습니다.

50분 안에 강력한 편집기의 사용법을 가르칠 수는 없으므로, 기본을 가르치고 고급 기능 일부를 보여 주며 도구를 숙달할 수 있는 자료를 제공하는 데 집중하겠습니다. Vim을 바탕으로 가르치지만, 대부분의 아이디어는 여러분이 사용하는 다른 강력한 편집기에도 적용됩니다(적용되지 않는다면 그 편집기는 사용하지 않는 편이 좋을 것입니다!).

![편집기 학습 곡선](/2019/files/editor-learning-curves.jpg)

<!-- source: https://blogs.msdn.microsoft.com/steverowe/2004/11/17/code-editor-learning-curves/ -->

편집기 학습 곡선 그래프는 사실과 다릅니다. 강력한 편집기의 기본을 배우기는 꽤 쉽습니다(숙달하려면 몇 년이 걸릴 수 있지만요).

요즘 어떤 편집기가 인기 있을까요? 이 [Stack Overflow 설문 조사](https://insights.stackoverflow.com/survey/2018/#development-environments-and-tools)를 보세요(Stack Overflow 사용자가 프로그래머 전체를 대표하지 않을 수 있으므로 어느 정도 편향이 있을 수 있습니다).

## 명령줄 편집기

결국 GUI 편집기를 쓰기로 결정하더라도, 원격 컴퓨터에서 파일을 쉽게 편집할 수 있도록 명령줄 편집기를 배워 둘 만합니다.

# Nano

Nano는 단순한 명령줄 편집기입니다.

- 방향키로 이동합니다.
- 다른 모든 단축키(저장, 종료)는 아래쪽에 표시됩니다.

# Vim

Vi/Vim은 강력한 텍스트 편집기입니다. 보통 어디에나 설치되어 있는 명령줄 프로그램이므로, 원격 컴퓨터에서 파일을 편집할 때 편리합니다.

Vim에는 GVim이나 [MacVim](https://macvim-dev.github.io/macvim/) 같은 그래픽 버전도 있습니다. 이들은 24비트 색상, 메뉴, 팝업과 같은 추가 기능을 제공합니다.

## Vim의 철학

- 프로그래밍할 때는 작성보다 읽기/편집에 대부분의 시간을 씁니다.
    - Vim은 **모드 기반** 편집기입니다. 텍스트 삽입과 텍스트 조작에 서로 다른 모드를 사용합니다.
- Vim은 프로그래밍할 수 있습니다(Vimscript뿐 아니라 Python 같은 다른 언어로도 가능합니다).
- Vim의 인터페이스 자체가 프로그래밍 언어와 같습니다.
    - (기억하기 쉬운 이름을 가진) 키 입력이 명령어입니다.
    - 명령어를 조합할 수 있습니다.
- 마우스를 사용하지 마세요. 너무 느립니다.
- 편집기는 여러분이 생각하는 속도로 동작해야 합니다.

## Vim 입문

### 모드

Vim은 왼쪽 아래에 현재 모드를 표시합니다.

- Normal 모드: 파일 안에서 이동하고 편집할 때 사용합니다.
    - 대부분의 시간을 여기서 보내세요.
- Insert 모드: 텍스트를 삽입할 때 사용합니다.
- Visual (visual, line, block) 모드: 텍스트 블록을 선택할 때 사용합니다.

어느 모드에서든 `<ESC>`를 누르면 Normal 모드로 돌아갈 수 있습니다. Normal 모드에서는 `i`로 Insert 모드에, `v`로 Visual 모드에, `V`로 Visual Line 모드에, `<C-v>`로 Visual Block 모드에 들어갑니다.

Vim을 사용할 때는 `<ESC>` 키를 자주 누르게 됩니다. Caps Lock을 Escape로 다시 매핑하는 것을 고려해 보세요.

### 기본

Vim의 ex 명령어는 Normal 모드에서 `:{command}`로 실행합니다.

- `:q` 종료(창 닫기)
- `:w` 저장
- `:wq` 저장하고 종료
- `:e {name of file}` 파일을 열어 편집
- `:ls` 열려 있는 버퍼 표시
- `:help {topic}` 도움말 열기
    - `:help :w`는 `:w` ex 명령어의 도움말을 엽니다.
    - `:help w`는 `w` 이동의 도움말을 엽니다.

### 이동

Vim은 효율적인 이동이 핵심입니다. Normal 모드에서 파일 안을 이동하세요.

- 나쁜 습관이 들지 않도록 방향키를 비활성화하세요.
```vim
nnoremap <Left> :echoe "Use h"<CR>
nnoremap <Right> :echoe "Use l"<CR>
nnoremap <Up> :echoe "Use k"<CR>
nnoremap <Down> :echoe "Use j"<CR>
```
- 기본 이동: `hjkl`(왼쪽, 아래, 위, 오른쪽)
- 단어: `w`(다음 단어), `b`(단어의 시작), `e`(단어의 끝)
- 줄: `0`(줄의 시작), `^`(공백이 아닌 첫 문자), `$`(줄의 끝)
- 화면: `H`(화면 위쪽), `M`(화면 가운데), `L`(화면 아래쪽)
- 파일: `gg`(파일의 시작), `G`(파일의 끝)
- 줄 번호: `:{number}<CR>` 또는 `{number}G`({number}번째 줄)
- 기타: `%`(대응하는 항목)
- 찾기: `f{character}`, `t{character}`, `F{character}`, `T{character}`
    - 현재 줄에서 앞/뒤 방향으로 {character}를 찾거나 그 직전까지 이동합니다.
- N번 반복: `{number}{movement}`. 예를 들어 `10j`는 10줄 아래로 이동합니다.
- 검색: `/{regex}`, `n` / `N`으로 일치하는 항목 사이를 이동합니다.

### 선택

Visual 모드:

- Visual
- Visual Line
- Visual Block

이동 키를 사용해 선택할 수 있습니다.

### 텍스트 조작

마우스로 하던 모든 일을 이제 키보드로 (그리고 조합할 수 있는 강력한 명령어로) 합니다.

- `i` Insert 모드로 들어갑니다.
    - 하지만 텍스트를 조작/삭제할 때는 백스페이스 이상의 기능을 사용하고 싶을 것입니다.
- `o` / `O` 아래 / 위에 줄을 삽입합니다.
- `d{motion}` {motion}을 삭제합니다.
    - 예를 들어 `dw`는 단어를 삭제하고, `d$`는 줄 끝까지 삭제하며, `d0`은 줄 시작까지 삭제합니다.
- `c{motion}` {motion}을 변경합니다.
    - 예를 들어 `cw`는 단어를 변경합니다.
    - `d{motion}` 다음에 `i`를 누르는 것과 비슷합니다.
- `x` 문자를 삭제합니다(`dl`과 같습니다).
- `s` 문자를 치환합니다(`xi`와 같습니다).
- Visual 모드 + 조작
    - 텍스트를 선택한 뒤 `d`로 삭제하거나 `c`로 변경합니다.
- `u`로 실행 취소, `<C-r>`로 다시 실행합니다.
- 배울 것은 훨씬 더 많습니다. 예를 들어 `~`는 문자의 대소문자를 뒤집습니다.

### 참고 자료

- Vim을 가르쳐 주는 명령줄 프로그램 `vimtutor`
- Vim을 배울 수 있는 게임 [Vim Adventures](https://vim-adventures.com/)

## Vim 커스터마이징

Vim은 `~/.vimrc`에 있는 일반 텍스트 설정 파일(Vimscript 명령어가 들어 있습니다)을 통해 커스터마이징합니다. 켜 두고 싶은 기본 설정이 많이 있을 것입니다.

GitHub에서 다른 사람의 닷파일을 살펴보며 아이디어를 얻되, 설정 전체를 복사해 붙여 넣는 일은 가급적 피하세요. 읽고 이해한 뒤 필요한 것만 가져오세요.

고려할 만한 커스터마이징 항목:

- 구문 강조: `syntax on`
- 색 구성표
- 줄 번호: `set nu` / `set rnu`
- 모든 곳에서 백스페이스 사용: `set backspace=indent,eol,start`

## 고급 Vim

편집기의 강력함을 보여 주는 몇 가지 예를 소개합니다. 이런 종류의 기능을 전부 가르칠 수는 없지만, 사용하면서 배우게 됩니다. 좋은 경험 법칙이 있습니다. 편집기를 사용하다가 "더 나은 방법이 있을 텐데"라는 생각이 든다면, 아마 있을 것입니다. 온라인에서 찾아보세요.

### 검색과 치환

`:s`(치환) 명령어([문서](https://vim.fandom.com/wiki/Search_and_replace)).

- `%s/foo/bar/g`
    - 파일 전체에서 foo를 bar로 치환합니다.
- `%s/\[.*\](\(.*\))/\1/g`
    - 이름이 있는 Markdown 링크를 일반 URL로 치환합니다.

### 여러 창

- `sp` / `vsp`로 창을 분할합니다.
- 같은 버퍼를 여러 뷰로 볼 수 있습니다.

### 마우스 지원

- `set mouse+=a`
    - 클릭, 스크롤, 선택을 할 수 있습니다.

### 매크로

- `q{character}`로 `{character}` 레지스터에 매크로 기록을 시작합니다.
- `q`로 기록을 중단합니다.
- `@{character}`로 매크로를 재생합니다.
- 오류가 나면 매크로 실행이 중단됩니다.
- `{number}@{character}`는 매크로를 {number}번 실행합니다.
- 매크로는 재귀적으로 실행할 수 있습니다.
    - 먼저 `q{character}q`로 매크로를 비웁니다.
    - `@{character}`로 매크로를 재귀적으로 호출하면서 매크로를 기록합니다(기록이 끝날 때까지는 아무 동작도 하지 않습니다).
- 예: xml을 json으로 변환하기([파일](/2019/files/example-data.xml))
    - "name" / "email" 키를 가진 객체의 배열
    - Python 프로그램을 사용할까요?
    - sed / 정규 표현식을 사용합니다.
        - `g/people/d`
        - `%s/<person>/{/g`
        - `%s/<name>\(.*\)<\/name>/"name": "\1",/g`
        - ...
    - Vim 명령어 / 매크로
        - `Gdd`, `ggdd`로 첫 줄과 마지막 줄을 삭제합니다.
        - 요소 하나의 형식을 지정하는 매크로(`e` 레지스터)
            - `<name>`이 있는 줄로 이동합니다.
            - `qe^r"f>s": "<ESC>f<C"<ESC>q`
        - 한 사람의 형식을 지정하는 매크로
            - `<person>`이 있는 줄로 이동합니다.
            - `qpS{<ESC>j@eA,<ESC>j@ejS},<ESC>q`
        - 한 사람의 형식을 지정하고 다음 사람으로 이동하는 매크로
            - `<person>`이 있는 줄로 이동합니다.
            - `qq@pjq`
        - 파일 끝까지 매크로를 실행합니다.
            - `999@q`
        - 마지막 `,`를 수동으로 지우고 구분자 `[`와 `]`를 추가합니다.

## Vim 확장

Vim을 확장하는 플러그인은 아주 많습니다.

먼저 [vim-plug](https://github.com/junegunn/vim-plug), [Vundle](https://github.com/VundleVim/Vundle.vim), [pathogen.vim](https://github.com/tpope/vim-pathogen) 같은 플러그인 관리자를 설정하세요.

고려할 만한 플러그인:

- [ctrlp.vim](https://github.com/kien/ctrlp.vim): 퍼지 파일 검색기
- [vim-fugitive](https://github.com/tpope/vim-fugitive): Git 연동
- [vim-surround](https://github.com/tpope/vim-surround): "둘러싼 문자" 조작
- [gundo.vim](https://github.com/sjl/gundo.vim): 실행 취소 트리 탐색
- [nerdtree](https://github.com/scrooloose/nerdtree): 파일 탐색기
- [syntastic](https://github.com/vim-syntastic/syntastic): 구문 검사
- [vim-easymotion](https://github.com/easymotion/vim-easymotion): 마법 같은 이동
- [vim-over](https://github.com/osyo-manga/vim-over): 치환 미리 보기

플러그인 목록:

- [Vim Awesome](https://vimawesome.com/)

## 다른 프로그램에서의 Vim 모드

많이 쓰이는 여러 편집기(예: vim, emacs)의 동작을 모방하는 기능을 다른 많은 도구가 지원합니다.

- 셸
    - bash: `set -o vi`
    - zsh: `bindkey -v`
    - `export EDITOR=vim`(`git` 같은 프로그램이 사용하는 환경 변수)
- `~/.inputrc`
    - `set editing-mode vi`

웹 [브라우저](https://vim.fandom.com/wiki/Vim_key_bindings_for_web_browsers)에도 Vim 키 바인딩 확장 프로그램이 있습니다. Google Chrome용 [Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?hl=en)과 Firefox용 [Tridactyl](https://github.com/tridactyl/tridactyl)이 널리 쓰입니다.


## 참고 자료

- [Vim 팁 위키](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
- [Vim Advent Calendar](https://vimways.org/2018/): 다양한 Vim 팁
- [Neovim](https://neovim.io/)은 더 활발하게 개발되는 현대적인 Vim 재구현입니다.
- [Vim Golf](https://www.vimgolf.com/): 다양한 Vim 도전 과제

{% comment %}
# Resources

TODO resources for other editors?
{% endcomment %}

# 연습 문제

1. 몇 가지 편집기를 시험해 보세요. 최소한 명령줄 편집기 하나(예: Vim)와 GUI 편집기 하나(예: Atom)를 사용해 보세요. `vimtutor` 같은 튜토리얼(또는 다른 편집기의 동등한 튜토리얼)로 배우세요. 새 편집기를 제대로 경험하려면 며칠 동안 일상적인 작업을 하면서 그 편집기만 사용하기로 결심하세요.

1. 편집기를 커스터마이징하세요. 온라인에서 팁과 요령을 찾아보고, 다른 사람의 설정도 살펴보세요(대개 설명이 잘 되어 있습니다).

1. 편집기용 플러그인을 시험해 보세요.

1. 적어도 2주 정도는 강력한 편집기를 사용하기로 결심하세요. 그때쯤이면 이점을 느끼기 시작할 것입니다. 어느 시점에는 생각하는 속도만큼 빠르게 편집기를 다룰 수 있을 것입니다.

1. 린터(예: python의 pyflakes)를 설치하고 편집기에 연결한 뒤 제대로 동작하는지 시험하세요.
