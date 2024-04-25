---
layout: post
title: "TLB, Paging"
date:   2023-04-25 19:26:01 +0900
tag: cs
---

page table 은 nonpaged pool인가 swap out되나라는 멍청한 의문에서 나온 TLB, Paging 정리
책에서도 많이 다루는 TLB 작동방식을 통해 가상 메모리를 어떻게 물리 메모리로 바꾸는 과정들은 이미 많은 글들이 있다.

# 가상 주소 -> 물리주소 변환

전제 : Physical Address Extension, PAE를 적용X

x64 64비트 가상 메모리는 실제로 물리주소로 매핑할때 48bit만 사용한다.

64비트 운영체제에서 메모리 변환 방식은 PML4 방식으로 사용하고 있다.

**가상주소의 각각의 비트 영역을 쪼개서 각 테이블의 인덱스로 사용한다.**

메모리 매핑은 다음 순서대로 이루어 진다.

1. PML4 (Page Map Level 4)
2. PDPT (Page Directory Pointer Table)
3. Page Directory
4. Page Table
5. Physical Memory

5를 제외한 나머지는 모두 프로세스 개별로 가지고 있다.

그리고 각각의 인덱스에 대응되는 Entry 에대해서 E를 붙인다

1.  PML4 / PML4E
2. PDPT / PDPTE(PDPE)
3. Page Directory / PDE 
4. Page Table  / PTE
5. Physical Memory

보면 눈치 챌 수도있겠지만 table 부터 PML 1이라고 생각하면 순서대로 PML4가 되었다. (PDPT까지 붙였다 이제 더 이름 붙이기 애매했나보다.)

32bit 시절에 물리주소로 변환할땐  PML4 이 존재하지 않았다.

PDPT의 테이블이 작은 상태로(전체 크기가 32비트인 한계로 비트할당이 적어서) 존재하는것을 제외하면 크게 다를바가 없다.

![Untitled](/images/tlbpaging/Untitled.png)

혹시 사용하는 가상주소의 크기가 커진다면 다음 Table의 이름은 PML5가 되나?

그렇다.

다시 돌아와서

64비트의 전체 변환과정이다. 

각각 intel manual 과 windows intenals 의 그림을 떼왔다.

![Untitled](/images/tlbpaging/Untitled%201.png)

![Untitled](/images/tlbpaging/Untitled%202.png)

물리메모리 index(12bit)를 제외한 각각의 테이블 offset은 9비트인걸 알 수 있는데.

물리메모리 frame의 단위는 4KB(2^12)다. → 따라서 mapping 하기위해 12비트가 필요하다.

**Entry의 크기는 8byte** (64bit, 2^3byte)이다. 512(2^9)개 만큼 씩하면 이 테이블 전체의 크기는 총 2^12로 4kb가된다.

```cpp
ULONGLONG* PML4[512];
```
여기에 인덱스로 사용하기 위해서 2^9를 나타낼 수 있게 9bit만큼 할당이 된것.
이러면 각각의 테이블을 하나의 page단위(4KB)로 관리할 수 있으니 이런 이유로 9비트만큼 할당했지않나싶다.

PML5도 생기면 9bit만큼 먹겠지.

각 Entry는 **다음 table의 물리 주소(Page Frame Number / PFN)**를 가르킨다.

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
- 이게 최종적으로 Physical Memory의 주소를 가르키는 방식이다.

PML4를 제외한 나머지 테이블은 알아서 주소를 찾게되니 PML4테이블을 가르키는 주소만 추가적으로 가지고 있으면 된다.

이 주소는 어디서 알 수 있을까.

각 프로세스마다 관리하는 KPROCESS structure에 이 테이블을 가르키는 물리 주소를 가지고있다.

그리고 CR3 레지스터가 현재 돌아가는 스레드의 테이블 주소를 들고있는다.

# Control Register

![Untitled](/images/tlbpaging/Untitled%204.png)

![Untitled](/images/tlbpaging/Untitled%205.png)

X86은 CR 이라는 레지스터가 존재하는데 컨트롤 레지스터라 부른다.

CR3 레지스터에 대해 좀더 살펴보면

![Untitled](/images/tlbpaging/Untitled%206.png)

PCID(Process-Context Identifiers) 는 12비트 식별자고 CR4(17번째 비트) PCIDE flag가 세팅되어있으면 CR3 11:0 에 할당된다. Process 간의 고유한 id가 이 레지스터에서 들고있는다.

