---
layout: post
title:  "안드로이드 장치 드라이버에 대한 효과적 취약점 탐지 기법"
date:   2020-05-23 21:53:01 +0900
tag: notes
---

# [안드로이드 장치 드라이버에 대한 효과적 취약점 탐지 기법](http://www.dbpia.co.kr/Journal/articleDetail?nodeId=NODE07049996)

해당 논문에서 정보를 얻을 만한 부분만 따로 추출하였음.


악의적인 공격을 사전에 방지하기 위해, US-CERT
혹은 ETSI standard별 시큐어 코딩(secure coding) 요
구조건[2,3]을 적용하거나 전통적인 취약점 탐지 기법인
모의해킹(penetration testing)을 적용하는 것은 Zero-day
attack을 포함한 새로운 공격과 관련된 보안 취약점 검
출에 한계가 있다.


정적 취약점 분석에 관한 다양한 연구 사례가 있다.
방지호의 연구[5]에서는, JAVA언어로 쓰여진 어플리케
이션에 대한 보안 취약점을 탐지하기 위해, 표 1의 공개
용 정적 분석 도구를 사용했다.

정적 분석 도구들은 JAVA 어플리케이션/ 소스코
드에 대해 다음과 같은 항목들을 검사한다.

1. 입력 데이터 검증 및 표현
2. 보안성이 취약한 암호알고리즘 사용, 평문으로 기록된 비밀번호 등
3. 시간 및 상태
4. 에러처리
5. 코드오류
6. 캡슐화
7. API오용




안드로이드 기반 모바일 시스템에 있어서 위협은, 여
러가지 요인과 연관되어 있다. 대표적인 요인은 공격표
면(attack surface)과 연관된 취약점이다. 안드로이드 시
스템에서 공격표면은 
1. 입력 값 처리
    1. 버퍼 오버플로우(buffer overflow)
    2. 정수 오버플로우(integer overflow)
    3. 경로 탐색(path traversal)
2. 자원(파일 등의 대상 객체)에 대한 접근권한(access right = permission,또는 보안수준) 처리
3. 프로세스간 통신(Inter-Process Communication, IPC) 


전통적인 리눅스 시스템에 있어서 자원에 대한 비정
상적인 접근을 방지하기 위하여 임의 접근제어(Discretionary
Access Control, DAC)모델[13]이 사용되고 있
고, 최근 시스템에는 강제 접근제어(Mandatory Access
Control, MAC)[14] 등의 진보된 접근제어 기술이 도입
되어 적용되고 있다.

공격자가 루트 권한 혹은 시스템 권한 어플리케이션
의 데이터를 접근하기 위해서, 사용자 권한의 특정 어플
리케이션에 악성 코드를 포함시키는데, 이 경우 악성 코
드의 공격 방법은 다음과 같다. 예를 들어, 카메라 디바
이스가 일반 사용자 권한으로 접근할 수 있도록 설정되
어 있고, 해당 디바이스 드라이버 코드에 버퍼 오버플로
우 취약점이 있으며, 해당 디바이스 드라이버에서 이미
지 저장 시 시스템 권한으로 파일이 작성된다. 이때 버
퍼 오버플로우는 이미지 저장 시 해당 함수가 헤더의
정보를 보고 그 크기에 따라 메모리 공간을 할당할 때
발생될 수 있는 취약점이라고 가정한다면, 공격자의 악
성 코드는 비정상적 이미지 크기 정보와 함께 버퍼보다
더 큰 이미지를 할당하여 버퍼 오버플로우를 유도한다.
버퍼 오버플로우가 유도되었을 때의 권한은 시스템권한
이고, 복귀되는 주소는 SMS를 추출하는 코드의 주소일
경우 SMS메시지를 불법으로 추출 가능하다.



|Vulnerabilities|Types of vulnerability related to privilege escalation|
|------|---|
|Buffer overflow |1, 2, 3|
|Integer error |3|
|Memory leaks | 3|
|Race condition |3|
|MMU addressing error |1, 2, 3|
|Files I/O error |3|
|Inconsistent operation |3|
|Syscall bridging |3|
|Wrong exit condition | 3|
|Pointers error | 1, 2, 3|


