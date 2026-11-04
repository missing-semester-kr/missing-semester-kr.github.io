---
layout: lecture
title: "웹과 브라우저"
presenter: Jose
date: 2019-01-31
order: 1
video:
  aspect: 62.5
  id: XpZO3S8odec
special: true
translation_status: draft
---

터미널 외에도 웹 브라우저는 여러분이 상당한 시간을 쓰게 될 도구입니다. 따라서 이를 효율적이고 생산적으로 사용하는 방법을 배워 둘 가치가 있습니다.

## 단축키

브라우저에서 여기저기 클릭하는 것이 가장 빠른 방법은 아닌 경우가 많습니다. 자주 쓰는 단축키에 익숙해지면 장기적으로 큰 도움이 됩니다.

- 링크에서 `Middle Button Click`을 하면 새 탭에서 열립니다.
- `Ctrl+T`는 새 탭을 엽니다.
- `Ctrl+Shift+T`는 최근에 닫은 탭을 다시 엽니다.
- `Ctrl+L`은 검색창의 내용을 선택합니다.
- `Ctrl+F`는 웹페이지 안에서 검색합니다. 자주 사용한다면 정규 표현식 검색을 지원하는 확장 기능이 도움이 될 수 있습니다.


## 검색 연산자

Google이나 DuckDuckGo 같은 웹 검색 엔진은 더 정교하게 웹을 검색할 수 있도록 검색 연산자를 제공합니다.

- `"bar foo"`는 bar foo와 정확히 일치하는 결과만 찾습니다.
- `foo site:bar.com`은 bar.com 안에서 foo를 검색합니다.
- `foo -bar `는 bar가 들어 있는 용어를 검색에서 제외합니다.
- `foobar filetype:pdf`는 해당 확장자의 파일을 검색합니다.
- `(foo|bar)`는 foo 또는 bar가 있는 결과를 검색합니다.

