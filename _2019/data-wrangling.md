---
layout: lecture
title: "데이터 랭글링"
presenter: Jon
date: 2019-01-17
order: 2
video:
  aspect: 56.25
  id: VW2jn9Okjhw
translation_status: draft
---

텍스트를 잔뜩 놓고 무언가 해 보고 싶었던 적이 있나요?
좋습니다. 그게 바로 데이터 랭글링입니다!
구체적으로는 원하는 결과를 정확히 얻을 때까지
데이터를 한 형식에서 다른 형식으로 바꾸는 일입니다.

우리는 이미 기본적인 데이터 랭글링을 보았습니다. `journalctl | grep -i intel`입니다.
 - Intel을 언급하는 모든 시스템 로그 항목을 찾습니다(대소문자 구분 없음).
 - 사실 데이터 랭글링은 대부분 어떤 도구가 있는지,
   어떻게 조합하는지 아는 일입니다.

처음부터 시작해 봅시다. 데이터 소스와 그 데이터로 할 일이 필요합니다.
로그는 좋은 활용 사례가 되는 경우가 많습니다. 로그에 관해 무언가 조사하고 싶은 일이
자주 있고, 전체를 다 읽는 것은 현실적으로 불가능하기 때문입니다.
제 서버의 로그를 살펴보면서 누가 제 서버에 로그인하려고 하는지
알아봅시다.

```bash
ssh myserver journalctl
```

양이 너무 많습니다. ssh 관련 내용으로 한정해 봅시다.

```bash
ssh myserver journalctl | grep sshd
```

파이프를 사용해 *원격* 파일을 로컬 컴퓨터의 `grep`으로 스트리밍하고 있다는 점에
주목하세요! `ssh`는 마법 같습니다. 하지만 여전히 우리가 원한 것보다
양이 훨씬 많습니다. 읽기도 꽤 어렵습니다. 더 나아지게 해 봅시다.

```bash
ssh myserver journalctl | grep sshd | grep "Disconnected from"
```

여전히 불필요한 내용이 많습니다. 이를 없애는 방법은 *아주 많지만*,
여러분의 도구 상자에서 가장 강력한 도구 중 하나인
`sed`를 살펴봅시다.

`sed`는 오래된 `ed` 편집기를 기반으로 만든 "스트림 편집기"입니다.
기본적으로 내용을 직접 조작하는 대신(물론 그렇게 할 수도 있습니다),
파일을 어떻게 수정할지 짧은 명령어를
입력합니다. 명령어는 아주 많지만, 가장 흔한 것 중 하나는 `s`,
즉 치환입니다. 예를 들어 다음과 같이 쓸 수 있습니다.

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

방금 우리가 쓴 것은 간단한 *정규 표현식*입니다. 텍스트를 패턴과 대조해
일치시킬 수 있는 강력한 구성 요소입니다. `s` 명령어는
`s/REGEX/SUBSTITUTION/` 형식으로 작성합니다. `REGEX`는
검색하려는 정규 표현식이고, `SUBSTITUTION`은
일치하는 텍스트를 바꿀 텍스트입니다.

## 정규 표현식

정규 표현식은 충분히 흔하고 유용하므로, 시간을 들여 동작 방식을
이해할 가치가 있습니다. 위에서 사용한 표현식인
`/.*Disconnected from /`부터 살펴봅시다. 정규 표현식은
보통(항상 그런 것은 아니지만) `/`로 둘러쌉니다. 대부분의 ASCII 문자는
그대로의 의미를 가지지만, 일부 문자는 "특별한"
일치 동작을 합니다. 정확히 어떤 문자가 어떤 일을 하는지는
정규 표현식 구현마다 다소 다르며, 이 때문에 무척 답답할 때가
많습니다. 매우 흔한 패턴은 다음과 같습니다.

 - `.`은 줄바꿈을 제외한 "임의의 한 문자"를 뜻합니다.
 - `*`는 앞에 오는 일치 항목이 0개 이상임을 뜻합니다.
 - `+`는 앞에 오는 일치 항목이 1개 이상임을 뜻합니다.
 - `[abc]`는 `a`, `b`, `c` 중 임의의 한 문자입니다.
 - `(RX1|RX2)`는 `RX1`이나 `RX2`에 일치하는 것입니다.
 - `^`는 줄의 시작입니다.
 - `$`는 줄의 끝입니다.

