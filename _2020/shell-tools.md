---
layout: lecture
title: "셸 툴과 스크립팅"
description: >
  셸 스크립트 작성법과 강력한 커맨드라인 도구 사용법을 배웁니다.
thumbnail: /static/assets/thumbnails/2020/lec2.png
date: 2020-01-14
ready: true
video:
  aspect: 56.25
  id: kgII-YWo3Zw
translation_status: complete
---

이번 강의에서는 배시(bash)를 스크립팅 언어로 사용하기 위해 필요한 기본 사항들과, 여러분이 앞으로 커맨드라인에서 가장 보편적인 작업을 할 때 사용되는 몇 개의 셸 도구들에 대해 소개할 것입니다.

# 셸(Shell) 스크립팅

지난 시간에 셸에서 명령을 추출하고 파이프로 명령을 연결시키는 방법에 대해서 배웠습니다. 그러나 많은 상황에선 일련의 명령을 수행하고 조건문이나 반복문과 같은 제어 흐름 표현식을 사용해야 하는 경우가 있습니다.

쉘 스크립트은 살짝 복잡합니다. 대부분의 셸에는 변수, 제어 흐름(control flow) 및 자체 구문(syntax)을 가진 자체 스크립팅 언어가 있습니다. 쉘 스크립트와 다른 프로그래밍 언어의 차이점은 쉘 관련 작업을 수행하기 위해 최적화되어 있다는 것입니다. 따라서 명령 파이프 라인을 만들고 파일을 저장하고 표준 입력을 받는 것은 쉘 스크립팅이 원시적이며 범용 스크립팅 언어보다 사용하기 쉽습니다. 이 섹션에서는 그중 가장 일반적인 배시(bash) 스크립팅에 중점을 둘 것입니다.

배시에서 변수에 값을 할당할 때는 `foo=bar`와 같이 입력합니다. 변수의 값에 접근할 때는 `$foo`로 액세스합니다. `foo = bar`와 같이 띄어쓰기를 사용하면 `=` 와 `bar`를 사용하여 `foo` 프로그램을 호출하는 것으로 해석되므로 작동하지 않습니다. 일반적으로 셸 스크립트에서 공간 문자는 인수 분할을 수행합니다. 이런 규칙은 일반적인 프로그래밍 언어와 다르기 때문에 헷갈릴 수가 있으니 항상 유의하세요.

배시에서 문자열은 `'`나 `"` 기호로 선언될 수 있지만 둘은 다른 의미를 가지게 됩니다. `'`로 둘러싸인 문자열은 문자열 자체를 뜻하지만  `"`로 둘러싸인 문자열은 변수값을 반환합니다.
```bash
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo
```

대부분의 프로그래밍 언어와 마찬가지로, 배시는 `if`, `case`, `while` 및 `for`를 포함한 제어 흐름 기술을 지원합니다. 마찬가지로, 배시는 인자를 만들고 사용하고 작동하는 기능들이 있습니다. 다음은 디렉토리를 만들고 `cd`를 수행하는 함수의 예시입니다.
```bash
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

`$1`은 script/function의 첫 번째 인자입니다. 다른 스크립팅 언어와 달리, bash는 인수, 오류 코드 및 기타 관련 변수를 참조하기 위해 다양한 특수 변수를 사용합니다. 아래에 그중 일부를 적어놨습니다. 더 포괄적인 목록은 [여기](https://tldp.org/LDP/abs/html/special-chars.html)서 찾을 수 있습니다.

- `$0` - 스크립트 이름
- `$1` ~ `$9` - 스크립트의 인자들. `$1`부터 첫번쨰 인자입니다.
- `$@` - 모든 인자들
- `$#` - 인자의 수
- `$?` - 이전 명령 수행결과 코드 (error code). C 프로그래밍 언어와 같이 0은 정상적으로 종료되었음을 의미합니다.
- `$$` - 현재 스크립트에 대한 프로세스 식별 번호 (PID)
- `!!` - 인수를 포함하여 마지막 명령 전체를 포함합니다. 일반적으로는 사용 권한이 누락되어 실패할 때 사용합니다. `sudo`를 함께 써서 실패한 명령을 신속하게 다시 실행할 수 있습니다.
- `$_` - 마지막 명령에서 나온 마지막 인수입니다. 대화형 셸에 있는 경우 `Esc`를 입력한 후 `.`을 입력하거나 `Alt+.`을 눌러 이 값을 신속하게 얻을 수 있습니다.

