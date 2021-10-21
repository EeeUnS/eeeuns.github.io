---
layout: post
title:  "Dreamhack : Reverse Engineering "
date:   2020-08-31 19:26:01 +0900
tag: arrangement
---

# 1

리버스 엔지니어링(Reverse Engineering) : 이미 만들어진 시스템이나 장치에 대한 해체나 분석을 거쳐 그 대상 물체의 구조와 기능, 디자인 등을 알아내는 일련의 과정 == 역공학


Static Analysis : 프로그램을 실행시키지 않고 분석

Dynamic Analysis :  프로그램을 실행시켜서 입출력과 내부 동작 단계를 살피며 분석

Disassemble : 바이너리 코드를 어셈블리 코드로 변환하는 과정

# 2

CPU : 다음 실행할 명령어를 읽어오고(Fetch) → 읽어온 명령어를 해석한 다음(Decode) → 해석한 결과를 실행하는(Execute)

Instruction Cycle : 기계 코드가 실행되는 한 번의 과정

레지스터(Register) : CPU의 동작에 필수적인 저장 공간의 역할을 하는 CPU의 구성 요소


## 범용 레지스터(General-Purpose Registers, GPR) 

용도를 특별히 정해두지 않고 다양하게 쓸 수 있는 레지스터

- rax : 함수가 실행된 후 리턴값을 저장 

- rcx, rdx, r8, r9 : Windows 64bit에서 함수를 호출할 때 필요한 인자들을 순서대로 저장

- rsp : 스택 포인터(Stack Pointer)로, 스택의 가장 위쪽 주소

- rip : 다음에 실행될 명령어가 위치한 주소

Data Size

x 16bit
e~ 32bit
r~ 64bit

## FLAGS

상태 레지스터인
현재 상태나 조건을 0과 1로 나타내는 레지스터

CF(Carry Flag) : 자리 올림(carry)이 생기는 경우 CF의 값이 1  연산에 사용된 값들에 부호가 없다(unsigned)
ZF(Zero Flag) : 연산의 결과가 0일 때 ZF는 1

SF(Sign Flag) : 결과가 양수일 때, 즉 최상위 비트가 0이면 SF=0, 반대로 결과가 음수가 되어 최상위 비트가 1이면 SF=1

OF(Overflow Flag) : 연산에 사용된 값들에 부호가 있을 경우에는 CF 대신 OF를 사용

## Instruction Format

기계 코드(Machine Code) 또는 명령 코드(Opcode) : 컴파일러가 만드는 결과물인 바이너리를 구성하고 있으며, CPU가 실제로 수행할 작업을 나타내는 숫자

Operand : 명령 코드가 연산할 대상을 피연산자


mov   = 
sub    -

[reg] : 레지스터에 저장된 메모리 주소를 참조한 값

byte ptr : Pointer Directive
Data Size가 실제 어셈블리 코드에서 사용된 케이스

byte : 1byte
dword : 하위 4byte


## Instructions

- mov    dst,src       ; dst = src
- lea    dst,addr      ; dst = addr
- inc    dst           ; ++dst
- dec    dst           ; --dst
- neg    dst           ; dst = -dst
- not    dst           ; dst = ~dst
- add    dst,src       ; dst += src
- sub    dst,src       ; dst -= src
- imul   dst,src       ; dst *= src
- and    dst,src       ; dst &= src
- or     dst,src       ; dst |= src
- xor    dst,src       ; dst ^= src
- shl    dst,k         ; dst << k
- shr    dst,k         ; dst >> k
                       ; logical
- sal    dst,k         ; dst << k 
- sar    dst,k         ; dst >> k
                       ; arithmetic
- test   rax,rax       ; ZF=1 if rax = 0
                        ; SF=1 if rax < 0

- cmp    rax,rdi       ; ZF=1 if rax = rdi
                     ; ZF=0 if rax!= rdi
                     ; CF=1 if rax < rdi
                     ; CF=0 if rax > rdi
- jmp    location
- je     location      ; equal (ZF=1)
- jne    location      ; not equal (ZF=0)
- jg     location      ; >   signed
- jge    location      ; >=  signed
- jl     location      ; <   signed
- jle    location      ; <=  signed
- ja     location      ; >   unsigned
- jb     location      ; <   unsigned
- js     location      ; negative (SF=1)
- jns    location      ; not negative (SF=0)


Intel x86-64 아키텍처에서 스택은 낮은 주소(=더 작은 숫자)를 향해 자라기 때문에 스택이 자랄수록 rsp에 저장된 메모리 주소는 점점 낮아집니다.

- push   rdi           ; sub  rsp,8
                     ; mov  [rsp],rdi
- pop    rdi           ; mov  rdi,[rsp]
                     ; add  rsp,8

- call   location      ; push retaddr
                     ; jmp  location
- ret                  ; pop  rip
                     ; jmp  rip

F2

소프트웨어 브레이크 포인트를 걸 때 사용하는 단축키입니다. 이미 브레이크 포인트가 걸려있는 주소에서 누를 경우 브레이크 포인트를 삭제합니다.

F7

어셈블리 코드를 한 줄 실행합니다. 만약 call을 실행하려 하면 call한 함수 내부로 진입합니다.

F8

어셈블리 코드를 한 줄 실행합니다. 만약 call을 실행하려 하면 call한 함수가 ret 명령어를 실행할 때까지(리턴할 때까지) 실행한 다음 멈춥니다.

F9

프로그램의 실행을 재개합니다.

ctrl + g

현재 창이 보여주는 주소를 바꿉니다. 디스어셈블 창에서 사용하면 디스어셈블 창이 해당 주소로 가고, 헥스덤프 창에서 사용하면 헥스덤프 창이 해당 주소로 가는 식입니다. 주소값 말고도 간단한 사칙 연산이나 함수명도 인식합니다.

-, +

이전 또는 다음 주소로 이동합니다. call 이나 jmp 명령어의 주소로 이동했을 때, 이전 주소로 돌아가거나 다시 이동할 때 자주 쓰입니다.

<enter> 키

call이나 jcc와 같은 PC(program counter)를 변경시키는 명령어를 선택한 상태에서 누르면 해당되는 주소로 이동합니다. ex) call 0x11223344 → 0x11223344로 이동

<space> 키

선택한 어셈블리어를 수정합니다. 잔존 바이트를 NOP로 채우기를 선택하면 수정된 코드 길이가 기존 코드의 길이보다 작을 때 남는 공간을 NOP으로 자동으로 채워줍니다.

