---
layout: lecture
title: "Version Control (Git)"
description: >
  Git의 데이터 모델과 버전 관리 및 협업을 위한 Git 사용법을 배웁니다.
thumbnail: /static/assets/thumbnails/2020/lec6.png
date: 2020-01-22
ready: true
video:
  aspect: 56.25
  id: 2sjqTHE0zok
translation_status: complete
---

버전 컨트롤 시스템(VCSs)은 소스코드(또는 기타 파일이나 폴더들의 모음)의 변화를 추적하기 위해 사용되는 도구입니다.
이름에서 알수 있듯이 이 도구는 변경내역을 유지시켜주며, 나아가 협업을 용이하게 합니다.
VCSs는 일련의 스냅샷에서 폴더와 폴더안의 내용물들의 변화를 추적합니다. 각각의 스냅샷은 최상위 폴더 내의 파일 / 폴더의 모든 상태를 캡슐화 합니다.
VCSs는 누군가가 만들었던 각가의 스냅샷과 스냅샷에 연결된 메시지 등의 메타데이터를 유지합니다.

왜 버전 관리는 유용할까요? 혼자 작업을 하더라도 프로젝트의 지난 스냡샷을 보며 왜 변경사항 생겼는지를 기록하고, 병렬 개발 분기에서 작업을 하는 등 많은 일들을 할수 있게 해줍니다. 
다른 사람들과 일을 할때 버전관리는 다른 사람들의 변화를 볼수있는 귀중한 도구입니다. 게다가 동시 개발에서의 갈등을 해결해 줍니다.

현대 VCSs 는 아래와 같은 질문에 대한 답변을 쉽게 할수 있게 해줍니다. 

- 누가 이 모듈을 작성하였는가
- 언제 특정파일의 특정행이 수정되었는가. 누구에 의한 것인가? 왜 수정 되었는가?
- 지난 1000번이 넘는 수정들 중 언제 / 왜 특정 단위 태스트가 중지 되었었는가.


