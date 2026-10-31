---
layout: lecture
title: "Debugging and Profiling"
description: >
  로깅, 디버거, 정적 분석으로 프로그램을 디버깅하고 성능을 프로파일링하는 방법을 배웁니다.
thumbnail: /static/assets/thumbnails/2020/lec7.png
date: 2020-01-23
ready: true
video:
  aspect: 56.25
  id: l812pUnKxME
translation_status: complete
---

프로그래밍에서 가장 중요한 원칙은 '기대한 대로 작동하는 것'이 아니라 '지시내린 대로 작동한다는 것'입니다. 이 격차를 메꾸기는 꽤 어려운 일입니다. 이번 강의에서는 버그가 있거나 리소스를 많이 소비하는 코드를 다루기 위한 기술인 디버깅과 프로파일링에 대해 배웁니다.

# 디버깅

## 출력을 통한 디버깅과 로깅

"코드의 상태를 신중하게 출력해가며 심사숙고하는 것보다 효과적인 디버깅 도구는 없습니다." — 브라이언 커니핸, _Unix for Beginners_.

디버깅을 위한 첫 번째 방법은 문제가 발견되는 곳 주위의 상태를 출력해보고 문제의 원인을 발견할 때까지 이 과정을 반복하는 것입니다.

두 번째 방법은 출력문을 사용하지 않고 로깅(logging)을 사용하는 것입니다. 아래는 로깅이 일반적인 출력문보다 더 나은 이유입니다.

- 파일이나 소켓뿐만 아니라 원격 서버에 대해서도 기록을 남길 수 있습니다.
- INFO, DEBUG, WARN, ERROR 등과 같은 심각도에 관한 로그 레벨을 지원합니다. 이 레벨에 따라 출력을 필터링 할 수도 있습니다.
- 새로운 이슈가 발생했을 때 무엇이 잘못되었는지를 감지하기도 합니다.

[아래](/static/files/logger.py)는 로그 메시지에 대한 예시 코드입니다.

```bash
$ python logger.py
# 출력문(print)을 사용하여 출력하기
$ python logger.py log
# 로그 형식으로 출력하기
$ python logger.py log ERROR
# ERROR 이상의 로그 레벨에 대해서만 출력하기
$ python logger.py color
# 색칠된 형식으로 출력하기
```

