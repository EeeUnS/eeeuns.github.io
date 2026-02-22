---
layout: post
title: "TLB, Paging"
date:   26-02-22 19:26:01 +0900
tag: cs
---

첫 글 : 2023-04-25, 글을 전반적으로 다듬고 후반부 TLB 플러시 내용 수정 

page table 은 nonpaged pool인가 swap out되나라는 멍청한 의문에서 나온 TLB, Paging 정리

스펙터와 멜트다운을 맞이하면서 현대의 CPU는 훨씬 복잡한 구조로 되어있고, 현대의 CPU를 기준으로 한번 파헤쳐 보고자 한다.
주로 intel, Windows Internals 관련 하여 참고하였다.

# 가상 주소 -> 물리주소 변환

가장 먼저 프로세스의 가상 주소를 물리 주소로 변환하는 과정을 알아보자.

전제 : Physical Address Extension, PAE(32비트 아키텍처에서 4GB이상 메모리를 사용할 수 있게 하는 기능)를 적용X


x64 64비트 가상 메모리는 실제로 물리주소로 매핑할때 48bit만 사용한다. 64비트 운영체제에서 메모리 변환 방식은 현재 PML4 방식으로 사용하고 있다.

**가상주소의 각각의 비트 영역을 쪼개서 각 테이블의 인덱스로 사용한다.**

메모리 매핑은 다음 순서대로 이루어 진다.

1. PML4 (Page Map Level 4)
2. PDPT (Page Directory Pointer Table)
3. Page Directory
4. Page Table
5. Physical Memory

**테이블은 모두 프로세스 개별**로 가지고 있다.

그리고 각각의 인덱스에 대응되는 Entry E를 붙인다

1. PML4 <> PML4E (9bit / (x32)2bit)
2. PDPT <> PDPTE(PDPE) (9bit)
3. Page Directory <> PDE (9bit)
4. Page Table  <> PTE (12bit)
5. Physical Memory

보면 눈치 챌 수도있겠지만 table 부터 PML 1이라고 생각하면 순서대로 PML4가 되었다. (PDPT까지 이름을 따로 붙였다 이제 더 이름 붙이기 애매했나보다.)

32bit 시절에 물리주소로 변환할땐  PML4 이 존재하지 않았다.

PDPT의 테이블이 작은 상태로(전체 크기가 32비트인 한계로 비트할당이 적어서) 존재하는것을 제외하면 크게 다를바가 없다.

![Untitled](/images/tlbpaging/Untitled.png)

> 혹시 사용하는 가상주소의 크기가 커진다면 다음 Table의 이름은 PML5가 되나?
> 그렇다.

다시 돌아와서 아래는 64비트의 전체 변환과정이다. 

각각 intel manual 과 windows intenals 의 그림을 떼왔다.

![Untitled](/images/tlbpaging/Untitled%201.png)

![Untitled](/images/tlbpaging/Untitled%202.png)

물리메모리 index(12bit)를 제외한 각각의 테이블 offset은 9비트인걸 알 수 있는데.

마지막 물리메모리 frame의 단위는 4KB($$2^{12}$$)다. → 따라서 Page Table에서 실제 mapping 하기위해 12비트가 필요하다.

**Entry의 크기는 8byte** (64bit, $$2^3$$byte)이다. 사실상 포인터 타입이고 이게 512($$2^9$$)개 씩 가지고 있는거니 이 테이블 전체의 크기는 총 $$2^{12}$$로 4KB가된다.

```cpp
ULONGLONG* PML4[512];
```

여기에 인덱스로 사용하기 위해서 $$2^9$$를 나타낼 수 있게 9bit만큼 할당이 된것.
이러면 각각의 테이블을 하나의 page단위(4KB)로 관리할 수 있으니 이런 이유로 9비트만큼 할당했지않나싶다.

PML5도 생기면 9bit만큼 먹겠지.

각 Entry는 **다음 table의 물리 주소(Page Frame Number / PFN)** 를 가르킨다.

page frame number database라는것도 있는데 잘 모르겠다;;

![Untitled](/images/tlbpaging/Untitled%203.png)

