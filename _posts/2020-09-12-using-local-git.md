---
title: "Git Local Repository 잘 운영하기"
tags: ["git", "rebase", "merge", "history"]
category: [tech, git]
---

`지난글`

> Git은 clone - commit - push 만 쓰기엔 불친절하다.

# 내 Branch는 어떻게 꼬여가는가

Git을 원격 repository 에서 관리하다 보면, branch 가 점점 늘어난다. 이건 local repository / remote repository에 구분 없이 끊임없이 발생하게 된다.

---
layout: post
tags: ["git", "rebase", "local", "merge", "history"]
categories: ["tech", "git"]
#date: 2019-06-25 13:14:15
#excerpt: ''
#image: 'BASEURL/assets/blog/img/.png'
#description:
#permalink:
title: "Git Local Repository 잘 운영하기"
---

"지난글"
ㅔㅕ
> Git은 clone - commit - push 만 쓰기엔 불친절하다.


# Git은 어떻게 작동하는가


Git을 관리하다 보면, branch 가 점점 늘어난다. 특히, 원격 repository 를 기반으로 한 저장소 운영을 하면, local repository / remote repository가 실시간으로 늘어난다. 그렇기에 GitHub에서는 Staled, Active Branch를 이용해서 최근에 이용된 내용 위주로 보여주는 등 사용자 편의를 돕고 있다.

하지만 개발자가 `git clone` 을 수행하는 순간, stale / active 여부와는 상관 없이 모든 `refs/heads/* ` branch가 local repository롤 들어온다. 또는 기존 branch에서 `remote update`, `pull` 을 이용해서 remote repository 의 저장소를 가져온다.

여기서 내 저장소가 어떻게 작동하는지를 이해하기 위해서, 어떤 커맨드를 사용하는 것이 적절할 지 아래 내용을 참고할 수 있다.

## remote repository에서 소스를 가져오는 방법들
local repository에서 remote repository의 데이터를 가져오는 흔한 방법은 아래와 같이 있다.

### git clone
1. git init: 비어있는 git을 만든다.
2. remote:HEAD를 pull 한다.

### git fetch
1. 현재 branch에 대해서 업데이트 한다.

### git remote update
1. local repository에 있는 전체 origin branch를 최신 상태로 업데이트 한다.

### git pull
1. 현재 branch를 remote의 최신 정보로 업데이트 한다.
2. remote/branch를 현재 branch로 merge 한다.(!)


# 꼬여있는 내 Git을 풀어내는 방법
이전 blog와 위 명령을 참고하면, 이미 운영중인 원격 repository와 내 저장소에 conflict를 만들지 않을 수 있다. 다만 언제든 원하지 않는 상황이 발생할 수 있고, 이럴 때는 conflict를 어떻게든 풀어내야 한다.
이럴 때는 아래와 같은 방법을 이용해서 merge 또는 conflict이 쌓인 git을 풀어낼 수 있다.

## Reset 하고 다시 시작하기
git reset 은 로컬 저장소의 HEAD를 되돌린다.  이걸 수행하기 전에, git의 속성을 간단히 알아둬야 한다. git 은 내 작업 공간 - stage - repository 로 구분할 수 있으며,  다음 그림과 같이 이해할 수 있다.
이 그림에서, reset은 .git repoisitory 를 기반으로 내 작업 공간 또는 stage를 되돌리는 역할을 한다. reset은 soft 또는 hard를 선택할 수 있다.
* --soft: 
* 
기본적으론 --soft 가 적용되며, --hard를 이용할 수 있으나 주의해야 한다.
### git commit --amend
### git push
### git stash

## rebase 하기
rebase는 conflict 발생 시, 내 commit을 하나하나 확인하면서 conflict를 해결해서 rebase 하고, 다시 다음 commit 이 rebase 가능한지 확인하는 과정을 거친다.
이 때, conflict 를 해결하는 방법과 내부 작동은 다음과 같으며, 키가 되는 command 는 `git add / git rm` 및 `git rebae --continue` 이며, 다음과 같이 실행한다.
```bash
$ git rebase origin/master
# <<< file edit
$ git add .
$ git rebase --continue
```

## merge 하기
git 에서 conflict를 해결하는데 커맨드적으로 제일 쉬운건 merge 다. (remote - local conflict이 많을 때 발생하는 변경사항 resolve의 어려움은 뒤로 하자)

```bash
$ git remote update
$ git checkout <LOCAL_BRANCH>
$ git merge origin/maser
# < file edit
$ git add .
$ git commit
```
* git commit -m "comment" 보다는 git commit 하고 editor를 쓰는 것이 낫다. 

이렇게 간단하게 conflict를 Resolve 할 수 있다.
