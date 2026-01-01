---
layout: post
title:  "alignment 정리"
date:   2025-12-27 19:26:01 +0900
tag: cs
---

기본적인 내용은 아래 글을 먼저 확인하세요.

- [패딩과 정렬](https://eeeuns.github.io/2024/02/12/alignment/)

이 글은 MSVC(x64) 관점에서 구조체 정렬 관련 키워드들의 차이와 주의점을 정리한 것입니다. 실제 코드에 적용할 때 바로 참고할 수 있도록 예제와 sizeof 결과, 경고 정보를 포함했습니다.

## 핵심 개념(간단)

- 정렬(alignment): 타입이 요구하는 메모리 경계입니다(예: int = 4, double = 8).
- 패킹(packing): 컴파일러가 적용하는 정렬 한계를 낮춰 멤버 간 패딩을 줄이는 방식입니다(예: `#pragma pack`).

## 핵심 키워드

- `#pragma pack(push, N)` / `#pragma pack(pop)` — 멤버 배치 기준을 N 바이트로 제한합니다.
- `alignas(N)` (C++11) — 타입/변수/멤버의 최소 정렬을 보장합니다(표준).
- `__declspec(align(N))` (MSVC) — MSVC 전용 정렬 지정으로, `alignas`와 유사하게 동작합니다.

## 비교 표

| 기능 | 문법 (MSVC) | 범위 | 효과 | 추천 경우 |
|---|---:|---|---|---|
| 패킹 | `#pragma pack(push,N)` / `#pragma pack(pop)` | 소스 범위 | 멤버 오프셋 계산 기준을 N으로 제한 → sizeof 감소 가능 | 파일/네트워크 포맷 정합성 필요 시 |
| 표준 정렬 지정 | `alignas(N)` | 타입/변수/멤버 | 타입의 요구 정렬 증가 → sizeof 증가 | 성능/SIMD/하드웨어 요구 시 |
| MSVC 확장 | `__declspec(align(N))` | 타입/변수 | `alignas`와 유사 (MSVC 전용) | MSVC 전용 코드에서 사용 |


## MSVC(x64) 예제 (결과 포함)

```cpp
// 기본 MSVC 정렬
struct A { char c; int i; };
static_assert(sizeof(A) == 8); // MSVC x64: sizeof(A) == 8, alignment: 4

#pragma pack(push, 1)
struct B { char c; int i; };
#pragma pack(pop)
static_assert(sizeof(B) == 5); // MSVC x64: sizeof(B) == 5, alignment: 1

#pragma pack(push, 16)
struct BB { char c; int i; };
#pragma pack(pop)
static_assert(sizeof(BB) == 8); // BB alignment: 4

struct alignas(16) C { char c; int i; };
static_assert(sizeof(C) == 16); // MSVC x64: sizeof(C) == 16

struct alignas(1) CC { char c; int i; };
// warning C4359: 'CC': Alignment specifier is less than actual alignment (4), and will be ignored.
static_assert(sizeof(CC) == 8);

struct alignas(32) CCC { char c; int i; };
// warning C4359: 'CCC': Alignment specifier is less than actual alignment (4), and will be ignored.
static_assert(sizeof(CCC) == 32); // alignment: 32

__declspec(align(1)) struct D { double d; int i; };
// warning C4359: 'D': Alignment specifier is less than actual alignment (8), and will be ignored.
static_assert(sizeof(D) == 16); // alignment: 8

__declspec(align(16)) struct DD { double d; int i; };
static_assert(sizeof(DD) == 16); // alignment: 16

__declspec(align(32)) struct DDD { double d; int i; };
static_assert(sizeof(DDD) == 32); // alignment: 16

struct BBB {
	B b; // 5 bytes
	int a;
};
static_assert(sizeof(BBB) == 12); // alignment: 4

struct CCCC {
	CCC c; // 32 bytes, alignment: 32
	int a;
};
static_assert(sizeof(CCCC) == 64); // alignment: 32

```

설명: MSVC는 구성 멤버의 최대 정렬 요구를 기준으로 구조체 정렬을 결정합니다. `alignas`나 `__declspec(align())`로 요구 정렬을 올리면 `sizeof`가 그 배수로 늘어나고, `#pragma pack`은 멤버 오프셋 계산 시 적용되는 최대 정렬을 낮춰 구조체 크기를 줄입니다.


간단 요약: MSVC 환경에서는 `#pragma pack`으로 패킹을 낮춰 크기를 줄이고, `alignas`/`__declspec`로 정렬을 높여 접근 성능이나 하드웨어 요구를 만족시킵니다. 크로스-플랫폼/호환성이 중요하면 `alignas`를 우선 고려하고, 패킹은 최소한의 범위에서만 사용하세요.

## 참고사항

앞서 쓴 글에서도 언급했듯, 정렬은 객체의 시작 주소에 따라 결정됩니다. 시작 주소 자체가 정렬되어 있지 않다면 `alignas`를 붙여도 실제 정렬이 보장되지 않을 수 있습니다.

과거에는 스택에서 16바이트 이상의 정렬이 항상 보장되지 않던 시기가 있었고, VS2019에서 관련 동작이 수정된 적이 있습니다

[관련 이슈](https://developercommunity.visualstudio.com/t/alignas-of-structured-bindings-c17-not-working/627792)

## 참고 링크

- [https://velog.io/@jellypower/CMSVC%EC%9D%98-alignment-%EC%A7%80%EC%A0%95%EC%9E%90%EA%B0%84%EC%9D%98-%EC%B0%A8%EC%9D%B4](https://velog.io/@jellypower/CMSVC%EC%9D%98-alignment-%EC%A7%80%EC%A0%95%EC%9E%90%EA%B0%84%EC%9D%98-%EC%B0%A8%EC%9D%B4)
- [https://learn.microsoft.com/ko-kr/cpp/preprocessor/pack?view=msvc-170](https://learn.microsoft.com/ko-kr/cpp/preprocessor/pack?view=msvc-170)
- [https://learn.microsoft.com/ko-kr/cpp/build/x64-software-conventions?view=msvc-170#x64-structure-alignment-examples](https://learn.microsoft.com/ko-kr/cpp/build/x64-software-conventions?view=msvc-170#x64-structure-alignment-examples)
- [https://learn.microsoft.com/ko-kr/cpp/cpp/alignas-specifier?view=msvc-170](https://learn.microsoft.com/ko-kr/cpp/cpp/alignas-specifier?view=msvc-170)
- [http://post.procademy.co.kr/archives/865](http://post.procademy.co.kr/archives/865)