대충 이렇게 Entry 구조가 짜여있다는데 별로 더 알고싶진않다.


```cpp
typedef pointer ULONGLONG

ptr GetPhysicalAddress(ptr* virtualAddress)
{
	register ptr CR3;
	ptr offset = 0b1'1111'1111; // 9bit on
	ptr** pml4 = (ptr * *)(CR3 & ~(4096 - 1)); // pointer* PML4[512];  4096 2^12

	ULONGLONG pysicalIndex = ((ptr)virtualAddress & (4096 - 1));
	ULONGLONG PageIndex = ((ptr)virtualAddress & (offset << 12));
	ULONGLONG PageDirectoryIndex = ((ptr)virtualAddress & (offset << 21));
	ULONGLONG PDPTIndex = ((ptr)virtualAddress & (offset << 30));
	ULONGLONG PML4Index = ((ptr)virtualAddress & (offset << 39));

	ptr* PML4E = pml4[PML4Index];
	ptr** PDPT = (ptr**)PML4E;
		
	ptr* PDPTE = PDPT[PDPTIndex];
	ptr** PageDirectory = (ptr**)PDPTE;

	ptr* PDE = PageDirectory[PageDirectoryIndex];
	ptr** PageTable = (ptr**)PDE;

	ptr* PTE = PageTable[PageIndex];
	ptr* pysicalFrame = GetFrame(PTE);

	return pysicalFrame[pysicalIndex];
}
```

- PML4는 512개의 PDPT 주소를 가지고 있고 
- PDPT는 512개의 Page Directory를 가르키고…
- Page Directory가 최종적으로 Physical Memory의 주소를 가르키는 방식이다.

PML4를 제외한 나머지 테이블은 알아서 주소를 찾게되니 최종적으로 PML4테이블을 가르키는 주소만 추가적으로 가지고 있으면 된다.

이 주소는 어디서 얻을 수 있을까. 

각 프로세스마다 관리하는 KPROCESS structure에 이 테이블을 가르키는 물리 주소를 가지고있다.

위 사진에 있지만 CR3 레지스터가 현재 돌아가는 스레드의 테이블 주소(PML4)를 들고있는다.

# Control Register

![Untitled](/images/tlbpaging/Untitled%205.png)

X86은 CR 이라는 레지스터가 존재하는데 컨트롤 레지스터라 부른다.

CR3 레지스터에 대해 좀더 살펴보면 (PCID가 1로 세팅된 케이스만 생각해보자)

![Untitled](/images/tlbpaging/Untitled%206.png)

- PCID(Process-Context Identifiers) : Process 간의 고유한 id 12비트 식별자(0 ~ 11bit 사용)
- CR4(17번째 비트) PCIDE flag가 세팅되어있으면 CR3 11:0 에 할당된다.
- M:12 상위 비트는 PML4 테이블(PAGE MAP LEVEL 4) 이 된다. 

CR3 레지스터는 Paging에서 굉장히 중요하게 다뤄지는 레지스터다.
PWT, PCD 는 모르겠으니 넘어가자

<!-- ![Untitled](/images/tlbpaging/Untitled%207.png)

CR3를 포함한 전체 Entry 구조 

한 번 구경만 하고 넘어가자. -->

흠 그렇군.

# TLB (Translation Lookaside Buffer)

위의 주소 변환과정을 보면 무려 메모리 접근을 4번이나한다. memory indirection을 두번만해도 거품 무는 low level 개발자들이 네번이나 하는걸 알면 기절할 얘기다.

그래서 이를 해결하기위해 TLB가 등장했다. CPU 내부에 있는 물리적인 캐시 메모리라서 굉장히 빠르다.

룩업의 룩업의 룩업의 룩업을 한 구조를 룩업한게 TLB다.

![Untitled](/images/tlbpaging/Untitled%208.png)

메모리에 존재도 하지않는 MMU에서 직접 Physical address를 들고 연결해준다.

4단 탐색을 하기 이전에 TLB에서 먼저 가상 주소에 대한 물리 주소가 있는지 확인한다.
있으면 바로 직접 접근하고 없으면, 4단 탐색을 한다.