다른 VCSs가 존재하지만, **Git**은 사실상 버전관리 도구의 표준 입니다.
이 [XKCD comic](https://xkcd.com/1597/) 은 깃의 명성을 보여줍니다.

![xkcd 1597](https://imgs.xkcd.com/comics/git.png)

깃 인터페이스의 허술한 추상화로 인하여 하향식으로 방식으로 Git을 배우는것은(Git의 인터페이스/ 명령어 인터페이스 부터 시작하는것) 많은 혼란을 줄수 있있습니다.
하향식 학습은 몇가지 명령어를 외우고, 그것들을 마법주문처럼 생각하며, 문제가가 생길때마다 위의 만화에서의 접근방법을 따르게 할수 있습니다.

깃은 분명 투박한 인터페이스를 가지고 있지만, 그 이면의 디자인과 아이디어들은 아름답습니다.
나쁜 인터페이스는 암기 되어져야 하지만, 아름다운 디자인은 이해되어 집니다.
이러한 이유로 우리는 깃에 대한 데이터 모델부터 이후 명령어 인터페이스를 포함하는 상향식 설명을 제공합니다.
한번 데이터 모델을 이해하면 아래에 있는 데이터 모델을 잘 다루는 방법의 측면에서 명령어들을 더 잘 이해할수 있습니다.


# 깃의 데이터 모델

버전 관리를 위한 많은 필수적 접근법이 있습니다. 
갓은 기록 유지, 분기를 지원, 협업 기능 과 같이 버전과 관리에 대한 모든 우수한 기능을 사용할수 있는 매우 정교한 모델을 가지고있습니다. 

<span id="snapshots"></span>

## 스냅샷

깃은 최상위 디렉토리 내의 폴더와 파일 목록에대한 기록을 일련의 스냅샷으로 모델링 합니다.
깃에서는 파일은 "blob" 이라고 하며 그것은 단지 바이트 묶음임입니다
디렉토리는 "tree" 라고 하며 이름을 blob 또는 tree 에 매핑합니다(디렉토리는 다른 디렉토리들을 포함할수 있습니다.).
스냅샷은 추적중인 최상위 트리입니다. 
예를 들면 아래와 같은 트리를 가질수 있습니다. :

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

최상위 트리에는 "foo" 트리("bar.txt" blob을 하나의 요로소로 포함한 자신)와 "baz.txt" blob 두 요소가 있습니다. 

## Modeling history: relating snapshots
어떻게 버전 제어 시스템을 스냅샵과 연관시켜야할까요? 간단한 모델 하나는 선형 히스토리를 갖는 것입니다. 
히스토리는 시간순으로 정렬된 스냅 샷의 목록입니다. 
여러 이유로 Git은 이와 같은 간단한 모델을 사용하지 않습니다.

Git에서 히스토리는 스냅 샷의 유향 비순환 그래프 (DAG)입니다. 
멋진 수학 용어처럼 들릴지 모르지만 겁 먹지 마세요. 
이 모든 것은 선행하는 "부모" 집합을 참조하는 Git의 각각의 스냅 샷을 뜻합니다.
두 개의 병렬 개발 분기를 결합 (merge)하는것처럼 하나의 스냅샹은 여러 부모로부터 내려올것이기 때문에
히스토리는 단일 부모 보다 부모 집합을 가집니다(선형 기록의 경우).

Git은 이러한 스냅 샷을 "커밋"이라고 부릅니다. 
커밋 기록을 시각화 해본다면 다음과 같습니다. :

```
o <-- o <-- o <-- o
            ^  
             \
              --- o <-- o
```

위의 ASCII문자로 만든 그림에서 `o`는 개별 커밋 (스냅 샷)에 해당합니다. 
화살표는 각 커밋의 부모를 가리 킵니다 ("이후"가 아니라 "이전" 관계). 
세 번째 커밋 후 히스토리는 두 개의 개별 분기로 분기됩니다. 
예를 들어, 서로 독립되어 병렬로 개발고있는 두 개의 개별 기능에 해당 될 수 있습니다. 
앞으로 이러한 분기를 병합하여 두 기능을 모두 통합하는 새 스냅 샷을 생성 할 수 있으며, 
만든어진 새로은 히스토리는 아래와 같습니다. 
새로 생성 된 merge 커밋은 굵게 굵은 글자로 보여집니다.:

<pre class="highlight">
<code>
o <-- o <-- o <-- o <---- <strong>o</strong>
            ^            /
             \          v
              --- o <-- o
</code>
</pre>

Git의 커밋은 변경할 수 없습니다. 그렇다고 실수를 고칠 수 없다는 의미는 아닙니다. 
커밋 히스토리에 대한 "편집"은 실제로 완전히 새로운 커밋을 생성하고, 참조들(아래쪽을 보세요)이 
새 커밋을 가리 키도록 업데이트됩니다.

## Data model, as pseudocode

의사 코드로 작성된 Git의 데이터 모델을 보는 것은 유용할수 있습니다.

```
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | blob>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

깔끔하고 간결한 히스토리 모델입니다.

## Objects and content-addressing

object는 blob, tree, commit 입니다.

```
type object = blob | tree | commit
```
Git 데이터 저장소에서 모든 객체는 [SHA-1 해시](https://en.wikipedia.org/wiki/SHA-1)로 콘텐츠 주소가 지정됩니다.


```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blob, 트리 및 커밋은 모두 객체이며 이와 같은 방식으로 통합됩니다. 
다른 객체를 참조 할 때 디스크상의 표현에는 _포함_ 되지 않지만 해시에 의해 참조됩니다.

예를 들어 [위의](#snapshots) 디렉토리 구조의 트리는 다음과 같습니다. (`git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d`를 사용하여 시각화)

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

트리 자체에는 baz.txt(blob) 과 foo (트리)에 대한 포인터가 포함되어 있습니다 . 
해시에 의해 `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85` 주소가 지정된 baz.txt 를 보면 다음과 같은 결과가 나타납니다.

```
git is wonderful
```

## References

이제 모든 스냅 샷은 SHA-1 해시로 식별 할 수 있습니다. 인간에게는 40 개의 16 진수 문자열을 잘 기억하지 못하기떄문에 불편합니다.

Git은 이문제를 해결하기 휘하여 SHA-1 해시에 "references" 라는 인간이 읽을수있는 이름을 사용합니다. 
References는 커밋에 대한 포인터입니다. 변경이 불가능한 객체와는 다르게 references는 변경 가능합니다 (새 커밋을 가리 키도록 업데이트 할 수 있음). 
예를 들어 'master' references는 일반적으로 주요 개발 분기의 최신 커밋을 가리 킵니다.

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

이를 통해 Git은 긴 16 진수 문자열 대신 "master"와 같이 사람이 읽을 수있는 이름을 사용하여 히스토리의 특정 스냅 샷을 참조 할 수 있습니다.

추가로 종종 히스토리 에서 "현재 위치"라는 개념을 웝합니다. 그래서 무엇과 연관 되었는지(우리가 어떻게 `parents` 커밋 필드들 설정하였는지) 알기 위하여 
새 스냅 샷을 남깁니다. 이떄 Git에서는 "현재 위치"는 "HEAD" 라는 특수 references를 사용합니다.
## Repositories

마침내, 우리는 Git 저장소가 `객체` 와 `references` 테이터 라는것을 (대략)정의 할 수 있습니다.

디스크에서 모든 Git 저장소는 객체와 references이며 이것은 Git의 데이터 모델의 전부입니다. 
모든 git명령은 객체를 추가하고 참조를 추가 / 수정 하는 커밋 DAG의 조작에 의해 매핑됩니다.

명령을 입력 할 때마다 명령어가 만들어내는 기본그래프 자료구조의 조작에 대해 생각해보세요.
반대로 커밋 DAG에 특정 종류의 변경을 시도하는 경우, 
예를 들어 "커밋되지 않은 변경 사항을 무시하고 'master' reference가 커밋`d83f9e`을 가르키도록 만들어 보세요"
이를 수행하는 명령은 있을 수 있습니다 (예 : git checkout master; git reset --hard 5d83f9e).


# Staging area

Staging area는 데이터 모델과 독립된 또 다른 개념이지만 커밋을 생성하는 인터페이스의 일부입니다.

위에서 설명한 것처럼 스냅샷을 구현하기위해 생각해볼수 있는 한가지 방법은 작업 디렉토리의 _현재 상태_를 기준으로 새로운 스냅샷을 생성하는 "create snapshot" 명렁어를 사용하는 것입니다.

일부 버전 제어 도구는 이와 같이 작동하지만 Git은 작동하지 않습니다. 

우리는 깨끗한 스냅 샷을 원하며, 현재 상태에서 스냅 샷을 만드는 것이 항상 이상적인 것은 아닙니다.

예를 들어 이러한 두 개의 개별 기능을 구현한후 첫번째 기능은 첫 번째 기능으로 소개하고 두 번째 기능은 두 번째 기능으로 소개하는 두 개의 개별 커밋을 생성하려고 하는 시나리오를 생상해보세요.
또는 버그 수정과 동시에 코드 전체에 디버깅 명세 문이 추가 되는 시나리오를 상상해보세요.
;모든 인쇄 문을 삭제하는 동안 버그 수정을 커밋하고 싶습니다.

Git은 "스테이징 영역" 이라는 메커니즘을 통해 스냅 샷에 포함시켜야 하는 
수정 사항을 지정할수 있도록 하여 이러한 시나리오를 수용합니다.

# Git command-line interface

정보가 중복되는 것을 막기 위해 우리는 명령어에 대해 자세히 설명하지는 않을 것입니다. 
자세한 내용은 강의 영상을 보거나, 더 많은 정보가 있는 Pro Git을 참고하기를 추천드립니다. 


## Basics

{% comment %}

`git init` 명령은  새로운 Git 저장소를 초기화 하고, 
저장소 메타테이터를 `.git` 디렉토리에 저장합니다 :

```console
$ mkdir myproject
$ cd myproject
$ git init
Initialized empty Git repository in /home/missing-semester/myproject/.git/
$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
```

이 출력결과를 어떻게 해석하면 좋을까요? "No commits yet"은 기본적으로 version
history가 비어 있음을 의미합니다. 수정 해 보겠습니다.


```console
$ echo "hello, git" > hello.txt
$ git add hello.txt
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   hello.txt

$ git commit -m 'Initial commit'
[master (root-commit) 4515d17] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 hello.txt
```

우리는 파일을 스테이징 영역에 `git add` 하였습니다,
그리고 수정사항을 "Initial commit" 이라는 커밋 메시지를 추가하여 `git commit` 하였습니다. 
만약 우리가 `-m`을 명시하지 않았다면 Git은 커밋 메시지를 입력 할수 있도록 텍스트 편집기를 실행 시킬 것십니다.

이제 비어 있지 않은 version history가 있으므로 history를 시각화 할수 있습니다. 
history를 DAG로 시각화하면 저장소의 현재 상태를 이해하고 이를 Git 데이터 모델에 대한 이해와 연결하는데 큰 도움이 될 수 있습니다.

`git log`명령은 history를 시각화 해줍니다. 기본적으로는 그래프 구조 가 아닌 평면화 된 버전을 보여줍니다. 
만약 `git log --all --graph --decorate` 와 같은 명령을 사용하면 그래프로 시각화된 저장소의 전체 버전의 history가 표시됩니다.


```console
$ git log --all --graph --decorate
* commit 4515d17a167bdef0a91ee7d50d75b12c9c2652aa (HEAD -> master)
  Author: Missing Semester <missing-semester@mit.edu>
  Date:   Tue Jan 21 22:18:36 2020 -0500

      Initial commit
```

단일 노드 만을 포함하기 때문에 그래프처럼 보이지는 않습니다. 
좀 더 수정사항을 만들고, 새 커밋을 작성하여 history를 다시 한 번 시각화 해보겠습니다.

```console
$ echo "another line" >> hello.txt
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")
$ git add hello.txt
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   hello.txt

$ git commit -m 'Add a line'
[master 35f60a8] Add a line
 1 file changed, 1 insertion(+)
```

이제 히스토리를 다시 시각화하면 그래프 구조의 일부를 볼 수 있습니다.

```
* commit 35f60a825be0106036dd2fbc7657598eb7b04c67 (HEAD -> master)
| Author: Missing Semester <missing-semester@mit.edu>
| Date:   Tue Jan 21 22:26:20 2020 -0500
|
|     Add a line
|
* commit 4515d17a167bdef0a91ee7d50d75b12c9c2652aa
  Author: Anish Athalye <me@anishathalye.com>
  Date:   Tue Jan 21 22:18:36 2020 -0500

      Initial commit
```

또한 현재 분기(master)와 함께 현재 HEAD를 표시합니다.

`git checkout`명령을 사용하여 이전 버전을 볼 수 있습니다.

```console
$ git checkout 4515d17  # previous commit hash; yours will be different
Note: checking out '4515d17'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 4515d17 Initial commit
$ cat hello.txt
hello, git
$ git checkout master
Previous HEAD position was 4515d17 Initial commit
Switched to branch 'master'
$ cat hello.txt
hello, git
another line
```

Git은 다음 `git diff` 명령을 사용하여 파일이 어떻게 진화했는지 (차이 또는 diffs) 보여줄 수 있습니다.


```console
$ git diff 4515d17 hello.txt
diff --git c/hello.txt w/hello.txt
index 94bab17..f0013b2 100644
--- c/hello.txt
+++ w/hello.txt
@@ -1 +1,2 @@
 hello, git
 +another line
```

{% endcomment %}

- `git help <command>`: git 명령어에 대한 도움말을 보여줍니다.
- `git init`: `.git` 디렉토리에 데이터가 저장된 새 git 저장소를 만듭니다.
- `git status`: 진행 상황을 알려줍니다.
- `git add <filename>`: 스테이징 영역에 파일을 추가합니다.
- `git commit`: 새로운 커밋을 만듭니다.
    - [좋은 커밋 메시지](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)를 작성하세요!
    - [좋은 커밋 메시지](https://chris.beams.io/posts/git-commit/)를 작성해야하는 더 많은 이유!
- `git log`: 평면화된 history의 로그를 표시합니다.
- `git log --all --graph --decorate`: DAG을 사용하여 history를 시각화 합니다.
- `git diff <filename>`: 스테이징 영역을 기준으로 작업한 변경 사항을 보여줍니다.
- `git diff <revision> <filename>`: 스냅샷 간 파일의 차이를 보여줍니다.
- `git checkout <revision>`: HEAD를 업데이트 합니다. 분기를 체크아웃하면 현재 분기도 업데이트 합니다.

## Branching and merging

{% comment %}

분기는 버전 history를 "fork"할 수 있습니다. 독립적 인 기능이나 버그 수정을 병렬로 작업하는 데 유용 할 수 있습니다. 
`git branch` 명령을 사용하여 새 분기를 만들 수 있습니다. 
`git checkout -b <branch name>` 로 브랜치를 만들고 확인합니다.

병합은 분기의 반대입니다. 예를 들어 기능의 분기를 master로 다시 병합하는 것과 같이 분기 된 버전 history를 결합 할 수 있습니다. 
`git merge` 명령은 병합에 사용됩니다.


{% endcomment %}

- `git branch`: 분기를 보여줍니다
- `git branch <name>`: 분기를 생성합니다.
- `git checkout -b <name>`: 분기를 생성하고 전환 합니다.
    - `git branch <name>; git checkout <name>` 과 동일합니다.
- `git merge <revision>`: 현재 분기를 병합 합니다.
- `git mergetool`: 병합 충돌을 해결하는 데 도움이되는 멋진 도구를 사용합니다.
- `git rebase`: 패치 세트를 새로운 베이스로 배치합니다.

## Remotes

- `git remote`: remote 를 나열합니다.
- `git remote add <name> <url>`: remote를 추가합니다
- `git push <remote> <local branch>:<remote branch>`: remote로 객체를 보내고 remote 참조를 업데이트합니다.
- `git branch --set-upstream-to=<remote>/<remote branch>`: 로컬과 remote branch 사이의 통신을 설정합니다.
- `git fetch`: 원격에서 객체 / 참조를 검색합니다.
- `git pull`: `git fetch; git merge`와 동일합니다
- `git clone`: 원격에서 저장소를 다운받습니다.

## Undo

- `git commit --amend`: 커밋 내용 / 메시지 를 편집합니다.
- `git reset HEAD <file>`: 파일을 unstage 
- `git checkout -- <file>`: 변경 사항을 취소합니다.

# Advanced Git

- `git config`: Git을 [고도의 사용](https://git-scm.com/docs/git-config)화 합니다.
- `git clone --depth=1`: 전체 버전 history 없는 shallow 클론을 합니다. 
- `git add -p`: 대화형 스테이징을 합니다.
- `git rebase -i`: 대화형 리베이싱을 합니다.
- `git blame`: 누가 특정 라인을 마지막으로 편집 하였는지 보여줍니다.
- `git stash`: 작업 디렉토리에 대한 수정 사항을 일시적으로 제거합니다.
- `git bisect`: history를 이진 탐색 합니다. (e.g. for regressions)
- `.gitignore`: 의도적으로 추적되지 않는 파일을 [지정](https://git-scm.com/docs/gitignore)합니다.

# Miscellaneous

- **GUIs**: Git용 [GUI 클라이언트](https://git-scm.com/downloads/guis)가 많이 있습니다. 
우리는 개인적으로 GUI클라이언트를 사용하지않고 명령어 인터페이스를 사용합니다.
- **Shell integration**: 깃의 상태를 셸 프롬프트([zsh](https://github.com/olivierverdier/zsh-git-prompt),[bash](https://github.com/magicmonty/bash-git-prompt))의 일부으로 사용하는 것은 매우 편리합니다.
종종 [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)과 같은 프레임워크는 이를 포함하고 있습니다.
- **Editor integration**: 위와 유사하게 많은 기능이있는 편리한 통합입니다.  
[fugitive.vim](https://github.com/tpope/vim-fugitive) Vim의 표준입니다.
- **Workflows**: 우리는 데이터모델과 몇가지 기본 명령어를 가르쳐 드렸습니다. 
우리는 큰 프로젝프 작업을 할때 따라야할 실천사항에 대해 말해주지 않았습니다.
([많은](https://nvie.com/posts/a-successful-git-branching-model/)
[다른](https://www.endoflineblog.com/gitflow-considered-harmful)
[접근방식](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow))이 있습니다.
- **GitHub**: 깃은 깃허브가 아닙니다. 깃허브는 [pull requests](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)라고 불리는 다른 프로첵트에 코드를 기여하는 특정한 방법이 있습니다. 
- **Other Git providers**: GitHub는 특별한 것이 아닙니다. [GitLab](https://about.gitlab.com/) 과
[BitBucket](https://bitbucket.org/)같이 많은 Git 저장소 호스트가 있습니다.

# Resources

- [Pro Git](https://git-scm.com/book/en/v2) is **읽기를 강력히 권합니다**.
데이터 모델을 이해 하고 있는 지금, 1 ~ 5 장을 살펴보면 Git을 능숙하게 사용하는 데 필요한 대부분의 내용을 배울 수 있습니다. 
그 다음 장에는 흥미롭고, 고급 자료가 있습니다.
- [Oh Shit, Git!?!](https://ohshitgit.com/)은 Git에서 일반적으로 일어날수 있는 실수들을 처리하는 방법에 대한 짧은 가이드 입니다.
- [Git for Computer Scientists](https://eagain.net/articles/git-for-computer-scientists/)은 데이터 모델에 대한 짧은 설명을 의사코드를 적게 사용하고 멋진 다이어그램을 활용한 강의 노트 입니다.
- [Git from the Bottom Up](https://jwiegley.github.io/git-from-the-bottom-up/)은 호기심을 위해 데이터 모델뿐만 아니라 Git의 구현 세부 사항에 대해 자세히 설명 하였습니다..
- [How to explain git in simple words](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
- [Learn Git Branching](https://learngitbranching.js.org/) Git 을 학습하는 브라우저 기반 게임입니다.

# Exercises

1. Git에 대한 경험이 없는 경우 [Pro Git](https://git-scm.com/book/en/v2) 의 처음 몇 장을 읽어 보거나 [Learn Git Branching](https://learngitbranching.js.org/) 과 같은 튜토리얼을 훑어보세요. 이러한 작업을 통해 Git 명령어와 데이터 모델을 관련시켜 보세요.
1. [수업을 위한 웹사이트를](https://github.com/missing-semester/missing-semester) 클론해 보세요.
    1. 버전 history를 그래프로 시각화하여 살펴보세요.
    1. 마지막으로 `README.md`를 수정한 사람은 누구 입니까? (힌트 : `git log` 인수를 사용해보세요)
    1. `_config.yml`의 `collections:` 라인의 마지막 수정과 관련된 커밋 메시지는 무엇입니까? (힌트 : git blame 과 git show 를 사용 사용해 보세요)
1. Git을 배울 때 흔히 저지르는 실수 중 하나는 대용량 파일을 커밋하거나 Git에서 관리해서는 안되는 민감한 정보를 추가하는 것입니다. 저장소에 파일을 추가하는 커밋을 몇번 해보고 history에서 해당 파일을 삭제 하세요. (당신은 아마 [이걸](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)을 보고 싶을 거에요).
1. GitHub에서 일부 저장소를 clone하고 기존 파일 중 하나를 수정합니다. 
`git stash`할때 어떤 일이 생기나요? `git log--all --oneline` 을 실행할때 무엇을 볼수 있나요? 
`git stash pop` 실행 하여  `git stash` 한 작업을 취소하세요. 어떤 시나리오가 유용할가요?
1. 많은 명령어 도구들과 마찬가지로 Git은 `~/.gitconfig` 라고 불리는 환경설정 파일 (또는 dotfile)을 제공합니다. 
`git graph`를 실행할 때 `git log --all --graph --decrypt --online` 의 출력을 얻을 수 있도록 `~/.gitconfig` 에 별칭을 생성해 보세요. 파일을 직접 [편집](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias)하거나 `git config` 명령으로 별칭을 추가할 수 있습니다. Git 별칭에 관한 설명은 [여기](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases)에서 볼 수 있습니다.
1. `git config --global core.excludesfile  ~/.gitignore_global` 을 실행 하면 `~/.gitignore_global` 에서 전역 무시 패턴을 정의 할 수 있습니다. 이 명령은 Git이 사용할 전역 무시 파일의 위치만 설정하므로, 해당 경로에 파일을 직접 만들어야 합니다. 이후 `.DS_Store` 와 같이 OS와 관련되거나 에디터와 관련된 임시 파일을 무시하도록 전역 gitignore 파일을 설정해 보세요.
1. [수업을 위한 웹사이트의 저장소](https://github.com/missing-semester/missing-semester)를 fork하고, 
오타 또는 기타 개선 사항을 찾아 GitHub에서 pull request 해보세요. [이 안내](https://github.com/firstcontributions/first-contributions)를 참고해도 좋습니다. 유용한 pull request만 보내주세요(스팸은 보내지 말아주세요!). 개선할 점을 찾지 못했다면 이 문제는 건너뛰어도 됩니다.
