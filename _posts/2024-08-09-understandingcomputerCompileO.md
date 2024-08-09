---
layout: post
title:  "C언어가 cpu에 작동하기까지 심화 : 컴파일"
date:   2024-08-09 19:26:01 +0900
tag: etc
---

- [기본글](https://eeeuns.github.io/2021/10/27/understandingcomputer/)

기본글을 쓰고 내가 아는 모든 걸 쓰겠다는 생각은 옛날부터 해왔다.
가장 기본적인 구성을 이렇게 두가지로 하려고했는데..
1. 컴파일 과정
2. 실제 실행 과정

링커글이 굉장히 괜찮은 글이 있어서 링크로만 달고 퉁치고 있다가,
이번에 어떤 계기로 컴파일 과정을 쓰고 있다 내용이 너무 많아지고 컴파일 글을 다 완성하고 올리려니 언제 완성될지 감도 안잡히기에 짜를시점에 전반부를 올린다..(예전 퉁친 글은 날렸다)
그리고 남은 내용들이 C언어가 cpu에 작동하기까지라는 주제에 좀 벗어난 얘기들이다. 후반부라고 올릴지, 별개로 올릴진 일단 정리해봐야알것같다..


이번 내용은 대부분 CSAPP 링커 장에 해당하는 내용들이다.

-----


가장 기본적인 전체 이미지

![Untitled](/images/understandingcomputerCompileO/Untitled.png)

저 과정전체를 해주는 주체를 컴파일러라고 한다. (hello.c를 컴파일하여 hello를 뱉어주는 주체)

현재 사용하는 컴파일러는 대표적으로 세 개 정도가 있다. 

- MSVC (Microsoft Visual Complier) : windows Visual studio ide를 사용할때 주로 사용
- Clang
- GCC

위의 사진에서 보통 가장 크게 두 단계를 거친다

컴파일은 상황에따라서 넓거나 좁은 의미로 쓸 수 있는데 아래로 갈수록 좁은 의미이다.

1. 코드를 컴파일하여 실행파일을 뱉는 전체 과정을 호칭한다.  위 이미지 전체 과정
2. 맨 마지막 링커단계를 제외하고 구분하여 컴파일, 링커로 구분하여 호칭하기도한다.
3. 더 세부적으로 매크로 작업이 끝난후 어셈파일로 뱉는 과정을 컴파일이라고도 한다.

글에서는 1, 2의 의미를 혼용하여 사용할 것이다.

간단하게 링커를 제외한 컴파일 전체 과정을 한번 훑고 넘어가자.

# Compile

## Preprocessor

1. **매크로 명령어를 처리한다**
2. 여기서 나온 결과물을 가지고 **실제로 컴파일하는 파일** 그 자체라고 생각하면 좋다. 진짜 전처리다.

#define symbol 숫자 이런거는 실제 symbol은 숫자로 변경되서, 코드상의 정보로 남지않기에 디버깅시에 참고가 아예 안되는 문제가 있다.

ifdef 로 걸러진 define이나 코드들은 실제로 **실행파일에 없는 코드**가 된다.

매우 중요한 정보, 로직이라서 배포 실행파일에 들어가는것자체가 문제가 되는 케이스는 아예 해당 단계에서 제외를 시켜서, 아예 배포파일에서 제외를 시켜버릴 수 있다.

**#include는 진짜로 복사 붙여넣기다.** 헤더는 관용상 붙지 .h란 확장자가 아니어도된다. 

각 단계를 정확히 이해한다면 cpp를 include해도 잘 작동한다.

50줄짜리 Cpp파일 하나를 컴파일한다고 생각해보자.

그런데 여기에 딸린 헤더파일이 2만 줄 정도 되면 실제로는 2만 50줄짜리 Cpp파일을 컴파일 하는거다

## Compiler

결과물을 실제로 어셈블리어로 변환하는 과정

해당 결과물이 우리가 마지막으로 그나마 읽을 수 있는 파일 중 가장 기계어에 가깝다.

이 다음부터 결과물은 이진 파일이다.

## Assembler

어셈블리어를 이진 파일로 변환한다. 여기서부터는 간단한 파일은 실행파일이 될 수도 있다.

이 결과물을 object 파일이라고 한다. 

컴파일러는 단일 Cpp 파일을 한번씩 컴파일하여 object파일을 뽑아내고 N개의 cpp파일에 대해서 N개의 object파일을 뽑아내고 N개의 object파일을 링킹하여 1개의 온전한 실행파일을 뽑아내는 과정이다.

자 여기까지 서론이 끝났다.

# 컴파일의 결과 : Object File

object 파일은 크게 세가지 종류가 있다.

1. reloacatable object file 재배치 가능 목적파일
    
    하자가 있어서 실행파일이 못 된 파일이다. 
    
    a.cpp을 컴파일했는데 a.cpp 에서 호출한 함수 func() 가  b.cpp 파일에 구현되어 있는 경우 해당 함수의 컴파일 할 수 없으니 실행파일이 될 수 없는 상황이다. 
    
    이런 목적파일을 모아서 링킹을 하면 실행 파일이 될 수 있다.
    
    리눅스 gcc에서 보통 .o 확장자을 사용하고 windows msvc 에서는 .obj 파일 확장자이다.
    
2. excuteable object file 실행 가능 목적 파일
    
    실행 가능 목적 파일은 우리가 원하는 최종 결과인 실행파일이다.
    windows에서는 .exe 확장자, linux에서는 .out 확장자(꼭 그렇진 않다)이다.
    
    실행 파일을 실제로 실행하는 것은 운영체제의 담당인지라 파일 형식은 운영체제가 정한 형식에 종속된다. windows의 실행 파일은 linux에서 실행 불가능하다.
    
    Linking단계까지 고려하여 object 파일 포맷이 OS에 종속되는데 linux에서 목적 파일 포맷은 ELF(Executable and Linkable Format**)** 포맷이 라하고 Windows 에서는 PE Format(Portable Exceuteable)이라 한다.
    
3. Shared object file 공유 목적 파일
    
    내가 작성한 코드가 나의 프로젝트 많이 아닌 완전 타인에게도 코드를 공유하기위해서 
    
    작성한 Cpp파일을 모두 받아서 컴파일까지 맞춰서 세팅하는것은 비효율적이다. 
    
    사실은 변경되지않는 코드를 컴파일하는것조차 비효율적이다. 
    
    묶인 파일을 하나만 전달받아서 사용하면 되지않을까 하는 아이디어에서 출발
    
    뒤에서 더 자세하게 설명
    
    # Object File 상세
    
    실제 PE File 공식 문서 : [https://learn.microsoft.com/ko-kr/windows/win32/debug/pe-format?redirectedfrom=MSDN](https://learn.microsoft.com/ko-kr/windows/win32/debug/pe-format?redirectedfrom=MSDN)
    
    본 Windows 에서 실행파일은 이미지/PE(이식 가능한 실행 파일) 파일, 재배치 가능 목적파일은 개체 /COFF(공용 개체 파일 형식) 파일 이라고한다.
    
     
    
    상세하게는 명칭 등 차이가 있지만 전체적인 섹션 개요는 ELF와 비슷한 구조이다.
    
    ![Untitled](/images/understandingcomputerCompileO/Untitled%201.png)
    
    ELF 재배치 파일 구조
    
    ![Untitled](/images/understandingcomputerCompileO/Untitled%202.png)
    
    각 구역 별로 Section이 나뉜다. 
    
    목적파일 전체에서 가장 기본이 되는 네가지
    
    - .bss : 초기화되지 않은 전역변수와 정적변수
    - .data : 초기화된 전역변수 및 정적 변수
    - .rdata : 초기화된 읽기 전용 데이터 (스트링), 스위치 케이스 점프 테이블
    - .text : 명령어 머신 코드
    
    프로세스 메모리 구조에서의 익숙한 요 내용에 해당 하는 section이다.
    
    ![Untitled](/images/understandingcomputerCompileO/Untitled%203.png)
    
    - .debug 섹션
        - **디버그 디렉터리**
        - **.debug$F(개체만 해당)**
        - **.debug$S(개체만 해당)** 이 섹션에는 Visual C++ 디버그 정보(심볼 정보)가 포함
        - **.debug$P(개체만 해당)** 이 섹션에는 Visual C++ 디버그 정보(미리 컴파일된 정보)가 포함. 개체를 사용하여 생성된 미리 컴파일된 헤더를 통해 컴파일된 모든 개체 간에 공유되는 형식
        - **.debug$T(개체만 해당)** 이 섹션에는 Visual C++ 디버그 정보(형식 정보)가 포함
    
    ## **Microsoft 디버그 정보에 대한 링커 지원**
    
    디버그 정보를 지원하기 위해 링커에서 다음을 수행합니다.
    
    - **.debug$F**, **debug$S**, **.debug$P** 및 **.debug$T** 섹션에서 모든 관련 디버그 데이터를 수집합니다.
    - 링커에서 생성한 디버깅 정보와 함께 해당 데이터를 PDB 파일로 처리하고, 이를 참조하는 디버그 디렉터리 항목을 만듭니다.
    
    - .**edata** : (이미지만 해당) : 내보내기 데이터 섹션.
    다른 이미지에서 동적 링크를 통해 액세스할 수 있는 심볼에 대한 정보가 포함
    - .reloc 섹션(이미지만 해당) : 이미지의 모든 베이스 재배치에 대한 항목이 포함
    
    - **.drectve**(개체만 해당) **:** IMAGE_SCN_LNK_INFO 세팅되어있어야함 ****링커 옵션
    - **.idata** 섹션 : 심볼 가져오기 정보
    - .pdata 섹션 :  예외 처리에 사용되는 함수 테이블 항목의 배열
    - .tls 섹션 : 스레드 로컬 스토리지 테이블 주소 및 크기
    - .rsrc 섹션 : 리소스 데이터 (GUI 프로그램에서 많이 볼 수 있는 섹션, 그래픽 정보 저장)
    - .sxdata 섹션
    - .cormeta 섹션(개체만 해당)
    
    또한 개체 파일은 각 섹션마다 **COFF 재배치(개체만 해당)**을 가지고 있다. 주소 지정이 필요한 심볼을나중에 해당 심볼의 주소를 써넣기 위한 주소를 기입하기 위한 offset을 기록하는 테이블이다. 
    
    그리고 제일 마지막에 COFF 심볼 테이블이 존재한다
    

# Linker

여러개의 오브젝트 파일을 하나의 실행가능한 오브젝트 파일로 합쳐버린다.

링커는 두단계로 진행된다.

1. Symbol resolution 심볼 해석
각 재배치 목적파일의 심볼테이블을 합쳐서 완전한 하나의 심볼 테이블 엔트리를 가진다.

2. Relocation 재배치 : 앞단계의 심볼 테이블 엔트리를 통해서 실제로 심볼에 주소를 할당한다.
    1. 섹션과 심볼 정의를 재배치 : 각 파일마다 겹치는 섹션을 하나의 단일화된 섹션으로 합친다.
    2. 섹션 내 심볼 참조를 재배치 : 모든 심볼 참조를 수정하여 정확한 주소를 가르키도록 작업 두가지 방식이 있다.
        - PC 상대 참조의 재배치 : PC (Program Counter)을 기준으로 기록하는 방식
        - 절대 참조의 재배치 : 절대적인 변수, 함수의 주솟값을 직접 박아 기록하는 방식

# Symbol resolution 시 중복 정의된 심볼 처리

이 처리를 위해서 리눅스에서는 강한심볼과 약한 심볼이라는 개념이 등장한다.

1. 강한심볼 : 초기화된 전역변수
2. 약한 심볼 : 초기화되지않은 전역변수

규칙

1. 동일한 이름을 갖는 복수의 강한 심볼은 허용하지않는다.
2. 동일한 이름의 강한 심볼과 다수의 약한 심볼들은 강한 심볼을 택한다
3. 동일한 이름의 여러개의 심볼은 어떤 약한 심볼을 택해도 상관없다.

초기화하고 초기화 하지않은 같은 이름의 전역변수가 각각 있다던가 타입이 다른 전역 변수가 이름이 같다던가 등의 상황은 의도하지않은 버그를 왕창 낼 수 있다.

전역변수는 사용하려면 최대한 static 을 달고 사용하는게 좋다.

외부에서 가져가서 사용시에는 해당 변수 레퍼런스(or 포인터)를 반환하는 Getter 함수를 만들어서 호출하는것이 정신건강에 이롭다.

# Shared object file 공유 목적 파일 : 라이브러리

# Static lib

정적 라이브러리

사용자 입장에서는 컴파일된 목적파일 집합만 필요하면 된다.

필드된 파일 또한 여러개면 번거로우니 통합된 한개의 파일만 있자라고 생각한게 바로 정적 라이브러리다.

최초의 컴파일 된 라이브러리 파일만 가지고, 링킹시에 합치기해서 최종 실행파일에 실행파일에 모든것이 다 들어가있다. (실행파일 용량도 그만큼 증가)

그래서 내용물을 묶어 하나의 object file만들고,  헤더파일을 제공 → 사용자는 헤더 파일을 include하여 함수를 호출하고 링커 단계에서 묶음 object file을 통해 실행파일로 만드는 방식이다.

리눅스에서는 .a 윈도우즈에서는 .lib 확장자를 사용한다.

# Dynamic Link Lib

동적 라이브러리 ([Widows DLL ref](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-libraries) )

static lib는 실행 파일을 만드는 과정에서 보면 군더더기 없이 깔끔한 방식이다.

하지만, 해당 라이브러리의 사용을 정말 많은 실행 파일에서 동시에 사용한다고 생각해보자.

그러면 모든 실행파일에 해당 라이브러리를 링킹하여 만들 수도있다. 

그러면 해당 프로그램이 실행될 때, 라이브러리 관련된 것이 개별 프로세스마다 메모리에 적재 될 것이다.

메모리 전체의 입장에서보면 동일한 코드 로직이 프로세스 별 메모리에 모두 올라가 있는 셈이다.

추가로 참조하는 모든 실행 파일 크기가 커진다.

이게 비효율적이라 판단하여 이 라이브러리를 메모리에 한번만 적재하고 여러 실행파일에서 해당 라이브러리 메모리를 참조만하여 사용할 수 있도록 구조를 잡았다.

리눅스에서는 .so 파일 확장자로 윈도우즈에서 흔히말하는 DLL 확장자 파일이 여기에 해당한다.

작동방식은 크게 두가지로 나뉜다. 

1. **implicit load-time linking :** Loader가 프로세스에 메모리를 적재하는 때에 DLL도 같이 적재를 한다.  이때 이 작업을 하는 주체를 Dynamic linker라 한다.

![Untitled](/images/understandingcomputerCompileO/Untitled%204.png)

DLL 을 처음에 생성시에 두가지 파일이 만들어진다 dll, lib 파일 

- dll이 실제 대부분의 main함수없는 파일이고, 실제 로딩시에 사용 (dll 파일이 제공하는 함수는 **.edata 섹션에 존재한다**)
- lib 파일은 링킹시에 사용 참조를 알려주는 역할

링킹시에 사용 함수의 구현체가 없는 경우 당연히 링킹에러가 발생한다. 이때 .lib 파일이 링킹시에 해당 함수의 구현체가 DLL에 있음을 인지한다. (이에 대한 정보는 .**idata 섹션에 존재한다**)

그러고 파일 실행시에  로더가 해당 DLL파일을 실제로 로드하면서 함수의 위치를 가르키게한다.

![Untitled](/images/understandingcomputerCompileO/Untitled%205.png)
실행하자마자 dll 파일이 없으면 이런 에러가 뜰거다.


2. **explicit run-time linking :** 소스파일에서 직접 DLL open(dlopen, LoadLibrary) 함수를 호출하여 DLL을 로딩하는 경우

![Untitled](/images/understandingcomputerCompileO/Untitled%206.png)

함수 호출을 하여 DLL을 작성 코드 로직상 직접 로드를 하는 방법이다.

로드한 DLL에서 특정 함수명에 따른 직접 함수 주소를 불러와 함수포인터로 저장하고 이 함수포인터를 호출하여 간접사용한다. 

당연히 컴파일단에서는 어떤 DLL을 로드하는지, 어떤 함수를 사용할지 **컴파일타임에 파악이 불가능**하여 실행파일은 본인이 DLL을 사용하는지 조차 알 수 없다.

## 모듈

최신 언어들의 사용 방법

include따위는 정말 옛날 언어에나 해당 기능이다.  최신 언어에는 모듈 방식을 채택하고 있다.

include는 보면 알겠지만 **헤더파일 전체**를 복사 붙여넣기한다. 

cpp 개별 파일 기준으로 보면 모든 헤더 파일에 있는 모든 정보가 필요할 일은 보통 없다.

요 헤더 파일조차 쪼개서 특정 함수 하나만 가져다 쓰자라는 방안에서 생겨났다.

# Windows FE 포맷 실제 분석

자 직접 한번 확인해보자

```jsx
static int staticVar = 0x12345678;

void func1();
void func()
{
    staticVar++;
}
int global = 0x12abcdef;

const char * str = "Hello";

int main()
{
    staticVar++;
    func1();
}
```

func1 파일이 없어서 **당연히 링커에러가 발생**한다.

그럼 obj파일은 생성 되어있으니 요 obj 파일을 한번 PEView 로 열어보자

**drectve :** 링커옵션이 들어있다 

![Untitled](/images/understandingcomputerCompileO/Untitled%207.png)

- debug$S 심볼 정보

![Untitled](/images/understandingcomputerCompileO/Untitled%208.png)

- data : 초기화된 전역변수 및 정적 변수의 값이 들어있다. global 및 staticVAr

![Untitled](/images/understandingcomputerCompileO/Untitled%209.png)

- rdata 초기화된 읽기 전용 데이터 (스트링)

![Untitled](/images/understandingcomputerCompileO/Untitled%2010.png)

자 이제 어셈을 까보자 어셈은 PE View에서 못보는 듯해서 다른 툴을 찾아봤을때

공식으로 제공하는 것 중에는 Visual Studio msvc 내장 툴인 [dumpbin.exe](https://learn.microsoft.com/en-us/cpp/build/reference/dumpbin-reference?view=msvc-170) 으로 직접 볼 수도 있다.

공식적으로 지원하는 툴인 만큼 가장 정확하고 확실하다고 볼 수 있다.

visual studio 설치 폴더안에서 해당 파일로 검색하면 나온다.

이 툴을 통해서 obj text section을 Disasm한 결과도 볼 수 있다

```jsx
dumpbin.exe /all 경로/파일.obj
```

```jsx
dumpbin.exe /DISASM 경로/파일.obj
```

```jsx
Microsoft (R) COFF/PE Dumper Version 14.39.33522.0
Copyright (C) Microsoft Corporation.  All rights reserved.

Dump of file 

File Type: COFF OBJECT

?func@@YAXXZ (void __cdecl func(void)):
  0000000000000000: 40 55              push        rbp
  0000000000000002: 57                 push        rdi
  0000000000000003: 48 81 EC E8 00 00  sub         rsp,0E8h
                    00
  000000000000000A: 48 8D 6C 24 20     lea         rbp,[rsp+20h]
  000000000000000F: 48 8D 0D 00 00 00  lea         rcx,[__416313BE_testproject@cpp]
                    00
  0000000000000016: E8 00 00 00 00     call        __CheckForDebuggerJustMyCode
  000000000000001B: 8B 05 00 00 00 00  mov         eax,dword ptr [?staticVar@@3HA]
  0000000000000021: FF C0              inc         eax
  0000000000000023: 89 05 00 00 00 00  mov         dword ptr [?staticVar@@3HA],eax
  0000000000000029: 48 8D A5 C8 00 00  lea         rsp,[rbp+0C8h]
                    00
  0000000000000030: 5F                 pop         rdi
  0000000000000031: 5D                 pop         rbp
  0000000000000032: C3                 ret

__JustMyCode_Default:
  0000000000000000: C2 00 00           ret         0

main:
  0000000000000000: 40 55              push        rbp
  0000000000000002: 57                 push        rdi
  0000000000000003: 48 81 EC E8 00 00  sub         rsp,0E8h
                    00
  000000000000000A: 48 8D 6C 24 20     lea         rbp,[rsp+20h]
  000000000000000F: 48 8D 0D 00 00 00  lea         rcx,[__416313BE_testproject@cpp]
                    00
  0000000000000016: E8 00 00 00 00     call        __CheckForDebuggerJustMyCode
  000000000000001B: 8B 05 00 00 00 00  mov         eax,dword ptr [?staticVar@@3HA]
  0000000000000021: FF C0              inc         eax
  0000000000000023: 89 05 00 00 00 00  mov         dword ptr [?staticVar@@3HA],eax
  0000000000000029: E8 00 00 00 00     call        ?func1@@YAXXZ
  000000000000002E: 33 C0              xor         eax,eax
  0000000000000030: 48 8D A5 C8 00 00  lea         rsp,[rbp+0C8h]
                    00
  0000000000000037: 5F                 pop         rdi
  0000000000000038: 5D                 pop         rbp
  0000000000000039: C3                 ret

  Summary

          98 .chks64
          14 .data
         34C .debug$S
          64 .debug$T
          82 .drectve
           1 .msvcjmc
          18 .pdata
           6 .rdata
           8 .rtc$IMZ
           8 .rtc$TMZ
          70 .text$mn
          20 .xdata
```

어셈상에 실제로 참조가 필요한 곳은 모두 0으로 밀려있다. 그럼 어떻게 disasemble로 변수를 정확하게 집어냈을까?

해당 정보들은 개별로 **COFF 재배치** 테이블에 존재한다.

```jsx
RELOCATIONS #A
                                                Symbol    Symbol
 Offset    Type              Applied To         Index     Name
 --------  ----------------  -----------------  --------  ------
 00000012  REL32                      00000000        15  __416313BE_testproject@cpp
 00000017  REL32                      00000000        2D  __CheckForDebuggerJustMyCode
 0000001D  REL32                      00000000        41  ?staticVar@@3HA (int staticVar)
 00000025  REL32                      00000000        41  ?staticVar@@3HA (int staticVar)
 0000002A  REL32                      00000000        28  ?func1@@YAXXZ (void __cdecl func1(void))

```

최종적인 전체 해당 OBJ의 COFF SYMBOL TABLE 이다

```jsx
COFF SYMBOL TABLE
000 010582F2 ABS    notype       Static       | @comp.id
001 80010390 ABS    notype       Static       | @feat.00
002 00000002 ABS    notype       Static       | @vol.md
003 00000000 SECT1  notype       Static       | .drectve
    Section length   82, #relocs    0, #linenums    0, checksum        0
    Relocation CRC 00000000
006 00000000 SECT2  notype       Static       | .debug$S
    Section length  1A0, #relocs    8, #linenums    0, checksum        0
    Relocation CRC 6F82BF45
009 00000000 SECT3  notype       Static       | .data
    Section length   14, #relocs    1, #linenums    0, checksum 9BE618FC
    Relocation CRC D3D0C3E9
00C 00000000 SECT3  notype       External     | ?global@@3HA (int global)
00D 00000008 SECT3  notype       External     | ?str@@3PEBDEB (char const * const str)
00E 00000000 SECT4  notype       Static       | .rdata
    Section length    6, #relocs    0, #linenums    0, checksum 60817DAB, selection    2 (pick any)
    Relocation CRC 00000000
011 00000000 SECT4  notype       External     | ??_C@_05COLMCDPH@Hello@ (`string')
012 00000000 SECT5  notype       Static       | .msvcjmc
    Section length    1, #relocs    0, #linenums    0, checksum 77073096
    Relocation CRC 00000000
015 00000000 SECT5  notype       Static       | __416313BE_testproject@cpp
016 00000000 SECT6  notype       Static       | .text$mn
    Section length   33, #relocs    4, #linenums    0, checksum 8A2C0D07, selection    1 (pick no duplicates)
    Relocation CRC 64879072
019 00000000 SECT7  notype       Static       | .debug$S
    Section length   9C, #relocs    4, #linenums    0, checksum        0, selection    5 (pick associative Section 0x6)
    Relocation CRC BD630DA2
01C 00000000 SECT8  notype       Static       | .text$mn
    Section length    3, #relocs    0, #linenums    0, checksum 922B422E, selection    2 (pick any)
    Relocation CRC 4A6A8444
01F 00000000 SECT9  notype       Static       | .debug$S
    Section length   6C, #relocs    2, #linenums    0, checksum        0, selection    5 (pick associative Section 0x8)
    Relocation CRC C5009019
022 00000000 SECTA  notype       Static       | .text$mn
    Section length   3A, #relocs    5, #linenums    0, checksum AAA72B5C, selection    1 (pick no duplicates)
    Relocation CRC A65313CD
025 00000000 SECTB  notype       Static       | .debug$S
    Section length   A4, #relocs    4, #linenums    0, checksum        0, selection    5 (pick associative Section 0xA)
    Relocation CRC 16E24314
028 00000000 UNDEF  notype ()    External     | ?func1@@YAXXZ (void __cdecl func1(void))
029 00000000 SECT6  notype ()    External     | ?func@@YAXXZ (void __cdecl func(void))
02A 00000000 SECTA  notype ()    External     | main
02B 00000000 UNDEF  notype ()    External     | _RTC_InitBase
02C 00000000 UNDEF  notype ()    External     | _RTC_Shutdown
02D 00000000 UNDEF  notype ()    External     | __CheckForDebuggerJustMyCode
02E 00000000 SECT8  notype ()    External     | __JustMyCode_Default
02F 00000000 SECT6  notype       Label        | $LN3
030 00000000 SECTA  notype       Label        | $LN3
031 00000000 SECTC  notype       Static       | .xdata
    Section length   10, #relocs    0, #linenums    0, checksum E3EC660A, selection    5 (pick associative Section 0x6)
    Relocation CRC 00000000
034 00000000 SECTC  notype       Static       | $unwind$?func@@YAXXZ
035 00000000 SECTD  notype       Static       | .pdata
    Section length    C, #relocs    3, #linenums    0, checksum  B42549E, selection    5 (pick associative Section 0x6)
    Relocation CRC 51E35C8D
038 00000000 SECTD  notype       Static       | $pdata$?func@@YAXXZ
039 00000000 SECTE  notype       Static       | .xdata
    Section length   10, #relocs    0, #linenums    0, checksum E3EC660A, selection    5 (pick associative Section 0xA)
    Relocation CRC 00000000
03C 00000000 SECTE  notype       Static       | $unwind$main
03D 00000000 SECTF  notype       Static       | .pdata
    Section length    C, #relocs    3, #linenums    0, checksum 140D4FB5, selection    5 (pick associative Section 0xA)
    Relocation CRC 70AAD6AF
040 00000000 SECTF  notype       Static       | $pdata$main
041 00000010 SECT3  notype       Static       | ?staticVar@@3HA (int staticVar)
042 00000000 SECT10 notype       Static       | .rtc$IMZ
    Section length    8, #relocs    1, #linenums    0, checksum        0, selection    2 (pick any)
    Relocation CRC BB70974B
045 00000000 SECT10 notype       Static       | _RTC_InitBase.rtc$IMZ
046 00000000 SECT11 notype       Static       | .rtc$TMZ
    Section length    8, #relocs    1, #linenums    0, checksum        0, selection    2 (pick any)
    Relocation CRC AACEFC19
049 00000000 SECT11 notype       Static       | _RTC_Shutdown.rtc$TMZ
04A 00000000 SECT12 notype       Static       | .debug$T
    Section length   64, #relocs    0, #linenums    0, checksum        0
    Relocation CRC 00000000
04D 00000000 SECT13 notype       Static       | .chks64
    Section length   98, #relocs    0, #linenums    0, checksum        0
    Relocation CRC 00000000
```

중간의 Static, External, Label 는 설명이 없어도 각각 어떤 의미인지 알 수 있을거라 생각한다

링크에러를 뱉은 func1 을 지우고 컴파일을 했을때

최종 컴파일 된 코드의 main 함수 disasm 이다

```jsx
main:
  00000001400117C0: 40 55              push        rbp
  00000001400117C2: 57                 push        rdi
  00000001400117C3: 48 81 EC E8 00 00  sub         rsp,0E8h
                    00
  00000001400117CA: 48 8D 6C 24 20     lea         rbp,[rsp+20h]
  00000001400117CF: 48 8D 0D 2A F8 00  lea         rcx,[140021000h]
                    00
  00000001400117D6: E8 7C FB FF FF     call        @ILT+850(__CheckForDebuggerJustMyCode)
  00000001400117DB: 8B 05 2F A8 00 00  mov         eax,dword ptr [14001C010h]
  00000001400117E1: FF C0              inc         eax
  00000001400117E3: 89 05 27 A8 00 00  mov         dword ptr [14001C010h],eax
  00000001400117E9: 33 C0              xor         eax,eax
  00000001400117EB: 48 8D A5 C8 00 00  lea         rsp,[rbp+0C8h]
                    00
  00000001400117F2: 5F                 pop         rdi
  00000001400117F3: 5D                 pop         rbp
  00000001400117F4: C3                 ret
```

참조가 필요한 변수들의 주소를 절대 참조하여 재배치하였다.

notepad의 import 파일을 한번 보자.

```jsx
dumpbin /imports C:\Windows\System32\notepad.exe
Microsoft (R) COFF/PE Dumper Version 14.39.33522.0
Copyright (C) Microsoft Corporation.  All rights reserved.

Dump of file C:\Windows\System32\notepad.exe

File Type: EXECUTABLE IMAGE

  Section contains the following imports:

    KERNEL32.dll
             1400268B8 Import Address Table
             14002D3E0 Import Name Table
                     0 time date stamp
                     0 Index of first forwarder reference

                         2B8 GetProcAddress
                          DC CreateMutexExW
                           1 AcquireSRWLockShared
                         114 DeleteCriticalSection
                   
           

    GDI32.dll
             140026800 Import Address Table
             14002D328 Import Name Table
                     0 time date stamp
                     0 Index of first forwarder reference

    USER32.dll
             140026B50 Import Address Table
             14002D678 Import Name Table
                     0 time date stamp
                     0 Index of first forwarder reference

    api-ms-win-crt-string-l1-1-0.dll
             140027088 Import Address Table
             14002DBB0 Import Name Table
                     0 time date stamp
                     0 Index of first forwarder reference

                          83 memset
                          A9 wcsnlen
                          9E wcscmp

    api-ms-win-crt-runtime-l1-1-0.dll
             140027060 Import Address Table
             14002DB88 Import Name Table
                     0 time date stamp
                     0 Index of first forwarder reference

                          15 _c_exit
                          3D _register_thread_local_exe_atexit_callback
                          37 _initterm_e
                          36 _initterm

    api-ms-win-crt-private-l1-1-0.dll
             140026F40 Import Address Table
             14002DA68 Import Name Table
                     0 time date stamp
                     0 Index of first forwarder reference

                          91 _o__callnewh
                          93 _o__cexit
                          9F _o__configthreadlocale
                  

    api-ms-win-core-com-l1-1-0.dll
             140026D80 Import Address Table
             14002D8A8 Import Name Table
                     0 time date stamp
                     0 Index of first forwarder reference

                          49 CoWaitForMultipleHandles
                          46 CoUninitialize
                

    api-ms-win-core-shlwapi-legacy-l1-1-0.dll
             140026E68 Import Address Table
             14002D990 Import Name Table
                     0 time date stamp
                     0 Index of first forwarder reference

....
  Summary

        3000 .data
        1000 .didat
        2000 .pdata
        A000 .rdata
        1000 .reloc
        1000 .rsrc
       25000 .text

```

대충 이런느낌이다.

- csapp 7장 링커
    - [[CS:APP 13-1] Linking 1 - 어떻게 링킹 에러를 피할까](https://blog.kurcreative.com/kur1701091416)
    - [[CS:APP 13-2] Linking 2 - 정적 링킹lib과 동적 링킹dll](https://blog.kurcreative.com/kur1701091636)