`sed`의 정규 표현식은 조금 특이해서, 이 문자들 대부분이 특별한
의미를 갖게 하려면 앞에 `\`를 붙여야 합니다. 또는
`-E`를 전달할 수도 있습니다.

그럼 `/.*Disconnected from /`을 다시 보면, 임의의 개수의 문자로
시작하고 그 뒤에 리터럴 문자열 "Disconnected from "이 오는
텍스트와 일치한다는 것을 알 수 있습니다. 우리가 원한 대로입니다. 하지만
주의하세요. 정규 표현식은 까다롭습니다. 누군가 "Disconnected from"이라는
사용자 이름으로 로그인을 시도한다면 어떨까요? 다음과 같은 로그가 생깁니다.

```
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
```

결국 어떤 결과가 나올까요? `*`와 `+`는 기본적으로 "탐욕적"입니다.
최대한 많은 텍스트와 일치하려고 합니다. 따라서 위의 경우에는
다음 내용만 남게 됩니다.

```
46.97.239.16 port 55920 [preauth]
```

우리가 원한 결과가 아닐 수도 있습니다. 일부 정규 표현식
구현에서는 `*`나 `+` 뒤에 `?`를 붙이기만 하면
탐욕적이지 않게 만들 수 있지만, 아쉽게도 `sed`는 이를 지원하지 않습니다.
하지만 이 구문을 *지원하는* perl의 명령줄 모드로 바꿀 *수는* 있습니다.

```bash
perl -pe 's/.*?Disconnected from //'
```

그래도 앞으로는 계속 `sed`를 사용하겠습니다. 이런 종류의 작업에
훨씬 더 흔하게 쓰이는 도구이기 때문입니다. `sed`는 주어진 패턴에 일치하는 줄 뒤의
줄들을 출력하거나, 한 번 실행하여 치환을 여러 번 하거나, 무언가를 검색하는 등
다른 편리한 일도 할 수 있습니다. 하지만 여기서는 그런 기능을 많이 다루지는
않습니다. `sed` 자체만으로도 사실 하나의 주제이지만,
더 나은 도구가 있는 경우가 많습니다.

좋습니다. 이제 없애고 싶은 접미사도 있습니다. 어떻게 하면
될까요? 특히 사용자 이름에 공백 같은 것이 들어갈 수 있다면,
사용자 이름 뒤에 오는 텍스트만 일치시키는 일은 조금 까다롭습니다!
우리가 해야 할 일은 줄 *전체*를 일치시키는 것입니다.

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//'
```