CR3 레지스터는 Paging에서 굉장히 중요하게 다뤄지는 레지스터다.

항상 64:M 상위 비트는 PML4 테이블( PAGE MAP LEVEL 4**)** 이 된다. 

PWT, PCD 는 모르겠으니 넘어가자

![Untitled](/images/tlbpaging/Untitled%207.png)

CR3를 포함한 전체 Entry 구조 

한 번 구경만 하고 넘어가자.

흠 그렇군.

# TLB

위의 주소 변환과정을 보면 무려 메모리 접근을 4번이나한다. memory indirection을 두번만해도 거품 무는 low level개발자들이 네번이나 하는걸 알면 기절할 얘기다.

그래서 이를 해결하기위해 TLB가 등장했다.

룩업의 룩업의 룩업의 룩업을 한 구조를 룩업한게 TLB다.

![Untitled](/images/tlbpaging/Untitled%208.png)

메모리에 존재도 하지않는 MMU에서 직접 Physical address 를 들고 연결해준다.

이로인해 가상메모리의 접근 성능을 극단적으로 끌어올렸다.

# 전체그림

![Untitled](/images/tlbpaging/Untitled%209.png)

**Paging 방식으로 다른 프로세스간 같은 가상 주소를 어떻게 다른 물리메모리로 매핑하는가???**

답은 위에 있다 CR3가 가지고 있는 물리 주소는 해당 프로세스에 고유한 테이블이라고.

그럼 TLB에서는??

# 미스 처리

TLB 에서 미스가 나면 page table을 자동으로 탐색한다.

라고만 되어있겠지만 이 탐색 과정을 좀 더 자세히 살펴보자.

# switcing 시의 TLB

context switching할때 TLB는 어떻게 처리해야하는가?

TLB를 냅둔다면 같은 가상 주소공간을 접근한다면 다른 프로세스의 물리메모리를 접근할거다.

첫번째로 할 수 있는건 TLB flush 하는거다 valid 를 0으로 민다.

너무 비효율적이다.

context switching을 할때마다 TLB가 다 날라간다니 전환마다 메모리 접근을 네번씩이나 하라고?

이건 못 참지하고 다른 수를 써서 flush를 모두하지않고 프로세스간 존재할 수 있게 효율적으로 만들었다 생각했더니

그렇게 멜트다운 버그라는 대형사고를 치고 첫번째로 회귀했다.

두번째로는 TLB에서 CR3의 PCID 값을 연관지어 처리하는 것이다.

TLB내부에서 구분지어서 사용할 수 있다.

# Process-Context Identifiers (PCIDs)

일단 이게 세팅이되는 cpu라면 TLB와 paging structure caches는 모두 PCID와 연관되어 작동한다.

# TLB 상세

TLB의 구조를 다시 알아보자. 

사실 첨부된 그림에 있는거보다 굉장히 자세하게 되어있는데 intel manual에 대부분의 얘기가 나와있다.

다음은 4.10 의 내용을 정리 한 것이다.

- 페이지 프레임
- paging structure entry(페이징에 사용한 구조 엔트리 PML4만이아닌 언급된 많은 종류들)의 접근 권한
- PTE 의 속성
    - dirty flag(위의 entry 구조를보면 6번째 bit의 D가 dirty flag표시이다.)
    - memory type

- The physical address corresponding to the page number (the page frame).
- The access rights from the paging-structure entries used to translate linear addresses with the page number (see Section 4.6):
— The logical-AND of the R/W flags.
— The logical-AND of the U/S flags.
— The logical-OR of the XD flags (necessary only if IA32_EFER.NXE = 1).
— The protection key (necessary only with IA-32e paging and CR4.PKE = 1).
- Attributes from a paging-structure entry that identifies the final page frame for the page number (either a PTE or a paging-structure entry in which the PS flag is 1):
— The dirty flag (see Section 4.8).
— The memory type (see Section 4.9)

TLB는 기본적으로 PCIDs에 의존한다.

**Global Pages**가 활성화되면 TLB는 현재 PCIDs와 다른 TLB entry도 들고있는다.

# Invalidation of TLBs and Paging-Structure Caches

TLB 무효화 정책은 OS개발자의 담당인듯하다.

메뉴얼에서는 여러가지 지우는 방식에 대한 설명이 나열되어있다.

