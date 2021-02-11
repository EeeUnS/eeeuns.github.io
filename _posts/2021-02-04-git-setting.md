---
layout: post
title:  "git setting"
date:   2021-02-04 02:26:01 +0900
tag: etc
---

git 구조
https://uxgjs.tistory.com/182

https://jeonghwan-kim.github.io/dev/2020/02/10/git-usage.html


fork update

https://velog.io/@k904808/Fork-%ED%95%9C-Repository-%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8-%ED%95%98%EA%B8%B0


push

```shell
git push origin
```

log 보기  / 변경사항보기
```shell
git log
git log -p
```


pintos, Pintos.pm을 git 추적을 못하게 해야한다.

```shell
git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   utils/Pintos.pm
        modified:   utils/pintos

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        bochsout.txt
        bochsrc.txt

no changes added to commit (use "git add" and/or "git commit -a")

euns@DESKTOP-6VB35LS:~/pintos/utils$ git update-index --assume-unchanged pintos
euns@DESKTOP-6VB35LS:~/pintos/utils$ git update-index --assume-unchanged Pintos.pm
euns@DESKTOP-6VB35LS:~/pintos/utils$ git status
On branch main
Your branch is up to date with 'origin/main'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        ../bochsout.txt
        ../bochsrc.txt

nothing added to commit but untracked files present (use "git add" to track)
```



https://coding-start.tistory.com/361


- revert : 커밋을 되돌린 기록이 남음.
- reset  : 기록이 남지않고 커밋만 남김. 만약 push를 이미 한 상태에서 commit 을 되돌릴경우에 원격 레포와 충돌이 일어남. 혼자사용하는 레포일경우에는 -f 옵션으로 push하면 해결.