---
layout: lecture
title: "Data Wrangling"
description: >
  sed, awk, 정규 표현식 같은 명령줄 도구로 데이터를 조작하고 변환하는 방법을 배웁니다.
thumbnail: /static/assets/thumbnails/2020/lec4.png
date: 2020-01-16
ready: true
video:
  aspect: 56.25
  id: sz_dsktIjt4
special: true
translation_status: complete
---

여러분은 아마 데이터의 한 형식을 다른 형식으로 변환하고 싶었던 적이 있었을 것입니다. 이번 강의는 이러한 주제를 다루고자 합니다. 특히, 데이터가 텍스트 형식이든 바이너리 형식이든, 여러분이 원하는 데이터가 될 때까지 데이터를 조작하는 방법을 다룹니다.

여러분은 이전의 강의에서 데이터를 다루는 기본적인 방법들을 배웠습니다. 여러분이 터미널에서 `|` 연산자(operator)를 사용할 때마다, 여러분은 일종의 데이터 조작(data wrangling)을 수행하는 것입니다. `journalctl | grep -i intel`과 같은 명령어는, (대소문자와 상관없이) 명시된 Intel의 모든 시스템 로그를 찾아냅니다. 아마 여러분은 이를 데이터를 조작하는 활동이라고 생각하지 않을 수 있지만, 이는 여러분의 시스템 로그 전체를 intel에 한정하여 가져와 유용하게 사용할 수 있다는 점에서, 데이터 조작이라고 할 수 있습니다. 대부분의 데이터 조작은 여러분의 목적에 적합한 도구에 대해 아는 것과, 이 둘을 결합하는 방법에 대한 것이라고 할 수 있습니다.

자, 이제 시작해보죠. 데이터를 조작하기 위해선 두 가지가 필요합니다. 바로 조작할 데이터와 이들을 가지고 무언가를 할 수 있는 도구입니다. 이전처럼 로그를 분석하는 사례가 도움이 될 것입니다. 왜냐하면 여러분이 로그 전체를 읽으면서 로그를 분석하는 일은 거의 불가능하기 때문입니다. 이제 서버 로그를 열어서 누가 서버에 접속하고자 했는지 확인해봅시다:

```bash
ssh myserver journalctl
```

여전히 너무 많은 결과가 나오네요. ssh에 관련된 내용만 나오도록 조정을 합시다:

```bash
ssh myserver journalctl | grep sshd
```

여기서 우리는  로컬 컴퓨터에서 `grep`을 통해 _원격_ 파일(_remote_ file)을 출력하고자 할 때, 파이프(pipe)를 사용한다는 점을 주목해야 합니다. `ssh`는 정말로 마법과도 같은데요, 이에 대해선  커맨드-라인 환경을 다루는 다음 강의에서 더 자세히 이야기 할 것입니다. 지금의 결과도 우리가 원하는 것보다 여전히 많은 내용을 담고 있군요. 그리고 읽기도 어렵습니다. 좀 더 나은 방법을 적용해보죠:

```bash
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' | less
```

추가적인 인용구(quoting)을 사용하는 이유는 무엇인가요? 음... 우리의 로그는 아마 꽤 클 것이기에, 컴퓨터로 하여금 그 모든 내용을 출력하고 필터링하는 일은 꽤나 낭비일 것입니다. 대신, 우리는 원격 서버에서 필터링을 하고, 데이터를 로컬에서 조작하는 일을 할 수 있습니다. `less`는 우리로 하여금 긴 출력을 위아래로 스크롤 할 수 있는 "페이저(pager)"를 제공합니다. 커맨드-라인을 디버그하는 동안 추가적인 트래픽을 절약하기 위해, 우리는 현재 필터링 된 로그를 파일에 넣음으로써, 다음 작업을 수행하는 동안 네트워크에 접근하지 않아도 됩니다:

```console
$ ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
$ less ssh.log
```

하지만 여전히 지저분하군요. 이를 처리하기 위한 _많은_ 방법이 존재하지만, 여러분이 가진 가장 강력한 도구, `sed`에 대해 알아봅시다.