이로인해 가상메모리의 접근 성능을 극단적으로 끌어올렸다.


# 전체그림

![Untitled](/images/tlbpaging/Untitled%209.png)

캐시 메모리를 추가해보자.

1. 가상 주소가 TLB에 있는지 확인한다.
2. TLB에 있으면 바로 물리 주소로 접근한다.
3. TLB에 없으면, 4단 탐색을 하여 물리 주소를 얻는다.
   - TLB에 캐싱한다.
4. 캐시 메모리에 해당 물리 주소가 있는지 접근한다.
5. 캐시 메모리에 있으면 바로 접근한다.
6. 캐시 메모리에 없으면, 물리 메모리에 접근한다.
   1. 캐시 메모리에 캐싱한다.


만약?

- swap out되어 보조기억장치에 저장되어있다면?
- page fault가 발생한다면? 
-> Page Table단에서 valid bit로 관리한다.


**Paging 방식으로 다른 프로세스간 같은 가상 주소를 어떻게 다른 물리메모리로 매핑하는가???**

답은 위에 있다 CR3가 가지고 있는 PML4 는 해당 프로세스마다 있는 테이블이다,
레지스터니 현재 실행 중인 스레드의 테이블을 들고 있을 것이다. context switching 할 때마다 CR3 레지스터의 값이 바뀌면서 다른 테이블을 가르키게 된다.

그럼 TLB에서는??


# context switching 시 TLB

context switching 할 때 TLB는 어떻게 처리해야하는가?

context switching이 발생하고도, TLB를 그대로 냅둔다면 같은 가상 주소공간을 접근한다면 다른 프로세스 상의 물리메모리를 접근할거다.

1. 첫번째로 할 수 있는건, TLB flush 하는거다 valid 를 0으로 민다.

너무 비효율적이다.

context switching을 할때마다 TLB가 다 날린다니 전환마다 메모리 접근을 네번씩이나 하라고?

이건 못 참지하고 다른 수를 써서 flush를 모두하지않고 프로세스간 존재할 수 있게 효율적으로 만들었다 생각했더니

그렇게 멜트다운 버그라는 대형사고를 치고 첫번째로 회귀했다.

2. 두번째로는 TLB에서 CR3의 PCID 값을 연관지어 처리하는 것이다.

이를 해결하기 위해 PCID(Process-Context Identifiers)라는 것을 도입했다. (intel 에선 해당 이슈 이전에 도입되었으나, OS 단에서 지원이 늦어 지원되지 않고있다가 멜트다운 스펙터 버그 이후에 관련 개선과 함께 지원하기 시작했다.)

TLB, Paging 작동 방식은 멜트다운, 스펙터 버그가 터지기 이전과 이후로 나뉜다.

# Process-Context Identifiers (PCIDs)

Cpu단에서 지원이 되어야하고, 일단 이게 세팅이되는 cpu라면 TLB와 paging structure caches는 모두 PCID와 연관되어 작동한다.
기존 TLB와 PCID가 맞물려 작동하지않는 cpu에서는 switching시마다 TLB flush가 일어나지만(아래 기술할 KLA Shadowing 적용된 상황에서는 커널 -> 유저모드 전환시에도 kernal space flush 발생), PCID가 지원되는 cpu에서는 TLB flush가 일어나지않는다.

# KVA shadowing

TLB를 더 들어가기 이전에 KVA shadowing이라는 것을 먼저 다루어야한다.

KVA shadowing 은 멜트다운 스펙터를 맞은 윈도우즈가 낸 보완책이다. 리눅스에 도입된 KPTI(Kernel Page-Table Isolation)의 윈도우판이라고 보면된다.

![Kernel_page-table_isolation](/images/tlbpaging/Kernel_page-table_isolation.svg.png)

주요 골자는 다음과 같다. page table자체를 user mode와 kernel mode일때로 분리해서 관리한다.