|Group | Selection criteria|
|---|---|
|0|Vulnerabilities found both in static and dynamic analysis, and can be used as targets for immediate attacks|
|1 |Vulnerabilities found either in static or dynamic analysis, and can be used as targets for immediate attacks|
|2|Vulnerabilities found either in static or dynamic analysis, but can’t be used as targets for immediate attacks. |But if combined with other vulnerabilities, permission can be upgraded for attacks|
|3|Vulnerabilities like in 2 Group, but permission can’t be upgraded for attacks

보안 취약점 분석은 일반적으로 코드 기반의 정적 분
석 방법, 코드 기반+런타임 기반 혹은 런타임 기반의 동
적 분석 방법으로 나누어진다(표 5 참조).

- Static
    - Code-based analysis
    - Code-based + Target binary-based analysis
    - Analysis of code that is reverse engineered
- Dynamic
    - Virtual /images/ analysis
    - Runtime analysis
    - Runtime + Reverse engineered code analysis



### 정적 분석 방법

<!--
<img src="이미지 url" width="원하는 크기">

/images//test/test-01.png
-->


<img src="/images//0523/img17.jpg" width="60%">


정적 분석 방법은 알려진 취약점 혹은 규칙에 따라서
소스 코드나 목적 코드를 분석한다. 효과적인 정적 분석
을 위하여 본 논문에서는 위협 분석을 CWE/CVE 기반
데이터로 구성하였으며, 실제 공격이 가능한 대상을 선
별하였다. US-CERT Secure Coding Guide에 따라서
분석 가능한 도구를 선정 및 개발 하였다. 도구 구성은
개발 코드의 언어별 종류, 사용하는 라이브러리, 시스템
에 따라서 다양하게 구분되며, 공격 대상이 되는 종류에
따라서 분석 도구를 선정한다. 선정된 도구를 기반으로
1차 분석하여 결과를 검토하며, 필요 시 몇몇 도구를 조
합하여 사용한다. 조합에 있어서는 분석해야 하는 코드
의 언어, 종류, 코드의 구현 방법이 다른 언어의 코드를
조합하여 사용하는 함수가 있는지, 그리고 해당 코드는
어떤 목적으로 사용되는지에 따라 다양하게 달라질 수
있으나, 본 논문에서는 ANSI표준 C 언어가 그 분석 대
상이고, C 언어로 된 코드에서 ASM코드가 포함이 되
어 있기 때문에 Flawfinder로 일반적인 C코드 취약점을
확인 하였다. 또한 AppScan이 근래 모바일 악성 코드
의 코드 구조를 모두 포함하고 있기 때문에 AppScan을
사용하였으며, Coverity의 결과와 비교 검토하였다.


### 동적 분석 방법

<img src="/images//0523/img19.jpg" width="60%">


일반적인 동적 분석 방법은 런타임에 다양한 입력값
을 발생시켜서 특정 타깃 동작을 확인하는 방법을 모두
동적 분석 방법으로 분류한다면, 본 논문의 방법은 시스
템 콜을 입력값으로 발생시키고, 비정상적인 커널 크래
쉬를 특정 타깃 동작으로 확인하는 동적 분석 방법이다.
이는, 디바이스 드라이버의 입/출력은 모두 시스템 콜을
기반으로 커널 입/출력인 것이고, 발생되는 결과는 정상
적인 디바이스 드라이버 동작 혹은 커널의 대응 동작이
수행되거나, 혹은 커널의 비정상적인 동작이 발생하는
것이 예상된다. 따라서 본 논문의 실험에서는 그 중 디
바이스드라이버 함수에서 비정상적인 오버플로우 이슈
가 발생하고, 그 영향으로 커널 크래쉬가 발생하는 것을
확인하고, 보안 취약점으로 확인한다.

