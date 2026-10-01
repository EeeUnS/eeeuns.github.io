---
layout: post
title:  "alignment 정리"
date:   2025-12-27 19:26:01 +0900
tag: cs
---

기본적인 정렬과 패딩은 이전 글에서 다뤘으니 생략.

- [패딩과 정렬](https://eeeuns.github.io/2024/02/12/alignment/)

구조체 정렬을 건드릴 때 쓰는 키워드는 몇 가지가 있다.

- `#pragma pack`
- `alignas`
- `__declspec(align())`

이름만 보면 전부 비슷한 일을 할 것 같은데, 같은 숫자를 넣어도 결과가 다르다. `pack(16)`을 쓴다고 구조체가 16byte 정렬이 되는 건 아니다.

MSVC x64에서 `char`, `int`, `double`을 넣은 간단한 구조체로 차이를 보자. 아래 예제는 별도의 패킹 옵션을 바꾸지 않은 환경을 기준으로 한다.

## pack은 멤버 정렬의 상한

```cpp
// 기본 MSVC 정렬
struct A { char c; int i; };
static_assert(sizeof(A) == 8);
static_assert(alignof(A) == 4);

#pragma pack(push, 1)
struct B { char c; int i; };
#pragma pack(pop)
static_assert(sizeof(B) == 5);
static_assert(alignof(B) == 1);

#pragma pack(push, 16)
struct BB { char c; int i; };
#pragma pack(pop)
static_assert(sizeof(BB) == 8);
static_assert(alignof(BB) == 4);
```

기본 구조체 `A`는 `char` 다음에 바로 `int`를 붙이지 않는다. `int`의 4byte 정렬을 맞추기 위해 3byte를 비워두고, offset 4부터 `int`를 배치한다. 그래서 전체 크기는 8byte다.

`pack(1)`을 적용한 `B`는 이 멤버 정렬의 상한을 1byte로 낮춘다. `char` 바로 다음인 offset 1에 `int`가 들어가니 크기가 5byte로 줄어든다.

그럼 `pack(16)`은?

16byte까지 허용한다는 뜻이지 모든 멤버를 16byte 정렬로 올리라는 뜻은 아니다. `int`는 원래 필요한 4byte 정렬이면 충분하니 `BB`는 `A`와 같은 크기와 정렬을 가진다. [pack 문서](https://learn.microsoft.com/ko-kr/cpp/preprocessor/pack?view=msvc-170)에서 설명하는 멤버 배치 기준을 보면 이 차이를 확인할 수 있다.

`push`와 `pop`은 현재 패킹 설정을 저장했다가 복원하는 용도다. `pop`을 빠뜨리면 이후에 선언하는 다른 구조체까지 영향을 받으니 적용할 범위만 감싸는 편이 좋다.

## alignas는 정렬 요구를 높인다

```cpp
struct alignas(16) C { char c; int i; };
static_assert(sizeof(C) == 16);
static_assert(alignof(C) == 16);

struct alignas(32) CCC { char c; int i; };
static_assert(sizeof(CCC) == 32);
static_assert(alignof(CCC) == 32);
```

`alignas(16)`은 구조체 자체에 16byte 정렬을 요구한다. 위의 `C`는 멤버 배치에 필요한 공간은 여전히 8byte지만 전체 크기는 16byte가 된다.

배열로 놓는다고 생각하면 이유가 간단하다. 첫 원소가 16byte 정렬이고 다음 원소가 8byte 뒤에 있으면 다음 원소는 16byte 정렬을 만족하지 못한다. 따라서 원소 간 간격인 `sizeof(C)`도 정렬의 배수가 되어야 한다. `CCC`가 32byte가 되는 것도 같은 이유다.

반대로 `alignas(1)`로 기존 정렬을 낮출 수 있을까?

```cpp
// 잘못된 지정의 예. 정상 예제와 분리해서 진단을 확인할 것.
struct alignas(1) CC { char c; int i; };
```

`int` 때문에 원래 4byte 정렬이 필요한데 1byte를 요구하고 있다. **alignas로 타입에 필요한 정렬보다 약한 정렬을 지정할 수는 없다.** [cppreference의 alignas 설명](https://en.cppreference.com/w/cpp/language/alignas.html)에서도 이 경우를 잘못된 프로그램으로 설명하고, [MSVC 문서](https://learn.microsoft.com/ko-kr/cpp/cpp/alignas-specifier?view=msvc-170)에도 같은 제한이 있다. MSVC에서 C4359 진단과 함께 무시되는 경우를 봤다고 해서 정렬을 낮추는 방법으로 쓰면 안 된다.

## MSVC의 __declspec(align())

```cpp
__declspec(align(16)) struct DD { double d; int i; };
static_assert(sizeof(DD) == 16);
static_assert(alignof(DD) == 16);

__declspec(align(32)) struct DDD { double d; int i; };
static_assert(sizeof(DDD) == 32);
static_assert(alignof(DDD) == 32);
```

MSVC 전용 지정자다. 여기서처럼 구조체의 정렬 요구를 올리는 용도는 `alignas`와 비슷하다. [공식 문서](https://learn.microsoft.com/ko-kr/cpp/cpp/align-cpp?view=msvc-170)에서도 정렬 제약을 높이는 방향으로만 사용할 수 있다고 설명한다.

`double`과 `int`를 넣은 기본 구조체는 8byte 정렬에 크기 16byte다. `DD`는 정렬을 16byte로 올렸지만 크기는 이미 그 배수라서 그대로 16byte다. `DDD`는 32byte 정렬을 요구하니 크기도 32byte로 늘어난다.

즉 정렬을 올린다고 `sizeof`가 항상 커지는 건 아니다. 기존 크기가 새 정렬의 배수인지도 같이 봐야 한다. 다른 컴파일러에서도 사용할 코드라면 표준 지정자인 `alignas`를 쓰는 편이 낫다.

## 구조체 안에 다시 넣는다면

```cpp
struct BBB {
	B b; // 5 bytes
	int a;
};
static_assert(sizeof(BBB) == 12);
static_assert(alignof(BBB) == 4);

struct CCCC {
	CCC c; // 32 bytes, alignment: 32
	int a;
};
static_assert(sizeof(CCCC) == 64);
static_assert(alignof(CCCC) == 32);
```

`B`를 만들 때의 `pack(1)`은 이미 `pop`으로 끝냈다. `B` 자체의 크기는 5byte지만, 이를 멤버로 넣는 `BBB`까지 1byte 패킹이 되는 건 아니다.

`b`가 offset 0부터 5byte를 쓰고, `a`는 4byte 정렬이 필요하니 offset 8에 들어간다. 그래서 `BBB`의 크기는 12byte다.

`CCCC`는 32byte 정렬이 필요한 `CCC`를 멤버로 가진다. `c`가 32byte, `a`가 4byte를 쓰고 전체 크기는 32의 배수로 맞춰지니 64byte가 된다. 바깥 구조체를 별도로 패킹하지 않은 이 예제에서는 멤버의 정렬 요구가 바깥 구조체에도 이어지는 셈이다.

## 실제 주소도 맞아야 한다

`alignas`를 붙인 객체를 정상적으로 선언하면 컴파일러가 그 정렬을 만족하도록 배치해야 한다. 요구 정렬이 컴파일 타임에 결정된다는 이유로 실제 주소의 정렬이 보장되지 않는다는 뜻은 아니다.

근데 임의의 버퍼 주소를 해당 타입의 포인터로 캐스팅한다고 그 주소가 정렬되는 건 아니다. 직접 확보한 메모리에 객체를 놓거나 데이터를 복사해서 접근한다면 저장 공간의 주소도 요구 정렬을 만족하는지 확인해야 한다. 캐스팅은 주소를 옮겨주지 않는다.

컴파일러 버그는 별개다. [구조적 바인딩의 alignas 관련 이슈](https://developercommunity.visualstudio.com/t/alignas-of-structured-bindings-c17-not-working/627792) 같은 사례도 있으니 특이한 동작을 확인했다면 컴파일러 버전과 재현 조건을 같이 봐야 한다.

정렬을 더 크게 잡거나 패킹을 줄이는 것 자체가 최적화는 아니다. 전자는 패딩으로 메모리 사용량이 늘 수 있고, 후자는 멤버가 정렬되지 않은 주소에 놓여 접근 비용이나 하드웨어 제약이 문제가 될 수 있다. 어떤 레이아웃과 접근이 필요한지 보고 선택해야 한다.

## 참고 링크

- [https://velog.io/@jellypower/CMSVC%EC%9D%98-alignment-%EC%A7%80%EC%A0%95%EC%9E%90%EA%B0%84%EC%9D%98-%EC%B0%A8%EC%9D%B4](https://velog.io/@jellypower/CMSVC%EC%9D%98-alignment-%EC%A7%80%EC%A0%95%EC%9E%90%EA%B0%84%EC%9D%98-%EC%B0%A8%EC%9D%B4)
- [https://learn.microsoft.com/ko-kr/cpp/preprocessor/pack?view=msvc-170](https://learn.microsoft.com/ko-kr/cpp/preprocessor/pack?view=msvc-170)
- [https://learn.microsoft.com/ko-kr/cpp/build/x64-software-conventions?view=msvc-170#x64-structure-alignment-examples](https://learn.microsoft.com/ko-kr/cpp/build/x64-software-conventions?view=msvc-170#x64-structure-alignment-examples)
- [https://learn.microsoft.com/ko-kr/cpp/cpp/alignas-specifier?view=msvc-170](https://learn.microsoft.com/ko-kr/cpp/cpp/alignas-specifier?view=msvc-170)
- [http://post.procademy.co.kr/archives/865](http://post.procademy.co.kr/archives/865)

