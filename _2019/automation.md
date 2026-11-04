---
layout: lecture
title: "자동화"
presenter: Jose
date: 2019-01-24
order: 3
video:
  aspect: 56.25
  id: BaLlAaHz-1k
special: true
translation_status: draft
---

어떤 일을 하는 스크립트를 작성한 뒤 주기적으로 실행하고 싶을 때가 있습니다. 백업 작업이 그 예입니다. 백그라운드에서 실행되다가 주기적으로 동작하는 *임시방편*을 직접 작성할 수도 있습니다. 하지만 대부분의 UNIX 시스템에는 간단한 규칙에 따라 최대 1분에 한 번의 빈도로 작업을 실행할 수 있는 cron 데몬이 포함되어 있습니다.

대부분의 UNIX 시스템에서는 cron 데몬인 `crond`가 기본적으로 실행되고 있지만, 언제든 `ps aux | grep crond`로 확인할 수 있습니다.

## crontab

cron의 설정 파일은 `crontab -l`을 실행해 표시하고 `crontab -e`를 실행해 편집할 수 있습니다. cron이 사용하는 시간 형식은 공백으로 구분한 필드 다섯 개와 사용자, 명령어로 이루어집니다.

- **minute** - 명령어를 매시 몇 분에 실행할지를 정하며, '0'에서 '59' 사이입니다.
- **hour** - 명령어를 몇 시에 실행할지를 정하며, 24시간제로 지정합니다. 값은 0에서 23 사이여야 합니다(0은 자정입니다).
- **dom** - 명령어를 실행하려는 날짜입니다. 예를 들어 매월 19일에 명령어를 실행하려면 dom은 19가 됩니다.
- **month** - 지정한 명령어를 실행할 달입니다. 숫자(0-12)나 달의 이름(예: May)으로 지정할 수 있습니다.
- **dow** - 명령어를 실행하려는 요일입니다. 숫자(0-7)나 요일 이름(예: sun)으로 지정할 수 있습니다.
- **user** - 명령어를 실행하는 사용자입니다.
- **command** - 실행하려는 명령어입니다. 이 필드에는 여러 단어나 공백이 들어갈 수 있습니다.

별표 `*`는 모든 값을 뜻하며, 별표 뒤에 슬래시와 숫자를 붙이면 n번째 값마다 실행한다는 뜻입니다. 따라서 `*/5`는 5 간격으로 실행한다는 뜻입니다. 몇 가지 예는 다음과 같습니다.

```shell
*/5   *    *   *   *       # Every five minutes
  0   *    *   *   *       # Every hour at o'clock
  0   9    *   *   *       # Every day at 9:00 am
  0   9-17 *   *   *       # Every hour between 9:00am and 5:00pm
  0   0    *   *   5       # Every Friday at 12:00 am
  0   0    1   */2 *       # Every other month, the first day, 12:00am
```
일반적인 crontab 일정의 예는 [crontab.guru](https://crontab.guru/examples.html)에서 더 많이 찾아볼 수 있습니다.

## 셸 환경과 로깅

cron을 사용할 때 흔히 빠지는 함정은 일반적인 셸이 불러오는 `.bashrc`, `.zshrc` 등의 환경 스크립트를 불러오지 않고, 기본적으로 출력을 어디에도 기록하지 않는다는 점입니다. 실행 빈도도 최대 1분에 한 번이므로, 처음에는 cron 스크립트를 디버깅하기가 꽤 어려울 수 있습니다.

환경 문제를 처리하려면 모든 스크립트에서 절대 경로를 사용하고, 스크립트가 정상적으로 실행되도록 `PATH`와 같은 환경 변수를 수정하세요. 로깅을 단순하게 하려면 다음과 같은 형식으로 crontab을 작성하는 것이 좋습니다.


```shell
* * * * *   user  /path/to/cronscripts/every_minute.sh >> /tmp/cron_every_minute.log 2>&1
```

그리고 스크립트는 별도 파일로 작성하세요. `>>`는 파일에 내용을 덧붙이며, `2>&1`은 `stderr`를 `stdout`으로 리다이렉션한다는 점을 기억하세요(둘을 분리해 두고 싶을 수도 있습니다).

## Anacron

cron을 사용할 때 주의할 점은 cron 스크립트가 실행되어야 할 시각에 컴퓨터가 꺼져 있거나 절전 상태라면 스크립트가 실행되지 않는다는 것입니다. 자주 실행하는 작업이라면 괜찮을 수 있지만, 드물게 실행하는 작업이라면 반드시 실행되도록 하고 싶을 수 있습니다. [anacron](https://linux.die.net/man/8/anacron)은 실행 주기를 일 단위로 지정한다는 점을 제외하면 `cron`과 비슷하게 동작합니다. cron과 달리 컴퓨터가 계속 실행 중이라고 가정하지 않습니다. 따라서 하루 24시간 실행되지 않는 컴퓨터에서도 매일, 매주, 매월 실행하는 작업과 같은 정기 작업을 제어하는 데 사용할 수 있습니다.


## 연습 문제

1. 매분 다운로드 폴더에서 그림 파일을 찾아 Pictures 폴더로 옮기는 스크립트를 만드세요([MIME 유형](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)을 알아보거나 정규 표현식으로 흔히 쓰이는 확장자를 찾아볼 수 있습니다).

1. 매주 시스템에서 오래된 패키지가 있는지 확인하고 업데이트를 요청하거나 자동으로 업데이트하는 cron 스크립트를 작성하세요.



{% comment %}

- [fswatch](https://github.com/emcrisostomo/fswatch)
- GUI automation (pyautogui) [Automating the boring stuff Chapter 18](https://automatetheboringstuff.com/chapter18/)
- Ansible/puppet/chef

- https://xkcd.com/1205/
- https://xkcd.com/1319/

{% endcomment %}
