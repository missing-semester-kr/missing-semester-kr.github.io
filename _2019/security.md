---
layout: lecture
title: "보안과 개인정보 보호"
presenter: Jon
date: 2019-01-31
order: 2
video:
  aspect: 56.25
  id: OBx_c-i-M8s
special: true
translation_status: draft
---

세상은 무서운 곳이고, 모두가 여러분을 노리고 있습니다.

좋아요, 그렇지는 않을 수도 있지만, 그렇다고 여러분의 모든 비밀을 드러내고 싶은 것은 아닐 겁니다. 보안(그리고 개인정보 보호)은 대체로 공격자의 진입 장벽을 높이는 일입니다. 여러분의 위협 모델이 무엇인지 파악한 뒤, 그것을 중심으로 보안 수단을 설계하세요! 위협 모델이 NSA나 모사드라면 _아마_ 힘든 시간을 보내게 될 것입니다.

디지털 세계의 여러분을 더 안전하게 만드는 방법은 _많습니다_. 여기서는 여러 가지를 큰 틀에서 살펴보겠지만, 보안은 하나의 과정이며 스스로 배우는 것은 여러분이 할 수 있는 가장 좋은 일 중 하나입니다. 그러니 다음을 실천하세요.

## 적절한 사람들을 팔로우하세요