[Google](https://ahrefs.com/blog/google-advanced-search-operators/)과 [DuckDuckGo](https://duck.co/help/results/syntax) 같은 인기 검색 엔진에 대해서는 더 자세한 목록을 찾아볼 수 있습니다.


## 검색창

검색창도 강력한 도구입니다. 대부분의 브라우저는 웹사이트에서 검색 엔진을 알아내 저장할 수 있습니다. 키워드 인자를 수정하면

- Google Chrome에서는 [chrome://settings/searchEngines](chrome://settings/searchEngines)에 있습니다.
- Firefox에서는 [about:preferences#search](about:preferences#search)에 있습니다.

예를 들어 `y SOME SEARCH TERMS`로 youtube에서 바로 검색하도록 만들 수 있습니다.

게다가 도메인을 소유하고 있다면 도메인 등록 업체를 통해 하위 도메인 포워딩을 설정할 수 있습니다. 예를 들어 저는 `https://ht.josejg.com`을 이 강의 웹사이트에 연결했습니다. 이렇게 하면 `ht.`만 입력해도 검색창이 자동으로 완성해 줍니다. 이 설정의 또 다른 장점은 북마크와 달리 모든 브라우저에서 작동한다는 것입니다.

## 개인정보 보호 확장 기능

요즘 웹 서핑은 광고 때문에 꽤 짜증 날 수 있고 추적기 때문에 사생활을 침해받을 수 있습니다. 게다가 좋은 광고 차단기는 대부분의 광고 콘텐츠를 차단할 뿐 아니라 수상하거나 악의적인 웹사이트도 차단합니다. 그런 사이트가 공통 차단 목록에 포함되어 있기 때문입니다. 또한 전송하는 요청 수를 줄여 페이지 로딩 시간을 줄여 주기도 합니다. 몇 가지를 추천합니다.

- **uBlock origin** ([Chrome](https://chrome.google.com/webstore/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm), [Firefox](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/)): 미리 정의된 규칙에 따라 광고와 추적기를 차단합니다. 지역이나 탐색 습관에 맞춰 더 많은 차단 목록을 활성화할 수 있으므로, 설정에서 현재 활성화된 차단 목록을 살펴보는 것도 좋습니다. [웹 곳곳에서 제공하는](https://github.com/gorhill/uBlock/wiki/Filter-lists-from-around-the-web) 필터를 설치할 수도 있습니다.

- **[Privacy Badger](https://privacybadger.org/)**: 추적기를 자동으로 탐지하고 차단합니다. 예를 들어 여러분이 여러 웹사이트를 오갈 때 광고 회사들은 어떤 사이트를 방문하는지 추적하여 여러분의 프로필을 만듭니다.

- **[HTTPS everywhere](https://www.eff.org/https-everywhere)**는 웹사이트에 HTTPS 버전이 있으면 자동으로 그쪽으로 리다이렉션하는 훌륭한 확장 기능입니다.

이런 종류의 부가 기능은 [여기](https://www.privacytools.io/privacy-browser-addons/)에서 더 찾아볼 수 있습니다.

## 스타일 맞춤 설정

웹 브라우저도 *여러분의 컴퓨터*에서 실행되는 소프트웨어이므로, 보통 무엇을 표시하고 어떻게 작동해야 할지에 대한 최종 결정권은 여러분에게 있습니다. 맞춤 스타일이 그 예입니다. 브라우저는 흔히 CSS라고 줄여 부르는 Cascading Style Sheets를 사용하여 웹페이지의 스타일을 어떻게 렌더링할지 결정합니다.

웹사이트를 검사하여 소스 코드에 접근하고, 내용과 스타일을 일시적으로 바꿀 수 있습니다(이것은 웹페이지 스크린샷을 절대로 믿으면 안 되는 이유이기도 합니다).

브라우저가 웹페이지의 스타일 설정을 계속 덮어쓰게 하려면 확장 기능을 사용해야 합니다. 우리가 추천하는 것은 **[Stylus](https://github.com/openstyles/stylus)**([Firefox](https://addons.mozilla.org/en-US/firefox/addon/styl-us/), [Chrome](https://chrome.google.com/webstore/detail/stylus/clngdbkpkpeebahjckkjfobafhncgmne?hl=en))입니다.


예를 들어 강의 웹사이트에 다음과 같은 스타일을 작성할 수 있습니다.


```css

body {
    background-color: #2d2d2d;
    color: #eee;
    font-family: Fira Code;
    font-size: 16pt;
}

a:link {
    text-decoration: none;
    color: #0a0;
}
```

또한 Stylus는 다른 사용자가 작성하여 [userstyles.org](https://userstyles.org/)에 공개한 스타일을 찾을 수 있습니다. 예를 들어 대부분의 인기 웹사이트에는 어두운 테마의 스타일시트가 하나 이상 있습니다. 참고로 Stylish는 사용자 데이터를 유출하는 것으로 밝혀졌으므로 사용하면 안 됩니다. 자세한 내용은 [여기](https://arstechnica.com/information-technology/2018/07/stylish-extension-with-2m-downloads-banished-for-tracking-every-site-visit/)를 참고하세요.


## 기능 맞춤 설정

스타일을 수정할 수 있는 것처럼, 맞춤 JavaScript를 작성하고 [Tampermonkey](https://tampermonkey.net/) 같은 웹 브라우저 확장 기능으로 불러와 웹사이트의 동작도 수정할 수 있습니다.

예를 들어 다음 스크립트는 J와 K 키로 vim처럼 이동할 수 있게 해 줍니다.

```js
// ==UserScript==
// @name         VIM HT
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  Vim JK for our website
// @author       You
// @match        https://hacker-tools.github.io/*
// @grant        none
// ==/UserScript==


(function() {
    'use strict';

    window.onkeyup = function(e) {
        var key = e.keyCode ? e.keyCode : e.which;

        if (key == 74) { // J is key 74
            window.scrollBy(0,500);;
        }else if (key == 75) { // K is key 75
            window.scrollBy(0,-500);;
        }
    }
})();
```

[OpenUserJS](https://openuserjs.org/)와 [Greasy Fork](https://greasyfork.org/en) 같은 스크립트 저장소도 있습니다. 하지만 주의하세요. 다른 사람이 만든 사용자 스크립트는 신용카드 번호를 훔치는 것처럼 사실상 무엇이든 할 수 있으므로 설치하는 것이 매우 위험할 수 있습니다. 스크립트 전체를 직접 읽고, 무엇을 하는지 이해하고, 의심스러운 일을 전혀 하지 않는다는 것을 확실히 알지 못한다면 절대로 설치하지 마세요. 축소되거나 난독화된 코드가 들어 있는 스크립트는 여러분이 그 코드를 읽을 수 없다면 절대로 설치하지 마세요!

## 웹 API

웹 서비스가 애플리케이션 인터페이스, 즉 웹 API를 제공하여 웹 요청으로 서비스와 상호작용할 수 있게 하는 일이 점점 더 흔해지고 있습니다.
이 주제를 더 깊이 소개하는 자료는 [여기](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Introduction)에서 볼 수 있습니다. [공개 API도 많습니다](https://github.com/toddmotto/public-apis). 웹 API는 아주 다양한 이유로 유용할 수 있습니다.

- **조회**. 웹 API로 지도, 날씨, 여러분의 공인 IP 주소 같은 정보를 상당히 쉽게 얻을 수 있습니다. 예를 들어 `curl ipinfo.io`는 공인 IP, 지역, 위치 등에 관한 세부 정보가 담긴 JSON 객체를 반환합니다. 적절히 파싱하면 이런 도구를 명령줄 도구와도 통합할 수 있습니다. 다음 bash 함수는 Google의 자동 완성 API와 통신하여 처음 열 개의 일치 결과를 반환합니다.

```bash
function c() {
    url='https://www.google.com/complete/search?client=hp&hl=en&xhr=t'
    # NB: user-agent must be specified to get back UTF-8 data!
    curl -H 'user-agent: Mozilla/5.0' -sSG --data-urlencode "q=$*" "$url" |
        jq -r ".[1][][0]" |
        sed 's,</\?b>,,g'
}
```

- **상호작용**. 웹 API 엔드포인트는 동작을 실행하는 데도 사용할 수 있습니다. 보통 해당 서비스에서 얻을 수 있는 일종의 인증 토큰이 필요합니다. 예를 들어 다음을 실행하면
`curl -X POST -H 'Content-type: application/json' --data '{"text":"Hello, World!"}' "https://hooks.slack.com/services/$SLACK_TOKEN"` 채널에 `Hello, World!` 메시지가 전송됩니다.

- **파이프로 연결하기**. 웹 API를 제공하는 일부 서비스는 상당히 인기가 있으므로, 흔히 쓰이는 웹 API "연결" 기능이 이미 구현되어 서버까지 포함해서 제공됩니다. [If This Then That](https://ifttt.com/)이나 [Zapier](https://zapier.com/) 같은 서비스가 여기에 해당합니다.


## 웹 자동화

웹 API만으로는 부족할 때가 있습니다. 읽기만 필요하다면 `pup` 같은 HTML 파서나 라이브러리를 사용할 수 있습니다. 예를 들어 Python에는 BeautifulSoup이 있습니다. 하지만 상호작용이나 JavaScript 실행이 필요하다면 이런 방법으로는 부족합니다.

- **WebDriver**. 예를 들어 다음 스크립트는 웹사이트 주소를 입력하는 상호작용을 흉내 내어, 지정된 URL을 Wayback Machine에 저장합니다.

```python
from selenium.webdriver import Firefox
from selenium.webdriver.common.keys import Keys


def snapshot_wayback(driver, url):

    driver.get("https://web.archive.org/")
    elem = driver.find_element_by_class_name('web-save-url-input')
    elem.clear()
    elem.send_keys(url)
    elem.send_keys(Keys.RETURN)
    driver.close()


driver = Firefox()
url = 'https://hacker-tools.github.io'
snapshot_wayback(driver, url)
```


## 연습문제

1. 웹 브라우저에서 자주 사용하는 키워드 검색 엔진을 편집하세요.
1. 위에서 언급한 확장 기능을 설치하세요. 특정 웹사이트에서 uBlock Origin이나 Privacy Badger를 비활성화하는 방법을 알아보세요. 어떤 차이가 보이나요? YouTube처럼 광고가 많은 웹사이트에서 해 보세요.
1. Stylus를 설치하고 제공된 CSS로 강의 웹사이트의 맞춤 스타일을 작성하세요. 자주 쓰이는 프로그래밍 문자 몇 가지입니다. `=   ==   ===   >=   =>   ++   /=   ~=` 글꼴을 Fira Code로 바꾸면 이 문자들은 어떻게 되나요? 더 알고 싶다면 프로그래밍 글꼴 합자를 검색해 보세요.
1. 여러분의 도시나 지역의 날씨를 가져오는 웹 API를 찾아보세요.
1. [Selenium](https://www.selenium.dev/documentation/) 같은 WebDriver 소프트웨어로 브라우저에서 자주 하는 반복적인 수동 작업을 자동화하세요.


