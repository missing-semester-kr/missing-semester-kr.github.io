---
layout: lecture
title: "Command-line Environment"
description: >
  작업 제어, 터미널 멀티플렉서, dotfiles와 SSH를 통한 원격 컴퓨터 사용법을 배웁니다.
thumbnail: /static/assets/thumbnails/2020/lec5.png
date: 2020-01-21
ready: true
video:
  aspect: 56.25
  id: e8BO_dYxk5c
translation_status: complete
---

이 강의에서는 shell을 사용할 때 워크플로우를 개선할 수 있는 몇 가지 방법을 살펴볼 것입니다. 
그동안 우리는 shell을 사용해왔지만, 주로 다른 명령을 실행했습니다. 
이제 여러 프로세스를 추적하면서 동시에 실행하는 방법, 특정 프로세스를 중지하거나 일시 중지하는 방법, 프로세스가 백그라운드에서 실행되도록 하는 방법 등을 살펴보겠습니다.

우리는 또한 별칭을 정의하고 dotfile을 사용하여 설정함으로써 여러분의 shell과 다른 도구들을 향상시키는 다른 방법에 대해서도 배울 것입니다. 
이 두 가지 모두 긴 명령을 입력할 필요 없이 모든 시스템에서 동일한 구성을 사용하여 시간을 절약할 수 있습니다. 
우리는 SSH를 이용하여 어떻게 원격 서버에서 작업을 하는지에 대해 알아볼 것입니다.


# 작업 제어

예를 들어 명령이 완료되는 데 시간이 너무 오래 걸리는 경우(예컨데 `find`를 사용하여 매우 큰 디렉토리 구조를 검색)와 
같이 작업이 실행 중일 때 중단해야 하는 경우가 있습니다.
대부분 경우 `Ctrl-C`를 하게 되면 명령이 중단됩니다.
하지만 이것이 실제로 어떻게 작동하고, 왜 때때로 프로세스를 멈추는데 실패할까요?

## 프로세스 제거하기

당신의 shell은 _신호_ 라고 불리는 UNIX 통신 메커니즘을 사용하여 정보를 프로세스에 전달하고 있습니다. 
프로세스가 실행을 중지하는 신호를 수신하면 해당 신호를 처리하고, 신호가 전달한 정보를 기반으로 실행 흐름을 잠재적으로 변화시킵니다. 
따라서 신호는 _소프트웨어 중단_ 입니다.

`Ctrl-C`를 입력하면 shell이 `SIGINT` 신호를 프로세스에 전달합니다.

다음은 `SIGINT`를 캡처하여 무시하고 더 이상 멈추지 않는 파이썬 프로그램의 작은 예제 입니다. 
이 프로그램을 제거하려면 `Ctrl-\`를 입력하여 `SIGQUIT` 신호를 보냅니다.

```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

이 프로그램에 두 번의 `SIGINT`를 보내고, 그 다음에 `SIGQUIT`를 보내면 다음과 같은 일이 일어납니다. 
`^`는 termial에 입력할 때 `Ctrl`이 표시되는 방식입니다.