보안 지식을 높이는 가장 좋은 방법 중 하나는 보안에 관해 적극적으로 이야기하는 사람들을 팔로우하는 것입니다. 몇 사람을 추천합니다.

 - [@TroyHunt](https://twitter.com/TroyHunt)
 - [@SwiftOnSecurity](https://twitter.com/SwiftOnSecurity)
 - [@taviso](https://twitter.com/taviso)
 - [@thegrugq](https://twitter.com/thegrugq)
 - [@tqbf](https://twitter.com/tqbf)
 - [@mattblaze](https://twitter.com/mattblaze)
 - [@moxie](https://twitter.com/moxie)

더 많은 추천은 [이 목록](https://heimdalsecurity.com/blog/best-twitter-cybersec-accounts/)도 참고하세요.

## 일반적인 보안 조언

Tech Solidarity에는 합리적인 조언이 많이 담겨 있고 내용도 꽤 최신인 훌륭한 [언론인을 위한 권장 사항과 금지 사항 목록](https://web.archive.org/web/20221123204419/https://techsolidarity.org/resources/basic_security.htm)이 있습니다. [@thegrugq](https://medium.com/@thegrugq)의 [여행 중 보안에 관한 조언](https://medium.com/@thegrugq/stop-fabricating-travel-security-advice-35259bf0e869)도 읽어 볼 만한 좋은 블로그 게시물입니다. 여기서는 그 자료들의 조언을 상당 부분 되풀이하고, 몇 가지를 더 다루겠습니다. 그리고 [USB 데이터 차단기](https://www.amazon.com/dp/B00QRRZ2QM/)를 구하세요. [USB는 무섭기](https://www.bleepingcomputer.com/news/security/heres-a-list-of-29-different-types-of-usb-attacks/) 때문입니다.

## 인증

아직 하지 않았다면, 가장 먼저 해야 할 일은 비밀번호 관리자를 다운로드하는 것입니다. 좋은 도구로는 다음이 있습니다.

 - [1password](https://1password.com/)
 - [KeePass](https://keepass.info/)
 - [BitWarden](https://bitwarden.com/)
 - [`pass`](https://git.zx2c4.com/password-store/about/)

특히 의심이 많다면 서버에 비밀번호를 평문으로 저장하는 도구 대신 여러분의 컴퓨터에서 로컬로 암호화하는 도구를 사용하세요. 지금 바로 이 도구로 여러분이 중요하게 여기는 모든 웹사이트의 비밀번호를 생성하세요. 그런 다음 이중 인증을 활성화하세요. 가능하면 [FIDO/U2F](https://fidoalliance.org/) 동글(예를 들어 [YubiKey](https://www.yubico.com/quiz/)는 [학생에게 20% 할인](https://www.yubico.com/why-yubico/for-education/)을 제공합니다)을 사용하세요. Google Authenticator나 Duo 같은 TOTP도 급할 때는 쓸 수 있지만, [피싱을 막아 주지는 못합니다](https://twitter.com/taviso/status/1082015009348104192). 위협 모델이 비밀번호가 전송되는 도중 낯선 사람이 우연히 가로채는 상황만 포함하는 것이 아니라면 SMS는 사실상 쓸모가 없습니다.

종이 키에 대해서도 한마디 하겠습니다. 서비스에서는 실제 두 번째 인증 수단을 잃었을 때 두 번째 인증 수단으로 쓸 수 있는 "백업 키"를 제공하는 경우가 많습니다(참고로, 예비 동글은 항상 안전한 곳에 보관하세요!). 이 키를 비밀번호 관리자에 넣어 둘 _수는_ 있지만, 누군가 비밀번호 관리자에 접근하면 완전히 당하게 된다는 뜻입니다(하지만 그런 위협 모델을 받아들일 수도 있겠죠). 정말 의심이 많다면 이 종이 키를 인쇄하고, 디지털 형태로는 절대 저장하지 말고, 현실 세계의 금고에 넣어 두세요.

## 비공개 통신

[Signal](https://www.signal.org/)을 사용하세요([설정 방법](https://medium.com/@mshelton/signal-for-beginners-c6b44f76a1f0). [Wire](https://wire.com/en/)도 [괜찮습니다](https://www.securemessagingapps.com/). WhatsApp도 괜찮습니다. [Telegram은 사용하지 마세요](https://twitter.com/bascule/status/897187286554628096)). 데스크톱 메신저는 상당히 문제가 많습니다(부분적으로는 신뢰해야 할 소프트웨어 계층이 매우 방대한 Electron에 대개 의존하기 때문입니다).

이메일은 PGP 서명을 하더라도 특히 문제가 많습니다. 일반적으로 전방향 안전성을 제공하지 않으며, 키 배포 문제도 상당히 심각합니다. [keybase.io](https://keybase.io/)가 도움이 되며, 그 밖에도 여러 이유로 유용합니다. 또한 PGP 키는 보통 데스크톱 컴퓨터에서 다루는데, 이는 보안이 가장 취약한 컴퓨팅 환경 중 하나입니다. 이와 관련해서 Chromebook을 구매하거나, 그냥 키보드를 연결한 태블릿에서 작업하는 것을 고려해 보세요.

## 파일 보안

파일 보안은 어렵고, 여러 수준에서 이루어집니다. 무엇으로부터 보호하려는 건가요?

[![5달러짜리 렌치](https://imgs.xkcd.com/comics/security.png)](https://xkcd.com/538/)

 - 오프라인 공격(누군가 꺼져 있는 노트북을 훔치는 경우): 전체 디스크 암호화를 활성화하세요. (Linux에서는 [cryptsetup + LUKS](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_a_non-root_file_system), Windows에서는 [BitLocker](https://fossbytes.com/enable-full-disk-encryption-windows-10/), macOS에서는 [FileVault](https://support.apple.com/en-us/HT204837)를 사용하세요. 공격자가 여러분까지 _함께_ 붙잡고 있고 여러분의 비밀을 정말 원한다면 이것도 도움이 되지 않는다는 점에 유의하세요.
 - 온라인 공격(누군가 켜져 있는 여러분의 노트북을 가지고 있는 경우): 파일 암호화를 사용하세요. 크게 두 가지 방법이 있습니다.
    - 암호화된 파일시스템: 계층형 파일시스템 암호화 소프트웨어는 암호화된 블록 장치를 사용하는 대신 파일을 개별적으로 암호화합니다. 복호화 키를 제공하여 이런 파일시스템을 "마운트"하면 그 안의 파일을 자유롭게 둘러볼 수 있습니다. 마운트를 해제하면 그 파일들에 모두 접근할 수 없게 됩니다. 최신 도구로는 [gocryptfs](https://github.com/rfjakob/gocryptfs)와 [eCryptFS](https://www.ecryptfs.org/)가 있습니다. 더 자세한 비교는 [여기](https://nuetzlich.net/gocryptfs/comparison/)와 [여기](https://wiki.archlinux.org/title/Data-at-rest_encryption#Comparison_table)에서 확인할 수 있습니다.
    - 암호화된 파일: 대칭 암호화(`gpg -c` 참고)와 비밀 키로 개별 파일을 암호화하세요. 또는 `pass`처럼 그 키도 공개 키로 암호화하여, 나중에 개인 키를 가진 여러분만 다시 읽을 수 있게 하세요. 세부적인 암호화 설정이 매우 중요합니다!
 - [그럴듯한 부인 가능성](https://en.wikipedia.org/wiki/Plausible_deniability)(무슨 문제라도 있나요, 경찰관님?): 보통 성능이 낮고, 데이터를 잃기도 더 쉽습니다. 실제로 [부인 가능한 암호화](https://en.wikipedia.org/wiki/Deniable_encryption)를 제공하는지 증명하기가 어렵습니다! [여기의 토론](https://security.stackexchange.com/questions/135846/is-plausible-deniability-actually-feasible-for-encrypted-volumes-disks)을 살펴본 뒤, [VeraCrypt](https://www.veracrypt.fr/en/Home.html)(친숙한 옛 TrueCrypt에서 갈라져 나와 유지 관리되고 있는 포크)를 사용해 볼지 생각해 보세요.
 - 암호화된 백업: [Tarsnap](https://www.tarsnap.com/)이나 [Borgbase](https://www.borgbase.com/)를 사용하세요.
    - 공격자가 여러분의 노트북을 손에 넣으면 백업도 삭제할 수 있을지 생각해 보세요!

## 인터넷 보안과 개인정보 보호

인터넷은 _정말_ 무서운 곳입니다. 개방형 WiFi 네트워크는 [무서운](https://www.troyhunt.com/the-beginners-guide-to-breaking-website/) [것입니다](https://www.troyhunt.com/talking-with-scott-hanselman-on/). 사용한 뒤에는 반드시 삭제하세요. 그렇지 않으면 나중에 여러분의 휴대전화가 같은 이름의 네트워크를 찾는다는 사실을 거리낌 없이 알리고 다시 연결할 것입니다!

신뢰할 수 없는 네트워크에 접속한다면 VPN이 쓸모가 _있을 수도_ 있지만, VPN 제공업체를 _많이_ 신뢰하게 된다는 점을 기억하세요. 정말 여러분의 ISP보다 그들을 더 신뢰하나요? VPN을 꼭 쓰고 싶다면 확실히 신뢰하는 제공업체를 이용하고, 아마 유료로 쓰는 편이 좋을 것입니다. 또는 [WireGuard](https://www.wireguard.com/)를 직접 설정하세요. [아주 훌륭합니다](https://web.archive.org/web/20210526211307/https://latacora.micro.blog/there-will-be/)!

[cipherlist.eu](https://cipherlist.eu/)에는 인터넷에 연결되는 여러 애플리케이션의 안전한 설정도 있습니다. 개인정보 보호를 특히 중시한다면 [privacytools.io](https://privacytools.io)도 좋은 자료입니다.

[Tor](https://www.torproject.org/)가 궁금한 분들도 있을 것입니다. Tor는 전 세계적으로 활동하는 강력한 공격자에게 특별히 강한 것은 _아니며_, 트래픽 분석 공격에 취약하다는 점을 기억하세요. 소규모로 트래픽을 숨기는 데는 도움이 될 수 있지만, 개인정보 보호 측면에서 그렇게 큰 이득을 주지는 않습니다. 처음부터 더 안전한 서비스(Signal, TLS + 인증서 고정 등)를 사용하는 편이 낫습니다.

## 웹 보안

그러니까, 웹도 사용하고 싶다고요?
이런, 정말 운을 너무 믿으시는군요.

[HTTPS Everywhere](https://www.eff.org/https-everywhere)를 설치하세요. SSL/TLS는 [매우 중요합니다](https://www.troyhunt.com/ssl-is-not-about-encryption/). 암호화만을 위한 것이 _아니라_, 애초에 올바른 서비스와 통신하고 있는지 검증할 수 있게 해 주기도 하기 때문입니다! 직접 웹 서버를 운영한다면 [테스트하세요](https://www.ssllabs.com/ssltest/index.html). TLS 설정은 [복잡해질 수 있습니다](https://wiki.mozilla.org/Security/Server_Side_TLS). HTTPS Everywhere는 대안이 있다면 HTTP 사이트로 이동하지 않도록 최선을 다합니다. 이것만으로 안전해지지는 않지만 도움이 됩니다. 정말 의심이 많다면 꼭 필요하지 않은 모든 SSL/TLS CA를 차단 목록에 넣으세요.

[uBlock Origin](https://github.com/gorhill/uBlock)을 설치하세요. 이 도구는 광고뿐 아니라 페이지가 시도할 수 있는 온갖 제삼자 통신도 차단하는 [광범위 차단기](https://github.com/gorhill/uBlock/wiki/Blocking-mode)입니다. 인라인 스크립트 같은 것도 차단합니다. 제대로 작동하도록 설정하는 데 시간을 쓸 의향이 있다면 [미디엄 모드](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-medium-mode)나 심지어 [하드 모드](https://github.com/gorhill/uBlock/wiki/Blocking-mode:-hard-mode)를 사용하세요. 이런 모드에서는 설정을 충분히 조정할 때까지 일부 사이트가 _분명_ 작동하지 않을 것이지만, 온라인 보안도 크게 향상됩니다.

Firefox를 사용한다면 [Multi-Account Containers](https://support.mozilla.org/en-US/kb/containers)를 활성화하세요. 소셜 네트워크, 은행, 쇼핑 등을 위한 컨테이너를 각각 만드세요. Firefox는 컨테이너마다 쿠키와 그 밖의 상태를 완전히 분리해서 보관하므로, 한 컨테이너에서 방문한 사이트가 다른 컨테이너의 민감한 데이터를 엿볼 수 없습니다. Google Chrome에서는 [Chrome 프로필](https://support.google.com/chrome/answer/2364824)을 사용하여 비슷한 결과를 얻을 수 있습니다.

## 연습문제

1. PGP로 파일을 암호화하세요.
1. veracrypt로 간단한 암호화 볼륨을 만드세요.
1. GMail, Dropbox, Github 등 가장 민감한 데이터를 가진 계정에 2FA를 활성화하세요.