본 논문에서는 런타임 기반 분석 방법 중 다양한 입
력을 생성하여 비정상 입력 조합을 구성할 수 있는
dumb fuzzing을 적용하였다. Dumb fuzzing은 복잡한
공격의 탐지에는 효과적으로 적용하기 힘든 방법이지만,
오버플로우 등 주요 취약점들에 대해서는 빠른 시간 내
에 탐지할 수 있고, fuzzer 구성이 어렵지 않다는 장점
이 있다. 그러나 동적 분석 방법은 단일 디바이스 혹은
단일 시스템의 검증에서 빠른 취약점 확인 방법으로 유
용하지만, 시스템 혹은 디바이스(HW chip)에 따라 인
터페이스 및 생성 시스템 콜 등을 모두 최적화 및 수정
해야 하는 단점이 있다. 본 논문에서는 위협 분석의 대
상으로 선정된 디바이스 드라이버에 대해 각 fuzzer를
구성하여 공격 시험하였다.
동적 분석 방법에 있어서 UI에서는 필요한 파라미터
선정 및 대상 드라이버 선정/탐색 단계를 진행하며,
Fuzzer core에서는 입력된 파라미터에 따라서 시스템
콜을 구성한다. Fuzzer driver에서는 정의된 시스템 콜
을 IOCTL로 생성하며, 생성된 시스템 콜의 응답이나
혹은 시스템 크래쉬 등을 확인하여 중요 취약 함수를
확인할 수 있다. 분석 대상 드라이버(Target driver)의
세부 호출 함수의 취약점을 확인하고 결과를 검토하였다.


-----


<img src="/images//0523/img26.jpg" width="60%"> 그림 4
<img src="/images//0523/img28.jpg" width="60%"> 그림 5

나타난 코드에서 결국 “copy_from_user”
함수가 사용되는데, 이는 시스템 권한의 커널 모듈에서
사용자 권한의 파일이나 데이터를 읽어오기 위한 함수
이며, 읽어올 때에는 시스템권한으로 읽어온다. 위 코드
에서 data와 task 스트럭쳐(structure)가 사용되는데,
task의 내용은 그림 5와 같이 정의되어 있다.


읽어온 데이터를 저장할 때에 배열의 인덱스와 데이
터를 복사하는 것은 위 그림 5 마지막 두 라인에 for
루프로 구성되어 있다. 해당 데이터는 잘못된 입력으로
인하여 예상되는 인덱스와 데이터 보다 큰 데이터의 입
력이 발생할 수 있기 때문에 버퍼 오버플로우 취약점이
된다. 분석으로 확인된 보안 취약점은 Exynos기반의 안
드로이드 기기에서 카메라 드라이버 관련 코드에 존재
하였다.


<img src="/images//0523/img26.jpg" width="60%"> 그림 6


해당 취약점에 대한 동적 분석에서는 IOCTL대상 디
바이스 드라이버 /dev/xxx만을 대상으로 하여 dumb
fuzzing을 시행하였으며, IOCTL cmd“0xC0C0????”에
서 디바이스 및 시스템 크래쉬를 확인하였다. 동적 분석
에서 전달된 ARG의 비정상적인 호출로 인하여 해당 값
이 copy_from_user 함수에서 task 스트럭쳐로 구성되
며, 해당 값으로 인하여 시스템 크래쉬가 발생 한다는
내용을 정적 분석에서 확인된 코드에서 검토 하였다. 검
토된 내용을 바탕으로 오퍼플로우가 발생되는 주소를
확인하기 위하여 특정 패턴으로 구성된 ARG를 구성하
여 호출하였으며, 정확한 오버플로우 발생 주소를 이용
하여, 이후 exploit 코드를 작성하여 PC값이 의도된
“A”의 ASCII로 조작되는지 확인하였다. 확인된 결과는
그림 6과 같다.


그림 6에서 붉은색으로 표시된 내용이 PC 값이며 의
도된 것과 같이 “A”로 변경된 것을 확인할 수 있다.
정리하면, 그림 4와 같이 정적 분석에서 확인되는 내
용이 동적 분석에서 동시에 그림 6과 같이 확인 되었다.
이는 공격자가 실제 동적 분석으로 취약점을 찾고 해당
취약점을 이용하여 시스템 권한을 시도 한다면 즉시 권
한 획득이 가능한 취약점이라는 것이 증명되는 것이다.






strncpy의 경우 입력값 크기만을 고려하면 NULL
termination 문자가누락될 수 있어 1byte의 해당 공간
이 반드시 필요하다. 그러나 많은 코드에서 위와 같이
단순히 strncpy로 변경되어 있기 때문에 잠재적 취약점
을 갖고 있다. 해당 코드의 경우 동적 분석에서는 버퍼
오버플로우 취약점이 확인되지 않고, 정적 분석에서만
확인 되었다. 검토 결과 해당 변수는 네트워크 장치의
작동이 시작되고 특정 함수가 요청되었을 때 발생하는
것으로서 본 실험에서 사용되지 않는 상황에서 동적 분
석이 시행되었기 때문에 함수가 호출되지 않았다.