```bash
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

`SIGINT`와 `SIGQUIT`는 보통 터미널 관련 요청과 관련되어 있는 반면, 보다 멋있게 종료하는 과정을 요청하는 일반적인 신호는 `SIGTERM` 신호입니다.
이 신호를 보내기 위해 [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html) 명령을 `kill -TERM <PID>`라는 구문과 함께 사용할 수 있습니다.

## 프로세스의 일시 중지와 백그라운드

신호는 프로세스를 제거하는 것 이상의 다른 일들을 할 수 있습니다. 
예를 들어 `SIGSTOP`은 프로세스를 일시 중지시킵니다. 
터미널에서 `Ctrl-Z`를 입력하면 shell이 Terminal Stop(터미널 버전 `SIGSTOP`)을 줄인 `SIGTSTP` 신호를 보냅니다.

그 다음 각각 [`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) 또는 [`bg`](https://man7.org/linux/man-pages/man1/bg.1p.html) 를 사용하여 포어그라운드 또는 백그라운드에서 일시 중지된 작업을 계속할 수 있습니다.

[`jobs`](https://www.man7.org/linux/man-pages/man1/jobs.1p.html) 명령어에는 현재 터미널 세션과 관련된 미완료 작업 목록을 보여줍니다. 
목록의 pid([`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 사용)를 이용하여 해당 작업을 참조할 수 있습니다.
보다 직관적으로 작업 번호(`jobs`로 보여지는) 다음에 해당하는 % 기호를 사용하는 프로세스도 참조할 수 있습니다. 
마지막 백그라운드 작업을 참조하려면 특수 매개 변수 `$!`를 사용하면 됩니다.

한 가지 더 알아야 할 것은 명령어의 `&` 접미사가 명령을 백그라운드에서 실행해 귀찮을 수 있는 쉘의 `STDOUT`를 여전히 사용하지만(이 경우 쉘 재조정을 사용) 프롬프트를 다시 제공한다는 점입니다.

이미 실행 중인 프로그램을 백그라운드 실행으로 변경하려면 `Ctrl-Z`에 이어 `bg`를 실행하면 됩니다.
백그라운드 프로세스는 여전히 터미널의 하위 프로세스로서 터미널을 닫으면 죽는다는 점에 유의하세요 (또 다른 신호인 `SIGHUP`를 전송). 
그런 일이 일어나는 것을 방지하기 위해 [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html) (`SIGHUP`를 무시하는 wrapper) 
으로 프로그램을 실행하거나 프로세스가 이미 시작되었다면 `disown`을 사용합니다.
대안으로 다음 섹션에서 볼 수 있듯이 터미널 멀티플렉서를 사용할 수 있습니다.

아래는 이러한 개념들 중 몇 가지를 소개하기 위한 예제입니다.

```bash
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ bg %1
[1]  - 18653 continued  sleep 1000

$ jobs
[1]  - running    sleep 1000
[2]  + running    nohup sleep 2000

$ kill -STOP %1
[1]  + 18653 suspended (signal)  sleep 1000

$ jobs
[1]  + suspended (signal)  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ jobs
[2]  + running    nohup sleep 2000

$ kill -SIGHUP %2

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000

$ jobs

```

특수신호는 프로세스를 캡처할 수 없고 항상 즉시 종료되기 때문에 `SIGKILL`입니다. 
하지만 고아가 된 하위 프로세스를 방치하는 등의 부작용을 낳을 수 있습니다.

이것과 다른 신호에 대해 자세히 알아보려면 [`여기`](https://en.wikipedia.org/wiki/Signal_(IPC)) 또는 [`man signal`](https://www.man7.org/linux/man-pages/man7/signal.7.html)을 입력 또는 `kill -l`을 입력하세요.


# 터미널 멀티플렉서

커맨드라인 인터페이스를 사용할 때 한 번에 두 개 이상의 작업을 실행하는 경우가 많을 겁니다.
예를 들어, 에디터와 프로그램을 나란히 실행하길 원한다고 합시다.
새로운 터미널 창을 열면 가능하겠지만 터미널 멀티플렉서를 사용하는 것이 더 다재다능한 해결방법 입니다.

[`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html)와 같은 터미널 멀티플렉서는 창과 탭을 사용하여 
터미널 창을 동시에 실행가능하게 분할할 수 있어 여러 shell 세션과 상호 작용이 가능합니다.
또한 터미널 멀티플렉서는 현재의 터미널 세션을 빠져나왔다가 나중에 다시 연결할 수 있습니다.
이는 `nohup`과 유사한 트릭을 사용할 필요가 없기 때문에 원격 컴퓨터로 작업할 때 워크플로우를 훨씬 더 좋게 만들 수 있습니다.

요즘 가장 인기 있는 터미널 멀티플렉서는 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) 입니다. 
`tmux`는 환경 관리 설정이 뛰어나며 관련된 키 바인딩을 사용하여 여러 탭과 창을 만들고 빠르게 탐색할 수 있습니다.

`tmux`는 키 바인딩을 알고 싶어하는데 (1) `Ctrl+b` 키보드를 누르고, (2) `Ctrl+b` 키보드를 놓은 후, 
(3) `x`를 누르는 `<C-b> x` 형식을 갖고 있습니다. 
`tmux`는 다음과 같은 개체 계층을 가지고 있습니다:
- **Sessions** - 세션은 하나 이상의 윈도우가 있는 독립된 작업 공간
    + `tmux` 새로운 세션 시작
    + `tmux new -s NAME` 세션 이름과 함께 새로운 세션 시작
    + `tmux ls` 현재 세션 목록 나열
    + `tmux` 실행 중 `<C-b> d`를 입력  현재 세션에서 빠져나오기
    + `tmux a` 마지막 세션으로 들어가기. `-t` 옵션을 써서 특정 세션으로 접속 가능

- **Windows** - 에디터 또는 브라우저의 탭과 동일하게 동일한 세션에서 시각적으로 분리된 부분
    + `<C-b> c` 새로운 윈도우를 생성. `<C-d>`를 입력하면 윈도우 닫기
    + `<C-b> N` _N_ 번째 윈도우로 이동. 윈도우마다 번호가 있음을 유의
    + `<C-b> p` 이전 윈도우로 이동
    + `<C-b> n` 다음 윈도우로 이동
    + `<C-b> ,` 현재 윈도우 이름 바꾸기
    + `<C-b> w` 현재 윈도우 목록 나열

- **Panes** - vim 분할처럼 창은 동일한 시각 디스플레이에 여러 개의 shell을 가질 수 있음
    + `<C-b> "` 현재 창을 가로로 나누기
    + `<C-b> %` 현재 창을 세로로 나누기
    + `<C-b> <direction>` _direction_ 방향의 창으로 이동. 여기서 direction은 화살표 키를 의미
    + `<C-b> z` 현재 창의 확대/축소 전환
    + `<C-b> [` scrollback 시작. `<space>`를 누르면 선택을 시작하고 `<enter>`를 누르면 선택 내용이 복사
    + `<C-b> <space>` 창 배열을 순환

추가적인 자료는 [`여기`](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)에 `tmux`에 대한 간단한 튜토리얼이 있고, 
[여기](https://linuxcommand.org/lc3_adv_termmux.php)에는 원래의 `screen` 명령을 다루는 보다 자세한 설명이 있습니다. 
대부분의 UNIX 시스템에 설치되므로 [`screen`](https://www.man7.org/linux/man-pages/man1/screen.1.html)을 숙지하는 것이 좋을 것입니다.

# 별칭

많고 장황한 옵션을 포함하는 긴 명령을 입력하는 것은 지루한 일입니다.
그런 이유로 대부분의 shell은 _별칭 지정_ 을 지원합니다.
별칭은 shell이 자동으로 대체할 수 있는 다른 명령을 위한 짧은 형식입니다.
예를 들어 bash의 별칭은 다음과 같은 구조를 가지고 있습니다.

```bash
alias alias_name="command_to_alias arg1 arg2"
```

[`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html)는 하나의 argument를 취하는 shell 명령이기 때문에 등호 `=` 주위에 빈 공간이 없다는 점을 유의하세요.

별칭은 많은 편리한 기능을 가지고 있습니다:

```bash
# 일반적인 플래그의 짧은 명령어 만들기
alias ll="ls -lh"

# 평소에 자주 사용하는 명령어 저장하기
alias gs="git status"
alias gc="git commit"
alias v="vim"

# 오타 방지
alias sl=ls

# 더 좋은 기본값을 위해 기존 명령어 덮어쓰기
alias mv="mv -i"           # -i 지정 위치에 동일 파일이 있을 경우 덮어 쓸때 물어봄
alias mkdir="mkdir -p"     # -p 상위 디렉토리가 필요하다면 만들기
alias df="df -h"           # -h 사람이 읽을 수 있는 형식으로 출력

# 별칭을 시용하여 별칭을 구성
alias la="ls -A"
alias lla="la -l"

# 앞에 \가 붙은 상태로 별칭 실행하면 빌칭 실행 무시
\ls
# 또는 unalieas를 사용하여 별칭 사용불가
unalias la

# 별칭 정의를 얻기으려면 별칭을 호출
alias ll
# ll='ls -lh' 이 출력됨
```

별칭은 기본적으로 shell 세션을 유지하지 않는다는 점을 알아두세요.
별칭을 영구적으로 만들려면 다음 절에서 소개할 `.bashrc`나 `.zshrc`와 같은 shell 시작 파일에 포함시켜야 합니다.


# 도트 파일(Dotfiles)

많은 프로그램은 _dotfiles_ 로 알려진 일반 텍스트 파일을 사용하여 구성됩니다.
(파일 이름이 `.`, `~/.vimrc` 예들로 시작하므로 기본적으로 디렉토리 목록에 숨겨져 있기 때문입니다.)

쉘은 이러한 파일로 구성된 프로그램의 한 예입니다. 시작시 쉘은 구성을 로드하기 위해 많은 파일을 읽습니다. 쉘에 따라 로그인을 시작하든 쌍방향의 작업을 시작하든 둘 중 어떤 것을 시작하든 전체 프로세스는 상당히 복잡할 수 있습니다. 주제에 대한 자료는 [`여기`](https://web.archive.org/web/20260329133158/https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html)에 있습니다. 

`bash` 의 경우, `.bashrc` 혹은 `.bash_profile`의 편집은 대부분의 시스템에서 작동합니다. 여기에 방금 설명한 별칭이나 'PATH' 환경 변수 수정과 같이 시작 시 실행할 명령을 포함할 수 있습니다. 실제로 많은 프로그램에서는 binaries를 찾을 수 있도록 쉘 구성 파일에 `export PATH="$PATH:/path/to/program/bin"`과 같은 행을 포함하도록 요청할 것입니다.

도트 파일을 통해 구성할 수 있는 도구의 다른 예는 다음과 같습니다 :

- `bash` - `~/.bashrc`, `~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` and the `~/.vim` folder
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

dotfiles를 어떻게 체계적으로 구성할 수 있을까요? 버전 관리 하에 자체 폴더에 있어야하며, 스크립트를 사용하여 **symlinked**를 배치해야합니다. 이것은 다음과 같은 이점이 있습니다.

- **쉬운 설치**: 새 컴퓨터에 로그인한 경우, 여러분의 customizations(사용자 정의)을 적용하는데 몇 분 밖에 걸리지 않습니다.
- **이식성**: 여러분의 도구는 어디서나 동일한 방식으로 작동할 것입니다.
- **동기화**: 어디서든 dotfile을 업데이트하고 모두 동기화 상태로 유지할 수 있습니다.
- **변경 내용 추적**: 여러분은 아마 프로그래머로서 일을 할 동안 dotfiles을 유지 관리할 것이고, 버전 기록은 오래 지속되는 프로젝트에 유용할 것입니다.

dotfiles에 무엇을 넣어야할까요?
온라인상의 자료 또는 [`매뉴얼 페이지`](https://en.wikipedia.org/wiki/Man_page)를 읽으며 여러 가지 도구에 대한 설정울 배울 수 있습니다. 또 다른 좋은 방법은 인터넷에서 특정 프로그램에 대한 블로그 게시물을 검색하는 것입니다. 여기에서 블로거는 그들이 선호하는 customizations(사용자 정의)에 대해 이야기할 것입니다. customizations(사용자 정의)에 대해 배우는 또 다른 방법은 다른 사람들의 dotfile을 살펴 보는 것 입니다. GitHub에서 수많은 [`dotfiles 저장소`](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories)를 찾을 수 있습니다. 가장 인기있는 저장소는 [`여기`](https://github.com/mathiasbynens/dotfiles)입니다.(하지만 configurations을 무턱대고 복사하는 것은 올바른 방법이 아님을 알려드립니다.) [`이것`](https://dotfiles.github.io/)은 이 주제에 대한 또 다른 좋은 자료입니다.

모든 강사들은 GitHub에서 공적으로 액세스할 수 있는 dotfiles을 가지고 있습니다 : [Anish](https://github.com/anishathalye/dotfiles), [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/jjgo/dotfiles).


## 이식성(Portability)

dotfile의 일반적인 문제점은 예를 들어 운영 체제나 쉘이 서로 다른 경우와 같이 여러 장치로 작업할 때 configurations이 작동하지 않을 수 있다는 것입니다. 때로는 특정 시스템에만 일부 configuration을 적용하기를 원할 수도 있습니다.

이것을 쉽게 만드는 몇 가지 요령들이 있습니다. 구성 파일이 지원하는 경우, if문의 비교연산을 사용하여 특정한 사용자 설정을 적용할 수 있습니다. 예를 들어 쉘은 다음과 같은 것들을 활용할 수 있습니다.

```bash
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# 쉘별 기능을 사용하기 전에 확인하기
if [[ "$SHELL" == "zsh" ]]; then {do_something}; fi

# 특정 장치별로 만들기 
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

구성 파일이 이를 지원하는 경우, includes를 활용할 수 있습니다. 예를 들어 `~/.gitconfig`는 다음과 같은 설정을 가질 수 있습니다.  

```
[include]
    path = ~/.gitconfig_local
```

그리고 각 장치에서 `~/.gitconfig_local`은 장치별 설정들을 포함할 수 있습니다. 분리된 저장소(리포지토리)에서 시스템별 설정을 추적할 수도 있습니다.

이것은 또한 다른 프로그램들이 일부 구성을 공유하기를 원하는 경우에도 유용합니다. 예를 들어, 여러분이 `bash`와 `zsh` 두 가지 모두 동일한 별칭으로 공유하고 싶은 경우 `.aliases` 아래에 표기하고 다음 블록을 포함시킬 수 있습니다.

```bash
# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

<span id="remote-machines"></span>

# 원격 장치(Remote Machines)

프로그래머들이 일상 업무에 원격 서버를 이용하는 것은 점점 더 보편화되었습니다. 백엔드 소프트웨어를 배치하기 위해 원격 서버를 사용해야 하거나 더 높은 계산 능력을 가진 서버가 필요한 경우, 여러분은 결국 SSH(Secure Shell)를 사용하게 될 것입니다. 대부분의 툴과 마찬가지로 SSH는 구성성이 뛰어나므로 이에 대해 학습할 가치가 있습니다.

서버에서 `ssh`를 사용하려면 다음과 같은 명령을 실행합니다.

```bash
ssh foo@bar.mit.edu
```

여러분은 서버 `bar.mit.edu`에서 사용자`foo`로 ssh를 사용하게 됩니다.
서버는 URL ( 예) `bar.mit.edu` ) 또는 IP ( 예) `foobar @ 192.168.1.42` )로 지정할 수 있습니다. 나중에 ssh 구성 파일을 수정하면 `ssh bar`와 같은 것을 사용하여 액세스 할 수 있음을 알 수 있습니다.


## 명령실행(Executing commands)

놓치기 쉬운 `ssh`의 특징은 명령을 직접 실행할 수 있다는 점입니다.
`ssh foobar @ server ls`는 foobar의 홈 폴더에서`ls`를 실행합니다.
파이프와 함께 작동하므로 `ssh foobar @ server ls | grep PATTERN`은 `ls`의 근처의 원격 출력을 grep하고, `ls | ssh foobar@server grep PATTERN`은 먼 로컬 출력을 grep합니다.


## SSH키(SSH Keys)

키 기반 인증은 공개키 암호화를 이용하여 클라이언트가 키를 노출하지 않고 비밀 개인 키(private key)를 소유하고 있음을 서버에 증명합니다. 이렇게하면 매번 암호를 다시 입력 할 필요가 없습니다. 그럼에도 불구하고 개인 키 (종종`~ / .ssh / id_rsa` 그리고 최근에는`~ / .ssh / id_ed25519`)가 사실상 암호이므로 암호처럼 취급하면 됩니다.


### 키 생성(Key generation)

쌍을 생성하려면 [`ssh-keygen`](https://www.man7.org/linux/man-pages/man1/ssh-keygen.1.html)을 실행합니다.
```bash
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```
인증된 서버에 액세스할 수 있는 여러분의 개인 키(private key)를 보유한 사람을 피하려면 암호(passphrase)선택해야합니다. [`ssh-agent`](https://www.man7.org/linux/man-pages/man1/ssh-agent.1.html) 또는 [`gpg-agent`](https://linux.die.net/man/1/gpg-agent)를 사용하면 매번 암호를 입력할 필요없습니다.

SSH 키를 사용하여 GitHub로 푸시를 구성한 적이 있다면, [here](https://help.github.com/articles/connecting-to-github-with-ssh/))에 설명된 단계를 수행한 후 유효한 키 쌍을 이미 가지고 있을 것입니다. 암호가 있는지 확인하고 유효성을 검사하려면 `ssh-keygen -y -f /path/to/key`를 실행할 수 있습니다.


### 키 기반 인증(Key based authentication)

`ssh`는 `ssh / authorized_keys`를 조사하여 허용해야하는 클라이언트를 결정합니다. 공용 키를 복사하려면 다음을 사용할 수 있습니다.

```bash
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

`ssh-copy-id`를 사용하여 더 간단하게 사용할 수 있습니다.

```bash
ssh-copy-id -i .ssh/id_ed25519 foobar@remote
```

## SSH를 통해 파일 복사(Copying files over SSH)

ssh를 통해 파일을 복사하는 방법에는 여러 가지가 있습니다.

- `ssh+tee`, 가장 간단한 방법은 `cat localfile | ssh remote_server tee serverfile`을 실행하여 `ssh`명령 실행(command execution) 그리고 STDIN 입력(input)을 사용하는 것입니다. [`tee`](https://www.man7.org/linux/man-pages/man1/tee.1.html)에서 STDIN의 출력을 파일로 작성합니다.
- [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) 많은 양의 파일/디렉토리를 복사 할 때, 보안 복사 `scp` 명령은 경로를 쉽게 반복 할 수 있으므로 더 편리합니다. 구문은 `scp path/to/local_file remote_host:path/to/remote_file`입니다. 
- [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html)는 로컬과 원격에서 동일한 파일을 탐지해 다시 복사하지 못하게 함으로써 `scp`를 향상시킵니다. 또한 심볼릭 링크(symlinks), 권한(permissions)에 대해 보다 세밀한 제어를 제공하며 이전에 중단 된 복사본에서 다시 시작할 수있는`--partial` 플래그(flag)와 같은 추가 기능이 있습니다. `rsync`는 `scp`와 유사한 구문을 가지고 있습니다. 


## 포트 포워딩(Port Forwarding)

많은 시나리오에서 여러분은 장치의 특정 포트를 수신하는 소프트웨어를 만나게 될 것입니다. 로컬 시스템에서 발생하게 된다면 `localhost : PORT` 또는 `127.0.0.1 : PORT`를 입력 할 수 있습니다. 그러나, 네트워크/인터넷을 통해 직접 사용할 수 있는 포트가 없는 원격 서버가 있다면 어떻게 해야할까요?  

이것을 _포트 포워딩(port forwarding)_ 이라고 하며 로컬 포트 포워딩(Local Port Forwarding)과 원격 포트 포워딩(Remote Port Forwarding)의 두 가지 형태로 제공됩니다. (자세한 내용은 사진을 참조해주십시오. 저작권은 [StackOverflow 게시물](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot)에 있습니다.) 

**로컬 포트 포워딩(Local Port Forwarding)**
![Local Port Forwarding](/static/media/images/local-port-forwarding.png "Local Port Forwarding")

**원격 포트 포워딩(Remote Port Forwarding)**
![Remote Port Forwarding](/static/media/images/remote-port-forwarding.png "Remote Port Forwarding")

가장 일반적인 시나리오는 로컬 포트 포워딩(local port forwarding)으로, 원격 장치의 서비스가 포트에서 수신 대기하고 로컬 장치의 포트를 원격 포트로 연결하는 경우입니다.
예를 들어 '8888' 포트를 수신하는 원격 서버에서 주피터 노트북을 실행하면, 이를 로컬 포트 `9999`로 전달하기 위해 `ssh -L 9999:localhost:8888 foobar@remote_server`를 수행한 다음 로컬 장치에서 `localhost:9999`로 이동합니다.


## SSH 구성(SSH Configuration)

우리는 전달할 수 있는 많은 논쟁을 다루었습니다. 가장 매력적은 대안은 다음과 같이 쉘 별칭을 만드는 것입니다.
```bash
alias my_server="ssh -i ~/.ssh/id_ed25519 --port 2222 -L 9999:localhost:8888 foobar@remote_server"
```

그러나 `~/.ssh/config`를 사용하면 더 좋은 대안을 만들 수 있습니다. 

```bash
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# Configs can also take wildcards
Host *.mit.edu
    User foobaz
```

별칭(alias)보다 `~/.ssh/config` 파일을 사용하는 것의 또 다른 장점은 `scp`, `rsync`, `mosh`, `&c` 등의 프로그램도 읽을 수 있다는 것과 설정을 해당 플래그로 변환할 수 있다는 것입니다.

`~/.ssh/config` 파일은 도트 파일(dotfile)로 간주될 수 있으며, 일반적으로 이것이 나머지 도트 파일과 같이 있어도 괜찮다는 점을 기억하세요. 그러나, 만약 여러분이 이것을 공개한다면, 여러분은 인터넷상의 낯선 사람들에게 잠재적으로 이런 정보를 제공할 수도 있습니다: 여러분의 서버 주소, 사용자, 개방된 포트, 등등. 이렇게 하면 일부 유형의 공격이 쉬워질 수 있으므로 SSH 구성을 공유하는 데 유의하십시오.

서버측 구성은 보통 `/etc/ssh/sshd_config`에 지정됩니다. 여러분은 여기서 암호 인증 사용 안 함, ssh 포트 변경, X11 포워딩 사용 등의 변경 작업을 수행할 수 있습니다. 구성 설정을 각 사용자별로 지정할 수 있습니다.

## 이외에 알아두면 좋은 사항

원격 서버에 연결할 때 나타나는 흔한 문제들 중 하나는 컴퓨터를 종료/절전하거나, 네트워크를 변경할 때 연결이 끊어지는 것입니다. 게다가 만약 여러분이 연결에서 심한 지연(lag)을 경험하게 된다면 ssh를 사용하기가 꺼려질 수도 있습니다. 모바일 쉘 [Mosh](https://mosh.org/)는 로밍 연결(roaming connections), 간헐적 연결(intermittent connectivity), 지능형 로컬 에코(intelligent local echo)가 가능하므로 ssh보다 더 낫습니다.

때로는 원격 폴더를 마운트 하는것이 편리합니다. [sshfs](https://github.com/libfuse/sshfs)는 로컬로 원격 서버에 폴더를 마운트 하여 로컬 에디터를 사용할 수 있게 합니다.


# 셸 & 프레임워크

셸 툴과 스크립팅에서 우리는 `bash` 셸을 다루었는데요. 그것은 `bash`셸이 단연코 가장 보편적이며, 대부분의 시스템이 그것을 기본 옵션으로 가지고 있기 때문입니다. 하지만 이것이 유일한 선택지는 아닙니다. 

예를 들어, `bash`의 수퍼셋(superset)인 `zsh` 셸은 다음과 같이 별도의 설치나 구성이 필요없이 사용 가능한 많은 편리한 기능을 제공한다.

- 더 깔끔한 글로빙(globbing), `**`
- 인라인 글로빙/와일드카드 확장
- 스펠링 수정
- 더 나은 탭 완성/선택
- 경로 확장 (예를 들어, `cd /u/lo/b` 은 `/usr/local/bin`로 확장됩니다.)

**프레임워크** 또한 여러분의 셸을 개션해 줍니다. 유명한 광범위 프레임워크 몇개를 소개하자면 [prezto](https://github.com/sorin-ionescu/prezto) 아니면 [oh-my-zsh](https://ohmyz.sh/)가 있고, 특정 기능에 초점을 맞춘 규모가 작은 것들에는[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) 아니면 [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)가 있습니다. [fish](https://fishshell.com/)같은 셸들은 많은 사용자 친화적 기능들을 기본 탑재하고 있습니다. 이러한 기능들 중에는 아래와 같은 것들이 포함되어 있습니다:

- 우측 프롬프트
- 명령어 하이라이팅
- 히스토리 서브스트링 검색
- 맨 페이지(manual page) 기반의 플래그 완성 기능
- 더 나은 자동 완성
- 프롬프트 테마들

이러한 프레임워크를 사용할 때 유의해야 할 점이 있습니다. 그것은 바로 실행하는 코드가 적절하게 최적화되지 않았거나 코드의 양이 너무 많을 때 프레임워크 사용이 셸 속도를 느리게 할 수 있다는 것입니다. 여러분은 이것에 대해 프로파일을 작성하고 자주 사용하지 않는 기능이나 속도 향상보다 더 중요하지 않은 기능들을 막아둘 수 있습니다. 

# 터미널 에뮬레이터

여러분의 터미널을 커스터마이징 하는것과 함께, 당신에게 딱 맞는 **터미널 에뮬레이터**와 그 설정을 찾아보는데 시간을 들이는 것도 가치가 있는 일입니다. 이 세상에는 정말 많은 터미널 에뮬레이터들이 있습니다. (여기 [비교]가 있습니다. (https://anarc.at/blog/2018-04-12-terminal-emulators-1/)).

여러분은 아마도 여러분의 터미널에서 수백 수천 시간을 보낼 것이기 때문에, 그 설정을 자세히 살펴보는게 확실히 도움이 됩니다. 여러분이 여러분의 터미널에서 수정하고 싶어할만한 것들은 다음과 같습니다. 

- 글꼴 선택
- 색채 배합 (color scheme)
- 키보드 단축기
- 탭/창 보조
- 스크롤백 설정
- 성능 ( [Alacritty](https://github.com/jwilm/alacritty) 혹은 [kitty](https://sw.kovidgoyal.net/kitty/)와 같은 최신 터미널들은  GPU 가속화를 지원합니다).

# 실습

## 작업 제어

1. 지금까지 공부한 바에 따르면, 우리는 명령어 `ps aux | grep` 로 PID를 조회하여 작업을 종료할 수 있습니다. 하지만 이것보다 더 나은 방법이 있습니다. 터미널에서 `sleep 10000`를  실행하고, `Ctrl-Z`로 그것을 백그라운드 실행하세요. 그리고 `bg`를 통해 이 작업을 계속합니다. 이제 [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html)을 사용해 pid를 찾은 다움 [`pkill`](https://man7.org/linux/man-pages/man1/pgrep.1.html)을 실행하면 pid를 직접 타이핑하지 않고 작업을 종료시킬 수 있습니다. (힌트: `-af` 플래그를 사용하세요).

2. 여러분이 다른 프로세스가 종료되기를 기다려 새 프로세스를 시작하려고 한다고 합시다, 이것을 어떻게 할 수 있을까요? 이 실습에서 프로세스 제한은 항상 `sleep 60 &`로 합니다.
이것을 가능케 하는 또 다른 방법은 [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html) 명령어를 사용하는 것입니다. sleep 명령어를 실행한 뒤 백그라운드 작업이 끝날 때까지 기다려 `ls`가 실행되도록 하세요.

    하지만 만일 각기 다른 bash 창에서 작업을 실행하는 중이라면, 이 시도는 실패할 것입니다. `wait`은 자식 프로세스에만 적용되기 때문입니다. 지금껏 우리가 다루지 않은 기능 중 하나는 명령어 `kill`이 성공했을때 exit status가 0(zero)이 되며, 그렇지 않을때는 0이 아니라는 것(nonzero)입니다. 명령어 `kill -0`은 시그널을 보내지 않습니다만 프로세스가 존재하지 않을때 0이 아닌 exit status를 반환할 것입니다. 
    `pidwait`이라는 이름의, pid를 가져오고 해당 프로세스가 끝날 때까지 기다리는 bash 명령어를 작성해 보세요. 불필요한 CPU 낭비 방지를 위해 `sleep` 명령어를 사용해야 합니다. 

## 터미널 멀티플렉서

1. 이 `tmux` [튜토리얼](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)을 따라해 보고 [이 단계](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/)에 따라 기본적인 커스터마이제이션 방법을 배워보세요. 

## 별칭

1. 오타 낼 경우를 대비해서 `cd`로 연결되는 별칭 `dc`를 생성합니다.

2.  `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10`를 실행해서 가장 많이 사용하는 명령어 10개를 확인합니다. 그리고 이들에게 짧은 별칭을 부여하는 것을 고려해 보세요. 주의: 이 방법은 Bash에서 유효합니다; 만일 ZSH를 쓰고 있다면, 그냥 `history` 말고 `history 1`를 사용하세요. 


## 도트 파일

이제 도트 파일을 사용할때 작업 속도를 높여봅시다. 
1. 도트 파일들을 위한 폴더를 생성하고, 버전 컨트롤을 구성합니다.
2. 최소한 프로그램 하나에 구성을 추가합니다, 예. 여러분의 셸에 약간의 커스터마이제이션 하기(시작하는 차원에서, `$PS1`을 설정해 셸 프롬프트를 커스터마이제이션 하는것과 같이 간단한 것도 좋습니다).
3. 새로운 머신에서 여러분의 도트파일을 빨리 (그리고 수작업 없이)설치하는 메서드를 생성하세요. 이것은 각 파일에 `ln -s`를 요청하는 셸 스크립트 처럼 간단한 것이어도 됩니다. 또는 [specialized
   utility](https://dotfiles.github.io/utilities/)를 사용할 수도 있습니다.
4. 여러분의 설치 스크립트를 새로운 가상 머신에서 테스트 해 보세요.
5. 현재 도구 구성의 모든 사항을 여러분의 도트파일 레포지토리로 옮겨(migrate)줍니다.
6. 여러분의 도트 파일을 GitHub에 공개하세요. 

## 원격 머신

이번 실습을 위해서 리눅스 가상 머신을 설치 (혹은 기존의 것을 사용) 하세요. 만약 여러분이 가상머신 사용에 익숙하지 않다면,  [여기](https://hibbard.eu/install-ubuntu-virtual-box/) 튜토리얼을 참고해 설치해 보세요.

1. `~/.ssh/`로 이동한 다음 그곳에 SSH 키페어가 있는지 확인합니다. 만일 키페어가 없다면, `ssh-keygen -a 100 -t ed25519`를 통해 생성하세요. 비밀번호와 `ssh-agent`를 사용하는 것이 권장됩니다. 더 많은 정보는 [여기](https://www.ssh.com/ssh/agent)서 확인하세요. 

2. `.ssh/config`를 수정해 다음과 같은 사항을 추가해 줍니다.

   ```bash
   Host vm
       User username_goes_here
       HostName ip_goes_here
       IdentityFile ~/.ssh/id_ed25519
       LocalForward 9999 localhost:8888
   ```

1. `ssh-copy-id vm`를 통해 ssh 키를 서버에 복사하세요.
2. `python -m http.server 8888`를 실행해 여러분의 가상 머신에서 웹서버를 시작합니다. 여러분의 머신에서 `http://localhost:9999`를 통해 해당 서버에 접근할 수 있습니다.
3. `sudo vim /etc/ssh/sshd_config`를 사용해 SSH 서버의 구성을 수정하세요. 그리고 `PasswordAuthentication`의 값을 변경해 비밀번호 인증을 해제합니다. `PermitRootLogin`의 값을 변경해 루트 로그인 기능을 해제합니다. `sudo service sshd restart` 명령어로 `ssh`를 재시작 합니다. ssh 접속을 다시 시도해 보세요. 
4. (도전) 가상 머신에 [`mosh`](https://mosh.org/)를 설치해 연결을 구성해 보세요. 그리고 서버/가상머신의 네트워크 어댑터를 연결 해제 합니다. mosh가 이 문제를 제대로 해결할 수 있나요?
5. (도전) `ssh`에서 `-N`과 `-f` 플래그가 어떤 역할을 하는지 살펴보세요. 그리고 백그라운드 포트 포워딩을 하기 위해서는 어떤 명령어를 써야 하는지 찾아보세요. 
