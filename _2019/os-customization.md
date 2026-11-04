---
layout: lecture
title: "운영체제 사용자 설정"
presenter: Anish
date: 2019-01-29
order: 3
video:
  aspect: 62.5
  id: epSRVqQzeDo
special: true
translation_status: draft
---

설정 메뉴에서 제공하는 것 외에도 운영체제를 원하는 대로 바꾸기 위해
할 수 있는 일은 많습니다.

# 키보드 재매핑

키보드에는 별로 사용하지 않는 키가 있을 것입니다. 쓸모없는 키로 두는 대신,
유용한 일을 하도록 재매핑할 수 있습니다.

## 다른 키로 재매핑하기

가장 간단한 방법은 키를 다른 키로 재매핑하는 것입니다. 예를 들어 caps lock 키를
별로 사용하지 않는다면 더 유용한 키로 재매핑할 수 있습니다.
Vim 사용자라면, 예를 들어 caps lock을 escape로
재매핑하고 싶을 수 있습니다.

macOS에서는 시스템 환경설정의 키보드 설정에서 일부 재매핑을 할 수 있습니다.
더 복잡한 매핑에는 별도의 소프트웨어가 필요합니다.

## 임의의 명령어로 재매핑하기

키를 꼭 다른 키로만 재매핑해야 하는 것은 아닙니다. 키(또는 키 조합)를
임의의 명령어로 재매핑할 수 있게 해 주는 도구도 있습니다. 예를 들어,
command-shift-t로 새 터미널 창을 열게 할 수 있습니다.

# 숨겨진 운영체제 설정 바꾸기

## macOS

macOS는 `defaults` 명령어로 유용한 설정을 많이 제공합니다. 예를 들어,
숨겨진 애플리케이션의 Dock 아이콘을 반투명하게 만들 수 있습니다.

```shell
defaults write com.apple.dock showhidden -bool true
```

가능한 모든 설정을 모아 둔 단일 목록은 없지만, Mathias Bynens의
[.macos](https://github.com/mathiasbynens/dotfiles/blob/master/.macos)처럼
구체적인 사용자 설정 목록을 온라인에서 찾을 수 있습니다.

# 창 관리

## 타일링 창 관리

[타일링 창 관리](https://en.wikipedia.org/wiki/Tiling_window_manager)는
창을 서로 겹치지 않는 프레임에 배치하는 창 관리 방식입니다.
Linux 기반 운영체제를 사용한다면 타일링 창 관리자를
설치할 수 있습니다. Windows나 macOS 같은 운영체제를 사용한다면
이 동작을 비슷하게 구현해 주는 애플리케이션을 설치할 수 있습니다.

## 화면 관리

여러 화면 사이에서 창을 조작하는 데 도움이 되도록
키보드 단축키를 설정할 수 있습니다.

## 배치

화면에 창을 배치하는 특정한 방식이 있다면,
그 배치를 수동으로 "실행"하는 대신 스크립트로 작성하여
아주 쉽게 배치를 만들 수 있습니다.

# 자료

- [Hammerspoon](https://www.hammerspoon.org/) - macOS 데스크톱 자동화
- [Rectangle](https://rectangleapp.com/) - macOS 창 관리자
- [Karabiner](https://karabiner-elements.pqrs.org/) - 정교한 macOS 키보드 재매핑
- [r/unixporn](https://www.reddit.com/r/unixporn/) - 사람들이 멋지게 꾸민 설정의
스크린샷과 문서

# 연습 문제

1. Caps Lock 키를 더 자주 사용하는 키(예를 들어 Escape나 Ctrl, Backspace)로
   재매핑하는 방법을 알아보세요.

1. 새 터미널 창이나 새 브라우저 창을 여는 사용자 지정 전역 키보드
   단축키를 만드세요.

{% comment %}

TODO

- Bitbar / Polybar
- 클립보드 관리자(스택/검색 가능한 기록)

{% endcomment %}