읽기 쉬운 로그를 위한 한 가지 팁은 색칠된 로그를 뽑는 것입니다. 방금 여러분은 색깔 덕분에 훨씬 더 읽기 쉬웠을 것입니다. 어떻게 할 수 있을까요? `ls` 나 `grep` 과 같은 프로그램은 색칠된 출력으로 바꾸어주는 특수문자 시퀀스인 [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code)를 사용합니다. 예를 들어, `echo -e "\e[38;2;255;0;0mThis is red\e[0m"` 를 실행하면 `This is red` 라는 메시지가 빨간색으로 출력됩니다. 이 예제는 터미널이 [트루 컬러(true color)](https://github.com/termstandard/colors#truecolor-support-in-output-devices)를 지원해야 합니다. macOS의 Terminal.app처럼 이를 지원하지 않는 터미널에서는 더 널리 지원되는 16색 ANSI escape codes를 사용할 수 있습니다. 예를 들어 `echo -e "\e[31;1mThis is red\e[0m"`를 실행해보세요.

아래는 트루 컬러를 지원하는 터미널에 많은 RGB 색을 출력하는 방법에 대한 스크립트입니다.

```bash
#!/usr/bin/env bash
for R in $(seq 0 20 255); do
    for G in $(seq 0 20 255); do
        for B in $(seq 0 20 255); do
            printf "\e[38;2;${R};${G};${B}m█\e[0m";
        done
    done
done
```

## 서드 파티 로그(Third party logs)

더 큰 소프트웨어 시스템을 구축한다면 별도 프로그램끼리의 의존성을 맞닥드리게 될 것입니다. 웹 서버, 데이터베이스, 메시지 브로커는 이런 종류의 의존성에 대한 일반적인 예시입니다. 이런 시스템과 상호작용 할 때에는 클라이언트 측의 에러 메시지 만으로는 충분치 않기 때문에 필수적으로 각 시스템의 로그를 읽어야합니다.

운좋게도, 대부분의 프로그램이 그들의 로그를 시스템 어딘가에 남겨놓습니다. 유닉스에서는 `/var/log` 이하에 남겨놓습니다. 예를 들어, [NGINX](https://www.nginx.com/) 웹서버는 `/var/log/nginx` 에 로그를 남깁니다. 최근 시스템들은 모든 로그 메시지가 가는 곳인 **시스템 로그**라는 것을 사용하기 시작했습니다. (전부는 아니지만) 거의 모든  리눅스 시스템은 `systemd` 를 사용합니다. `systemd` 는 어떤 서비스가 활성화 되어있고 실행가능한지 등을 제어하는 시스템 데몬입니다. `systemd` 는 `/var/log/journal` 에 로그를 특수한 양식으로 기록합니다. [`journalctl`](https://www.man7.org/linux/man-pages/man1/journalctl.1.html) 를 사용하면 이 로그 메시지를 볼 수 있습니다. 맥OS에서는 여전히 `/var/log/system.log` 에 기록되고 있지만 [`log show`](https://www.manpagez.com/man/1/log/) 명령어를 사용하여 볼 수 있는 시스템 로그 도구들이 늘어나는 중입니다. 대부분의 유닉스 시스템은 커널 로그에 접근하기 위한 [`dmesg`](https://www.man7.org/linux/man-pages/man1/dmesg.1.html) 명령어도 사용합니다.

시스템 로그 이하의 로그에 접근하기 위해서는 [`logger`](https://www.man7.org/linux/man-pages/man1/logger.1.html) 라는 쉘 프로그램을 사용합니다. 아래는 `logger` 를 사용하여 시스템 로그에 작성된 로그를 체크하는 방법에 대한 예시입니다. 또한 대부분의 프로그래밍 언어는 시스템 로그에 대한 바인딩 로깅이 있습니다.

```bash
logger "Hello Logs"
# On macOS
log show --last 1m | grep Hello
# On Linux
journalctl --since "1m ago" | grep Hello
```

데이터 랭글링 강의에서 보았던 것처럼 로그는 장황하기 때문에 원하는 정보를 얻기 위해서는 전처리와 필터링이 필요합니다. 만약 `journalctl` 나 `log show` 를 사용하여 자주 필터링 한다면 출력을 첫 번째로 필터링하기 위한 플래그 사용을 고려해볼 수 있습니다. 또한 로그 파일을 더욱 잘 보이도록 도와주는 [`lnav`](https://lnav.org/) 같은 도구도 있습니다.

## 디버거(Debugger)

출력을 이용한 디버깅이 충분치 않을 때는 디버거를 사용해야 합니다. 디버거는 실행 프로그램과 상호작용하며 아래 기능이 가능한 프로그램입니다.

- 특정 줄에 도달하면 프로그램 실행을 중지합니다.
- 프로그램을 한 번에 하나씩 단계별로 수행합니다.
- 프로그램 오류가 나면 변수의 값을 검사합니다.
- 주어진 조건을 만족하면 프로그램 실행을 중지합니다.
- 이런 기능들 외에도 다른 기능을 가지고 있습니다.

많은 프로그래밍 언어는 몇 개의 디버거를 가지고 있습니다. 파이썬에는 파이썬 디버거인 [`pdb`](https://docs.python.org/3/library/pdb.html) 가 있습니다. 

아래는 `pdb` 가 지원하는 짧은 명령어들입니다.

- **l**(ist) - 현재 줄 주위에 있는 11줄을 보여주거나 이전 목록을 그대로 보여줍니다.
- **s**(tep) - 현재 줄을 실행하고 바로 중지합니다.
- **n**(ext) - 현재 함수의 끝에 도달하거나 함수를 반환할 때까지 실행합니다.
- **b**(reak) - 입력한 인자에 따라 브레이크 포인트를 설정합니다.
- **p**(rint) - 현재 코드 표현을 평가하고 평가한 값을 출력합니다. [`pprint`](https://docs.python.org/3/library/pprint.html)를 사용하기 위한 **pp** 명령어도 있습니다.
- **r**(eturn) - 현재 함수가 반환될 때까지 함수를 계속합니다.
- **q**(uit) - 디버거를 종료합니다.

다음은 버그가 있는 파이썬 코드를 수정하기 위해서 `pdb` 를 사용하는 예시입니다. (강의 비디오를 봐주세요.) 

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(n):
            if arr[j] > arr[j+1]:
                arr[j] = arr[j+1]
                arr[j+1] = arr[j]
    return arr

print(bubble_sort([4, 2, 1, 8, 7, 6]))
```

파이썬은 인터프리터 언어기 때문에 `pdb` 쉘을 사용하여 명령(commands and instruction)을 실행할 수 있습니다. [`IPython`](https://ipython.org) REPL을 사용하는 [`ipdb`](https://pypi.org/project/ipdb/) 는 `pdb` 를 개선한 디버거 입니다.  `pdb` 와 동일한 인터페이스를 가지고 있으면서도 탭 완성이나 코드 구문 강조, 더 좋은 트레이스백과 내부 검사 기능을 제공합니다.

로우 레벨 프로그래밍 언어를 사용한다면 [`gdb`](https://www.gnu.org/software/gdb/)(및 이를 개선한 [`pwndbg`](https://github.com/pwndbg/pwndbg)) 와 [`lldb`](https://lldb.llvm.org/) 가 있습니다. 이들은 C 같은 언어의 디버깅에 최적화되어 있습니다. 하지만 거의 모든 프로세스를 조사하고 registers, stack, program counter 등 현재 기계의 상태를 얻을 수 있습니다.


## 전문화된 도구

디버깅하려는 대상이 블랙박스 바이너리(black box binary)라도 이를 돕는 도구들이 있습니다. 프로그램은 커널으로만 할 수 있는 작업을 수행해야 할 때마다 [시스템 호출(System Calls)](https://en.wikipedia.org/wiki/System_call)을 사용합니다. 프로그램이 만들어 내는 시스템 호출을 추적할 수 있는 명령어가 있습니다. 리눅스에서는 [`strace`](https://www.man7.org/linux/man-pages/man1/strace.1.html)를 사용하고 맥OS와 BSD에서는 [`dtrace`](https://dtrace.org/about/)를 사용합니다. `dtrace` 는 `D`언어로 작성되어 있기 때문에 사용하기 까다롭습니다. 하지만 `strace`와 유사한 인터페이스를 갖는 [`dtruss`](https://www.manpagez.com/man/1/dtruss/)라는 래퍼가 있습니다. (자세한 사항은 [이곳](https://8thlight.com/blog/colin-jones/2015/11/06/dtrace-even-better-than-strace-for-osx.html)을 참고해주세요)

아래는 `strace` 나 `dtruss` 를 사용하여 `ls`의 실행에 대한 [`stat`](https://www.man7.org/linux/man-pages/man2/stat.2.html) 시스템 호출을 추적하는 예시입니다. `strace`에 대하여 더욱 자세히 알고 싶다면 [이 글](https://blogs.oracle.com/linux/strace-the-sysadmins-microscope-v2)과 [이 소책자](https://jvns.ca/strace-zine-unfolded.pdf)를 참고해주세요. 

```bash
# 리눅스에서
sudo strace -e lstat ls -l > /dev/null
# 맥OS에서
sudo dtruss -t lstat64_extended ls -l > /dev/null
```

이슈 사항을 보여주는 네트워크 패킷이 필요한 상황이 있을 수 있습니다. [`tcpdump`](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) 와 [Wireshark](https://www.wireshark.org/) 등은 네트워크 패킷을 보거나 특정 기준에 따라서 필터링할 수 있는 네트워크 패킷 분석도구 입니다.

웹 개발에서 크롬/파이어폭스 개발자 도구는 매우 다루기 편합니다. 이들은 아래의 기능을 포함한 많은 기능을 가지고 있습니다.

- 소스 코드 - 어떤 사이트라도 HTML/CSS/JS 소스 코드를 조사할 수 있습니다.
- 실시간 HTML, CSS, JS 수정 - 테스트 용으로 웹 사이트의 콘텐츠나 스타일, 동적 기능을 바꿔볼 수 있습니다.
- 자바스크립트 쉘 - JS REPL에서 명령을 수행할 수 있습니다.
- 네트워크 - 요청 타임라인을 분석할 수 있습니다.
- 저장 - 쿠키나 로컬 애플리케이션 저장소를 살펴볼 수 있습니다.

## 정적(Static) 분석

어떤 이슈는 어떤 코드를 실행할 필요가 없기도 합니다. 예를 들어 코드를 유심히 보면 반복문의 변수가 이미 존재하는 변수나 함수의 이름을 덮어버릴 수 있습니다. 또는 어떤 변수를 정의하기도 전에 프로그램이 읽어버릴 수도 있습니다. 이 때가 [정적 분석](https://en.wikipedia.org/wiki/Static_program_analysis) 도구를 필요로하는 시점입니다. 정적 분석 프로그램은 소스 코드를 입력값으로 가져와서 그 코드가 올바른 코딩 규칙을 따르고 있는지 분석합니다.

아래의 파이썬 스니펫에는 몇 가지 실수가 있습니다. 첫 번째는 반복문의 변수인 `foo` 가 이전에 정의된 함수인 `foo`를 가려버립니다. 그리고 마지막에는 `bar` 대신에 `baz` 라고 썼습니다. 그래서 아래 코드는 1분간의 `sleep` 호출을 완료한 후에 오류가 날 것입니다.

```python
import time

def foo():
    return 42

for foo in range(5):
    print(foo)
bar = 1
bar *= 0.2
time.sleep(60)
print(baz)
```

정적 분석 도구는 이런 이슈를 잡아냅니다. 코드에서 [`pyflakes`](https://pypi.org/project/pyflakes)를 실행해준다면 두 버그에 관련된 에러를 얻을 수 있습니다. [`mypy`](https://mypy-lang.org/)는 변수 타입 오류를 탐지할 수 있는 또 다른 도구입니다. 아래 스크립트에서, `mypy`가 `bar`변수는 `int`로 정의되었지만 `float`로 변했다고 경고하고 있습니다. 코드를 실행하지 않고도 이런 이슈를 탐지할 수 있다는 것에 다시 한 번 주목해주세요.

```bash
$ pyflakes foobar.py
foobar.py:6: redefinition of unused 'foo' from line 3
foobar.py:11: undefined name 'baz'

$ mypy foobar.py
foobar.py:6: error: Incompatible types in assignment (expression has type "int", variable has type "Callable[[], Any]")
foobar.py:9: error: Incompatible types in assignment (expression has type "float", variable has type "int")
foobar.py:11: error: Name 'baz' is not defined
Found 3 errors in 1 file (checked 1 source file)
```

쉘 도구 강의에서 다루었던 [`shellcheck`](https://www.shellcheck.net/)과 유사한 도구입니다. 

대부분의 코드 에디터나 IDE는 경고나 에러의 위치를 강조하여 보여주는 도구를 자체적으로 가지고 있습니다. 이 도구는 **코드 린팅(linting)** 이라고 불리며 코드 스타일 위반이나 안전하지 않은 코드 구축 등의 여러 타입에 대한 이슈를 보여줍니다.

vim에서는 [`ale`](https://vimawesome.com/plugin/ale) 이나 [`syntastic`](https://vimawesome.com/plugin/syntastic) 플러그인을 통해 위의 것들을 할 수 있습니다. [`pylint`](https://github.com/PyCQA/pylint) 와 [`pep8`](https://pypi.org/project/pep8/) 는 파이썬을 위한 코드 스타일 린터(linter)입니다. 그리고 [`bandit`](https://pypi.org/project/bandit/) 은 파이썬 코드에서 일반적인 보안 이슈를 찾아내기 위해서 설계된 도구입니다. [Awesome Static Analysis](https://github.com/mre/awesome-static-analysis) 나 [Awesome Linters](https://github.com/caramelomartins/awesome-linters) 등과 같이 다른 언어에 대해서도 정적 분석 도구나 린터의 목록을 작성해놓은 페이지가 있습니다.

코드 스타일 린팅을 보완하는 도구로는 파이썬을 위한 [`black`](https://github.com/psf/black), Go언어를 위한 `gofmt`, 러스트를 위한 `rustfmt`, HTML/CSS/JS를 위한 [`prettier`](https://prettier.io/) 등이 있습니다. 이런 도구들로 코드를 자동 포맷하면 사용하는 언어에 맞는 일반적인 코드 스타일 패턴을 유지할 수 있습니다. 여러분의 코드 스타일을 제어하는 것을 원치 않을 수도 있습니다. 하지만 형식화된 코드 포맷은 다른 사람이 코드를 잘 읽을 수 있도록 하고 여러분이 다른 사람의 코드를 읽는 것도 도와줍니다.

# 프로파일링

코드가 작동 도중에 CPU나 메모리를 과도하게 잡아먹는다면, 그저 원하는대로 움직인다는 사실만으로 만족하기는 이릅니다. 알고리즘 수업에서 빅-O 표기법에 대해서 배우기는 하지만 여러분의 프로그램에서 주요 지점을 찾아내는 방법은 가르치지 않습니다. [섣부른 최적화(premature optimization)는 만악의 근원](https://wiki.c2.com/?PrematureOptimization)이기 때문에, 여러분은 프로파일러와 모니터링 도구에 대해서 배워야 합니다. 이런 도구들은 여러분이 시간과 자원을 많이 소비하는 부분을 찾아내고 그 부분을 최적화하는 데에 도움을 줄 것입니다.

<span id="timing"></span>

## 시간

디버깅과 유사하게 두 점 사이에 있는 코드의 실행 시간을 출력해보는 것만으로도 충분한 도움이 됩니다. 아래는 파이썬에서 [`time`](https://docs.python.org/3/library/time.html) 모듈을 사용한 예시입니다.

```python
import time, random
n = random.randint(1, 10) * 100

# Get current time
start = time.time()

# Do some work
print("Sleeping for {} ms".format(n))
time.sleep(n/1000)

# Compute time between start and now
print(time.time() - start)

# Output
# Sleeping for 500 ms
# 0.5713930130004883
```

여러분의 컴퓨터가 동시에 다른 프로세스를 실행하고 있거나 특정 이벤트를 기다리고 있다면 이렇게 기록된 시간은 오해의 소지가 있습니다. 그래서 시간과 관련된 도구들은 보통 *실제 시간, 사용자 시간, 시스템 시간* 을 구분합니다. 일반적으로 *사용자 시간 + 시스템 시간* 은 CPU에서 여러분의 작업이 얼마나 긴 시간을 필요로 하는지 말해줍니다(더욱 자세한 설명은 [이곳](https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1)을 참고).

- *실제 시간* - 다른 프로세스에 걸린 시간과 차단된 동안 지난 시간(I/O, 네트워크 대기 시간 등)을 포함하여 프로그램 시작과 종료까지 소요된 시간

- *사용자 시간* - CPU에서 사용자 코드를 실행하는 데 걸린 시간
- *시스템 시간* - CPU에서 커널 코드를 실행하는 데 걸린 시간 

예를 들어, HTTP 요청을 명령한 후에 [`time`](https://www.man7.org/linux/man-pages/man1/time.1.html)을 사용하여 시간을 재보세요. 연결이 느리다면 아래와 같은 결과물을 얻게 될 것입니다. 아래에서 요청이 완료되는 데에 2초가 넘게 걸렸지만 CPU 사용자 시간과 커널 CPU 시간은 각각 15ms, 12ms만 걸린 것을 볼 수 있습니다.

```bash
$ time curl https://missing.csail.mit.edu &> /dev/null
real    0m2.561s
user    0m0.015s
sys     0m0.012s
```

## 프로파일러

### CPU

사람들이 보통 *프로파일러* 라고 말할 때 *CPU 프로파일러* 를 의미합니다. CPU 프로파일러는 크게 *트레이싱 프로파일러*와 *샘플링 프로파일러*로 나눌 수 있습니다. 트레이싱 프로파일러는 프로그램이 실행하는 함수 호출을 모두 기록합니다. 반면 샘플링 프로파일러는 주기적으로(보통 1ms마다) 프로그램을 조사하고 이 내용을 프로그램 스택에 기록합니다. 그들은 기록한 내용을 사용하여 프로그램이 가장 많은 시간을 소비한 작업에 대해 통계치를 보여줍니다. 이 주제에 대해 자세한 사항을 알고 싶다면 [이곳](https://jvns.ca/blog/2017/12/17/how-do-ruby---python-profilers-work-)에 좋은 소개 기사가 있습니다. 

대부분의 프로그래밍 언어에는 여러분의 코드를 분석하기 위한 커맨드라인 프로파일러 몇 개가 내장되어 있습니다. 이들은 보통 IDE에 통합되어 있기는 하지만 이번 강의에서는 커맨드라인 도구 자체에 대해서만 집중적으로 알아볼 것입니다.

파이썬에서는 함수의 호출 시간을 알아보기 위해 `cProfile` 을 사용합니다. 다음은 파이썬에서 기초적인 grep을 구현하는 코드 예시입니다.

```python
#!/usr/bin/env python

import sys, re

def grep(pattern, file):
    with open(file, 'r') as f:
        print(file)
        for i, line in enumerate(f.readlines()):
            pattern = re.compile(pattern)
            match = pattern.search(line)
            if match is not None:
                print("{}: {}".format(i, line), end="")

if __name__ == '__main__':
    times = int(sys.argv[1])
    pattern = sys.argv[2]
    for i in range(times):
        for file in sys.argv[3:]:
            grep(pattern, file)
```

아래 명령어를 사용하면 위 코드를 프로파일링 할 수 있습니다. 출력을 분석해보면 IO가 대부분의 시간을 잡아먹고 있고 정규표현식을 컴파일링 하는 것 역시 상당량의 시간을 잡아먹고 있습니다. 정규 표현식 컴파일은 한 번만 해도 충분하기 때문에 for 반복문 밖으로 꺼낼 수 있습니다.

```
$ python -m cProfile -s tottime grep.py 1000 '^(import|\s*def)[^,]*$' *.py

[omitted program output]

 ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     8000    0.266    0.000    0.292    0.000 {built-in method io.open}
     8000    0.153    0.000    0.894    0.000 grep.py:5(grep)
    17000    0.101    0.000    0.101    0.000 {built-in method builtins.print}
     8000    0.100    0.000    0.129    0.000 {method 'readlines' of '_io._IOBase' objects}
    93000    0.097    0.000    0.111    0.000 re.py:286(_compile)
    93000    0.069    0.000    0.069    0.000 {method 'search' of '_sre.SRE_Pattern' objects}
    93000    0.030    0.000    0.141    0.000 re.py:231(compile)
    17000    0.019    0.000    0.029    0.000 codecs.py:318(decode)
        1    0.017    0.017    0.911    0.911 grep.py:3(<module>)

[omitted lines]
```

(다른 프로파일러도 마찬가지지만) 파이썬 `cProfile` 프로파일러의 주의 사항은 함수 호출마다 시간을 보여준다는 것입니다. 함수 호출은 너무나도 빠르게 호출되기 때문에 직관적이지 않습니다. 특히 서드파티 라이브러리를 사용한다면 라이브러리 내부 함수의 호출이 모두 고려되기 때문에 더욱 보기가 어려워집니다. 이럴 때 직관적으로 프로파일링 정보를 보여주려면  _line profilers_ 가 하는 것처럼 코드 라인마다 걸리는 시간을 포함하는 것입니다.

다음 파이썬 코드는 클래스 웹 사이트에 대한 요청을 수행하고 응답을 파싱하여 페이지 내에 있는 모든 URL을 가져옵니다.

```python
#!/usr/bin/env python
import requests
from bs4 import BeautifulSoup

# This is a decorator that tells line_profiler
# that we want to analyze this function
@profile
def get_urls():
    response = requests.get('https://missing.csail.mit.edu')
    s = BeautifulSoup(response.content, 'lxml')
    urls = []
    for url in s.find_all('a'):
        urls.append(url['href'])

if __name__ == '__main__':
    get_urls()
```

만약에 `cProfile` 을 사용하한다면 2500줄이 넘는 결과물을 받게 됩니다. 게다가 이 출력물은 시간이 오래 걸린 순서대로 정렬되기 때문에 더욱 이해하기 어렵습니다. 하지만 [`line_profiler`](https://github.com/pyutils/line_profiler)는 줄마다 걸린 시간을 보여줍니다.

```bash
$ kernprof -l -v a.py
Wrote profile results to urls.py.lprof
Timer unit: 1e-06 s

Total time: 0.636188 s
File: a.py
Function: get_urls at line 5

Line #  Hits         Time  Per Hit   % Time  Line Contents
==============================================================
 5                                           @profile
 6                                           def get_urls():
 7         1     613909.0 613909.0     96.5      response = requests.get('https://missing.csail.mit.edu')
 8         1      21559.0  21559.0      3.4      s = BeautifulSoup(response.content, 'lxml')
 9         1          2.0      2.0      0.0      urls = []
10        25        685.0     27.4      0.1      for url in s.find_all('a'):
11        24         33.0      1.4      0.0          urls.append(url['href'])
```

### 메모리

C나 C++ 등의 프로그래밍 언어에서 메모리 누수는 프로그램이 필요하지 않은 메모리를 해제하지 않는 현상을 유발합니다. 이런 경우 메모리 디버깅을 위해서 [Valgrind](https://valgrind.org/) 등의 도구를 고려해볼 수 있습니다.

파이썬과 같은 가비지 컬렉팅이 되는 언어 역시 메모리 프로파일러를 사용해야 합니다. 메모리에 객체에 대한 포인터가 있다면 그 객체는 가비지 컬렉팅이 되지 않기 때문입니다. 아래는 [memory-profiler](https://pypi.org/project/memory-profiler/) 를 사용하여 메모리를 분석한 예시입니다(`line-profiler` 같은 데커레이터에 주목해주세요).

```python
@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a

if __name__ == '__main__':
    my_func()
```

```bash
$ python -m memory_profiler example.py
Line #    Mem usage  Increment   Line Contents
==============================================
     3                           @profile
     4      5.97 MB    0.00 MB   def my_func():
     5     13.61 MB    7.64 MB       a = [1] * (10 ** 6)
     6    166.20 MB  152.59 MB       b = [2] * (2 * 10 ** 7)
     7     13.61 MB -152.59 MB       del b
     8     13.61 MB    0.00 MB       return a
```

### 이벤트 프로파일링

`strace`를 사용하여 디버깅을 할 때, 코드 중 일부를 프로파일링 하고 싶지 않은 경우도 있을 것입니다. [`perf`](https://www.man7.org/linux/man-pages/man1/perf.1.html) 명령어는 CPU 변화를 요약하고 시간이나 메모리를 보고하지 않는 대신, 프로그램과 관련된 시스템 이벤트를 보고합니다. 예를 들어, `perf`는 poor cache locality, 다량의 페이지 오류나 라이브락(livelock)을 보고합니다. 아래는 명령어의 예시입니다.

- `perf list` - perf 가 추적할 수 있는 사건의 목록을 보여줍니다.
- `perf stat COMMAND ARG1 ARG2` - 프로세스나 명령어와 관련된 서로 다른 사건의 수를 보여줍니다.
- `perf record COMMAND ARG1 ARG2` - 명령 실행 내역을 기록하고 `perf.data`라고 불리는 통계 데이터를 저장합니다.
- `perf report` - `perf.data` 내에 모아진 데이터를 형식화하여 출력합니다.


### 시각화

소프트웨어 프로젝트가 매우 복잡하기 때문에 실제로 출력된 프로파일링 결과는 너무나도 많은 양의 정보를 포함하고 있습니다. 사람은 많은 숫자를 읽고 이해하는 것에 대한 거부감이 있으며 시각 자료를 좋아합니다. 그래서 프로파일링 출력물을 훨씬 보기 쉽게 해주는 많은 시각화 도구들이 나오게 되었습니다.

[Flame Graph](https://www.brendangregg.com/flamegraphs.html)는 샘플링 프로파일러를 사용한 CPU프로파일링 정보를 보여주는 가장 일반적인 방법입니다. Y축에는 함수 호출을 계측적으로 보여주고, X축에는 소요된 시간의 비율을 보여줍니다. 또한 특정 부분을 확대하여 볼 수 있도록 매우 인터랙티브하다는 장점이 있습니다(아래 이미지를 클릭해보세요).

[![FlameGraph](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)

Call graphs나 control flow graphs는 각 함수를 노드로, 호출되는 과정을 엣지로 나타내어 프로그램 내부 서브루틴 간의 관계를 표현합니다. 수행 시간이나 호출 횟수 등의 프로파일링 정보과 같이 본다면 이런 그래프는 프로그램의 흐름을 보기에 유용한 도구입니다. 파이썬에서는 [`pycallgraph`](https://pycallgraph.readthedocs.io/) 라이브러리를 사용하여 이런 그래프를 볼 수 있습니다.

![Call Graph](https://upload.wikimedia.org/wikipedia/commons/2/2f/A_Call_Graph_generated_by_pycallgraph.png)


## 리소스 모니터링

리소스가 얼마나 소비되는 지 이해하는 것은 프로그램의 성능을 분석하기 위한 첫 번째 단계로 고려되기도 합니다. 네트워크 연결이 느리거나 충분하지 않은 메모리 등으로 리소스가 제한되면 프로그램이 느려집니다. 이를 위해 CPU 사용량, 메모리 사용량, 디스크 사용량 등의 시스템 리소스를 보여주는 커맨드라인 도구들이 있습니다.

- **범용 모니터링** - 가장 유명한 것은 [`top`](https://www.man7.org/linux/man-pages/man1/top.1.html)을 개선한 버전인 [`htop`](https://htop.dev/)일 것입니다. `htop`은 현재 실행되고 있는 작업에 대한 다양한 통계량을 보여줍니다. `htop`에는 많은 옵션과 키바인드가 있습니다. 예를 들어, `<F6>`은 프로세스를 정렬해주고 `t`는 계층적 트리를 부여주며 `h`는 스레드를 전환합니다. [`glances`](https://nicolargo.github.io/glances/)도 뛰어난 UI로 `htop`과 비슷하게 구현되어 있는 도구입니다. 모든 프로세스에 대해 집계한 측정값을 얻기 위해서는  [`dool`](https://github.com/scottchiefbaker/dool) 라는 도구를 사용합니다. 이 도구는 I/O, 네트워킹, CPU 사용률, 컨텍스트 스위치 등에 대한 리소스 수치를 실시간으로 보여줍니다.
- **I/O 작업** - [`iotop`](https://www.man7.org/linux/man-pages/man8/iotop.8.html)은 실시간 I/O 사용량에 대한 정보를 보여주어 무거운 I/O 디스크 작업을 사용하는 프로세스를 체크하기 편리합니다.
- **디스크 사용량** - [`df`](https://www.man7.org/linux/man-pages/man1/df.1.html)는 각 파티션의 수치를 보여주고 [`du`](https://man7.org/linux/man-pages/man1/du.1.html)(**d**isk **u**sage)는 현재 폴더에 있는 각 파일의 디스크 사용량을 보여줍니다. 이런 도구에서 `-h` 플래그는 프로그램에게 사람이 읽을 수 있는 형식으로 출력하도록 지시합니다. [`ncdu`](https://dev.yorhel.nl/ncdu)는 `du`의 더욱 인터랙티브한 버전으로 폴더를 탐색하면서 파일이나 폴더를 지울 수 있도록 합니다.
- **메모리 사용량** - [`free`](https://www.man7.org/linux/man-pages/man1/free.1.html)는 시스템에 있는 사용하고 있는 메모리와 남아있는 메모리의 양이 얼마나 되는지 보여줍니다. 메모리 사용량은 `htop`과 같은 도구를 사용해서도 볼 수 있습니다.
- **열린 파일** - [`lsof`](https://www.man7.org/linux/man-pages/man8/lsof.8.html)은 프로세스에 의해서 열린 파일에 대한 목록 정보를 보여줍니다. 특정 파일이 프로세스에 의해 열리는지 아닌지를 체크하기에 유용합니다.
- **네트워크 연결과 구성** - [`ss`](https://www.man7.org/linux/man-pages/man8/ss.8.html)는 수신 및 발신에 대한 네트워크 패킷과 인터페이스에 대한 통계를 모니터링합니다. `ss`는 보통 주어진 포트를 사용하는 프로세스를 파악합는데 사용합니다. 라우팅과 네트워크 장치, 인터페이스를 볼 때는 [`ip`](https://man7.org/linux/man-pages/man8/ip.8.html)를 사용할 수도 있습니다.  `netstat` 이나 `ifconfig`은 위와 같은 도구들 때문에 더 이상 사용되지 않습니다.
- **네트워크 사용량** - [`nethogs`](https://github.com/raboof/nethogs) 와 [`iftop`](https://pdw.ex-parrot.com/iftop/) 은 네트워크 사용량을 모니터링 할 수 있는 좋은 인터랙티브 CLI 도구입니다.

만약 이런 도구를 테스트하고 싶다면 [`stress`](https://linux.die.net/man/1/stress)라는 명령어를 사용하여 인위적으로 부하를 줄 수 있습니다.

### 특화된 도구들

블랙 박스 벤치마킹은 사용할 소프트웨어를 결정하기 위한 전부입니다. [`hyperfine`](https://github.com/sharkdp/hyperfine) 등의 도구는 커맨드 라인 프로그램을 빠르게 벤치마크 합니다. 예를 들어, 쉘 도구와 스크립트 강의에서 `find`보다  `fd` 를 권장했습니다. `hyperfine`은 이 둘 중 어느 것이 좋은지 비교하는 작업에 사용할 수 있습니다. 아래 예시에서는 제 컴퓨터에서  `fd` 가 `find` 보다 20배나 빠름을 보여주고 있습니다.

```bash
$ hyperfine --warmup 3 'fd -e jpg' 'find . -iname "*.jpg"'
Benchmark #1: fd -e jpg
  Time (mean ± σ):      51.4 ms ±   2.9 ms    [User: 121.0 ms, System: 160.5 ms]
  Range (min … max):    44.2 ms …  60.1 ms    56 runs

Benchmark #2: find . -iname "*.jpg"
  Time (mean ± σ):      1.126 s ±  0.101 s    [User: 141.1 ms, System: 956.1 ms]
  Range (min … max):    0.975 s …  1.287 s    10 runs

Summary
  'fd -e jpg' ran
   21.89 ± 2.33 times faster than 'find . -iname "*.jpg"'
```

디버깅과 마찬가지로 브라우저에는 웹 페이지 로딩을 프로파일링 하기 위한 좋은 도구가 제공됩니다. 이 도구로 시간을 많이 소요하는 과정이 로딩인지 렌더링인지 스크립팅인지 등을 파악할 수 있습니다. 더 많은 정보는 [Firefox](https://profiler.firefox.com/docs/)나 [Chrome](https://developers.google.com/web/tools/chrome-devtools/rendering-tools)을 확인해주세요.

# 연습문제

## 디버깅
1. 리눅스라면 `journalctl`, 맥OS라면 `log show`를 사용하여 마지막 슈퍼 유저의 접근과 명령어에 대해 출력하세요. 만약 실행한 명령이 없으면 `sudo ls` 같은 무해한 명령어를 실행한 뒤에 다시 확인할 수 있습니다.

1. 다양한 명령어에 친숙해지기 위해서 [이곳](https://github.com/spiside/pdb-tutorial)에서 `pdb` 튜토리얼을 해보세요. 더욱 심화된 튜토리얼을 하고 싶다면 [이곳](https://realpython.com/python-debugging-pdb)을 참조하세요.

1. [`shellcheck`](https://www.shellcheck.net/)을 설치하고 다음 스크립트를 체크해보세요. 어떤 부분이 틀렸는지 알아보고 고치세요. 경고 메시지를 자동으로 볼 수 있도록 코드 에디터에 린터(linter) 플러그인을 설치하세요.  

   ```bash
   #!/bin/sh
   ## Example: a typical script with several problems
   for f in $(ls *.m3u)
   do
     grep -qi hq.*mp3 $f \
       && echo -e 'Playlist $f contains a HQ file in mp3 format'
   done
   ```

1. (심화) [reversible debugging](https://undo.io/resources/reverse-debugging-whitepaper/)에 대해 읽어보고 [`rr`](https://rr-project.org/) 혹은 [`RevPDB`](https://morepypy.blogspot.com/2016/07/reverse-debugging-for-python.html)를 사용하여 간단한 작업을 해보세요.
## 프로파일링

1. [이곳](/static/files/sorts.py)에 다양한 정렬 알고리즘이 구현되어 있습니다. [`cProfile`](https://docs.python.org/3/library/profile.html) 과 [`line_profiler`](https://github.com/pyutils/line_profiler)를 사용하여 삽입 정렬과 퀵 정렬의 수행 시간을 비교해보세요. 각 알고리즘에서 병목 현상이 일어나는 곳은 어디인가요? `memory_profiler`를 사용하여 메모리 사용량을 비교해보고 왜 삽입 정렬이 더 나은지를 알아보세요. 내장된 버전의 퀵 소트를 체크해보세요. 도전 : `perf`를 사용하여 각 알고리즘의 주기 횟수 및 캐시 히트/미스를 확인해보세요.

1. 다음은 각 숫자에 대한 함수를 사용하여 피보나치 숫자를 계산하기 위한 파이썬 코드입니다. (다음 코드는 논란의 여지가 있습니다.) 

   ```python
   #!/usr/bin/env python
   def fib0(): return 0
   
   def fib1(): return 1
   
   s = """def fib{}(): return fib{}() + fib{}()"""
   
   if __name__ == '__main__':
   
       for n in range(2, 10):
           exec(s.format(n, n-1, n-2))
       # from functools import lru_cache
       # for n in range(10):
       #     exec("fib{} = lru_cache(1)(fib{})".format(n, n))
       print(eval("fib9()"))
   ```

   코드를 파일에 넣고 실행 가능하게 만드세요. 필요한 도구인 [`pycallgraph`](https://lewiscowles1986.github.io/py-call-graph/)와 [`graphviz`](https://graphviz.org/)를 설치하세요. (`dot`을 실행할 수 있다면 GraphViz가 이미 설치되어 있습니다.) `pycallgraph graphviz -- ./fib.py`라는 명령어와 함께 위 코드를 실행한 뒤 `pycallgraph.png`를 체크해보세요. `fib0`은 몇 번이나 호출되었을까요? 메모이제이션(memoization)하면 위 함수를 개선할 수 있습니다. 주석처리된 부분의 주석을 제거하고 이미지를 다시 생성해보세요. 이번에는 `fibN` 함수가 몇 번이나 호출되었나요?

1. 수신하려는 포트가 이미 다른 프로세스에 사용되고 있는 것은 일반적인 문제입니다. 이를 위해 프로세스의 pid를 알아내는 방법을 배워봅시다. `4444` 포트에서 수신 대기하는 최소한의 웹 서버를 만들기 위해서 `python -m http.server 4444` 명령을 실행합니다. 다른 터미널에서 `lsof | grep LISTEN`을 사용하여 모든 수신 프로세스와 포트를 출력합니다. 해당 프로세스의 pid를 찾고 `kill <PID>` 명령을 실행하여 프로세스를 종료합니다.

1. 프로세스 리소스를 제한하는 것은 편리한 도구가 될 수 있습니다. `stress -c 3` 을 실행해보고 `htop`으로 CPU사용량을 시각화해보세요. 그리고 `taskset --cpu-list 0,2 stress -c 3`을 실행한 뒤 이를 다시 시각화 해보세요. `stress`가 3개의 CPU에 걸쳐있나요? 그렇지 않다면 그 이유는 무엇일까요? [`man taskset`](https://www.man7.org/linux/man-pages/man1/taskset.1.html)을 읽어봅시다. 도전 : [`cgroups`](https://www.man7.org/linux/man-pages/man7/cgroups.7.html)를 사용하여 동일한 문제를 풀어보세요. `stress -m`의 메모리 소비를 제한해보세요.

1. (심화) `curl ipinfo.io` 명령어는 HTTP 요청을 수행하고 공용 IP에 대한 정보를 가져옵니다. [Wireshark](https://www.wireshark.org/)를 열고 요층을 스니핑한 뒤에 `curl`이 주고받은 패킷에 응답하세요. (힌트 : HTTP 패킷을 보려면 `http` 필터를 사용하세요.)
