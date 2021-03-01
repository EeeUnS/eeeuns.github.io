---
layout: post
title:  "pintos 세팅중 만난 에러들과 해결법"
date:   2021-01-26 02:26:01 +0900
tag: pintos
---

[원사이트](https://web.stanford.edu/class/cs140/projects/pintos/pintos.html)

- 사용 환경 : wsl2 ubuntu 20.04 TLS

wsl 세팅은 다음을 [참고](https://youtu.be/anZmL2bs-xY) 


참고 사이트
- [다음 사이트를 기준으로 시작하였음](https://bowbowbow.tistory.com/9)
- [같이 참고한 사이트](https://blog.koriel.kr/how-to-install-pintos/)

## 문제 1 gcc설치
gcc4.4가 깔리지 않는다. 두번째 사이트에도 이경우 이렇게 해결하라고 나와있기에 그대로 따라함.

```shell
deb http://dk.archive.ubuntu.com/ubuntu/ trusty main universe
deb http://dk.archive.ubuntu.com/ubuntu/ trusty-updates main universe
```
을 추가하였다.
이후는 


## 문제 2 Bochs make error

파일받고 압축해제하고 설치진행중 make 부분에서 다음과같은 문제가 뜨면서 계속 error가 발생
```shell
g++ -o bximage -g -O2 -D_FILE_OFFSET_BITS=64 -D_LARGE_FILES misc/bximage.o
/usr/bin/ld: misc/bximage.o: relocation R_X86_64_32 against `.rodata.str1.8' can not be used when making a PIE object; recompile with -fPIE
```

끝끝내 bochs-2.6.2 설치자체를 포기 2.6.11을 설치 두번째 사이트에서 나온 url을 숫자만 변경해서 사용해도 무방.

```
https://sourceforge.net/projects/bochs/files/bochs/2.6.11/
```

## 문제 3 pintos 컴파일

다음의 문제가 정확히 동일하게 나타남.

[stackoverflow](https://stackoverflow.com/questions/45656966/how-to-resolve-pintos-unrecognized-character-x16)

직접 파일 수정.


## 문제 4 pintos 실행

폴더 넣고 make하고 환경변수에도 등록했다. 근데 실행이 안된다.

[참고 사이트](https://m.blog.naver.com/PostView.nhn?blogId=hwu5&logNo=220808645877&proxyReferer=https:%2F%2Fwww.google.com%2F)

차례대로 커널도 찾을수 없다하고 로더도 찾을수 없단다 다음 사이트로 해결. 상대경로를 모두 절대경로로 박아넣었다.
이는 매우 비추하는 해결방법이며 절대경로를 넣지않아도

이제 아무데서 실행은 된다.



## 문제 5 다시 bochs 문제

실행에서 이런 에러가 뜬다.

bochs자체가 버전이 달라지면서 다른 사이트들과 전체적인 모습도 다른데 아무래도 버전때문에 생긴 문제같다..

어디에도 해결법은 못찾았었는데.

```shell
bochsrc.txt:8: 'user_shortcut' is deprecated - use 'keyboard' option instead.
```

[해결](http://bochs.sourceforge.net/doc/docbook/user/bochsrc.html)

pintos 파일 561 줄쯤에 보면 `user_shortcut: keys=ctrlaltdel` 줄을   `keyboard: user_shortcut=ctrl-alt-del` 로 바꿔준다.


해결 
?


일단 원래는 utils 폴더에서 make 를 하여 실행되는지 확인하여야하는데 일단 이부분은 make가 되지않았고 make를 하지않아도 일단은 진행되기에 그대로 진행할 예정.


## 문제 6 utils 폴더 make error

```
Prototype mismatch: sub main::SIGVTALRM () vs none at /home/euns/pintos/utils/pintos line 934.
```
make 를 안한 문제였다.

[참고](http://courses.mpi-sws.org/os-ss13/assignments/pintos/pintos_12.html)

Compile the Pintos utilities by typing make in src/utils. Bochs needs squish-pty, VMware Player needs squish-unix. If they don't compile on your recent Linux distribution, comment out #include <stropts.h> in both squish-pty.c and squish-unix.c, and comment out lines 288-293 in squish-pty.c.


make를 안한 문제 인 줄 알았다.

Prototype mismatch: sub main::SIGVTALRM () vs none at /home/euns/pintos/utils/pintos line 934.
Constant subroutine SIGVTALRM redefined at /home/euns/pintos/utils/pintos line 926.
Cannot find kernel








https://stackoverflow.com/questions/20822969/pintos-programming-project-2





https://stackoverflow.com/questions/52472084/pintos-userprog-all-tests-fail-is-kernel-vaddr

```
pintos -f -q
Prototype mismatch: sub main::SIGVTALRM () vs none at /home/euns/pintos/utils/pintos line 934.
Constant subroutine SIGVTALRM redefined at /home/euns/pintos/utils/pintos line 926.
squish-pty bochs -q
00000000000i[      ] BXSHARE not set. using compile time default '/usr/local/share/bochs'
00000000000i[      ] reading configuration from bochsrc.txt
========================================================================
                       Bochs x86 Emulator 2.6.11
              Built from SVN snapshot on January 5, 2020
                Timestamp: Sun Jan  5 08:36:00 CET 2020
========================================================================
00000000000i[      ] installing nogui module as the Bochs GUI
00000000000i[      ] using log file bochsout.txt
PiLo hda1
Loading.........
Kernel command line: -f -q
Kernel PANIC at ../../threads/vaddr.h:84 in vtop(): assertion `is_kernel_vaddr (vaddr)' failed.
Call stack: 0xc0028a4c 0xc002b2df 0xc002a960 0xc0020cdc 0xc00210e5 0xc0021172 0xc0022c8b 0xc0022f53 0xc002145d 0xc00217a2 0xc00205aa.
The `backtrace' program can make call stacks useful.
Read "Backtraces" in the "Debugging Tools" chapter
of the Pintos documentation for more information.
Timer: 0 ticks
Thread: 0 idle ticks, 0 kernel ticks, 0 user ticks
Console: 477 characters output
Keyboard: 0 keys pressed
Exception: 0 page faults
Powering off...
========================================================================
Bochs is exiting with the following message:
[UNMAP ] Shutdown port: shutdown requested
========================================================================
```