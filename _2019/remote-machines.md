---
layout: lecture
title: "원격 컴퓨터"
presenter: Jose
date: 2019-01-29
order: 4
video:
  aspect: 62.5
  id: X5c2Y8BCowM
translation_status: draft
---

프로그래머가 일상적인 업무에 원격 서버를 사용하는 일이 점점 더 흔해지고 있습니다. 백엔드 소프트웨어를 배포하기 위해 원격 서버를 사용해야 하거나 연산 능력이 더 높은 서버가 필요하다면, 결국 보안 셸(SSH)을 사용하게 됩니다. 이 강의에서 다루는 대부분의 도구와 마찬가지로 SSH도 매우 다양하게 설정할 수 있으므로 배워 둘 가치가 있습니다.


## 명령어 실행하기

`ssh`에서 흔히 간과되는 기능은 명령어를 직접 실행할 수 있다는 점입니다.

- `ssh foobar@server ls`는 foobar의 홈 폴더에서 ls를 실행합니다.
- 파이프와도 함께 작동하므로, `ssh foobar@server ls | grep PATTERN`은 원격의 `ls` 출력을 로컬에서 grep으로 처리하고, `ls | ssh foobar@server grep PATTERN`은 로컬의 `ls` 출력을 원격에서 grep으로 처리합니다.

## SSH 키

키 기반 인증은 공개 키 암호 기법을 활용하여, 클라이언트가 비밀인 개인 키를 소유하고 있다는 사실을 키를 드러내지 않고 서버에 증명합니다. 이렇게 하면 매번 비밀번호를 다시 입력할 필요가 없습니다. 하지만 개인 키(예: `~/.ssh/id_rsa`)는 사실상 여러분의 비밀번호이므로 비밀번호처럼 다루세요.

- 키 생성. 키 쌍을 생성하려면 `ssh-keygen -t rsa -b 4096`을 실행하기만 하면 됩니다. 암호 구문을 설정하지 않으면 개인 키를 손에 넣은 사람은 누구나 인증이 허용된 서버에 접근할 수 있으므로, 암호 구문을 설정하고 `ssh-agent`로 셸 세션을 관리하는 것을 권장합니다.