- INVLPG : 한마디로  TLB를 통째로 flush 시킨다.
- INVPCID
- MOV to CR0
- MOV to CR3.
- Task switch : CR3 값 변경

그리고 지울때 일반적으로 Global page에대한 entry는 남겨놓는다.(물론 얘네 까지 지우는 방식도 존재)

# TLB flushing algorithm

KVA shadowing 이 꺼져있는 일반적일때

- kernel page 는 non global
- user page는 global

PCID 사용 :  이때부터는 global  non global이 의미가 없어진다.

pcid를 이용하여 user page table 를 리로드하고  kernel TLB entry들은 알아서 날린다.

whereas on systems with PCID support, the user-page tables are reloaded using the
User PCID, which automatically invalidates all the stale kernel TLB entries.

![Untitled](/images/tlbpaging/Untitled%2010.png)

KVA shadowing 은 멜트다운 스펙터를 쳐맞은 윈도우즈가 낸 보완책이다. KPTI의 윈도우판이라고 보면된다.

# page table 은 swap out되는가?

nonpaged pool인가?

nonpage pool 이라고 할 수도있는데 애초에 page table은 memory pool영역이 아니라 nonpage/page 그 어느 영역에도 속하지 않는다고 봐야한다  

- Intel Corporation (2016). "4.10.1 Process-Context Identifiers (PCIDs)". *[Intel 64 and IA-32 Architectures Software Developer's Manual](https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-vol-3a-part-1-manual.pdf)* (PDF). Vol. 3A: System Programming Guide, Part 1.
[https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-vol-3a-part-1-manual.pdf](https://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-vol-3a-part-1-manual.pdf)
- Windows Internals part 1 7th
- Windows Internals part 2 7th
- [https://en.wikipedia.org/wiki/Control_register](https://en.wikipedia.org/wiki/Control_register)
- [https://42osstudy.github.io/os-study/jekyll/2022-08-05-ch21.html](https://42osstudy.github.io/os-study/jekyll/2022-08-05-ch21.html)
- operating system concepts 10th
- [https://en.wikipedia.org/wiki/Translation_lookaside_buffer](https://en.wikipedia.org/wiki/Translation_lookaside_buffer)
- [https://shhoya.github.io/hv_paging.html#--4-level-paging-and-5-level-paging](https://shhoya.github.io/hv_paging.html#--4-level-paging-and-5-level-paging)
- [http://recipes.egloos.com/5232056](http://recipes.egloos.com/5232056)
- [https://techcommunity.microsoft.com/t5/windows-blog-archive/pushing-the-limits-of-indows-paged-and-nonpaged-pool/ba-p/723789](https://techcommunity.microsoft.com/t5/windows-blog-archive/pushing-the-limits-of-windows-paged-and-nonpaged-pool/ba-p/723789)
- [https://connormcgarr.github.io/paging/](https://connormcgarr.github.io/paging/)
- [https://en.wikipedia.org/wiki/Intel_5-level_paging](https://en.wikipedia.org/wiki/Intel_5-level_paging)
- [https://www.kernel.org/doc/Documentation/x86/pti.txt](https://www.kernel.org/doc/Documentation/x86/pti.txt)
- [https://www.felixcloutier.com/x86/invpcid](https://www.felixcloutier.com/x86/invpcid)
- [https://stackoverflow.com/questions/72291786/sharing-a-tlb-entry-between-two-logical-cpus-intel](https://stackoverflow.com/questions/72291786/sharing-a-tlb-entry-between-two-logical-cpus-intel)
- [https://www.felixcloutier.com/x86/invlpg](https://www.felixcloutier.com/x86/invlpg)
- [https://learn.microsoft.com/ko-kr/cpp/intrinsics/invlpg?view=msvc-170](https://learn.microsoft.com/ko-kr/cpp/intrinsics/invlpg?view=msvc-170)
- [https://sata.kr/entry/보안-Issue-Meltdown멜트다운-취약점을-파헤쳐보자-1](https://sata.kr/entry/%EB%B3%B4%EC%95%88-Issue-Meltdown%EB%A9%9C%ED%8A%B8%EB%8B%A4%EC%9A%B4-%EC%B7%A8%EC%95%BD%EC%A0%90%EC%9D%84-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-1)
- [https://msrc.microsoft.com/blog/2018/03/kva-shadow-mitigating-meltdown-on-windows/](https://msrc.microsoft.com/blog/2018/03/kva-shadow-mitigating-meltdown-on-windows/)