---
layout: lecture
title: "Security and Cryptography"
description: >
  Hash 함수와 키 유도 함수 같은 암호학 기본 요소와 Git, SSH가 이를 활용하는 방법을 배웁니다.
thumbnail: /static/assets/thumbnails/2020/lec9.png
date: 2020-01-28
ready: true
video:
  aspect: 56.25
  id: tjwobAmnKTo
special: true
translation_status: complete
---

지난 해의 [security and privacy lecture](/2019/security/)는 컴퓨터 _사용자_ 측면에서 어떻게 더 안전해 질 수 있는가에 초점을 맞췄습니다. 이번에는, Git에서의 Hash 함수 혹은 키 유도 함수의 사용과 SSH 에서의 대칭/비대칭 암호 시스템과 같이 지난 수업에서 다룬 도구를 이해하는데 관련이 있는 보안과 암호학 개념에 초점을 맞출 것입니다.

이 강의는 컴퓨터 시스템 보안 ([6.858](https://css.csail.mit.edu/6.858/)) 혹은 암호학([6.857](https://courses.csail.mit.edu/6.857/) and 6.875)에 대한 완벽한 대체 강의가 아닙니다. 보안 업무를 하기 위해서는 보안에 관한 정규 교육을 받아야 합니다. 전문가가 아니라면, [본인이 만든 암호를 사용하지 마세요](https://www.schneier.com/blog/archives/2015/05/amateurs_produc.html). 이 원칙은 시스템 보안에도 적용됩니다.

이 강의는 기본적인 암호 개념을 비공식적(이지만 효율적)으로 다루고 있습니다. 이 강의는 독자가 어떻게 안전한 시스템이나 암호학 프로토콜을 _설계_ 할 수 있는지 가르치기에는 부족하지만, 독자가 이미 사용하는 프로그램과 프로토콜에 대해 일반적인 이해를 할 수 있도록 도움을 주고자 합니다.

# 엔트로피(Entropy)

[엔트로피](https://en.wikipedia.org/wiki/Entropy_(information_theory))는 임의성의 척도입니다. 이는 예를 들면 패스워드 강도를 결정할 때 유용합니다.

![XKCD 936: Password Strength](https://imgs.xkcd.com/comics/password_strength.png)

위의 [XKCD comic](https://xkcd.com/936/) 그림에서, "correcthorsebatterystaple"라는 패스워드는 "Tr0ub4dor&3"라는 패스워드보다 안전합니다. 하지만, 어떻게 이를 측정 할 수 있을까요?

엔트로피는  _bits_ 단위로 측정되며, 가능한 결과 집합에서 무작위로 균일하게 선택 할 때, 엔트로피는 `log_2(# of possibilities)`와 동일합니다. 동전 던지기는 1 bit의 엔트로피를 제공합니다. 6면을 가진 주사위 굴리기는 ~2.58 bits의 엔트로피를 가집니다.

공격자는 암호 _모델_ 을 알고 있지만, 특정 암호를 선택하는 데 사용되는 임의성(예 - [주사위 던지기](https://en.wikipedia.org/wiki/Diceware))은 알 수 없다는 점을 고려해야 합니다.

몇 bits의 엔트로피가 충분할까요? 이는 당신의 위협 모델에 달렸습니다. 온라인 추측의 경우 위의 XKCD 만화에서 확인했듯이, ~40 bits의 엔트로피로 충분합니다. 오프라인 추측에 대응 할 경우 더 강력한 암호가 필요합니다. 

# Hash 함수

[암호화를 위한 Hash 함수](https://en.wikipedia.org/wiki/Cryptographic_hash_function)는 임의 크기의 데이터를 고정된 크기로 매핑하고, 몇 가지 특징을 가지고 있습니다. Hash 함수의 대략적인 명세는 아래와 같습니다.

```
hash(value: array<byte>) -> vector<byte, N>  (for some fixed N)
```

Hash 함수의 한 예는 Git에서 사용하고 있는 [SHA1](https://en.wikipedia.org/wiki/SHA-1) 입니다. 이는 임의 크기의 입력값을 160 bit의 결과(40개의 16진수 문자로 표현할 수 있음)로 매핑합니다. SHA1 Hash는 `sha1sum` 명령어를 통해 사용 해 볼 수 있습니다.

```console
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'hello' | sha1sum
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
$ printf 'Hello' | sha1sum 
f7ff9e8b7bb2e09b70935a5d785e0cc5d9d0abf0
```

높은 수준에서, Hash 함수는 도치하기 어렵고 무작위로 보이(지만 결정적인)는 함수로 보일 수 있으며, 이는 [Hash 함수의 이상적 모델](https://en.wikipedia.org/wiki/Random_oracle)입니다. Hash 함수는 아래의 특징을 가지고 있습니다.

- 결정적(Deterministic) : 같은 입력 값은 항상 같은 결과가 도출
- 복구 불가능(Non-invertible) : 원하는 출력값 `h`에 대하여 `hash(m) = h`를 만족하는 `m`을 찾기 어려움
- 약한 충돌 저항성(Target collision resistant) : `m_1`이 주어졌을 때, `hash(m_1) = hash(m_2)`를 만족하는 `m_2`를 찾기 어려움
- 강한 충돌 저항성(Collision resistant) : `hash(m_1) = hash(m_2)`를 만족하는 `m_1`과 `m_2`를 찾기 어려움(약한 충돌 저항성보다 강한 속성)

참고 : 특정 목적으로 사용할 수도 있지만, SHA-1은 [더 이상](https://web.archive.org/web/20260207211148/https://shattered.io/) 안전한 암호화를 위한 Hash 함수로 취급하지 않습니다. [lifetimes of cryptographic hash functions](https://valerieaurora.org/hash.html)가 아마 흥미로울 것 입니다. 그러나, 특정 Hash 함수를 추천하는 것은 이 강의의 범위를 벗어납니다. 만약 이 글을 읽는 독자가 이러한 문제를 가지고 있다면, 보안/암호학에 대한 정규 교육이 필요합니다.

## 어플리케이션(Applications)

- Git, 컨텐츠 주소 지정 스토리지용. [hash 함수](https://en.wikipedia.org/wiki/Hash_function)의 개념은 더 일반적입니다.(암호화의 목적이 아닌 Hash 함수가 있음). 왜 Git이 암호화 Hash 함수를 사용할까요?
- 파일 내용을 짧게 요약. 사용자들은 소프트웨어를 (덜 신뢰할 수 있는) 미러 사이트에서 다운받곤 하는데(예 - Linux ISO), 이는 신뢰하지 않는 것이 좋습니다. 공식 사이트에서는 보통 타사의 미러 페이지로 이동하는 다운로드 링크와 함께 파일의 Hash 값을 게시 합니다. 이를 통해 사용자는 파일을 다운로드 받은 후 다운로드 받은 파일의 Hash를 확인할 수 있습니다.
- [Commitment schemes](https://en.wikipedia.org/wiki/Commitment_scheme). 특정 값을 커밋하고 싶지만, 나중에 공개하고 싶다고 가정합니다. 예를 들어, 나는 두 당사자가 볼 수 있고 신뢰할 수 있는 공유 동전 없이 "내 머리속에" 공정한 동전 던지기를 하고 싶습니다. 나는 `r = random()`인 값을 선택한 다음 `h = sha256(r)`을 공유할 수 있습니다. 그 후, 앞면 혹은 뒷면을 외칠 수 있습니다(`r`이 짝수 일 경우는 앞면이며, 홀수일 경우에는 뒷면을 의미한다고 약속합니다.). 당신이 외친 후, 내 값인 `r`을 공개할 수 있고, `sha256(r)`이 이전에 공유 한 Hash와 일치하는지 확인하여 내가 속이지 않았음을 확인할 수 있습니다.

# 키 유도 함수(Key derivation functions)

암호화 Hash와 관련된 개념인 [key derivation
functions](https://en.wikipedia.org/wiki/Key_derivation_function) (KDFs)는 다른 암호 알고리즘에서 키로 사용하기 위한 고정 길이 출력값을 생성하는 것도 포함하여, 여러 응용 프로그램에서 사용됩니다. 일반적으로, KDF는 오프라인 무차별 대입 공격의 속도를 늦추기 위해 의도적으로 느립니다.

## 어플리케이션(Applications)

- 다른 암호화 알고리즘에서 사용하기 위한 암호구문 키 생성 (예 - 대칭키 암호, 아래 참조)
- 로그인 자격증명 저장. 패스워드 평문 저장은 적절하지 않습니다. 올바른 접근 법은 각 사용자 별로 `salt = random()`인 랜덤한 [salt](https://en.wikipedia.org/wiki/Salt_(cryptography))를 생성하고, `KDF(password + salt)`를 저장합니다. 그 후 입력 된 패스워드와 저장된 salt로 제공된 KDF를 다시 계산하는 방법으로 로그인 시도를 검증합니다.

# 대칭키 암호(Symmetric cryptography)

메세지를 숨긴다는 개념은 아마 독자가 암호학에 대해 생각했을 때 가장 먼저 떠오르는 개념일 것입니다. 대칭키 암호는 다음 기능들을 사용하여 이를 수행합니다.

```
keygen() -> key  (this function is randomized)

encrypt(plaintext: array<byte>, key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, key) -> array<byte>  (the plaintext)
```

암호화 함수는 출력(암호문)을 제공하는 기능을 가지며, 키 없이 입력값(평문)을 알아내기 어렵습니다. 복호화 함수는 `decrypt(encrypt(m, k), k) = m`라는 명확한 정확성 속성을 가지고 있습니다.

오늘날 널리 쓰이는 대칭키 암호의 한 예는 [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 입니다.

## 어플리케이션(Applications)

- 신뢰할 수 없는 클라우드 서비스에 파일을 저장할 때 파일을 암호화. 이는 KDF와 결합되어 암호로 파일을 암호화 할 수 있습니다. `key = KDF(passphrase)`로 키를 생성하고, `encrypt(file, key)`로 암호화 한 파일을 저장합니다.


# 비대칭 암호화(Asymmetric cryptography)

"비대칭"이라는 용어는 서로 다른 역할을 가진 두 개의 키를 의미합니다.

이름에서 알 수 있듯이, 개인키는 비공개로 유지되는 반면 공개 키는 공개적으로 공유될 수 있으며 보안에 영향을 미치지 않습니다(대칭 암호 시스템에서 키를 공유하는 것과 다릅니다).

비대칭 암호화 시스템은 암호화/복호화 및 서명/검증을 위한 다음과 같은 기능 집합을 제공합니다.

```
keygen() -> (공개키, 개인키) (이 기능은 임의로 지정됨)

encrypt(plaintext: array<byte>, public key) -> array<byte>  (the ciphertext)
decrypt(ciphertext: array<byte>, private key) -> array<byte>  (the plaintext)

sign(message: array<byte>, private key) -> array<byte>  (the signature)
verify(message: array<byte>, signature: array<byte>, public key) -> bool  (whether or not the signature is valid)
```

encrypt/decrypt 함수에는 대칭형 암호 시스템의 아날로그와 유사한 속성이 있습니다.

메시지는 _공용_ 키를 사용하여 암호화할 수 있습니다.

출력(암호문)을 주어지면, _개인_ 키 없이 입력(평문)을 결정하기는 어렵습니다.

복호화 기능은 `decrypt(encrypt(m, public key), private key) = m`라는 명백한 정확성 속성을 갖고 있습니다. 

대칭과 비대칭 암호화를 물리적 잠금과 비교할 수 있습니다.
대칭 암호 시스템은 도어 잠금 장치와 같습니다: 키가 있는 사람은 누구나 잠그고 해제할 수 있습니다.
비대칭 암호화는 열쇠가 달린 자물쇠와 같습니다.
잠금 해제된 자물쇠를 누군가에게 줄 수 있고(공개키), 메시지를 상자에 넣어 다음 자물쇠를 채울 수 있습니다. 그 후, 오로지 당신만이 열쇠를 가지고 있기 때문에(개인키) 자물쇠를 열 수 있습니다.

서명/검증 기능은 서명을 위조하기 어렵다는 것에서 물리적 서명이 갖고 있는 것과 동일한 속성을 가지고 있습니다.
No matter the
message, without the _private_ key, it's hard to produce a signature such that
`verify(message, signature, public key)` returns true. And of course, the
verify function has the obvious correctness property that `verify(message,
sign(message, private key), public key) = true`.
메시지든, _개인_ 키 없이, `verify(message, signature, public key)`의 결과가 참인 서명을 생성하기 어렵습니다. 그리고 물론, 검증 함수는 `verify(message,
sign(message, private key), public key) = true`라는 명백한 정확성을 가지고 있습니다.

## 어플리케이션(Application)

- [PGP email 암호화](https://en.wikipedia.org/wiki/Pretty_Good_Privacy). 사람들은 자신의 공개키를 온라인상에 올릴 수 있습니다. (예 - PGP Keyserver나 [Keybase](https://keybase.io/) 등). 그러면 누구나 그 사람에게 암호화된 메일을 보낼 수 있습니다.
- 개인 메시지. [Signal](https://signal.org/)나 [Keybase](https://keybase.io/)와 같은 어플리케이션은 개인 통신 채널을 설정하기 위해 비대칭 키를 사용합니다.
- Signing software. Git에 GPG-signed commits과 tags를 사용할 수 있습니다. 공개된 공용 키를 사용하면, 누구나 다운로드된 소프트웨어의 신뢰성을 확인할 수 있습니다. 

## 키 분배(Key distribution)

비대칭 키 암호화는 훌륭하지만, 공용 키를 배포하고 실제 ID에 공개키를 매핑하는 데는 큰 어려움이 있습니다. 이 문제에 대한 많은 해결책이 있습니다.
Signal에는 첫 번째 사용에 대한 신뢰와 대역 외 공개 키 교환 지원 (직접 친구의 "안전 번호"확인)이라는 간단한 솔루션이 있습니다.
PGP에는 [web of trust] (https://en.wikipedia.org/wiki/Web_of_trust)라는 다른 솔루션이 있습니다.

Keybase에는 [사회적 증명](https://keybase.io/blog/chat-apps-softer-than-tofu) (다른 깔끔한 아이디어와 함께)의 또 다른 솔루션이 있습니다. 각 모델에는 장점이 있습니다. 우리 (강사)는 Keybase의 모델을 좋아합니다.

# Case studies

## Password managers

이 도구는 모든 사용자가 사용해야하는 필수적인 도구입니다.(예:[KeePassXC](https://keepassxc.org/), [pass](https://git.zx2c4.com/password-store/about/), [1Password](https://1password.com)).
암호 관리자를 사용하게 되면 임의로 만든 고유한 high-entropy 암호를 모든 로그인에 편리하게 사용할 수 있고, KDF를 사용해 생성된 키를 사용한 대칭 암호로 암호화된 모든 패스워드를 한 곳에 저장할 수 있습니다.

암호 관리자를 사용하면 암호 재사용을 피할 수 있고(이를 통해 웹사이트에 문제 발생 시 덜 영향을 받을 수 있음), high-entropy 암호를 사용할 수 있으며(손상될 가능성이 낮음), high-entropy 암호 하나만 기억하면 됩니다. 

## Two-factor authentication

[Two-factor authentication](https://en.wikipedia.org/wiki/Multi-factor_authentication)(2FA)에서는 도난된 암호와 [phishing](https://en.wikipedia.org/wiki/Phishing) 공격으로부터 보호하기위해 2FA 인증자([YubiKey](https://www.yubico.com/), "보유한 것"등)와 함께 암호("알고있는 것")를 사용해야 합니다.

## 전체 디스크 암호화

노트북의 전체 디스크를 암호화된 상태로 유지하면 노트북을 도난당한 경우 데이터를 쉽게 보관할 수 있습니다. 
Linux에서 [cryptsetup + LUKS](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_a_non-root_file_system)을 사용하거나 Windows에서 [BitLocker](https://fossbytes.com/enable-full-disk-encryption-windows-10/)을 사용할 수 있고, masOS에서는 [FileVault](https://support.apple.com/en-us/HT204837)를 사용할 수 있습니다.
이는 보호되는 키를 사용하여 전체 디스크를 대칭 암호화를 사용하여 암호화 합니다.

## 개인 메시지

[Signal](https://signal.org/) 또는 [Keybase](https://keybase.io/)를 사용합니다.
End-to-end의 보안은 비대칭 키 암호화에서 부트스트랩 됩니다.
연락할 사람의 공용 키를 얻는 것이 중요한 단계입니다.
안전한 보안을 원한다면, out-of-band(Signal 또는 keybase를 사용하여)로 공개 키를 인증하거나, 사회적 증명(keybase 사용)을 신뢰해야 합니다. 

## SSH

전 강의에서 SSH와 SSH키 사용에 대해 다루었습니다. [earlier lecture](/2020/command-line/#remote-machines)
이것의 암호적 측면을 살펴봅시다.

`ssh-keygen` 실행 시, `public_key, private_key`의 비대칭 키쌍이 생성됩니다. 이는 운영체제에서 제공되는 엔트로피를 사용하여 무작위로 생성됩니다(하드웨어 이벤트 등에서 수집). 
공용 키는 그대로 저장되며(공개키 이므로 비밀 유지하는 것은 중요하지 않음), 개인키는 디스크에 암호화되어 저장되어야 합니다.
`ssh-keygen` 프로그램은 사용자에게 암호를 묻고, 이는 키 유도 함수를 통해 제공되어 키를 생성 한 다음 대칭 암호로 개인 키를 암호화하는 데 사용됩니다.

사용 시, 서버가 클라이언트의 공용 키(`.ssh/authorized_keys`에 저장된 파일)를 알게 되면, 연결 클라이언트는 비대칭 서명을 사용해 신원을 증명할 수 있습니다.
이것은 [challenge-response](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication)을 통해 이루어집니다.
상위 수준에서, 서버는 임의의 숫자를 선택하여 클라이언트에게 전송합니다.
그 후 고객이 이 메시지에 서명하고 서명을 서버로 다시 보내면, 서버는 서명을 기록 된 공개키와 비교합니다.
이는 클라이언트가 서버의 `.ssh/authorized_keys`파일의 공개키에 해당하는 개인키를 소유하고 있음을 효과적으로 입증하며, 서버는 해당 클라이언트의 로그인을 허용할 수 있습니다. 

{% comment %}
extra topics, if there's time

security concepts, tips
- biometrics
- HTTPS
{% endcomment %}

# Resources
- [작년 참고사항](/2019/security/): 이 강의가 컴퓨터 사용자로서 보안과 개인 정보 보호에 더 중점을 두었을 때부터
- [암호화 권한 답변](https://latacora.micro.blog/2018/04/03/cryptographic-right-answers.html): 많은 일반적인 X에 대하여 "X에 사용해야 하는 암호?"에 대한 답변

# 예제

1. **Entropy.**
    1. 패스워드는 사전 내의 4개의 소문자 단어를 합한 것으로 가정하고, 각 단어는 100,000 크기의 사전에서 무작위로 선택합니다.
       예를 들면 `correcthorsebatterystaple`입니다. 이 때, 이 패스워드의 엔트로피는 몇 비트인가?
    1. 암호가 8개의 임의 영문자(소문자, 대문자 모두 포함)와 숫자를 선택하는 다른 암호를 고려 합니다. 예를 들면 `rg8Ql34g` 입니다. 엔트로피는 몇 비트인가요?
    1. 어느 쪽이 더 강력한 패스워드인가요?
    1. 공격자가 초당 10,000개의 암호를 추측할 수 있다고 가정합니다. 각 암호들을 해독하는데 평균적으로 얼마나 걸립니까?

1. **Cryptographic hash functions.** [mirror](https://www.debian.org/CD/http-ftp/)에서 Debian이미지를 다운로드합니다. (예- [from this Argentinean mirror](http://debian.xfree.com.ar/debian-cd/current/amd64/iso-cd/)).  Argentinean mirror에서 다운로드 한 경우, `debian.org`로 호스팅 되는 공식 Debian 사이트에서 해쉬 값을 가져 와(예 - [this file](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS) 해쉬 값을 교차 확인(예- `sha256sum`명령어 사용)합니다.)
 
1. **대칭키 암호** 
   [OpenSSL](https://www.openssl.org/) : `openssl aes-256-cbc -salt -in {input filename} -out {output filename}`를 사용하여 AES암호화로 파일 암호화를 합니다.
   `cat` 또는 `hexdump`를 사용 해 내용을 보십시오. `openssl aes-256-cbc -d -in {input filename} -out {output filename}`를 사용하여 파일을 해독하고 `cmp`를 사용하여 해당 파일이 원본 파일과 동일한지 비교하세요.
   
1. **비대칭키 암호**
    1. [SSH keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)를 접근해야 하는 컴퓨터에 설정합니다(Kerberos가 SSH keys와 이상하게 동작하기 때문에, Athena가 아닌 것으로 합니다).
       개인 키가 비밀번호로 암호화 되었는지 확인하여 저장시 보호되도록 합니다.
    1. [GPG설정](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages)
    1. Anish에게 암호화된 전자 메일([공개키](https://keybase.io/anish))을 보냅니다.
    1. `git commit -S`로 Git commit에 서명하거나, `git tag -s`로 서명된 Git tag를 사용합니다. `git show --show-signature`를 사용하여 서명이나 `git tag -v`를 사용하여 태그를 검증합니다.
