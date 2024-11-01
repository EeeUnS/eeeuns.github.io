---
layout: post
title:  "패딩과 정렬"
date:   2024-02-12 19:26:01 +0900
tag: cpp
---
기본적인 정렬과 패딩에 대한 설명을 생략.

windows os에서 64비트일때,

- 스택 프레임은 16byte
- 구조체는 default 8byte 정렬로 되어있다.

32비트일때는

- 스택은 4byte
- 구조체도 4byte정렬이다.

[https://learn.microsoft.com/en-us/cpp/build/reference/zp-struct-member-alignment?view=msvc-170](https://learn.microsoft.com/en-us/cpp/build/reference/zp-struct-member-alignment?view=msvc-170)

[https://learn.microsoft.com/en-us/cpp/build/stack-usage?view=msvc-170](https://learn.microsoft.com/en-us/cpp/build/stack-usage?view=msvc-170)

그외의 정렬 관련한것은

- **캐시라인은 64byte정렬**,
- **xmm register의 경우 16byte 정렬**

이정도가 정렬 관련한 스펙을 정리한 것 이다. 이것저것 정보를 찾아보다가.

갑자기 의문점이 든 것이 있다.

- **__declspec(align(#))**
- **#pragma pack**
- **alignas**

위의 키워드들은 구조체들의 정렬을 맞추기위해 통상적으로 사용하는 키워드들이다.

**그러면 구조체 정렬을 맞춰주면 정말로 해당 구조체에 접근하는 주솟값이 정렬됨이 보장이 되는가?**

란 의문이 들었다.

정답은 아니요다.

**런타임에 파악가능한것과 컴파일 타임에 파악가능한것**을 한번 생각해보면 간단하다.

어떤 구조체의 시작 주소는 컴파일 타임에 알 수 없다.

그럼 컴파일타임에 쟤들이 해주는게 뭘까?

시작 주소로부터 offset(패딩)을 확정한다.


![breakpoint-settings.png](/images/packing/rayout.png)

Visual Studio에 최근에 들어온 구조체 레이아웃인데, 이를보면 구조체 사이즈가 확정되어있다.
구조체의 사이즈는 컴파일타임에 결정된다. 그래서 static_assert (sizeof()~) 가 가능한거다.
예를들어 

```cpp
IOCompletionPort a
a.mlOWorkerThread
```
이런 코드를 보자.

시작 주소로부터 + 몇 바이트에 어떤 멤버 변수가 위치해있는지 컴파일러는 룰로서 확정하여 컴파일한다.

a.mlOWorkerThread을 컴파일하면서 어셈상에서 보면 해당 값을 어셈으로 (&a + 0x30) 에 접근하는거다 (alignment가 바뀌면, 0x30도 그에 따라 바뀐다.)

엄밀히말해서 객체의 메모리가 몇 바이트로 **정렬되어있음**을 의미하지않는다.
객체의 정렬은 컴파일타임에 결정할 수 없다.

물론 일반적으로 시작주소가 모두 OS단에서 정렬이 되어있는 상태로 주소가 주어져 정렬을 하게 된다.

**그런데 포인터를 직접 접근하게된 구조체들은 이런 규칙들에서 벗어 나게된다. (포인터는 온갖 이상한 짓을 할 수 있다.)**

예를들면 이런 방식이 가능하다.

```C
struct a
{ int c}

struct a * b = 0x~3;
```

sturct a가 packed 8로 되어있다고하자

&(b->c) 의 주소는 8byte 정렬되어있을까?

그래서 정렬이 의도치않게 망가지면 어떤 문제가 될까?

정렬과 관련된 문제는 당연히 메모리에 할당되어있는 상태를 나타내고

**메모리에 접근하는 명령어 중에는 정렬이 보장되어야하는 명령어가 존재한다**

대표적으로 XMM 레지스터를 활용한 SSE쪽 명령어를 사용하려고하면, movaps 명령어를 알게될건데 이 명령어의 경우 접근하는 메모리의 정렬이 깨져있을 경우 exception이 발생한다. (divide zero와 비슷한 하드웨어 인터럽트 발생)

[https://www.felixcloutier.com/x86/movaps](https://www.felixcloutier.com/x86/movaps)

재밌게도 일반적인 개발자들보다 ROP를 시도한 해커 꿈나무(…)에게서 많이 해당 상황을 직접 겪는데, 컴파일러가 부분부분 이런 정렬이 보장되어야하는 명령어들을 생성하기 때문이다. (64bit 컴파일부터는 xmm 레지스터를 굉장히 많이 사용한다.)

의외로 믿음과 신뢰로(…) 명령어를 생성하는데, 이런 어느정도 합의가 있지않으면, 

alignment가 보장되어야하는 명령어 사용을 아예 못 할 거다.

이런 문제는 특히 오래된 32비트 프로그램 64비트 프로그램으로 포팅하면서 온갖 문제가 발생하는데, 외부 라이브러리나, os api 들을 호출하는데 내부에서 만약 이상한 문제가 발생한다면, 의심해 볼 법하다.

그리고 요새 xmm 레지스터 쪽이 점점 16byte 정렬을 요구하기에.. 어느 순간부터는 시스템을 전체적으로 16byte기준으로 올려서 정렬을 요구할지도 모르는데, 여기에 근본적인 대비가 필요할지도 모른다.

이런 부분은 컴파일러 수준에서 힌트나 경고를 준다면 가장 좋을거같긴한데 아직은 힘들 듯 하다.

[https://learn.microsoft.com/ko-kr/windows/win32/winprog64/common-compiler-errors#warning-c4311-example-4](https://learn.microsoft.com/ko-kr/windows/win32/winprog64/common-compiler-errors#warning-c4311-example-4)

[https://learn.microsoft.com/en-us/cpp/cpp/align-cpp?view=msvc-170](https://learn.microsoft.com/en-us/cpp/cpp/align-cpp?view=msvc-170)

[https://megayuchi.com/2017/10/26/compiler-intrinsic을-사용해서-simd코드를-작성할때-주의할-점/](https://megayuchi.com/2017/10/26/compiler-intrinsic%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%84%9C-simd%EC%BD%94%EB%93%9C%EB%A5%BC-%EC%9E%91%EC%84%B1%ED%95%A0%EB%95%8C-%EC%A3%BC%EC%9D%98%ED%95%A0-%EC%A0%90/)

[https://megayuchi.com/2007/04/15/x64-어셈블리-쓰기/](https://megayuchi.com/2007/04/15/x64-%ec%96%b4%ec%85%88%eb%b8%94%eb%a6%ac-%ec%93%b0%ea%b8%b0/)

[https://megayuchi.com/2007/08/23/64비트-어셈에서-스택-사용시-주의사항/](https://megayuchi.com/2007/08/23/64%EB%B9%84%ED%8A%B8-%EC%96%B4%EC%85%88%EC%97%90%EC%84%9C-%EC%8A%A4%ED%83%9D-%EC%82%AC%EC%9A%A9%EC%8B%9C-%EC%A3%BC%EC%9D%98%EC%82%AC%ED%95%AD/)