[정규 표현식 디버거](https://regex101.com/r/qqbZqh/2)로 무슨 일이 일어나는지
살펴봅시다. 좋습니다. 시작 부분은 여전히 이전과 같습니다.
그다음에는 "user"의 여러 형태 중 하나와 일치시킵니다(로그에는
접두사가 두 개 있습니다). 그다음에는 사용자 이름 자리에 있는 임의의
문자열과 일치시킵니다. 그다음에는 단어 하나와
일치시킵니다(`[^ ]+`; 공백이 아닌 문자가 하나 이상 이어지는 것).
그다음에는 "port"라는 단어와 그 뒤에 이어지는 숫자들과 일치시킵니다.
그 뒤에는 접미사 ` [preauth]`가 있을 수도 있고, 그다음은 줄의 끝입니다.

이 방법을 사용하면 사용자 이름이 "Disconnected from"이어도
더는 혼란을 일으키지 않는다는 점에 주목하세요. 이유를 알겠나요?

하지만 한 가지 문제가 있습니다. 로그 전체가 비어 버린다는
것입니다. 결국 사용자 이름은 *유지하고* 싶습니다. 이를 위해
"캡처 그룹"을 사용할 수 있습니다. 괄호로 둘러싼 정규 표현식과 일치하는
텍스트는 번호가 매겨진 캡처 그룹에 저장됩니다. 치환 부분에서
(일부 엔진에서는 패턴 자체에서도!) `\1`, `\2`, `\3` 등으로
사용할 수 있습니다. 따라서 다음과 같이 합니다.

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

아마 짐작할 수 있겠지만, *정말* 복잡한 정규 표현식을
만들 수도 있습니다. 예를 들어 [이메일 주소](https://www.regular-expressions.info/email.html)를
일치시키는 방법을 다룬 글이 있습니다.
[쉽지 않습니다](https://web.archive.org/web/20221223174323/http://emailregex.com/). 그리고 [논의도 많습니다](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982).
사람들이 [테스트를 작성하기도 했습니다](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php).
[테스트 행렬](https://mathiasbynens.be/demo/url-regex)도 있습니다.
심지어 주어진 숫자가 [소수인지](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/)
판별하는 정규 표현식도 작성할 수 있습니다.

정규 표현식은 올바르게 작성하기 어렵기로 악명이 높지만,
도구 상자에 넣어 두면 아주 유용하기도 합니다!

## 데이터 랭글링으로 돌아가기

좋습니다. 이제 다음과 같은 코드가 있습니다.

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

`sed`만으로 할 수도 있지만, 굳이 왜 그럴까요? 재미있으니까요.

```bash
ssh myserver journalctl
 | sed -E
   -e '/Disconnected from/!d'
   -e 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

여기서 `sed`의 기능을 일부 볼 수 있습니다. `sed`는 텍스트를 삽입하거나
(`i` 명령어), 줄을 명시적으로 출력하거나(`p` 명령어),
인덱스로 줄을 선택하는 등 다른 일도 많이 할 수 있습니다. `man sed`를 확인하세요!

아무튼, 지금 만든 코드는 로그인을 시도한 모든 사용자 이름의
목록을 제공합니다. 하지만 이것만으로는 별로 도움이 되지 않습니다. 흔히 나오는
이름을 찾아봅시다.

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
```

`sort`는 말 그대로 입력을 정렬합니다. `uniq -c`는 같은 내용이
연속으로 나오는 줄들을 하나로 합치고, 앞에 등장 횟수를
붙입니다. 아마 이것도 정렬한 다음
가장 흔한 로그인만 남기고 싶을 것입니다.

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
```

`sort -n`은 (사전순 대신) 숫자순으로 정렬합니다. `-k1,1`은
"공백 문자로 구분된 첫 번째 열만으로 정렬하라"는 뜻입니다. `,n`
부분은 "`n`번째 필드까지 정렬하라"는 뜻이며, 기본값은 줄의
끝입니다. 이 *특정한* 예에서는 줄 전체로 정렬해도
상관없겠지만, 우리는 배우러 왔잖아요!

*가장 드물게* 나오는 이름을 원한다면 `tail` 대신 `head`를
사용할 수 있습니다. 역순으로 정렬하는 `sort -r`도 있습니다.

좋습니다. 꽤 멋지지만, 사용자 이름만 출력하고 싶고,
한 줄에 하나씩 나오지 않게 하고 싶을 수도 있겠죠?

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd,
```

`paste`부터 시작합시다. 주어진 한 문자 구분자(`-d`)로
줄들을 합칠 수 있습니다(`-s`). 그런데 이 `awk`라는 것은 뭘까요?

## awk -- 또 다른 편집기

`awk`는 마침 텍스트 스트림을 처리하는 데 아주 뛰어난
프로그래밍 언어입니다. 제대로 배우려면 `awk`에 관해 할 말이 *아주 많지만*,
여기서 다루는 다른 여러 도구와 마찬가지로
기초만 살펴보겠습니다.

먼저 `{print $2}`는 무슨 일을 할까요? `awk` 프로그램은
선택적인 패턴과, 그 패턴이 주어진 줄에 일치하면 무엇을 할지 정하는
블록으로 이루어집니다. 기본 패턴(위에서 사용한 것)은 모든 줄과
일치합니다. 블록 안에서는 `$0`에 줄 전체의 내용이 설정되고,
`$1`부터 `$n`에는 `awk` 필드 구분자(기본값은 공백 문자이며,
`-F`로 바꿀 수 있습니다)로 나누었을 때 그 줄의 첫 번째부터 `n`번째
*필드*가 각각 설정됩니다. 이 경우에는 각 줄에서 두 번째 필드의
내용을 출력하라고 하고 있으며, 그 필드가 바로 사용자 이름입니다!

좀 더 멋진 일을 할 수 있는지 봅시다. `c`로 시작하고 `e`로 끝나며
한 번만 사용된 사용자 이름의 개수를 계산해 봅시다.

```bash
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l
```

여기에는 짚어 볼 내용이 많습니다. 먼저, 이제 패턴이 있다는 점에
주목하세요(`{...}` 앞에 오는 부분입니다). 패턴은 줄의 첫 번째
필드가 1과 같아야 하고(`uniq
-c`가 구한 개수입니다), 두 번째 필드가 주어진 정규 표현식과
일치해야 한다고 합니다. 블록은 사용자 이름을 출력하라고만 합니다.
그다음 `wc -l`로 출력의 줄 수를 셉니다.

하지만 `awk`가 프로그래밍 언어라는 점을 기억하나요?

```awk
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

`BEGIN`은 입력의 시작과 일치하는 패턴입니다(`END`는 끝과
일치합니다). 이제 각 줄에 실행되는 블록은 첫 번째 필드의 개수를
더하기만 하고(이 경우에는 항상 1이겠지만), 마지막에 그 값을
출력합니다. 사실 `awk`는 [모든 것을 할 수 있으므로](https://web.archive.org/web/20251210045942/https://backreference.org/2010/02/10/idiomatic-awk/)
`grep`과 `sed`를 완전히 없앨 *수도* 있지만,
독자 여러분의 연습 문제로 남겨 두겠습니다.

## 데이터 분석하기

수학 계산도 할 수 있습니다!

```bash
 | paste -sd+ | bc -l
```

```bash
echo "2*($(data | paste -sd+))" | bc -l
```

다양한 방법으로 통계를 구할 수 있습니다.
[`st`](https://github.com/nferraz/st)는 꽤 괜찮지만, 이미
R이 설치되어 있다면 다음과 같이 할 수 있습니다.

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --slave -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
```

R은 데이터 분석과 [그래프 그리기](https://ggplot2.tidyverse.org/)에
뛰어난 또 다른 (이상한) 프로그래밍 언어입니다. 너무 자세히
들어가지는 않지만, `summary`는 행렬의 요약 통계를 출력하며,
우리는 숫자 입력 스트림에서 행렬을 계산했으므로
R이 원하는 통계를 제공해 준다고 해 두면 충분합니다!

간단한 그래프만 그리고 싶다면 `gnuplot`이 도움이 됩니다.

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | gnuplot -p -e 'set boxwidth 0.5; plot "-" using 1:xtic(2) with boxes'
```

## 인자를 만들기 위한 데이터 랭글링

긴 목록을 바탕으로 설치하거나 제거할 것을 찾기 위해
데이터 랭글링을 하고 싶을 때도 있습니다. 지금까지 이야기한
데이터 랭글링과 `xargs`는 강력한 조합이 될 수 있습니다.

```bash
rustup toolchain list | grep nightly | grep -vE "nightly-x86|01-17" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

# 연습 문제

1. 정규 표현식이 익숙하지 않다면,
   기본적인 내용 대부분을 다루는 짧은 대화형 튜토리얼이
   [여기](https://regexone.com/) 있습니다.
1. `sed s/REGEX/SUBSTITUTION/g`는 일반적인 sed와 어떻게 다른가요?
   `/I`나 `/m`은 어떤가요?
1. 원래 파일에 직접 치환하려고 할 때
   `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt` 같은 코드를 쓰고 싶어지기 쉽습니다.
   하지만 이것은 좋지 않은 생각입니다. 왜일까요? `sed`에만 해당하는 문제인가요?
1. 익숙한 언어에서 정규 표현식을 사용해 간단한 grep과 같은 도구를 구현하세요. grep처럼 출력에 색상을 강조하고 싶다면 ANSI 색상 이스케이프 시퀀스를 검색해 보세요.
1. 파일 이름 바꾸기 같은 작업은 `mv` 같은 기본 명령어로 하기 까다로울 때가 있습니다. `rename`은 이런 작업을 하기 좋은 도구이며 sed와 비슷한 문법을 사용합니다. 이름에 공백이 있는 파일을 여러 개 만든 뒤 `rename`으로 공백을 밑줄로 바꿔 보세요.
1. 지난 세 번의 재부팅 사이에서 공통으로 나타나지 *않는* 부팅 메시지를
   찾아보세요(`journalctl`의 `-b` 플래그를 보세요). 모든 부팅 로그를
   한 파일에 합쳐 놓으면 일이 더 쉬워질
   수도 있습니다.
1. 다음 메시지의 로그 타임스탬프를 사용하여
   최근 열 번의 부팅에 걸친 시스템 부팅 시간에 관한
   통계를 구하세요.
   ```
   Logs begin at ...
   ```
   그리고
   ```
   systemd[577]: Startup finished in ...
   ```
1. (`/usr/share/dict/words`에서) `a`가 세 개 이상 들어 있고
   `'s`로 끝나지 않는 단어의 수를 구하세요. 그 단어들의
   마지막 두 글자 중 가장 흔한 세 가지는 무엇인가요? `sed`의 `y` 명령어나
   `tr` 프로그램이 대소문자를 구분하지 않는 데 도움이 될 수 있습니다.
   그런 두 글자 조합은 몇 개 있나요? 그리고 도전 문제입니다.
   나타나지 않는 조합은 무엇인가요?
1. [이 데이터 세트](https://commons.wikimedia.org/wiki/Data:Wikipedia_statistics/data.tab)나
   [이 데이터 세트](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1) 같은 온라인 데이터 세트를 찾으세요.
   [여기](https://www.springboard.com/blog/data-science/free-public-data-sets-data-science-project/)에서
   다른 것을 찾아도 좋습니다.
   `curl`로 가져온 다음 숫자 데이터가 있는 열 두 개만
   추출하세요. HTML 데이터를 가져온다면
   [`pup`](https://github.com/EricChiang/pup)이 도움이 될 수 있습니다. JSON
   데이터에는 [`jq`](https://stedolan.github.io/jq/)를 써 보세요. 하나의 명령어로 한 열의
   최솟값과 최댓값을 구하고, 다른 명령어로 두 열 사이의 차이를
   합한 값을 구하세요.