SSH 키를 사용하여 Github에 푸시하도록 설정했다면, 아마 [여기](https://help.github.com/articles/connecting-to-github-with-ssh/)에 나오는 단계를 이미 거쳤고 유효한 키 쌍도 가지고 있을 것입니다. 암호 구문이 설정되어 있는지 확인하고 그 암호 구문이 맞는지 검증하려면 `ssh-keygen -y -f /path/to/key`를 실행하면 됩니다.

- 키 기반 인증. `ssh`는 어떤 클라이언트의 접속을 허용할지 결정하기 위해 `.ssh/authorized_keys`를 살펴봅니다. 공개 키를 복사하려면 다음을 사용할 수 있습니다.

```bash
cat .ssh/id_dsa.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

`ssh-copy-id`를 사용할 수 있다면 더 간단하게 처리할 수 있습니다.

```bash
ssh-copy-id -i .ssh/id_dsa.pub foobar@remote
```

## ssh로 파일 복사하기

ssh로 파일을 복사하는 방법은 여러 가지가 있습니다.

- `ssh+tee`: 가장 간단한 방법은 `ssh`의 명령어 실행 기능과 표준 입력을 사용하여 `cat localfile | ssh remote_server tee serverfile`을 실행하는 것입니다.
- `scp`: 많은 파일이나 디렉토리를 복사할 때는 보안 복사 명령어 `scp`가 경로를 따라 재귀적으로 처리하기 쉬워 더 편리합니다. 문법은 `scp path/to/local_file remote_host:path/to/remote_file`입니다.
- `rsync`는 로컬과 원격에서 동일한 파일을 감지해 다시 복사하지 않는다는 점에서 `scp`보다 발전한 도구입니다. 심볼릭 링크와 권한을 더 세밀하게 제어할 수도 있고, 중단된 복사를 이어서 진행할 수 있는 `--partial` 플래그 같은 추가 기능도 제공합니다. `rsync`의 문법은 `scp`와 비슷합니다.


## 프로세스를 백그라운드에서 실행하기

기본적으로 ssh 연결이 끊어지면 부모 셸의 자식 프로세스도 셸과 함께 종료됩니다. 이를 피하는 방법이 몇 가지 있습니다.

- `nohup` - `nohup` 도구는 터미널이 종료되어도 프로세스가 계속 살아 있도록 해 줍니다. 때로는 `&`와 `disown`으로도 가능하지만, 기본적으로는 nohup이 더 나은 선택입니다. 자세한 내용은 [여기](https://unix.stackexchange.com/questions/3886/difference-between-nohup-disown-and)에서 확인할 수 있습니다.

- `tmux`, `screen` - `nohup`은 프로세스를 백그라운드에서 실행하게 해 주지만 대화형 셸 세션에는 편리하지 않습니다. 그런 경우에는 `screen`이나 `tmux` 같은 터미널 멀티플렉서를 사용하면 해당 셸에서 쉽게 분리했다가 다시 연결할 수 있어 편리합니다.

마지막으로, 프로그램을 disown한 뒤 현재 터미널에 다시 연결하고 싶다면 [reptyr](https://github.com/nelhage/reptyr)를 살펴보세요. `reptyr PID`는 ID가 PID인 프로세스를 가져와 현재 터미널에 연결합니다.

## 포트 포워딩

여러 상황에서 컴퓨터의 포트에서 요청을 수신하는 방식으로 작동하는 소프트웨어를 만나게 됩니다. 로컬 컴퓨터에서는 `localhost:PORT`나 `127.0.0.1:PORT`를 사용하면 되지만, 네트워크나 인터넷에서 포트에 직접 접근할 수 없는 원격 서버라면 어떻게 해야 할까요? 이때 사용하는 것이 포트 포워딩이며, 로컬 포트 포워딩과 원격 포트 포워딩이라는 두 종류가 있습니다(자세한 내용은 그림을 참고하세요. 그림 출처는 [이 SO 게시물](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot)입니다).


**로컬 포트 포워딩**
![로컬 포트 포워딩](https://i.stack.imgur.com/a28N8.png)

**원격 포트 포워딩**
![원격 포트 포워딩](https://i.stack.imgur.com/4iK3b.png)


가장 흔한 상황은 원격 컴퓨터의 서비스가 특정 포트에서 요청을 수신하고 있고, 로컬 컴퓨터의 포트를 그 원격 포트로 전달하도록 연결하려는 로컬 포트 포워딩입니다. 예를 들어 원격 서버에서 `jupyter notebook`을 실행하여 `8888` 포트에서 요청을 수신한다고 합시다. 이를 로컬 포트 `9999`로 전달하려면 `ssh -L 9999:localhost:8888 foobar@remote_server`를 실행한 뒤 로컬 컴퓨터에서 `localhost:9999`로 접속하면 됩니다.

## 그래픽 포워딩

서버에서 GUI 기반 프로그램을 실행하려는 경우에는 포트 포워딩만으로 부족할 때가 있습니다. 전체 데스크톱 환경을 전송하는 원격 데스크톱 소프트웨어(예: RealVNC, Teamviewer 등)를 사용할 수도 있습니다. 하지만 GUI 도구 하나만 사용하려는 경우에는 SSH가 좋은 대안을 제공합니다. 바로 그래픽 포워딩입니다.

`-X` 플래그를 사용하면 SSH에 포워딩하도록 지시합니다.

 신뢰할 수 있는 X11 포워딩에는 `-Y` 플래그를 사용할 수 있습니다.

마지막으로, 이 기능이 작동하려면 서버의 `sshd_config`에 다음 옵션이 설정되어 있어야 합니다.

```bash
X11Forwarding yes
X11DisplayOffset 10
```

## 로밍

원격 서버에 접속할 때 흔히 겪는 불편은 컴퓨터를 종료하거나 절전 모드로 전환하거나 네트워크를 변경하여 연결이 끊어지는 것입니다. 게다가 지연이 큰 연결에서는 ssh 사용이 상당히 답답할 수 있습니다. 모바일 셸인 [Mosh](https://mosh.org/)는 로밍 연결과 간헐적인 연결을 허용하고 지능적인 로컬 에코를 제공하여 ssh를 개선합니다.

Mosh는 모든 주요 배포판과 패키지 관리자에서 제공됩니다. Mosh를 사용하려면 서버에서 ssh 서버가 작동하고 있어야 합니다. mosh를 설치하는 데 슈퍼유저 권한은 필요하지 않지만, 서버의 60000번부터 60010번까지의 포트가 열려 있어야 합니다(특권 포트 범위에 속하지 않으므로 대개 열려 있습니다).

`mosh`의 단점은 포트 포워딩이나 그래픽 포워딩의 로밍을 지원하지 않는다는 것입니다. 따라서 이런 기능을 자주 사용한다면 `mosh`는 큰 도움이 되지 않을 것입니다.

## SSH 설정

### 클라이언트

지금까지 전달할 수 있는 여러 인자를 살펴보았습니다. `alias my_server="ssh -X -i ~/.ssh/id_rsa -L 9999:localhost:8888 foobar@remote_server"` 같은 셸 별칭을 만드는 방법이 매력적으로 보일 수 있지만, `~/.ssh/config`를 사용하는 더 나은 방법이 있습니다.

```bash
Host vm
    User foobar
    HostName 172.16.174.141
    Port 22
    IdentityFile ~/.ssh/id_rsa
    RemoteForward 9999 localhost:8888

# Configs can also take wildcards
Host *.mit.edu
    User foobaz
```


별칭 대신 `~/.ssh/config` 파일을 사용하면 `scp`, `rsync`, `mosh` 같은 다른 프로그램도 이 파일을 읽고 설정을 해당 플래그로 변환할 수 있다는 장점도 있습니다.


`~/.ssh/config` 파일도 닷파일로 볼 수 있으며, 일반적으로 다른 닷파일과 함께 포함해도 괜찮습니다. 하지만 이를 공개할 때는 인터넷의 낯선 사람들에게 어떤 정보를 제공할 수 있는지 생각해 보세요. 서버 주소, 사용하는 사용자 계정, 열려 있는 포트 등이 여기에 해당합니다. 이런 정보는 일부 유형의 공격을 쉽게 만들 수 있으므로 SSH 설정을 공유할 때는 신중하게 판단하세요.

경고: RSA 키( `~/.ssh/id_rsa*` )는 절대로 공개 저장소에 포함하지 마세요!

### 서버 측

서버 측 설정은 보통 `/etc/ssh/sshd_config`에 지정합니다. 여기에서 비밀번호 인증을 비활성화하거나, ssh 포트를 바꾸거나, X11 포워딩을 활성화하는 등의 변경을 할 수 있습니다. 사용자별로 설정을 지정할 수도 있습니다.

## 원격 파일시스템

원격 폴더를 마운트하면 편리할 때가 있습니다. [sshfs](https://github.com/libfuse/sshfs)는 원격 서버의 폴더를 로컬에 마운트하여 로컬 편집기로 작업할 수 있게 해 줍니다.

## 연습문제

1. SSH가 작동하려면 호스트에서 SSH 서버가 실행 중이어야 합니다. 나머지 연습문제를 수행할 수 있도록 가상 머신에 SSH 서버(예: OpenSSH)를 설치하세요. 컴퓨터의 IP를 알아내려면 `ip addr` 명령어를 실행하고 inet 필드를 찾으세요(루프백 인터페이스에 해당하는 `127.0.0.1` 항목은 무시하세요).

1. `~/.ssh/`로 가서 SSH 키 쌍이 있는지 확인하세요. 없다면 `ssh-keygen -t rsa -b 4096`으로 생성하세요. 비밀번호를 설정하고 `ssh-agent`를 사용하는 것을 권장합니다. 자세한 내용은 [여기](https://www.ssh.com/ssh/agent)를 참고하세요.

1. `ssh-copy-id`를 사용하여 키를 가상 머신에 복사하세요. 비밀번호 없이 ssh로 접속할 수 있는지 확인하세요. 그런 다음 서버의 `sshd_config`에서 `PasswordAuthentication` 값을 수정하여 비밀번호 인증을 비활성화하세요. `PermitRootLogin` 값을 수정하여 root 로그인을 비활성화하세요.

1. 서버의 `sshd_config`를 편집하여 ssh 포트를 바꾸고 여전히 ssh로 접속할 수 있는지 확인하세요. 공개적으로 접근할 수 있는 서버를 운영하게 된다면 기본값과 다른 포트와 키만 사용하는 로그인으로 상당수의 악의적인 공격을 억제할 수 있습니다.

1. 서버나 VM에 mosh를 설치하고 연결한 다음, 서버나 VM의 네트워크 어댑터를 분리하세요. mosh가 연결을 제대로 복구할 수 있나요?

1. 로컬 포트 포워딩의 또 다른 용도는 특정 호스트로 가는 연결을 서버를 통해 터널링하는 것입니다. 네트워크에서 `reddit.com` 같은 웹사이트를 필터링한다면 다음과 같이 서버를 통해 터널링할 수 있습니다.

    - `ssh remote_server -L 80:reddit.com:80`을 실행하세요.
    - `reddit.com`과 `www.reddit.com`을 `127.0.0.1`로 `/etc/hosts`에 설정하세요.
    - 서버를 통해 해당 웹사이트에 접속하고 있는지 확인하세요.
    - 확실하지 않다면 호스트의 공인 IP에 따라 결과가 달라지는 [ipinfo.io](https://ipinfo.io/) 같은 웹사이트를 사용하세요.


1. 플래그를 몇 개 더 사용하면 포트 포워딩을 쉽게 백그라운드에서 실행할 수 있습니다. `-N`과 `-f` 플래그가 `ssh`에서 하는 일을 알아보고, `ssh -N -f -L 9999:localhost:8888 foobar@remote_server` 같은 명령어가 어떤 일을 하는지 알아보세요.


## 참고 자료

- [SSH 활용 기법](https://matt.might.net/articles/ssh-hacks/)
- [보안 셸을 안전하게 설정하기](https://stribika.github.io/2015/01/04/secure-secure-shell.html)

{% comment %}
Lecture notes will be available by the start of lecture.
{% endcomment %}