- 커널 페이지 테이블 
- 유저 페이지 테이블 (섀도우 테이블 이라고도 함)
  - 커널영역으로는 페이지 테이블 전환, 커널스택, 인터럽트 처리, 시스템 콜, 트랩 등 최소한만 존재 (translation page라고도 한다)


# TLB 플러시 작동

KVA Shadowing 지원을 위해, PCID 값을 커널용, 유저용 두개를 사용하여 분리한다.

- 커널페이지 PCID 2 지정
- 유저페이지 PCID 1 지정

그러니 추가로 mode switching이 일어날 때마다 CR3 레지스터의 PCID 값도 변경된다.
그러니 각 환경별로 접근하는 페이지에 대해 구분이 가능해졌다.

![Untitled](/images/tlbpaging/Untitled%2010.png)

context switching시에는 오래된 kernel TLB entry들은 알아서 날리도록 pcid를 이용하여 user page table 를 리로드한다.

TLB는 기본적으로 PCIDs에 의존하여 접근하게 한다.

# page table 은 swap out되는가?

nonpaged pool인가? nonpage pool이다.

---

참고

주요 참고

- Intel Corporation (2016). "4.10.1 Process-Context Identifiers (PCIDs)". *[Intel 64 and IA-32 Architectures Software Developer's Manual](https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-vol-3a-part-1-manual.pdf)* (PDF). Vol. 3A: System Programming Guide, Part 1.
- Windows Internals part 1 7th
- Windows Internals part 2 7th
- [KVA Shadow: Mitigating Meltdown on Windows
](https://www.microsoft.com/en-us/msrc/blog/2018/03/kva-shadow-mitigating-meltdown-on-windows)


---
- [임베디드 레시피](http://recipes.egloos.com/5232056)
- [https://en.wikipedia.org/wiki/Control_register](https://en.wikipedia.org/wiki/Control_register)
- [https://42osstudy.github.io/os-study/jekyll/2022-08-05-ch21.html](https://42osstudy.github.io/os-study/jekyll/2022-08-05-ch21.html)
- operating system concepts 10th
- [https://en.wikipedia.org/wiki/Translation_lookaside_buffer](https://en.wikipedia.org/wiki/Translation_lookaside_buffer)
- [https://shhoya.github.io/hv_paging.html#--4-level-paging-and-5-level-paging](https://shhoya.github.io/hv_paging.html#--4-level-paging-and-5-level-paging)
- [https://techcommunity.microsoft.com/t5/windows-blog-archive/pushing-the-limits-of-indows-paged-and-nonpaged-pool/ba-p/723789](https://techcommunity.microsoft.com/t5/windows-blog-archive/pushing-the-limits-of-windows-paged-and-nonpaged-pool/ba-p/723789)
- [https://connormcgarr.github.io/paging/](https://connormcgarr.github.io/paging/)
- [https://en.wikipedia.org/wiki/Intel_5-level_paging](https://en.wikipedia.org/wiki/Intel_5-level_paging)
- [https://www.kernel.org/doc/Documentation/x86/pti.txt](https://www.kernel.org/doc/Documentation/x86/pti.txt)
- [https://www.felixcloutier.com/x86/invpcid](https://www.felixcloutier.com/x86/invpcid)
- [https://stackoverflow.com/questions/72291786/sharing-a-tlb-entry-between-two-logical-cpus-intel](https://stackoverflow.com/questions/72291786/sharing-a-tlb-entry-between-two-logical-cpus-intel)
- [https://www.felixcloutier.com/x86/invlpg](https://www.felixcloutier.com/x86/invlpg)
- [https://learn.microsoft.com/ko-kr/cpp/intrinsics/invlpg?view=msvc-170](https://learn.microsoft.com/ko-kr/cpp/intrinsics/invlpg?view=msvc-170)
- [https://sata.kr/entry/보안-Issue-Meltdown멜트다운-취약점을-파헤쳐보자-1](https://sata.kr/entry/%EB%B3%B4%EC%95%88-Issue-Meltdown%EB%A9%9C%ED%8A%B8%EB%8B%A4%EC%9A%B4-%EC%B7%A8%EC%95%BD%EC%A0%90%EC%9D%84-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-1)