명령어들은 종종 `STDOUT`을 사용해서 출력을 하는데, 오류를 보고할 때는 좀 더 스크립트 친화적인 방식으로 `STDERR`를 사용해서 코드를 반환합니다. 리턴 코드 또는 종료 상태는 스크립트 또는 명령어들이 어떤 방식으로 실행 방법을 전달하는지를 보여주는 것입니다. 값 0은 일반적으로 모든 것이 정상임을 의미합니다. 0이 아닌 다른 값은 오류가 발생했음을 의미합니다.

종료 코드는 `&&`(and 연산자) 와 `||`(or 연산자) 즉, [short-circuiting](https://en.wikipedia.org/wiki/Short-circuit_evaluation) 연산자들을 사용하여 조건부로 명령을 실행하는 데 사용할 수 있습니다. 그리고 세미콜론 ; 을 사용하여 동일한 행 내에서 명령을 분리할 수도 있습니다. 
참(true)인 프로그램은 항상 0 리턴 코드를 가지며 거짓(false) 명령은 항상 1 리턴 코드를 갖습니다.몇 가지 예를 통해 한 번 봅시다. 

```bash
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
#

true ; echo "This will always run"
# This will always run

false ; echo "This will always run"
# This will always run
```

또 다른 일반적인 패턴은 명령의 출력을 변수로 가져오는 것입니다. 이것은 _명령어대체_ 로 수행 할 수 있습니다. `$(CMD)`를 기입할 때마다 `CMD`가 실행되고 명령의 출력을 가져와 제자리에 대체합니다. 예를 들어, `for file in $ (ls)` 라는 코드를 수행하면 쉘은 먼저 `ls` 를 호출한 다음 해당 값을 반복합니다. 위와 비슷하지만 상대적으로 덜 알려진 유사한 기능은 _프로세스 치환(process replacement)_ 입니다. `<(CMD)` 는 `CMD` 를 실행하고 출력을 임시 파일에 배치하고 `<()` 를 해당 파일의 이름으로 대체합니다. 이 명령어는 보통 STDIN이 아닌 파일 형태로 값이 전달 될 때 유용합니다. 예를 들어, `diff <(ls foo) <(ls bar)` 는 디렉토리 `foo` 와 `bar` 에 있는 파일 간의 차이점을 표시합니다.

위에 너무 많은 정보만을 제공했기 때문에, 이러한 기능 중 일부를 보여주는 예를 살펴보겠습니다. 아래의 예시는 우리가 제공하는 인수인 `foobar` 문자열에 대해 `grep` 을 반복하고, 찾을 수 없는 경우 주석으로 파일에 추가합니다.

```bash
#!/bin/bash

echo "Starting program at $(date)" # Date will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

비교에서 우리는`$?`와 0이 같지 않은지 테스트했습니다. Bash는 이런 종류의 많은 비교를 구현합니다. manpage에서 [test](https://www.man7.org/linux/man-pages/man1/test.1.html) 에 대한 자세한 목록을 찾을 수 있습니다. bash에서 비교를 수행할 때는 간단한 대괄호`[]`대신 이중 대괄호`[[]]`를 사용하십시오. 'sh'로 이식할 수는 없지만 실수할 가능성은 낮습니다. 자세한 설명은 [여기](https://mywiki.wooledge.org/BashFAQ/031)서 확인할 수 있습니다. 

스크립트를 시작할 때 유사한 인수를 제공해야 하는 경우가 종종 있을 것입니다. Bash에는 이런 상황에서 파일 이름을 확장하여 명령을 보다 쉽게 확장할 수 있는 방법을 제공합니다. 이러한 기술 중 하나가 쉘 _globbing_ 입니다.
- Wildcards - 일종의 와일드 카드 일치를 수행 할 때마다 `?` 및 `*` 를 사용하여 각각 하나 또는 원하는 양의 문자를 일치시킬 수 있습니다. 예를 들어 `foo`, `foo1`, `foo2`, `foo10` 및 `bar` 파일이 있을 때, `rm foo?` 명령어를 치면, `foo1` 과 `foo2` 를 삭제하는 반면에 `rm foo *` 는 `bar`를 뺀 모든 파일을 삭제합니다. 
- 중괄호(Curly braces) `{}` - 일련의 명령에 공통 부분 문자열이 있을 때마다 bash에 중괄호를 사용하여 이를 자동으로 확장할 수 있습니다. 이것은 파일을 이동하거나 변환할 때 매우 편리합니다.

```bash
convert image.{png,jpg}
# Will expand to
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# Will expand to
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder
# Will move all *.py and *.sh files


mkdir foo bar
# This creates files foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h
touch {foo,bar}/{a..h}
touch foo/x bar/y
# Show differences between files in foo and bar
diff <(ls foo) <(ls bar)
# Outputs
# < x
# ---
# > y
```

<!-- Lastly, pipes `|` are a core feature of scripting. Pipes connect one program's output to the next program's input. We will cover them more in detail in the data wrangling lecture. -->

`bash` 스크립트를 작성하는 것은 까다롭고 직관적이지 않을 수 있습니다. sh / bash 스크립트에서 오류를 찾는 데 도움이 되는 [shellcheck](https://github.com/koalaman/shellcheck)와 같은 도구가 있으니 참고해보세요.

참고로, 터미널에서 호출하기 위해 반드시 bash로 스크립트를 작성할 필요는 없습니다. 예를 들어 다음은 인수를 역순으로 출력하는 간단한 Python 스크립트입니다.

```python
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```


커널은 스크립트 상단에 [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) 줄을 포함했기 때문에 이 스크립트를 쉘 명령 대신 파이썬 인터프리터로 실행하는 것을 알고 있습니다. [env](https://www.man7.org/linux/man-pages/man1/env.1.html) 명령을 사용하여 명령이 시스템 어디에 있든 확인하는 것을 shebang 줄을 작성하여 연습하는 것이 좋습니다. 이것이 스크립트의 이식성을 증가시킵니다. 위치를 확인하기 위해`env`는 첫 번째 강의에서 소개한 `PATH` 환경 변수를 사용합니다. 이 예에서 shebang 줄은 `#!/usr/bin/env python`과 같습니다.

기억해야 할 셸 함수와 스크립트 간의 몇 가지 차이점은 다음과 같습니다.
- 함수는 쉘과 동일한 언어로 작성되어야 하며 스크립트는 모든 언어로 작성될 수 있습니다. 이것이 스크립트에 shebang을 포함하는 것이 중요한 이유입니다.
- 함수는 정의를 읽을 때 한 번 로드됩니다. 스크립트는 실행될 때마다 로드됩니다. 이렇게 하면 함수를 로드하는 속도가 약간 빨라지지만 변경할 때마다 정의를 다시 로드해야 합니다.
- 함수는 현재 쉘 환경에서 실행되는 반면, 스크립트는 자체 프로세스에서 실행됩니다. 따라서 함수는 환경 변수를 수정할 수 있습니다. 예를 들어, 현재 디렉토리를 변경하는 작업을 스크립트는 할 수 없습니다. 스크립트는 [export](https://www.man7.org/linux/man-pages/man1/export.1p.html)를 사용하여 내보낸 값을 환경 변수로 전달합니다.
- 모든 프로그래밍 언어와 마찬가지로 함수는 모듈성, 코드 재사용 및 쉘 코드의 명확성을 달성하기 위한 강력한 구성입니다. 종종 쉘 스크립트는 자체 함수 정의를 포함합니다.

# Shell Tools

## 명령 사용 방법 찾기

이 시점에서 `ls -l`, `mv -i` 및 `mkdir -p`와 같은 별칭 섹션에서 명령에 대한 플래그를 찾는 방법이 궁금 할 것입니다. 조금 더 일반적으로 말해, 명령이 주어지면 당신은 어떻게 그 명령어의 기능과 다른 옵션을 찾을 수 있습니까? 당신은 언제든지 구글링을 통해서 검색을 시작할 수 있지만, UNIX가 StackOverflow 보다 이전이므로 특정 명령어에 대한 정보를 가져오는 기본적으로 제공되는 방법이 따로 있습니다.

쉘 강의에서 보았듯이 1차 접근 방식은`-h` 또는 `--help` 플래그를 사용하여 해당 명령들을 찾아보는 것입니다. 혹은, 더 자세한 정보를 알기 위해 `man` 명령을 사용할 수 있습니다. manual의 줄임말인 [man](https://www.man7.org/linux/man-pages/man1/man.1.html)은 사용자가 지정한 명령에 대한 매뉴얼 페이지 (manpage라고 함)를 제공합니다. 예를 들어 `man rm`은 앞서 보여 준 `-i` 플래그를 포함하여 사용할 수 있는 모든 플래그와 함께 `rm` 명령의 동작을 출력합니다. 사실 지금까지의 모든 명령어에 대해 링크는 명령에 대한 Linux 매뉴얼 페이지의 온라인 버전입니다. 기본이 아닌 명령도 개발자가 작성하여 설치 프로세스의 일부로 포함하면 매뉴얼 페이지 항목이 있습니다. ncurses 기반 도구와 같은 대화형 도구의 경우, 프로그램 내에서 `: help` 명령을 사용하거나 `?` 를 입력하여 도움말에 액세스 할 수 있습니다.

때때로 man 페이지는 명령에 대한 지나치게 자세한 설명을 제공하여 일반적인 사용 사례에 사용할 플래그 / 구문을 해독하기 어렵게 만듭니다. [TLDR 페이지](https://tldr.sh/)는 사용할 옵션을 신속하게 파악할 수 있도록 명령의 사용 사례를 제공하는 데 초점을 맞춘 멋진 보완 솔루션입니다. 예를 들어, 저의 경우 [tar](https://tldr.inbrowser.app/pages/common/tar) 및 [ffmpeg](https://tldr.inbrowser.app/pages/common/ffmpeg)에 대한 tldr 페이지를 메뉴얼 페이지보다 더 자주 참조합니다. 

## 파일 찾기

모든 프로그래머가 직면하는 가장 일반적인 반복 작업 중 하나는 파일이나 디렉토리를 찾는 것입니다. 모든 유닉스 계열 시스템은 파일을 찾는 훌륭한 셸 도구인 [find](https://www.man7.org/linux/man-pages/man1/find.1.html)를 패키지로 함께 제공합니다. `find`는 일부 기준과 일치하는 파일을 재귀적으로 검색합니다. 몇 가지 예 :

```bash
# Find all directories named src
find . -name src -type d
# Find all python files that have a folder named test in their path
find . -path '*/test/*.py' -type f
# Find all files modified in the last day
find . -mtime -1
# Find all zip files with size in range 500k to 10M
find . -size +500k -size -10M -name '*.tar.gz'
```
파일 나열 외에도 `find`는 쿼리와 일치하는 파일에 대한 작업을 수행할 수도 있습니다. 이 속성(작업)은 상당히 단조로운 작업을 단순화하는 데 매우 유용할 수 있습니다.

```bash
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \;
# Find all PNG files and convert them to JPG
find . -name '*.png' -exec magick {} {}.jpg \;
```

`find`의 편리성에도 불구하고, 그 구문을 기억하기가 까다로울 수 있습니다.예를 들어, 어떤 패턴 `PATTERN`과 일치하는 파일을 찾으려면 `find -name PATTERN` (또는 패턴의 대소문자를 구분하지 않으려면 `-iname`)를 실행해야 합니다. 이러한 상황에 대비해서 별칭을 만들어서 편리하게 쓸 수 있지만, 쉘 철학의 일부는 대안을 탐색하는 것이 좋다는 것입니다. 쉘의 가장 좋은 속성 중 하나는 프로그램을 호출하는 것이므로 일부 대체 방법을 찾거나 직접 작성할 수 있다는 점을 기억하십시오. 예를 들어 [fd](https://github.com/sharkdp/fd)는`find`에 대한 간단하고 빠른 사용자 친화적 대안입니다. `fd`는 색상화된 출력, 기본 정규식 일치 및 유니 코드 지원과 같은 멋진 기본값을 제공합니다. 또한 제 생각에는 보다 직관적인 것 같습니다. 예를 들어 패턴 `PATTERN`을 찾는 구문은 `fd PATTERN`입니다.

대부분의 사람들은`find`와`fd` 둘 다 좋다는 데 동의할 것입니다. 그러나 여러분 중 일부는 빠른 검색을 위해 어떤 종류의 인덱스나 데이터베이스를 컴파일하는 것과 비교하여 매번 파일을 찾는 것의 효율성에 대해 궁금해 할 것입니다. 그것이 바로 [locate](https://www.man7.org/linux/man-pages/man1/locate.1.html)의 목적입니다. `locate`는 [updatedb](https://www.man7.org/linux/man-pages/man1/updatedb.1.html)를 사용하여 업데이트 된 데이터베이스를 사용합니다. 대부분의 시스템에서`updatedb`는 [cron](https://www.man7.org/linux/man-pages/man8/cron.8.html)을 통해 매일 업데이트됩니다. 따라서 둘 사이에서 한 가지 고려해야 할 점은 탐색 속도와 최신화입니다. 또한 `find` 와 유사한 도구는 파일 크기, 수정 시간 또는 파일 권한과 같은 속성을 사용하여 파일을 찾을 수도 있지만 `find`는 파일 이름만 사용합니다. 보다 자세한 비교는 [여기](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other)에서 확인할 수 있습니다.

## 코드 찾기

이름으로 파일을 찾는 것은 유용하지만, 당신은 자주 파일 **내용**을 기준으로 검색하는 경우가 있을 것입니다. 일반적인 시나리오는 패턴이 발생한 파일의 위치와 일부 내용(패턴)을 포함하여 모든 파일을 검색하는 것입니다. 이를 위해 대부분의 UNIX 계열 시스템은 입력에서 패턴을 일치시키는 일반적인 도구인 [grep](https://www.man7.org/linux/man-pages/man1/grep.1.html)을 제공합니다. `grep`은 믿을 수 없을 정도로 유용한 쉘 도구로, data wrangling 강의에서 더 자세히 다룰 것입니다. 

지금은 `grep`에 매우 다양한 도구를 제공하는 많은 플래그가 있음을 알고 있습니다. 제가 자주 사용하는 것들은 일치하는 줄 주위에 **C**ontext를 가져 오기 위해 `-C` 옵션을 사용하고, 일치하는 줄을 **V**erting하는 데 사용하기 위한 `-v`입니다. 즉, 일치하지 **않는** 모든 줄을 출력합니다. 예를 들어, `grep -C 5`는 일치하는 전 후 다섯 줄을 출력해줍니다. 많은 파일을 빠르게 검색할 때 `-R`을 사용하는 것이 좋습니다. 이는 **R**ecursively 하게 디렉토리로 이동하여 일치하는 문자열에 대한 파일을 검색하기 때문입니다.
 
그러나`grep -R`은 다중 CPU 지원, & c를 사용하여`.git` 폴더를 무시하는 등 여러가지 방법으로 개선할 수 있습니다. [ack](https://github.com/beyondgrep/ack3), [ag](https://github.com/ggreer/the_silver_searcher) 및 [rg](https://github.com/BurntSushi/ripgrep)를 포함한 많은 것들이 `grep`의 대안으로 개발되었습니다. `grep`의 대안으로 나온 저 모든 것들은 환상적이며 거의 동일한 기능을 제공합니다. 지금은 얼마나 빠르고 직관적인지를 고려할 때 ripgrep(`rg`)를 사용하고 있습니다. 몇 가지 예 :

```bash
# Find all python files where I used the requests library
rg -t py 'import requests'
# Find all files (including hidden files) without a shebang line
rg -u --files-without-match "^#\!"
# Find all matches of foo and print the following 5 lines
rg foo -A 5
# Print statistics of matches (# of matched lines and files )
rg --stats PATTERN
```

`find` /`fd`와 마찬가지로 이러한 문제는 이러한 도구 중 하나를 사용하여 빠르게 해결할 수 있는 게 중요하지, 어떤 걸 사용하는 지는 중요하지 않습니다.

## shell commands 찾기

지금까지 파일과 코드를 찾는 방법을 살펴보았지만, 셸을 더 많이 활용하기 시작하면 당신은 어느 시점에 입력한 특정 명령을 찾고 싶을 수 있습니다. 가장 먼저 알아야 할 것은 위쪽 화살표를 누르면 마지막에 쳤던 명령을 보여주고, 계속 누르면 쉘 히스토리를 천천히 살펴볼 수 있다는 것입니다.

`history` 명령을 사용하면 프로그래밍 방식으로 쉘 히스토리에 액세스 할 수 있습니다. 쉘 히스토리를 표준 출력으로 인쇄합니다. 그 히스토리에서 검색을 하고 싶다면 그 출력을 `grep` 으로 파이프하고 패턴을 검색할 수 있습니다. `history | grep find` 는 하위 문자열 "find"를 포함하는 명령을 인쇄합니다. 

대부분의 셸에서 `Ctrl + R`을 사용하여 history을 역방향으로 검색 할 수 있습니다. `Ctrl + R`을 누른 후 히스토리에있는 명령과 일치시킬 하위 문자열을 입력 할 수 있습니다. 계속 누르고 있으면 history에서 cycle을 순환합니다. [zsh](https://github.com/zsh-users/zsh-history-substring-search)에서 위쪽 / 아래쪽 화살표를 사용하여 활성화 할 수도 있습니다. `Ctrl + R` 에 추가 기능은 [fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r) 바인딩을 사용하여 제공됩니다. `fzf`는 많은 명령과 함께 사용할 수있는 범용 퍼지 파인더입니다. 이것은 당신의 history를 모호하게 일치시켜 결과를 좀 더 편리하고 시각적으로 즐거운 방식으로 제시해줍니다. 

내가 정말 좋아하는 또 다른 멋진 기록 관련 트릭은 **기록 기반 자동 제안** 입니다. [fish](https://fishshell.com/) 셸에서 처음 도입된 이 기능은 공통 접두사를 공유하는 가장 최근에 입력한 명령으로 현재 셸 명령을 동적으로 자동 완성합니다. [zsh](https://github.com/zsh-users/zsh-autosuggestions)에서 활성화 할 수 있으며 셸을 사용하는데 엄청난 도움이 됩니다. 

마지막으로 명심해야 할 점은 `.bashrc`에 `HISTCONTROL=ignorespace`를 설정하거나 `.zshrc`에 `setopt HIST_IGNORE_SPACE`를 설정하면, 선행 공백으로 시작하는 명령이 쉘 히스토리에 추가되지 않는다는 것입니다. 이것은 암호 또는 기타 민감한 정보로 명령을 입력할 때 유용합니다. 만약 선행 공백을 추가하지 않는 실수를 한 경우, 시작 부분을 수동으로 제거하고 `.bash_history` 또는 `.zsh_history`를 편집하면 됩니다. 

## 디렉토리 탐색

지금까지 이러한 작업을 수행하기 위해 필요한 위치에 이미 있다고 가정했습니다. 그러나 디렉토리를 빠르게 탐색하는 방법은 무엇입니까?
쉘 별칭을 작성하거나 [ln -s](https://www.man7.org/linux/man-pages/man1/ln.1.html)를 사용하여 심볼릭 링크를 만드는 등 이를 수행할 수 있는 간단한 방법이 많이 있습니다. 하지만 개발자들은 이미 매우 영리하고 정교한 솔루션을 만들어놓았습니다.

이 과정의 주제와 마찬가지로, 당신은 종종 일반적인 경우를 최적화하고자 할 것입니다. [fasd](https://github.com/clvv/fasd) 및 [autojump](https://github.com/wting/)와 같은 도구를 통해 자주 and / or 최근 파일과 디렉토리를 찾을 수 있습니다. Fasd는 [_frecency_](https://web.archive.org/web/20210421120120/https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm), 즉 _frequency_ 및 _recency_ 별로 파일 및 디렉토리의 순위를 매 깁니다.
기본적으로 `fasd` 는 _frecent_ 디렉토리의 하위 문자열을 사용하여 빠르게`cd`하는 데 사용할 수있는 `z` 명령을 추가합니다. 예를 들어`/ home / user / files / cool_project`로 자주 이동하는 경우 `z cool` 을 사용하여 바로 이동할 수 있습니다. autojump를 사용하면 `j cool` 을 사용하여 동일한 디렉토리 변경을 수행할 수 있습니다.

[tree](https://linux.die.net/man/1/tree), [broot](https://github.com/Canop/broot) 또는 [nnn](https://github.com/jarun/nnn) 또는 [ranger](https://github.com/ranger/ranger)와 같은 본격적인 파일 관리자 디렉토리 구조에 대한 개요를 신속하게 파악하기위한 더 복잡한 도구도 존재합니다. 

# 연습문제

1. [man ls](https://www.man7.org/linux/man-pages/man1/ls.1.html)를 읽고 다음과 같은 방식으로 파일을 나열하는`ls` 명령을 작성합니다.

    - 모든 파일 포함, 모든 숨겨진 파일 포함
    - 사이즈는 사람이 읽을 수 있을 법한 형식으로 (e.g. 454M instead of 454279954)
    - 최신순 파일 정렬
    - 색상화 되어 출력

    출력 예시는 아래와 같습니다. 

    ```
    -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
    drwxr-xr-x   5 user group  160 Jan 14 09:53 .
    -rw-r--r--   1 user group  514 Jan 14 06:42 bar
    -rw-r--r--   1 user group 106M Jan 13 12:12 foo
    drwx------+ 47 user group 1.5K Jan 12 18:08 ..
    ```

{% comment %}
ls -lath --color=auto
{% endcomment %}

1. 다음을 수행하는 bash 함수`marco` 및`polo`를 작성합니다.
`marco`를 실행할 때마다 현재 작업 디렉토리가 어떤 방식으로 저장되어야합니다. 그러면`polo`를 실행할 때 어떤 디렉토리에 있든 상관없이 `polo` 가`cd`를 수행해서 `marco` 를 실행한 디렉토리로 돌아갑니다.
디버깅을 쉽게 하기 위해 `marco.sh` 파일에 코드를 작성하고 `source marco.sh`를 실행하여 쉘에 정의를 (재)로드 할 수 있습니다.

{% comment %}
marco() {
    export MARCO=$(pwd)
}

polo() {
    cd "$MARCO"
}
{% endcomment %}

1. 거의 실패하지 않는 명령이 있다고 가정 해보십시오. 그것을 디버그하려고 출력을 캡처해야하지만, 실패를 실행하는 데 시간이 오래 걸릴 수 있습니다.
실패 할 때까지 표준 출력 및 오류 스트림을 파일로 캡처하고 마지막에 앞의 모든 것을 출력하는 bash 스크립트를 작성하십시오. 스크립트가 실패하는 데 걸린 실행 횟수도 보고할 수 있다면 보너스 포인트입니다.

    ```bash
    #!/usr/bin/env bash

    n=$(( RANDOM % 100 ))

    if [[ n -eq 42 ]]; then
       echo "Something went wrong"
       >&2 echo "The error was using magic numbers"
       exit 1
    fi

    echo "Everything went according to plan"
    ```

{% comment %}
#!/usr/bin/env bash

count=0
until [[ "$?" -ne 0 ]];
do
  count=$((count+1))
  ./random.sh &> out.txt
done

echo "found error after $count runs"
cat out.txt
{% endcomment %}

1. 강의에서 다루었 듯이`find`의`-exec`는 검색하는 파일에 대한 작업을 수행하는데 매우 강력합니다. 
그러나 zip 파일을 만드는 것과 같이 **모든** 파일로 작업을 수행하려면 어떻게해야합니까?
지금까지 본 것처럼 명령은 인수와 STDIN 모두에서 입력을 받습니다.
명령을 타이핑 할 때 STDOUT을 STDIN에 연결하지만 `tar`와 같은 일부 명령은 인수에서 입력을 받습니다.
이러한 문제를 해결하기 위해 STDIN을 인수로 사용하여 명령을 실행하는 [xargs](https://www.man7.org/linux/man-pages/man1/xargs.1.html) 명령이 있습니다.
예를 들어`ls | xargs rm`은 현재 디렉토리의 파일을 삭제합니다.

    당신의 임무는 폴더에서 모든 HTML 파일을 재귀적으로 찾아서 zip 파일을 만드는 명령을 작성하는 것입니다. 파일에 공백이 있어도 명령은 작동되어야 합니다. (hint: check `-d` flag for `xargs`)
    {% comment %}
    find . -type f -name "*.html" | xargs -d '\n'  tar -cvzf archive.tar.gz
    {% endcomment %}

    macOS에서는 기본 BSD `find`가 [GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands)에 포함된 버전과 다르다는 점에 주의하세요. `find`의 `-print0` 옵션과 `xargs`의 `-0` 옵션을 사용할 수 있습니다. macOS에 포함된 명령줄 도구는 GNU 버전과 다를 수 있으며, 원한다면 [brew를 사용하여](https://formulae.brew.sh/formula/coreutils) GNU 버전을 설치할 수 있습니다.

1. (고급) 명령 또는 스크립트를 작성하여 디렉토리에서 가장 최근에 수정된 파일을 재귀적으로 찾으시오. 그리고 모든 파일을 최신순으로 나열할 수 있습니까?
