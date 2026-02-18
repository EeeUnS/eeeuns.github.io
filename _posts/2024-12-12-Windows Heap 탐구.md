---
layout: post
title:  "Windows Heap 탐구"
date:   2024-12-12 19:26:01 +0900
tag: notes
---

![image.png](/images/windowsheap/image.png)

![image.png](/images/windowsheap/image%201.png)

![image.png](/images/windowsheap/image%202.png)

![image.png](/images/windowsheap/image%203.png)

![image.png](/images/windowsheap/image%204.png)

해제시에 기본적으로 체크하고 크래시를 내는데 이미 문제가 발생한 후라 버그 찾기가 어려움

![image.png](/images/windowsheap/image%205.png)

![image.png](/images/windowsheap/image%206.png)

![image.png](/images/windowsheap/image%207.png)

![image.png](/images/windowsheap/image%208.png)

만약 6808byte를 할당한다면

![image.png](/images/windowsheap/image%209.png)

16byte 얼라인을 맞추기위해 초과분의 8byte는

HEAP_ENTRY 구조체는 앞의 8byte를 안쓰니까 뒷 블럭의 HEAP_ENTRY 8byte에다가 쓴다.

질문. 8byte넘어가면 쓸 공간 없는데 만약 0xC사용하면 어떻게되는지?

debug버전일땐 CRT Heap이 붙는다

![image.png](/images/windowsheap/image%2010.png)

![image.png](/images/windowsheap/image%2011.png)

기본 48byte ++ 요청 메모리 +  4byte = 기본 52byte가 붙는다

![image.png](/images/windowsheap/image%2012.png)

OS에서 제공하는 기능

![image.png](/images/windowsheap/image%2013.png)

tail checking을 위한 0xab 16byte + 0x00 16byte(== padding) 추가

![image.png](/images/windowsheap/image%2014.png)

![image.png](/images/windowsheap/image%2015.png)

이 D는 0~12byte 왔다갔다함

![image.png](/images/windowsheap/image%2016.png)

debug에서는 어셈코드상에서 HeapAlloc : 6852요청 → 근데 실제 할당한 사이즈는 

- 6912 : tail checking free checking 활성화
- 6864 : CRT Heap 활성화
- 릴리즈 경우 windbg → Tail Checking Free Checking 활성화

![image.png](/images/windowsheap/image%2017.png)