---
layout: post
title:  "alignment 정리"
date:   2025-12-27 19:26:01 +0900
tag: cpp
---

기본적인 내용은 아래에 정리되어있다. 
- [패딩과 정렬](https://eeeuns.github.io/2024/02/12/alignment/)

실제 정렬에 맞춰서 해당 기능을 쓸때 참고하기 위해 각 기능별 차이를 이 글에서 기술한다.

## 핵심 개념(간단)

- 정렬(alignment): 타입이 요구하는 메모리 경계(예: int=4, double=8).
- 패킹(packing): 컴파일러가 적용하는 정렬 한계를 낮춰 멤버간 패딩을 줄이는 것(예: `#pragma pack`).

## 핵심 키워드

- `#pragma pack(push, N)` / `#pragma pack(pop)` — 멤버 배치 기준을 N 바이트로 제한.
- `alignas(N)` (C++11) — 타입/변수/멤버의 **최소 정렬**을 보장(표준).
- `__declspec(align(N))` (MSVC) — MSVC 전용 정렬 지정, `alignas`와 동일.


## 비교 표

| 기능 | 문법 (MSVC) | 범위 | 효과 | 추천 경우 |
|---|---:|---|---|---|
| 패킹 | `#pragma pack(push,N)` / `#pragma pack(pop)` | 소스 범위 | 멤버 오프셋 계산 기준을 N으로 제한 → sizeof 감소 가능 | 파일/네트워크 포맷 정합성 필요 시 |
| 표준 정렬 지정 | `alignas(N)` | 타입/변수/멤버 | 타입의 요구 정렬 증가 → sizeof 증가 | 성능/SIMD/하드웨어 요구 시 |
| MSVC 확장 | `__declspec(align(N))` | 타입/변수 | `alignas`와 동일 (MSVC 전용) | `alignas` 동일 |


## MSVC(x64) 예제 (결과 포함)

```cpp
// 기본 MSVC 정렬
struct A { char c; int i; };
static_assert(sizeof(A) == 8);
// default MSVC x64: sizeof(A) == 8, alignment : 4

#pragma pack(push, 1)
struct B { char c; int i; };
#pragma pack(pop)
static_assert(sizeof(B) == 5);
// MSVC x64: sizeof(B) == 5, alignment : 1

#pragma pack(push, 16)
struct BB { char c; int i; };
#pragma pack(pop)
static_assert(sizeof(BB) == 8); // BB alignment : 4

struct alignas(16) C { char c; int i; };
// MSVC x64: sizeof(C) == 16
static_assert(sizeof(C) == 16);

struct alignas(1) CC { char c; int i; };
//warning C4359: 'CC': Alignment specifier is less than actual alignment (4), and will be ignored.
static_assert(sizeof(CC) == 8);

struct alignas(32) CCC { char c; int i; };
//warning C4359: 'CC': Alignment specifier is less than actual alignment (4), and will be ignored.
static_assert(sizeof(CCC) == 32); // alignment : 32

__declspec(align(1)) struct D { double d; int i; };
// warning C4359: 'D': Alignment specifier is less than actual alignment (8), and will be ignored.
static_assert(sizeof(D) == 16);// alignment : 8

__declspec(align(16)) struct DD { double d; int i; };
static_assert(sizeof(DD) == 16);// alignment : 16

__declspec(align(32)) struct DDD { double d; int i; };
static_assert(sizeof(DDD) == 32);// alignment : 16

struct BBB {
	B b; // 5 bytes
	int a;
};
static_assert(sizeof(BBB) == 12); // alignment : 4

struct CCCC
{
	CCC c; // 32 bytes alignment 32
	int a;
};
static_assert(sizeof(CCCC) == 64); // alignment : 32

```

설명: MSVC는 구성 멤버의 최대 정렬 요구를 기준으로 구조체 정렬을 결정합니다. `alignas`/`__declspec(align())`로 요구 정렬을 올리면 sizeof가 그 배수로 늘어나고, `#pragma pack`은 멤버 오프셋 계산 시 적용되는 최대 정렬을 낮춰 sizeof를 줄입니다.


간단 요약: MSVC 환경에서는 `#pragma pack`으로 패킹을 낮춰 크기를 줄이고, `alignas`/`__declspec`로 정렬을 높여 접근 성능이나 하드웨어 요구를 만족시킨다. 크로스-플랫폼/호환성이 중요하면 `alignas`를 우선 고려하고, 패킹은 최소한의 범위에서만 사용하세요.

# 참고사항

앞글에서 언급했지만 정렬이란 결국 시작 주소에 따라 결정나기에 시작 주소가 정렬되어있지않다면 아무리 구조체에 alignas를 붙여도 소용이없다.

일제로 스택에서는 16byte 정렬로 움직이고 있기에 꽤 오랬동안 해당 키워드가 스택에서 16byte를 넘는 정렬이 보장되지않다가 이것이 버그로 판단되어 VS19에서 추후 수정되었다. [링크](https://developercommunity.visualstudio.com/t/alignas-of-structured-bindings-c17-not-working/627792)


## 참고 링크

- [https://velog.io/@jellypower/CMSVC%EC%9D%98-alignment-%EC%A7%80%EC%A0%95%EC%9E%90%EA%B0%84%EC%9D%98-%EC%B0%A8%EC%9D%B4](https://velog.io/@jellypower/CMSVC%EC%9D%98-alignment-%EC%A7%80%EC%A0%95%EC%9E%90%EA%B0%84%EC%9D%98-%EC%B0%A8%EC%9D%B4)
- [https://learn.microsoft.com/ko-kr/cpp/preprocessor/pack?view=msvc-170](https://learn.microsoft.com/ko-kr/cpp/preprocessor/pack?view=msvc-170)
- [https://learn.microsoft.com/ko-kr/cpp/build/x64-software-conventions?view=msvc-170#x64-structure-alignment-examples](https://learn.microsoft.com/ko-kr/cpp/build/x64-software-conventions?view=msvc-170#x64-structure-alignment-examples)
- [https://learn.microsoft.com/ko-kr/cpp/cpp/alignas-specifier?view=msvc-170](https://learn.microsoft.com/ko-kr/cpp/cpp/alignas-specifier?view=msvc-170)
- [http://post.procademy.co.kr/archives/865](http://post.procademy.co.kr/archives/865)


