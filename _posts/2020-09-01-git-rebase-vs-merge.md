---
title:  "Git rebase vs merge"
tags: ["git", "rebase", "merge"]
category: [tech, git]
---

> rebase 와 merge를 설명해야 할 일이 있어서 준비해볼 겸 짧게 작성한다.

Git 을 이용한 형상관리를 진행하다 보면, master branch 이외에 다른 brach도 사용을 하게 된다. 
더 많이 사용하다 보면 stash를 사용할 수도 있겠으나, 이 부분에 대한 설명은 이번엔 하지 않을 것이다.

매우 간단한 git graph를 보도록 하자.
![default-tree](/assets/images/2020-09-01/git-mr-default.png)

이러한 이력으로 개발이 진행되고 있다고 가정하자.

이제 나는 d까지 개발을 마치고, master의 b와 합치려고 한다. (여기서 b는 내가 개발을 했어도 무관하고, remote 의 master에서 가져왔을 수도 있다.)  
이 시점에서 작업자가 master에 내 변경 사항을 더하려면 rebase 또는 merge를 해야 한다.  간단하게 설명해서 rebase는 target을 base 로 현재 branch 의 commit을 다시 작성한다. merge 는 두 branch의 head를 기준으로 충돌이 없는 경우라면 fast-forward 로 병합하고, 아니라면 merge commit 을 만든다.

자동으로 처리할 수 없는 경우(충돌이 발생하는 경우) 는 merge conflict 이 발생하고 이때는 개발자가 직접 충돌을 풀어서 merge commit 을 만들어야 한다.

이 글에서는, merge 와 rebase 각각을 어떻게 실행해서 어떤 결과를 낼 것인지를 간단하게 그림으로 보려고 한다.


---

1. feature에 master로 rebase 하는 경우
간단하게 설명하면, 현재 위치는 feature 이고, master 를 현재 commit의 base로 설정하고 싶다는 것이다.  
이렇게 설명하는 것보다 실습과 그림이 나을 듯 하여 아래와 같이 추가한다.
상단과 같이 준비된 git repo 에서 아래의 명령을 실행한다.
```bash
$ git checkout feature
$ git rebase master
```
이렇게 실행하면 다음과 같은 형태로 리베이스가 된다.
![default-tree](/assets/images/2020-09-01/git-mr-1.png)
여기서 주목해야 하는 것은, b(master) 가 원본을 유지하고, 내가 작업중인 feature 브랜치가 master 의 전방으로 움직인 것이다. 그리고 주목해야 할 것은 c' 와 d' 이다. 이 두 commit은 기존의 c 와 d 가 아닌 새로운 commit 으로 rewrite 된다.

따라서, 내가 모든 작업을 마치고 remote/feature 에 push +pull request 를 한다면, 작업자는 fast-forward 가 가능한 상태로 commit을 작성할 수 있으며, 명료한 git graph를 만들기에 도움이 된다.


---

2. feature에 master로 rebase 하는 경우 (never do that)
이 경우는 위와 동일하되 명령을 실행하는 순서만 다르다.
```bash
$ git checkout master
$ git rebase feature
```
이 명령을 실행하면 이런 그림이 나온다.
![default-tree](/assets/images/2020-09-01/git-mr-2.png)
일단 봐서는 위와 같이 리니어한 branch 가 나온다. 하지만 여기서 주목해야 되는 부분이 있다.  
바로 b' 이다. Git을 많이 쓴다면 일반적으로 remote와 연결된 branch는 직접 수정하기보다는 branch out -> commit -> 병합의 과정을 거치게 되는데, 여기서는 remote와 같은 변경사항을 갖고 있는 b가 b' 로 변경되버린 것이다.

결국 이 커밋은 remote로 push할 경우 conflict이 발생할 가능성이 아주 높아지고, 어떤 경우는 -f 옵션을 걸어서 이력을 더럽혀야 할 수도 있다. 그렇기에 이 명령 및 branch 병합 전략은 절대로 권장하지 않는다.


---

3. feature 에 master를 merge 하는 경우
merge 는 더 간단하다. 두 branch를 검토하여 fast-forward병합이 가능한지, merge commit을 통한 병합이 가능한지, 둘다 불가능한지에 대해 git이 판단하고 알아서 병합을 한다.

일단, fast-forward 가 가능한 경우부터 보자. 1번의 경우, master에서 git merge feature 를 수행하면 다음과 같은 그래프가 완성된다.
![default-tree](/assets/images/2020-09-01/git-mr-3.png)
따라서, 현재 HEAD는 master, feature가 동일한 commit을 보게 되며, 별도의 병합 과정이 없이 fast-forward 로 수행되었음을 git console에서 표시된다.
다음은 merge commit 이 생성되는 경우이다. 기본 그림에서 다음과 같이 수행한다.
```bash
$ git checkout feature
$ git merge master
```
즉, 현재 feature 에 master의 내용을 병합하겠다는 것이며, 다음과 같이 git graph가 작성된다.

여기서 점선인 부분은 병합이 되는 경로를 의미한다. 그리고 최종 산출물로서 merge commit e 가 새로 생성된다. 이 명령을 수행하면, console에서 새로운 commit 에 대한 comment 작성을 하도록 에디터가 열린다. 그리고 이 에디터의 내용에서, 어떤 commit들이 merge가 되는지에 대한 정보가 log에 추가로 작성된다.

이렇게 생성된 merge commit 을 remote/feature 에 push + pull request를 함으로써 충돌 없이 로컬에서 잘 정돈된 commit을 등록할 수 있다.


---

4. master 에 feature를 merge 하는 경우
1 - 2에서 보았듯, 명령은 현재 branch - target 만 반대로 수행한다.
```bash
$ git checkout master
$ git merge feature
```
이 결과는, 지금과 같이 단순한 병합 과정에서는 변경사항이 3과 같다. 다만 변경점이 큰 경우는 병합 결과가 달라질 수 있을 것이다. 그렇기에, 3과 4의 merge commit 은 그 commit id 가 다르다.
![default-tree](/assets/images/2020-09-01/git-mr-4.png)
이렇게 만든 결과물도, 실제로는 권장하지는 않는다. 일반적으로 2인 이상이 개발을 하게 되는 경우 master를 관리해야 하기 때문에 개발자의 로컬 환경에서 merge commit 이 master 에 있다 한들 실제로 remote/feature로 push 해야 하기 때문이다.


---

이상으로 간단하게 merge - rebase를 비교해 보았다. remote git 저장소를 관리하게 된다면 fast forward only, merge if necessary, rebase if necessary 로 branch 병합 전략을 정하고 commit graph나 conflict를 관리하게 된다.

개발자가 병합을 마치고 올리도록 강제하여 conflict 생성 자체를 막는 방법도 의미가 있으며, 반대로 review와 patchset / cherrypick 을 다양하게 다루는 조직에서는 merge 를 열어주는것이 좋을 수 있다. (어느쪽이든 master에 직접 쓰는건 좋지 않을 것이다. 심지어는 혼자 개발을 해도 마찬가지일듯)
다만 로컬에서 merge 를 할지, rebase를 할지, 그 결과가 어떤 내용이 나올지에 대해서 내부 머지 룰에 따라 적용하면 된다.