`sed`는 옛날 `ed` 에디터 위해 구축된 "스트림 에디터"입니다. 이 에디터에서는, (여러분이 그렇게 할 수 있음에도 불구하고) 기본적으로 파일의 내용을 직접 조작하는 대신, 파일을 수정하는 방법에 대한 짧은 명령들을 제공합니다. 수많은 명령들이 있지만, 가장 자주 쓰이는 명령어로 `s`, 즉 치환(substitution)이 있습니다. 예를 들어, 우리는 다음과 같이 쓸 수 있을 것입니다:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

우리는 방금, 텍스트를 패턴에 따라 일치시킬 수 있는 강력한 단어인 _정규 표현식(regular expression)_을 작성했습니다. `s` 명령어는 다음의 형식을 가집니다: `s/REGEX/SUBSTITUTION/`. 여기서 `REGEX`는 여러분이 찾고자 하는 정규 표현식이며, `SUBSTITUTION`은 정규표현식에 일치하는 텍스트를 치환할 텍스트입니다.

(Vim [강의 노트](/2020/editors/#검색과-치환)의 "검색과 치환" 부분에서 본 문법일 것입니다! Vim은 검색과 치환에 `sed`의 치환 명령과 비슷한 문법을 사용합니다. 한 도구를 배우면 다른 도구를 더 능숙하게 사용하는 데도 도움이 됩니다.)

## 정규 표현식(Regular expressions)

정규 표현식은 자주 사용되고 유용하게 쓰이기에, 이들이 어떻게 작동하는지 이해하는 시간은 가치가 있을 것입니다. 위에서 우리가 방금 사용하였던 `/.*Disconnected from /` 부터 살펴봅시다. 정규 표현식은 (항상 그렇지는 않지만) 보통 `/`로 둘러싸여 있습니다. 대부분의 ASCII 문자들은 일반적인 의미를 지니고 있지만, 몇몇 문자들은 "특별한" 의미를 가집니다. 정확히는 어떤 문자가 정규 표현식의 구현에 따라 다소 차이가 나게 되는데, 이는 정규 표현식을 접하는 사람에게 커다란 장애물이 되어왔습니다. 자주 쓰이는 패튼들은 다음과 같습니다:

- `.`는 개행 문자(newline)를 제외한 "모든 한 글자"를 의미합니다
- `*`는 0개 이상의 일치 항목을 의미합니다
- `+`는 1개 이상의 일치 항목을 의미합니다
- `[abc]`는 `a`, `b`, 그리고 `c` 중 한 글자를 의미합니다
- `(RX1|RX2)`는 `RX1` 또는 `RX2`와 일치하는 문자열을 의미합니다
- `^`는 해당 줄의 시작을 의미합니다
- `$`는 해당 줄의 끝을 의미합니

`sed`의 정규 표현식은 조금 이상하고, 정규 표현식을 적용하기 위해선 앞에 `\`를 붙여야 합니다. `-E`를 붙이면 그런 작업은 필요하지 않지만요.

다시 `/.*Disconnected from /`로 돌아오면, 우리는 정규 표현식이 임의의 길이의 문자에 이어서 "Disconnected from"이라는 문자열을 가지는 텍스트와 일치한다는 점을 알 수 있습니다. 바로 우리가 원하던 결과네요. 하지만 조심해야 합니다. 정규 표현식은 까다로운 녀석입니다. 만약 누군가 "Disconnected from"이라는 유저 이름으로 로그인하는 경우에는 어떻게 될까요?

```
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
```

만약 이러한 경우라면 결과는 어떻게 될까요? `*`와 `+`는 기본적으로 "탐욕적(greedy)"입니다. 이들은 일치하는 텍스트를 가능한 많이 찾으려고 하죠. 그렇기에 위의 경우는, 아래와 같이 우리가 원하지 않은 결과가 나오게 됩니다:

```
46.97.239.16 port 55920 [preauth]
```

몇몇의 정규 표현식의 실행에 있어서, 여러분은 `*`와 `+`에 `?`를 붙여 탐욕적이지 않게 만들 수 있지만, 안타깝게도 `sed`는 이를 지원하지 않습니다. 대신 우리는 이러한 구성을_허용하는_ perl의 커맨드-라인 모드로 _전환할 수 있습니다_:

```bash
perl -pe 's/.*?Disconnected from //'
```

이번 강의에서는 `sed`를 계속 사용할 것입니다. 왜냐하면 `sed`는 이러한 종류의 작업에 흔히 쓰이는 도구이기 때문입니다. `sed`는 주어진 패턴과 일치하는 줄을 출력하거나, 호출된 만큼 여러번 치환할 수 있거나, 검색 등 기타 편리한 작업을 수행할 수 있습니다. 하지만 여기에서는 너무 많이 다루지는 않겠습니다. `sed`는 이번 강의 전체를 아우르는 기본 주제이지만, 이보다 더 나은 도구가 존재한다는 점 또한 중요합니다.

좋습니다, 우리는 없애야 할 접미사(suffix)를 가지게 되었습니다.이제  어떻게 해야 할까요? 유저 이름에 공백 등이 있을 경우, 유저 이름 뒤에 오는 텍스트만 일치시키는 일은 꽤 까다롭습니다. 우리가 해야 할 일은_전체_ 라인을 일치시키는 것입니다:

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//'
```

[regex debugger](https://regex101.com/r/qqbZqh/2)에서 어떤 일이 일어나는지 확인해보세요. 정규 표현식의 시작은 이전과 같지만, 유저의 모든 변형 요소들을 포함합니다(로그를 보면 두 가지 접두사가 있습니다). 그런 다음 유저 이름에 해당하는 모든 문자를 일치시킵니다. 이후 모든 단일 문자와 (`[^ ]+`; 공백을 포함하지 않는 비어있지 않은 모든 문자열), 단어 "port" 뒤에 나오는 일련의 숫자를 일치시키고, 접미사가 `[preauth]`인 가능성을 포함하여 줄이 끝남을 표현합니다. 이제 유저 이름이 "Disconnected from"인 경우는 더 이상 문제가 되지 않게 됩니다. 여러분은 왜 그런지 아시겠나요?

이러한 기술에도 불구하고, 여전히 한 가지 문제점이 남아있는데, 바로 모든 로그가 비어버리게 된다는 것입니다. 우리는 결국 유저 이름이 _유지_되기를 원합니다. 이를 위해, 우리는 "그룹화 구문(capture groups)"을 사용할 수 있습니다. 괄호로 둘러싸여 정규식에 일치하는 모든 텍스트는 번호가 매겨진 그룹에 저장됩니다. 이는 `\1`, `\2`, `\3`과 같이, 치환 작업에서도 사용할 수 있습니다(그리고 몇몇 엔진에서는 패턴 그 자체로 사용할 수도 있습니다):

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

아마 여러분은 _정말로_ 복잡한 정규 표현식을 마주하게 되는 상상을 할 수 있을 것입니다. 예를 들어, 정규 표현식으로 [e-mail 주소](https://www.regular-expressions.info/email.html)를 일치시키는 방법은 이와 같습니다. [정말 어렵지요](https://web.archive.org/web/20221223174323/http://emailregex.com/). 이에 대해 [많은 논의들이 있습니다](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982). 그리고 사람들은 이를 위한 [테스트](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php)를 작성하기도 하였습니다. [테스트 행렬도요](https://mathiasbynens.be/demo/url-regex). 여러분은 주어진 숫자가 [소수인지](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/). 확인하는 정규식을 작성할 수도 있습니다.

정규 표현식은 사용하기 어렵기로 악명이 높지만, 그만큼 굉장히 편리한 도구이므로, 여러분의 도구 상자에 넣어 사용할 만한 충분한 가치가 있습니다.

## 데이터 조작으로 돌아와서(back to data wrangling)

좋습니다, 이제 우리는 아래의 명령어를 가지게 되었군요:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

`sed`는 이외에도 흥미로운 여러가지 일들을 할 수 있습니다. 가령 (`i` 명령어를 이용하여) 텍스트를 주입하거나, (`p` 명령어를 이용하여) 명확하게(explicitly) 줄을 출력하거나, 인덱스를 이용하여 줄을 선택하거나, 등등 다양한 작업들이 있습니다. `man sed`를 확인해보세요!

아무튼, 이제 우리는 로그인을 시도한 모든 유저의 이름으로 구성된 목록을 가집니다. 하지만 상당히 보기 불편한 결과로군요. 조금 더 일반적으로 보이도록 만들어봅시다:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
```

`sort`는, 음... 주어진 입력을 정렬할 것입니다. `uniq -c`는 모든 단일한 행을 대상으로, 중복되는 수를 앞에 붙여 출력하고, 중복되는 행들은 하나의 행으로 축약됩니다. 우리는 여전히 이 출력을 정렬하여, 가장 빈번한 유저 이름만을 가지길 원할 수도 있습니다:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
```

`sort -n`은 (사전 순이 아닌) 숫자 순서대로 정렬합니다. `-k1,1`은 "공백으로 구분된 첫 번째 열을 기준으로 정렬할 것"을 의미합니다. `,n` 부분은 "기본값은 행의 끝인, n번째 필드까지 정렬할 것"을 의미합니다. 이 _특정한_ 예제에선, 전체 행을 정렬하는 것은 문제가 되지 않습니다. 하지만 우리는 여기 배우러 온 것이기에, 더 자세히 들어가봅시다!

만약 우리가 _가장 흔하지 않은_ 결과를 찾고자 한다면, 우리는 `tail` 대신 `head`를 사용할 수 있지만, 역순으로 정렬을 가능하게 하는 `sort -r`을 사용할 수도 있습니다.

좋습니다, 이쯤 되어도 꽤 훌륭한 것 같네요. 하지만 설정 파일 등에 사용하기 위해 유저 이름만 추출하여, 한 줄에 하나씩 출력하는 대신 쉼표로 구분된 목록으로 만드는 방법은 어떨까요?

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd,
```

macOS를 사용한다면 위 명령이 macOS에 포함된 BSD `paste`에서는 작동하지 않는다는 점에 주의하세요. BSD와 GNU coreutils의 차이 및 macOS에 GNU coreutils를 설치하는 방법은 [셸 도구 강의의 연습 문제 4번](/2020/shell-tools/#연습문제)을 참고하세요.

`paste`에 대해 알아보는 것부터 시작하죠. 이는 여러분으로 하여금 주어진 단일 문자 구분 기호(`-d`, 여기서는 `,`)로 행을 결합(`-s`)할 수 있게 합니다. 그런데 여기서 `awk`는 어떤 일을 하는 것일까요?

## awk -- 또 다른 에디터

`awk`은 텍스트 스트림 처리에 아주 좋은 프로그래밍 언어입니다. 여러분이 `awk`를 제대로 배우고자 한다면 알려드려야 할 것들이 _정말_ 많지만, 이번 강의에서 늘 그래왔듯, 우리는 기본적인 내용을 다룰 것입니다.

먼저, `{print $2}`가 하는 일은 무엇일까요? 음... `awk` 프로그램은 패턴이 주어진 행과 일치할 경우 어떻게 해야 하는지 알려주는 블럭(block)과 선택적인 패턴(optional pattern)의 형태를 가집니다. (위에서 우리가 사용한 것과 같은) 기본적인 패턴은 모든 행과 일치합니다. 블럭 안을 들여다보면, `$0`은 모든 행의 내용으로 설정되고, `$1`부터 `$n`까지는 해당 행의 `$n`번째 _필드_로 설정됩니다. 이들은  `awk`의 필드 구분자(separator)로 구분됩니다(기본적으로 공백이 구분자로 지정되지만, `-F`를 이용하여 변경할 수 있습니다). 이 경우, 우리는 모든 행에 대해, 유저 이름이 되는 두 번째 필드의 내용을 인쇄하도록 요청하였습니다!

이를 좀 더 멋지게 처리할 수 있을지 살펴보죠. `c`로 시작하여 `e`로 끝나고, 단 한 번만 사용된 유저 이름이 있는지 계산해봅시다.

```bash
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l
```

풀어야 할 내용들이 많군요. 먼저, 이제 우리는 (`{...}` 이전에 오는 것들인) 패턴을 가짐을 주목해야 합니다. 해당 패턴은 (`uniq -c`에 의해 계산된) 첫 번째 필드가 1과 일치해야 하고, 두 번째 필드는 주어진 정규 표현식과 일치해야 함을 나타냅니다. 그리고 블럭은 오직 유저 이름을 출력하라고 말합니다. 그리고 나서 `wc -l`을 이용하여 출력된 행의 개수를 셉니다.

그나저나, `awk`가 프로그래밍 언어라는 사실을 기억하시나요?

```awk
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

`BEGIN`은 입력의 시작과 일치하는 패턴입니다(그리고 `END`는 끝과 일치합니다). 이제 블럭은 행마다 첫 번째 필드의 숫자를 세어서(비록 이 경우에는 언제나 1이겠지만요) 마지막에 출력합니다. 사실, 우리는 `grep`과 `sed`를 완전히 _제거할 수_ 있는데, 왜냐하면, `awk`는 [이 모든 것을 다 할 수 있기 때문입니다](https://web.archive.org/web/20251210045942/https://backreference.org/2010/02/10/idiomatic-awk/). 이에 대해선 독자의 연습문제로 남겨놓도록 하겠습니다.

## 데이터 분석하기

표준 입력(STDIN)을 읽을 수 있는 계산기 `bc`를 사용하면 셸에서 직접 계산할 수 있습니다! 예를 들어, 각 행의 숫자를 `+`로 이어 붙여 모두 더하는 식은 다음과 같습니다:

```bash
 | paste -sd+ | bc -l
```

또는 보다 정교한 표현으로:

```bash
echo "2*($(data | paste -sd+))" | bc -l
```

여러분은 다양한 방법으로 통계를 적용할 수 있습니다. [`st`](https://github.com/nferraz/st)는 꽤 멋진 도구인데, 만약 여러분이 이미 [R](https://www.r-project.org/)을 가지고 있다면 다음과 같이 적용할 수 있습니다:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --no-echo -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
```

R은 데이터 분석과 [그래프 표현(plotting)](https://ggplot2.tidyverse.org/)에 특화된 또 다른 (기묘한) 프로그래밍 언어입니다. 여기서는 자세한 내용을 다루지 않겠지만, 우리는 입력으로 받은 숫자 스트림을 담은 벡터를 만들었고, `summary`는 벡터에 대해 요약된 통계를 출력하여, 결과적으로 R은 우리가 원하는 통계를 제공하였다는 점을 다루고 넘어가면 충분할 것 같습니다.

만약 여러분이 간단한 그래프 표현을 원한다면, `gnuplot`은 여러분의 친구가 될 것입니다:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | gnuplot -p -e 'set boxwidth 0.5; plot "-" using 1:xtic(2) with boxes'
```

## 인자를 만들기 위한 데이터 조작 (Data wrangling to make arguments)

때떄로 여러분은 긴 목록을 바탕으로 하여 제거하거나 설치할 항목을 찾기 위해 데이터 조작을 수행하기를 원하는 경우가 있을 것입니다. 앞서 이야기하였던 내용들과 `xargs`의 조합은 굉장히 강력한 콤보가 될 수 있습니다.

예를 들어, 강의에서 본 것처럼 다음 명령으로 시스템에서 오래된 Rust nightly 빌드를 제거할 수 있습니다. 데이터 조작 도구로 오래된 빌드 이름을 추출한 뒤, `xargs`로 제거 명령에 전달합니다:

```bash
rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

## 이진 데이터 조작하기 (Wrangling binary data)

지금까지 우리는 텍스트 데이터를 조작하는 방법에 대해 이야기해왔습니다. 하지만 파이프는 이진 데이터를 처리하는데에도 유용합니다. 예를 들어, 우리는 ffmpeg를 사용하여 카메라로부터 이미지를 포착하고, 이를 흑백으로 변환하고, 압축하고, SSH를 통해 원격 장치로 전송하고, 그곳에서 압축해제하고, 복사본을 만들어, 나타낼 수 있습니다.

```bash
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.jpg | env DISPLAY=:0 feh -'
```

# 연습문제

1. [짧은 반응형 정규식 튜토리얼](https://regexone.com/)을 완수하세요.
2. (`/usr/share/dict/words` 안에서) 적어도 세 개의 `a`가 포함되고, `s`로 끝나지 않는 단어의 개수를 찾아보세요. 그러한 단어들의 마지막 두 글자로 무엇이 가장 흔한가요? `sed`의 `y` 명령어, 또는 `tr` 프로그램은 대소문자 구분 문제를 해결하는데 도움이 될 것입니다. 그러한 두 글자 조합은 얼마나 존재하나요? 그리고 도전문제로: 두 글자 조합 중 등장하지 않는 조합은 무엇이 있나요?
3. `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`와 같은 in-place한 치환 작업은 꽤 유혹적이지만, 그리 좋지 않은 아이디어입니다. 왜 그럴까요? 이는 `sed`의 사용에 있어서만 그런 것일까요? 이를 위해 `man sed`를 이용하여 어떻게 더 나은 방법으로 해결할 수 있는지 찾아보세요.
4. 최근 10번의 부팅을 가지고 시스템 부팅 시간의 평균, 중앙값, 그리고 최대값을 찾아보세요. Linux의 경우 `journalctl`을, macOS의 경우 `log show`를 이용하면 됩니다. 그리고 각 부팅의 시작과 끝 부분에서 로그 타임스탬프를 찾아보세요. Linux의 경우, 이는 다음과 같이 생겼을 것입니다:
```
Logs begin at ...
```
and
```
systemd[577]: Startup finished in ...
```
macOS의 경우는, [다음을 살펴보세요](https://eclecticlight.co/2018/03/21/macos-unified-log-3-finding-your-way/):
```
=== system boot:
```
and
```
Previous shutdown cause: 5
```
5. 이전 세 번의 재부팅 간에 공유되지 _않은_ 부팅 메세지를 찾아보세요(`journalctl`의 `-b` 플래그를 참조하세요). 이 작업을 여러 단계로 나눕니다. 먼저, 지난 세 번의 부팅에₩서 나온 로그만 얻을 수 있는 방법을 찾아보세요. 여러분이 부팅 로그를 추출하는데 사용하는 도구에 적용 가능한 플래그를 적용하거나, `sed '0,/STRING/d'`를 사용하여 `STRING`과 일치하는 모든 행을 지우는 방법을 적용할 수 있을 것입니다. 다음으로, (타임스탬프와 같이) _항상_ 달라지는 행의 모든 부분을 제거하세요. 그런 다음, 각 입력 행의 중복의 정도를 세면서, 행의 중복을 제거합니다(`uniq`는 여러분의 친구라는 점을 잊지 마세요). 마지막으로, 중복의 수가 3 이상인 모든 행을 제거합니다(왜냐하면 이는 모든 부팅에서 _공유되었음을_ 의미하기 떄문입니다).
6. [이것이나](https://commons.wikimedia.org/wiki/Data:Wikipedia_statistics/data.tab) [요것](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1), 아니면 [여기에 있는 데이터](https://www.springboard.com/blog/data-science/free-public-data-sets-data-science-project/)와 같은 온라인 데이터 셋을 찾아보세요. `curl`을 사용하여 데이터를 가져와(fetch) 두 개의 숫자 데이터 열만 추출합니다. 만약 여러분이 HTML 데이터를 가져오는 경우, [`pup`](https://github.com/EricChiang/pup)이 아마 도움이 될 것입니다. JSON 데이터인 경우, [`jq`](https://stedolan.github.io/jq/)를 사용해보세요. 단일 명령으로 하나의 열의 최소값과 최대값을 찾고, 다른 명령으로 각 열의 합 사이의 차이를 구해보세요.
