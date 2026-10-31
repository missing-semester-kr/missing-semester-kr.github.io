---
layout: lecture
title: "Metaprogramming"
description: >
  빌드 시스템, 의존성 관리, 테스트와 지속적 통합을 배웁니다.
details: build systems, dependency management, testing, CI
thumbnail: /static/assets/thumbnails/2020/lec8.png
date: 2020-01-27
ready: true
video:
    aspect: 56.25
    id: _Ms1Z4xfqv4
translation_status: complete
---

우리가 메타프로그래밍(metaprogramming)이라고 할 때, 이 용어는 무얼 뜻하는 것일까요?  
이는 코드를 작성하는 것이나 더 효율적으로 작업하는 것보다는, _"프로세스(process)"_ 에 더 관련된 것들을 두루 일컫는 총칭이라고 할 수 있습니다.  
이 강의에서, 우리는 코드를 빌드하고 테스트하며, 의존성([dependency](https://en.wikipedia.org/wiki/Dependency_injection#Constructor_injection))들을 관리하기 위한 체계들을 살펴볼 것입니다.  
이러한 것들은 학생으로서의 여러분의 일상에서 마치 별로 중요하지 않은 것처럼 보일 지도 모르지만, 여러분이 인턴십을 통해서 더 큰 코드베이스([code base](https://en.wikipedia.org/wiki/Codebase))와 상호작용하거나 일단 "실제 세계"에 발을 들이게 되면, 이를 어디서든 보게 될 것입니다.  
이 강의의 목적을 위해 우리가 사용하고 있는 정의는 아니지만, 우리는 "메타프로그래밍"이 "[프로그램들을 대상으로 작동하는 프로그램들](https://en.wikipedia.org/wiki/Metaprogramming)" 또한 의미할 수 있음에 유의해야 합니다.

# [빌드 시스템(Build systems)](https://web.archive.org/web/20240913174421/http://developinghappiness.com/?p=26)

문서를 [Latex(레이텍 또는 라텍)](https://namu.wiki/w/LaTeX)으로 작성할 때, 문서를 작성하기 위해 실행해야 하는 명령어는 무엇입니까?  
[벤치마크](https://namu.wiki/w/%EB%B2%A4%EC%B9%98%EB%A7%88%ED%81%AC)를 실행하고, 그 결과를 그래프로 그리고, 그 그래프를 문서에 삽입할 때는요?  
또는 사용하고 있는 클래스 안에 제공되어 있는 코드를 컴파일하고 테스트를 실행할 때는 어떤 명령어를 사용합니까?

대부분의 프로젝트들에는, 그것들이 코드를 포함하든 포함하지 않든, "**빌드 프로세스**"라고 하는 것이 있습니다. 이는 인풋(Input)에서 아웃풋(Output)으로 가기 위해서 수행할 필요가 있는 몇몇 순차적인 작업들을 뜻합니다.  
종종, 그 프로세스에는 많은 단계들과 많은 [브랜치(branch)](https://backlog.com/git-tutorial/kr/stepup/stepup1_1.html)들이 있을 수도 있습니다.  
이 도표를 생성하기 위해서 이걸 하고, 저 결과들을 생성하기 위해 저걸 하고, 최종 문서를 만들어 내기 위해서 또 뭔가 다른 걸 해야하는 것처럼 말이죠.  
우리가 이 강의에서 보아온 아주 많은 것들과 마찬가지로, 여러분은 제일 처음으로 이러한 골칫거리에 마주한 사람들이 아니고, 운 좋게도 여러분들을 도와줄 많은 도구들이 존재합니다!

이것들은 보통 "**빌드 시스템(build systems)**"이라고 불리고, _많은_ 것들이 있습니다.  
여러분이 무엇을 사용하는지는 당면한 과제, 선호하는 프로그래밍 언어, 그리고 프로젝트의 규모에 달려 있습니다.  
그렇지만 핵심적인 면에서는 모두 아주 비슷합니다.
여러분은 어떤 것으로부터 다른 것을 만들어 내기 위한 많은 _의존성(dependencies)_, _타겟_, 그리고 _규칙_ 들을 정의하게 됩니다.  
여러분이 빌드 시스템에게 여러분이 특정 타겟을 원한다는 것을 전하면, 빌드 시스템이 하는 일은 그 타겟의 [이행성을 띠는(transitive)](https://terms.naver.com/entry.nhn?docId=856345&cid=50376&categoryId=50376) 모든 의존성들을 찾아내고서는, 최종 타겟이 만들어지는 때까지 계속해서 중간의 타겟들을 만들어내는 규칙들을 적용하는 것입니다.
이상적으로는, 빌드 시스템은 의존성이 바뀌지 않은 타겟들과, 이전의 빌드에 의한 결과가 이미 사용가능한 경우에 대해서는, 불필요하게 규칙들을 실행하지 않는 방식으로 이를 수행합니다.

`make`는 가장 흔한 빌드 시스템 중 하나이고, 거의 모든 [UNIX](https://namu.wiki/w/%EC%9C%A0%EB%8B%89%EC%8A%A4)-기반 컴퓨터에 설치되어 있습니다. `make`는 결점이 있기는 하지만, 간단한 ~ 중간 정도의 복잡성을 지닌 프로젝트들에는 꽤 잘 작동합니다.  
여러분이 `make`를 실행하면 이것은 현재 디렉토리에 있는 `Makefile`이라고 하는 파일을 참조합니다.  
모든 타겟들과 의존성들, 그리고 규칙들은 그 파일 안에 정의되어 있습니다. 함께 하나 살펴보죠:

```make
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@
```

이 파일 안의 각각의 명령(directive)은 우변을 이용하여 좌변을 만들어 내는 방법에 대한 규칙입니다.
다르게 말하면, 우변에 이름지어진 것들은 의존성들이고, 좌변은 타겟입니다.  
들여쓰기된 블럭은 의존성들로부터 타겟을 만들어 내기 위한 프로그램들의 순서입니다.  
`make`에서는, 첫번째 명령(directive)은 최종 타겟의 초기값(default) 또한 정의합니다. 만약 여러분이 어떠한 인자도 없이 `make`를 실행한다면, 이 초기값이 바로 `make`가 만들어낼 최종 타겟이 되는 것입니다.  
대신에 여러분이 `make plot-data.png`와 같은 무언가를 실행시킨다면 `make`는 그 타겟을 빌드할 것입니다.

우변의 규칙(rule)에 있는 `%`는 하나의 "패턴"이고, 이것은 좌변과 우변에 있는 같은 문자열을 매치시킬 것입니다.  
예를 들면, 만약 타겟 `plot-foo.png`가 요청되면 `make`는 `foo.dat`과 `plot.py` 의존성들이 있는지 찾을 것입니다.  
자 이제 만약 우리가 `make`를 빈 소스 디렉토리에서 실행하면 무슨 일이 일어나는지 살펴봅시다.

```console
$ make
make: *** No rule to make target 'paper.tex', needed by 'paper.pdf'.  Stop.
```

`make`는 유용하게도, `paper.pdf`를 빌드하기 위해서는 `paper.tex`가 필요하며, 그 파일을 만들기 위한 방법을 알려주는 규칙을 가지고 있지 않다는 사실을 말해주고 있습니다.  
한 번 만들어 봅시다!

```console
$ touch paper.tex
$ make
make: *** No rule to make target 'plot-data.png', needed by 'paper.pdf'.  Stop.
```

흠, 흥미롭네요, `plot-data.png`를 만드는 규칙은 *존재*합니다. 하지만 그 규칙은 패턴 규칙(pattern rule)입니다.(구체적으로 어떻게 타겟 파일을 만들어내야 하는지 명시되어 있지 않음)  
소스 파일(`data.dat`)이 존재하지 않기 때문에, `make`는 단지 파일을 만들 수 없다고만 이야기하고 있습니다.  
한 번 모든 파일을 다 만들어 보죠:

```console
$ cat paper.tex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
\includegraphics[scale=0.65]{plot-data.png}
\end{document}
$ cat plot.py
#!/usr/bin/env python
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('-i', type=argparse.FileType('r'))
parser.add_argument('-o')
args = parser.parse_args()

data = np.loadtxt(args.i)
plt.plot(data[:, 0], data[:, 1])
plt.savefig(args.o)
$ cat data.dat
1 1
2 2
3 3
4 4
5 8
```

이제 `make`를 실행해보면 어떻게 될까요?

```console
$ make
./plot.py -i data.dat -o plot-data.png
pdflatex paper.tex
... lots of output ...
```

이제 보면, `make`가 PDF 파일을 하나 만들어 주었습니다. `make`를 다시 한 번 더 실행시키면 어떻게 될까요?

```console
$ make
make: 'paper.pdf' is up to date.
```

`make`가 아무 일도 하지 않았습니다. 왜냐고요? 왜냐면 그럴 필요가 없었기 때문이죠.
`make`는 모든 이전에 빌드된 타겟들이 나열된 의존성들에 대해서 최신의 상태라는 것을 체크했습니다.  
우리는 `paper.tex`를 수정하고 `make`를 재실행함으로써 이를 테스트해볼 수 있습니다:

```console
$ vim paper.tex
$ make
pdflatex paper.tex
...
```

`make`가 `plot.py`는 다시 실행하지 _않았다는_ 사실에 주목하세요, 그럴 필요가 없었으니까요 ( `plot-data.png`의 의존성들 중 어느 것도 바뀌지 않았기 때문이죠.)

# 의존성 관리(Dependency management)

더 거시적인 수준에서는, 여러분의 소프트웨어 프로젝트들은 또 다른 프로젝트들에 의존하고 있을 수 있습니다.  
여러분은 설치된 프로그램(예: `파이썬(python)`), 시스템 패키지(예: `openssl`), 또는 여러분이 사용하는 프로그래밍 언어 안에 있는 라이브러리(예: `matplotlib`)에 의존하고 있을지도 모릅니다.  
요즘에는, 대부분의 의존성들이 _레포지토리(repository)_ 를 통해서 가능할 텐데, 레포지토리는 단일한 공간에서 수많은 의존성들을 관리하고, 그것들을 설치할 수 있는 편리한 방법을 제공합니다.  
레포지토리 예시:

-   우분투 시스템 패키지(Ubuntu system packages)를 위한 [우분투 패키지 레포지토리(the Ubuntu package repository)](https://packages.ubuntu.com) - `apt` [툴(tool)](http://taewan.kim/tip/apt-apt-get/)을 통해 접근할 수 있음
-   프로그래밍 언어 루비(Ruby) 라이브러리들을 위한 [루비젬(RubyGem)](https://rubygems.org)
-   파이썬(Python) 라이브러리들을 위한 [파이파이(PyPI)](https://pypi.org)
-   [아치 리눅스](https://www.archlinux.org) 사용자-기여 패키지들을(Arch Linux user-contributed packges) 위한 [아치 사용자 레포지토리(Arch User Repository)](https://aur.archlinux.org)

이러한 레포지토리들과 상호작용하는 정확한 방법은 레포지토리마다 다르고 툴마다 다르기 때문에, 우리는 이 강의에서 특정 레포지토리의 너무 세부적인 사항까지는 들어가지 않을 것입니다. 우리가 _다룰_ 것은 레포지토리들이 모두 사용하는 몇몇 공통적인 용어입니다.  
그 중 첫 번째는 _버저닝(versioning)_ 입니다.  
다른 프로젝트들이 의존하는 대부분의 프로젝트들은 모든 배포(release)와 함께 _버젼 넘버(version number)_ 를 발행합니다. 보통 8.1.3 또는 64.1.20192004 와 같은 것입니다. 항상 그런 것은 아니지만, 보통 그것들은 숫자입니다.  
버전 넘버는 많은 목적에 기여하는데, 그 중 가장 중요한 것 중 하나는 소프트웨어가 계속 작동할 것이라는 것을 보장하는 것입니다.  
예를들면 제가 특정한 함수의 이름을 바꾼 라이브러리의 새 버전을 배포한다고 한 번 상상해 보세요. 제가 그 업데이트를 배포한 이후에 누군가가 제 라이브러리에 의존하는 소프트웨어를 빌드하려고 시도한다면, 그 빌드는 실패할 지도 모릅니다, 왜냐하면 그 빌드는 더 이상 존재하지 않는 함수를 호출할 수도 있기 때문이죠!  
버저닝(versioning)은 프로젝트가 다른 프로젝트의 어떤 특정한 버전, 또는 버전들의 범위에 의존하는지를 나타내도록 함으로써 이러한 문제를 해결하려 시도합니다.  
그런 식으로, 비록 라이브러리에 변화가 생길 지라도, 그 라이브러리에 의존하는 소프트웨어는 그 라이브러리의 더 예전 버전을 사용함으로써 계속 빌드할 수 있게 됩니다.

그렇긴 하지만 완벽하진 않죠!  
만약 제가 보안 업데이트를 하는데, 그 새로운 버전이 제 라이브러리의 퍼블릭 인터페이스(public interface, "[API](https://ko.wikipedia.org/wiki/API)")를 바꾸지 _않고_, 오래된 버전에 의존했던 어떤 프로젝트도 즉시 사용할 수 있는 것이라면 어떨까요?
여기서 서로 다른 숫자들을 가진 버전이 나오는 것입니다.
각각의 정확한 의미는 프로젝트들마다 다르지만, 상대적으로 공통적인 기준은 [시멘틱 버저닝(Semantic Versioning)](https://velog.io/@slaslaya/Semantic-Versioning-2.0.0-MAJOR-MINOR-PATCH와-명세에-관하여)입니다.  
시맨틱 버저닝에서 모든 버전 넘버는 다음의 형식을 따릅니다: **major.minor.patch**  
규칙은 다음과 같습니다:

-   만약 새로운 버전 배포가 API를 바꾸지 않는다면, **patch** 버전을 증가시킨다.
-   만약 이전의 버전들과 호환성을 유지하면서 API에 _추가_ 되는 것이 있다면, **minor** 버전을 증가시킨다.
-   만약 이전의 버전들과 호환이 되지 않는 방식으로 API를 바꾸게 된다면, **major** 버전을 증가시킨다.

이 방식은 이미 몇몇 이점을 제공합니다.
자, 만약 저의 프로젝트가 여러분의 프로젝트에 의존하고 있다면, 제가 그 프로젝트를 빌드할 때 이용했던 것과 같은 **major** 버전의 최신 배포 버전을 사용하는 것이 _안전해야_ 합니다, 제 프로젝트에서 사용하고 있는 **minor** 버전이 적어도 빌드할 그 당시의 것 이상인 한 말이죠.  
달리 말하면, 만약 제가 여러분의 라이브러리 버전 `1.3.7`에 의존하고 있다면, `1.3.8`, `1.6.1`, 심지어는 `1.3.0`을 이용해서 빌드하는 것 또한 _안전해야_ 합니다. 버전 `2.2.4`는 아마 괜찮지 않을 겁니다, 왜냐면 **major** 버전이 증가되었기 때문이죠.  
우리는 파이썬의 버전 넘버에서 시맨틱 버저닝의 예시를 확인할 수 있습니다. 여러분 중 다수는 아마 파이썬 2와 파이썬 3의 코드가 함께 잘 어울릴 수 없다는 것을 알고 있을 겁니다, **major** 버전 증가의 이유죠. 마찬가지로, 파이썬 3.5를 위해 쓰인 코드는 파이썬 3.7 버전에서는 실행되지만 아마 3.4에서는 실행되지 않을지도 모릅니다.

의존성 관리 시스템(dependency management systems)을 이용해 작업하면, 여러분은 _잠금 파일(lock files)_ 의 개념 또한 접할 수도 있습니다.
잠금 파일(lock file)이란, 그아먈로 여러분이 각각의 의존성에 _현재_ 의존하고 있는 정확한 버전을 나열하는 파일입니다. 보통, 여러분은 의존성들의 더 새로운 버전으로 업그레이드 하기 위해서는 명시적으로 업데이트 프로그램을 실행시킬 필요가 있습니다.
여기에는 많은 이유가 있는데, 쓸데없는 리컴파일(recompile)을 피하고, 재현가능한(reproducible) 빌드를 하고, (고장났을 수도 있는) 최신 버전으로 자동 업데이트를 하지 않기 위함이죠.  
이러한 종류의 의존성 잠금(dependency locking) 중 극단적인 경우가 _벤더링(vendoring)_ 인데, 여러분의 모든 의존성들의 모든 코드를 여러분의 프로젝트 안에 복사하는 겁니다. 이렇게 하면 여러분은 프로젝트에 대한 어떤 변화에 대해서도 통제력을 가지게 되고 프로젝트에 자기 나름의 변화들을 도입할 수 있지만, 시간이 지나면서 생기는 어떤 업데이트도 업스트림 관리자(upstream maintainer)로부터 명시적으로 풀(pull)해줘야 한다는 것을 의미하기도 합니다.

# [지속적 통합](https://www.redhat.com/ko/topics/devops/what-is-ci-cd) 시스템 (Continuous integration systems)

여러분이 더 큰 프로젝트에서 작업하게 될 수록, 프로젝트에 변화를 줄 때마다 추가적으로 수행해야 하는 일들이 있다는 것을 알게 될 것입니다.
여러분은 아마 기록문서(documentation)의 새로운 버전을 업로드하고, 컴파일된 버전을 어딘가에 업로드하고, 코드를 파이파이(pypi)에 배포하고, [테스트 스위트(test suite)](https://m.blog.naver.com/PostView.nhn?blogId=netrance&logNo=110184403382&proxyReferer=https:%2F%2Fwww.google.com%2F)를 실행하고, 별의 별 것들을 다 해야 할 수도 있습니다.  
아마 누군가가 깃허브(GitHub)에서 여러분에게 풀 리퀘스트(pull request;PR)을 보낼 때마다, 여러분은 그 코드가 스타일 체크(style check)되기를 바라고 성능 테스트(benchmark)를 실행하고 싶을 지도 모르겠죠?  
이러한 필요가 생길 때가 바로 지속적 통합(continuous integration)을 고려해 봐야 할 때입니다.

지속적 통합(continuous integration), 또는 CI는 "코드가 바뀔 때마다 실행되는 것"에 대한 포괄적인 용어입니다, 그리고 다양한 종류의 CI를 제공하는 많은 회사들이 있습니다 (흔히 오픈소스 프로젝트인 경우 무료).
큰 것들 중에는 트레비스 CI(Travis CI), 아주르 파이프라인(Azure Pipelines), 그리고 깃허브 액션즈(GitHub actions)가 있습니다.
이것들은 거의 같은 방식으로 작동합니다:  
여러분이 여러분의 레포지토리에 다양한 일들이 발생할 때 어떤 일이 일어나야 하는지에 대해 설명하는 파일을 그 레포지토리에 추가합니다.
단연코 가장 흔한 것은 "누군가 코드를 푸쉬(push)하면, 테스트 스위트(test suite)를 실행한다"와 같은 규칙입니다. 그 이벤트가 촉발될 때, CI 제공자는 가상머신(virtual machine) 혹은 그 이상의 것을 돌리고, 여러분이 작성한 "레시피" 안에 있는 명령들을 실행시키고는, 보통 그 다음에는 결과들을 어딘가에 적어둡니다.
여러분은 테스트 스위트가 통과되는 것이 중지될 때 여러분이 통지를 받도록, 또는 테스트가 통과되는 한 여러분의 레포지토리에 작은 뱃지가 나타나도록 설정을 할 수도 있을 것입니다.

CI 시스템의 예시로서, 수업 웹사이트가 깃허브 페이지스(GitHub Pages)를 이용해서 준비되어 있습니다.  
페이지스(Pages)는 `master` 브랜치에 올라오는 모든 푸쉬(push)에 대해서 제킬 블로그 소프트웨어([Jekyll](http://jekyllrb-ko.github.io))를 실행하고 빌드된 사이트가 특정 깃허브 도메인에서 사용 가능하도록 만드는 CI 동작입니다.
이게 우리가 웹사이트를 업데이트하는 걸 굉장히 쉽게 만들어 주죠!  
우린 그냥 로컬에서 변화를 만들고, 깃(git)을 통해서 커밋(commit)하고, 그 다음에 푸쉬(push)할 뿐이에요. 나머지는 CI가 책임져 주는 거죠.

## 테스팅에 관한 간단한 여담

대부분의 큰 소프트웨어 프로젝트에는 "테스트 스위트(test suite)가 딸려 있습니다. 여러분은 이미 테스팅의 일반적인 개념에 익숙할지도 모르지만, 우리는 여러분이 야생에서 마주할 수도 있는 몇몇 테스팅 접근법들과 테스팅 용어들에 대해 짧게 언급하는 게 나을 거라 생각했습니다:

-   테스트 스위트(Test suite): 모든 테스트들에 대한 총칭
-   유닛 테스트(Unit test, 단위 테스트): 특정 기능만 떼어놓고 테스트하는 "마이크로-테스트(micro-test)"
-   통합 테스트(Integration test): 그 달라진 기능이나 요소들이 _함께_ 제대로 작동하는 지 확인하기 위해 시스템의 더 큰 부분을 실행하는 "매크로-테스트(macro-test)"
-   회귀 테스트(Regression test): _이전에_ 버그를 일으켰던 특정 패턴을 구현함으로써 그 버그가 다시 나타나지 않는다는 것을 보장하기 위한 테스트
-   [모킹(Mocking)](https://www.daleseo.com/python-unittest-mock/): 관련 없는 기능을 테스팅하는 것을 피하기 위해 가짜 구현(fake implementation)으로 함수, 모듈, 또는 타입을 대체하는 것. 예를 들면, "네트워크를 모킹"하거나 "디스크를 모킹"할 수 있다.

# 연습

1. 대부분의 메이크파일(Makefile)은 `clean`이라고 하는 타겟을 제공합니다. 이건 `clean`이라는 파일을 생성하기 위한 것이 아니라 make에 의해서 다시 빌드될 수 있는 모든 파일들을 싹 지우기 위한 것입니다. 모든 빌드 스텝들을 "되돌리기(undo)"할 수 있는 방법이라고 생각하시면 되요.  
   위에 있는 `paper.pdf` 메이크파일을 위한 `clean` 타겟을 구현해보세요. [포니](https://pinocc.tistory.com/131) 타겟(phony target)을 구현해야할 겁니다. 아마 [git ls-files](https://git-scm.com/docs/git-ls-files) [명령어](https://m.blog.naver.com/PostView.nhn?blogId=cyberpass&logNo=221037298316&proxyReferer=https:%2F%2Fwww.google.com%2F)가 유용할 겁니다.  
   아주 흔한 다른 `make` 타겟들은 [여기](https://www.gnu.org/software/make/manual/html_node/Standard-Targets.html#Standard-Targets)에 나열되어 있습니다.
2. [Rust's build
   system](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html)에서 의존성들에 대한 버전 요구사항들을 명시하는 다양한 방법들을 살펴보세요. 대부분의 패키지 레포지토리가 비슷한 문법을 지원합니다. 각각의 방법에 대해서 그 특정한 종류의 요구사항이 말이 되는 활용 사례를 제시해 보세요.
3. 깃(Git)은 그 자체로 단순한 CI 시스템으로서 기능할 수 있습니다. 어떤 깃 레포지토리(git repository)에든 있는 `.git/hooks` 안에서, 특정 행위가 일어날 때 스크립트(script)로서 실행되는 (현재 비활성인) 파일들을 발견할 수 있을 것입니다. `make paper.pdf`를 실행시키고 `make` 명령어가 실패하면 커밋을 거부하는 [프리커밋 훅(`pre-commit` hook)](https://git-scm.com/book/ko/v2/Git맞춤-Git-Hooks)을 작성해보세요.
   이러면 빌드가 안 되는 버전의 paper를 어떤 커밋도 가지고 있지 않도록 방지해줄 것입니다.
4. [깃허브 페이지스(GitHub Pages)](https://pages.github.com/)를 사용해서 자동으로 게재되는 단순한 페이지를 하나 만들어 보세요. 레포지토리에 [깃허브 액션(GitHub Action)](https://github.com/features/actions)을 추가해서 레포지토리에 있는 모든 셸(shell) 파일들에 대해 `shellcheck`을 실행해보세요.(여기에 [그 방법 중 하나](https://github.com/marketplace/actions/shellcheck)가 있습니다.) 잘 작동하는 지 한 번 확인해보세요!
5. [자신만의 깃허브 액션(GitHub Action)을 빌드](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/building-actions)해서 레포지토리에 있는 모든 `.md` 파일들에 대해서 [`proselint`](https://github.com/amperser/proselint) 또는 [`write-good`](https://github.com/btford/write-good)을 실행해보세요. 그걸 여러분의 레포지토리에서 활성화시키고, 안에 오타가 포함된 풀 리퀘스트(pull request; PR)를 보내서 잘 작동하는 지 확인해 보세요.
