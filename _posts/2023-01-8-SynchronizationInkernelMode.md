---
layout: post
title:  "커널 모드 동기화"
date:   2023-1-8 19:26:01 +0900
tag: windows
---

오개념을 잡아보자.


windows api관련해서 공부하다보면 유저모드 동기화라고 부르는 동기화 종류가있다
- [interlock compiler intrinsic/함수들](https://learn.microsoft.com/en-us/windows/win32/sync/interlocked-variable-access)
- [CriticalSection](https://learn.microsoft.com/en-us/windows/win32/sync/critical-section-objects)
- [condition variable](https://learn.microsoft.com/en-us/windows/win32/sync/condition-variables)
- [SRWLock](https://learn.microsoft.com/en-us/windows/win32/sync/slim-reader-writer--srw--locks)

그리고 그의 반대 급부로 커널모드 동기화라고도 소개를 많이한다.
- Semaphore
- Mutex
- Event

보통 이렇게 소개를 많이하는데. (물론 몇개 더있다.)
**결론부터 얘기하자면 이들에 대해서 커널모드 동기화라고 하는 것 자체가 틀렸다.**
커널모드 동기화라는 용어 자체에 대해서 한번 따져볼 필요가 있다.

| interlock은 사실상 어셈단의 단일 명령어기에.. windows에서 제공하는 동기화가 아니라고하고 아래 내용을 전개하겠다.

windows api로서는 레퍼런스 급의 명성을 지닌 windows via C/C++의 목차를보면 목차에 유저모드 동기화와 그다음에 소개되어있는거는 커널 모드 동기화가 아닌 커널 오브젝트를 이용한 동기화라고 부른다. 

![Untitled](/images/SynchronizationInkernelMode/1.png)



커널 오브젝트를 이용한 동기화인데 유저 모드 동기화를 처음에 알고나서 그대로 반대개념으로 생각을 하고서 중간을 안읽고 자기 생각대로 넘어가고 이게 그대로 글로 써지면서 이런 일이 일어난것이다.

커널 모드 동기화라 부르는 동기화 객체들은 동기화 방식이 커널오브젝트를 쓰는거지, 커널 모드간 동기화라는건 말이 애초에 성립이 안되는거라 사용자 입장에선 모두 유저모드에서 돌아가는 동기화다.
실제로도 사용례로서 커널 오브젝트를 사용해서도 프로세스간 동기화를 하는경우는 그렇게 많지않고 대부분 스레드간에 사용한다. (커널모드로 들어가면서 프로세스간의 동기화의 의미가 그렇게 크지않다는걸)

커널모드 동기화의 의미에 대해서 나눠보자면.

1. 커널 모드(커널 코드 사용)를 사용하는 동기화
2. 커널 모드 내부의 코드단을 동기화

두 가지 의미로 해석 될 수 있다. 

1번의 의미로 생각했을때는 CriticalSection이 반례가 될 수 있는데 이넘이 CriticalSection도 스핀카운트를 다 채우면 커널이 재운다. 
이 경우 1번에 해당하지않는 동기화는 스케줄링에 계속 남아있는 spinlock 말고는 없는데 spinlock만을 지원하는건 windows에는 없으니 모든 동기화 방식이 커널모드 동기화에 해당한다.
결론적으로 얘도 커널모드 들어가니까 커널모드 동기화라고 할 수 있다. 그런데 얘는 명백하게 유저 모드 동기화로서 다루고 결론적으로 모든 동기화 방식이 커널 모드 동기화가 되는거라 의미가 없어진다.

| [CriticalSection에 대해 좀 더 알아보고싶으면](http://www.kudryavka.me/?p=991)

2번의 의미로 보면 커널 모드 내부 코드단을 동기화를 한다는 의미로 쓰게되면 커널모드는 애초에 사용자 입장에서 접근이 안되기에 커널모드 동기화라는 단어가 성립이 안된다고 봐야한다.


창피하게 나도 완전히 똑같은 오류를 범했고 이걸 회사 발표에서조차 이렇게 언급을해버렸다.
웃긴건 이걸 발표하고나서도 그걸 눈치채지못하고, 팀원들의 ppt를 보다가 이런 문제가 있다는걸 깨달았다.
(역시 남 훈수두려고 볼 때 사람이 가장 똑똑해지는것같다.)


# 그래서 커널모드 동기화는 없는건가요?

당연히 아니다. 2번의 의미의 커널 모드 동기화는 진짜로 있다. 커널 내부 os단에서 사용하는 개별의 동기화 객체가 있고 그게 커널모드 동기화가 되는거다. Synchronization In kernel Mode라고 영어로 검색해봐도 진짜 커널단의 동기화에 대해서 나와있다. 
값비싼 윈도우가 커널을 우리에게 코드를 보여줄리는 없고 오픈소스 linux에서 [커널모드 동기화 장치](https://docs.kernel.org/locking/locktypes.html)를 볼 수 있다. 커널이라고 딱히 개념적으로 특별한건 없다.












