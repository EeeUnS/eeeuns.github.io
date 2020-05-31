---
layout: post
title:  "DreamHack 시스템 해킹 커리큘럼 01 기초,Memory Corrunption 1,2"
date:   2020-05-30 21:53:01 +0900
categories: paper
---

# [System Exploitation Fundamental 코스](https://dreamhack.io/lecture/curriculums/2)

# 실습에대한 답이 있으므로 직접 해결하실분은 보지마세요

# System Exploitation Fundamental 

중요도

1. Reliably Exploitable Vulnerability
2. Exploitable
3. Vulnerable
4. Buggy


## Attack Vector

공격자가 소프트웨어와 상호 작용할 수 있는 곳

Attack Surface : Attack Vector의 집합


## 취약점 종류

- 메모리 커럽션 취약점
    - Buffer Overflow : 할당한 크기의 버퍼보다 더 큰 데이터를 입력받아 메모리의 다른 영역을 오염시킬 수 있는 취약점
    - Out-Of-Boundary : 버퍼의 길이 범위를 벗어나는 곳의 데이터에 접근할 수 있는 취약점
    - Off-by-one : 경계 검사에서 하나 더 많은 값을 쓸 수 있을 때 발생하는 취약점
    - Format String Bug : printf나 sprintf와 같은 함수에서 포맷 스트링 문자열을 올바르게 사용하지 못해 발생하는 취약점
    - Double Free / Use-After-Free : 동적 할당된 메모리를 정확히 관리하지 못했을 때 발생하는 취약점
    - etc
-  로지컬 버그
   - Command Injection : 사용자의 입력을 셸에 전달해 실행할 때 정확한 검사를 실행하지 않아 발생하는 취약점
   - Race Condition : 여러 스레드나 프로세스의 자원 관리를 정확히 수행하지 못해 데이터가 오염되는 취약점
   - Path Traversal : 프로그래머가 가정한 디렉토리를 벗어나 외부에 존재하는 파일에 접근할 수 있는 취약점
   - etc  

미티게이션

- Stack Smashing Protector(SSP) : 버퍼 오버플로우를 방지하기 위해 버퍼의 뒤에 랜덤한 값을 넣어두고 이를 특정 시점에 검사해 버퍼가 오염되었는지 확인하는 것

------------
# Memory Corrunption 1

------------
## buffer overflow

메모리를 덮어써서 Undefined Behavior를 일으키거나 Segmentation Fault발생

### ex 1
0x001100110011001100110011001100110000000041414141을 넣자.
그러면 buf+20뒷부분(해당 실습에선 ret)이0x41414141로 덮인다.
```Cpp
// stack-1.c
#include <stdio.h>
#include <stdlib.h>
int main(void) {
    char buf[16];
    gets(buf);
    
    printf("%s", buf);
}
```


### ex2

```Cpp
// stack-2.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int check_auth(char *password) {
    int auth = 0;
    char temp[16];
    
    strncpy(temp, password, strlen(password));
    
    if(!strcmp(temp, "SECRET_PASSWORD"))
        auth = 1;
    
    return auth;
}
int main(int argc, char *argv[]) {
    if (argc != 2) {
        printf("Usage: ./stack-1 ADMIN_PASSWORD\n");
        exit(-1);
    }
    
    if (check_auth(argv[1]))
        printf("Hello Admin!\n");
    else
        printf("Access Denied!\n");
}
```

해당 코드의 문제는 check_auth :의 strncpy부분에 password을 strlen 인자로 사용하고있다 password에 16바이트 크기이상의 값을넣어서 temp 뒤에 존재하는 auth의 값을 덮어씌워서 의도적으로 0이아니게 값을 만들시에 해당 코드가 통과할 수 있다.

### ex3

```Cpp
// stack-3.c
#include <stdio.h>
#include <unistd.h>
int main(void) {
    char win[4];
    int size;
    char buf[24];
    //32 
    scanf("%d", &size);
    //1111111111111111111111111111ABCD
    read(0, buf, size);
    if (strncmp(win, "ABCD", 4)){
        printf("Theori{-----------redeacted---------}");
    }
}
```
덮어 씌워서 win영역에 접근가능

### ex4
```Cpp
// stack-4.c
#include <stdio.h>
int main(void) {
	char buf[32] = {0, };
	read(0, buf, 31);
	sprintf(buf, "Your Input is: %s\n", buf);
	puts(buf);
}
```

------------
## out of boundary
버퍼의 길이 범위를 벗어나는 인덱스에 접근할 때 발생하는 취약점

```Cpp
// oob-1.c
#include <stdio.h>
int main(void) {
    int win;
    int idx;
    int buf[10];
    
    printf("Which index? "); 
    scanf("%d", &idx); // 11
    printf("Value: ");
    scanf("%d", &buf[idx]); //31337
    printf("idx: %d, value: %d\n", idx, buf[idx]);
    if(win == 31337){
        printf("Theori{-----------redeacted---------}");
    }
}
```
해당 실습에서는 int가 32bit로 설정되어있다. 따라서 4byte당 한칸으로 치며
buf 뒤에는 idx가 자리를 잡고있고 그뒤에 win이 위치한다 따라서 buf[10]이 wind의 자리와 동일


```cpp
// oob-2.c
#include <stdio.h>
int main(void) {
    int idx;
    int buf[10];
    int win;
    
    printf("Which index? ");
    scanf("%d", &idx); // -1
    
    idx = idx % 10;
    printf("Value: ");
    scanf("%d", &buf[idx]); // 31337
    printf("idx: %d, value: %d\n", idx, buf[idx]);
    if(win == 31337){
        printf("Theori{-----------redeacted---------}");
    }
}
```
1과 같다. 
전과 다르게 이번에는 buf인덱스에 10을넘어서 접근을 하지는 못하지만 모듈러는 음수값에는 모듈러 값이나와 음수범위에 접근을 할수가있다. 이때 win의 주소가 buf보다 윗쪽에 있으므로 접근을 할수있다

```cpp
//oob-3.c
#include <stdio.h>
int main(void) {
    int idx;
    int buf[10];
    int dummy[7];
    int win;
    printf("Which index? ");
    scanf("%d", &idx); //2147483648
    
    if(idx < 0)
        idx = -idx;
    idx = idx % 10; // No more OOB!@!#!
    printf("Value: ");
    scanf("%d", &buf[idx]); //31337
    printf("idx: %d, value: %d\n", idx, buf[idx]);
    if(win == 31337){
        printf("Theori{-----------redeacted---------}");
    }
}
```
여기서 좀 고민했는데 결국 방법은 -가 나오게 하는법 밖에없어서 그냥 오버플로우가 일어날거라 생각하고 큰값을 때려넣었다.
int max값보다 큰 2147483648을 넣으니 -8이 인덱스로 찍혔다.

int는 2^31-1 ~ -2^31 까지 표현 할 수 있는데 2^31은 -2^31이 된다.

-2^31은 0x1000_0000인데 이것의 역수는 0x1000_0000로 캐리를 버리는 방식밖에 없다.

------------
## Off-by-one

취약점은 경계 검사에서 하나의 오차가 있을 때 발생하는 취약점입니다. 이는 버퍼의 경계 계산 혹은 반복문의 횟수 계산 시 < 대신 <=을 쓰거나, 0부터 시작하는 인덱스를 고려하지 못할 때 발생합니다.

### ex1
```cpp
// off-by-one-1.c
#include <stdio.h>
void copy_buf(char *buf, int sz) {
    char temp[16];
    
    for(i = 0; i <= sz; i++)
        temp[i] = buf[i];
}
int main(void) {
    char buf[16];
    
    read(0, buf, 16);
    copy_buf(buf, sizeof(buf));
}
```




# Memory Corrunption 2


## Format String Bug

printf나 sprintf와 같이 포맷 스트링을 사용하는 함수에서 발생하는 취약점으로, "%x"나 "%s"와 같이 프로그래머가 지정한 문자열이 아닌 사용자의 입력이 포맷 스트링으로 전달될 때 발생하는 취약점

UB발생

```cpp
// fsb-1.c
#include <stdio.h>
int main(void) {
    char buf[100] = {0, };
    
    read(0, buf, 100);
    printf(buf);
}
```

buf에 만약 10,abcd와 같은 것을 입력했다면 그대로 출력되겠지만 %x,%d 같은것을 넣었다면 뒤의 가변인자를 받지 않기에 쓰레기값이 출력된다.


```cpp
// fsb-2.c
#include <stdio.h>
#include <stdlib.h>
int main(void) {
    FILE *fp = fopen("log.txt", "w");
    char buf[100] = {0, };
    
    read(0, buf, 100-1);
    
    fprintf(fp, "BUFFER-LOG: ");
    fprintf(fp, buf);
    
    fclose(fp);
    return 0;
}
```

두번째 fprintf 에서 buf를 포맷 스트링으로 입력 받기에 버그가 생길 수 있다.

표준 C 라이브러리에서 포맷스트링을 쓰는 함수들
- printf
- sprintf / snprintf
- fprintf
- vprintf / vfprintf
- vsprintf / vsnprintf

```cpp
// fsb-easy.c
#include <stdio.h>
int main(void) {
    int flag = 0x41414141;
    char buf[32] = {0, };
    
    read(0, buf, 31);
    printf(buf);
}
```
%x%x%x%x%x%x%x%x%x%x

처음에 %x를 했을때
41fe41ff가 출력되었다 뭔가 싶었더니 printf_stackframe의 값이었다. 리들엔디안으로 인해서 실제 저장 순서는 조금 뒤틀려있었다.
주소순으로 0xff,0x41,0xfe,0x41로 저장되어있다.
%x는 int를 값으로 받는지 4바이트 단위로 받고 따라서 buf 배열은 %x 8번 부르면 넘어간다. 그뒤에 flag를 부르기위해 %x를 부른다면 해결 꼭 %x를 10번 하는것이아닌 %d,%c를 이용해서 해결 할 수도있다.


## Double Free & Use After Free

Double Free : 해제된 메모리를 다시 해제하는 취약점
Use-After-Free(UAF) : 해제된 메모리에 접근해서 값을 쓰는 취약점

```cpp
// df-1.c
#include <stdio.h>
#include <malloc.h>
int main(void) {
    char* a = (char *)malloc(100);
    char *b = (char *)malloc(100);
    memset(a, 0, 100);
    strcpy(a, "Hello World!");
    
    memset(b, 0, 100);
    strcpy(b, "Hello Pwnable!");
    
    printf("%s\n", a);
    printf("%s\n", b);
    
    free(a);
    free(b);
    
    free(a);
}
```


```cpp
// uaf1.c
#include <stdio.h>
#include <string.h>
#include <malloc.h>
int main(void) {
    char *a = (char *)malloc(100);
    memset(a, 0, 100);
    
    strcpy(a, "Hello World!");
    printf("%s\n", a);
    free(a);
    
    char *b = (char *)malloc(100); 
    strcpy(b, "Hello Pwnable!");
    printf("%s\n", b);
    
    strcpy(a, "Hello World!");
    printf("%s\n", b);
}
```
이미 해제된 후 a가 가르키는 주솟값은 b와 같다 따라서 해제된 후 a를 건들면 b에도 수정된다


## 초기화되지 않은 메모리

```cpp
// uninit1.c
typedef struct person {
    char *name;
    int age;
} Person;
int main(void) {
    Person p;
    int name_len;
    
    printf("Name length: ");
    scanf("%d", &name_len);
    
    if(name_len < 100)
        p.name = (char *)malloc(name_len);
    read(0, p.name, name_len);
    
    printf("Age: ");
    scanf("%d", &p.age);
    
    printf("Name: %s\n", p.name);
    printf("Age: %d\n", p.age);
}
```

read는 값을 입력받을때 뒤에 null같은걸 붙이지 않는다 따라서  p.name뒤에 문자열로 출력이 끝없이 출력되어 정보들이 유출 될 수 있음

## integer issues

정수의 형 변환을 제대로 고려하지 못해 발생하는 취약점

size_t와 long 자료형은 아키텍쳐에 따라 표현할 수 있는 수의 범위가 달라집니다. long 자료형은 32비트인 경우 int와 동일하고, 64비트인 경우 long long과 동일합니다. size_t 자료형은 32비트일 때 unsigned int와 동일하며, 64비트일 때는 unsigned long과 같습니다.

### 묵시적 형변환

- 대입 연산시 묵시적 형변환 일어남 자료형 작은 정수형에 큰 정수형 대입시 상위바이트 소멸
-  정수 승격 은 char이나 short같은 자료형이 연산될 때 일어납니다. 이는 컴퓨터가 int형을 기반으로 연산하기 때문에 일어납니다.
-  피연산자가 불일치할 경우 형 변환 이 일어납니다. 이 경우 int< long< long long < float< double < long double 순으로 변환되며, 작은 바이트에서 큰 바이트로, 정수에서 실수로 형 변환이 일어나게 됩니다. 예를 들어, int와 double을 더하면 int가 double 형으로 변환된 후 연산이 진행됩니다.


```cpp
// int-1.c
#include <stdio.h>
#include <stdlib.h>
int main(void) {
    char *buf;
    int len;
    
    printf("Length: ");
    scanf("%d", &len);
    
    buf = (char *)malloc(len + 1);
    
    if(!buf) {
        printf("Error!");
        return -1;
    }
    
    read(0, buf, len);
}
```

그렇다면 공격자가 len 값으로 -1을 넣었을 때 프로그램의 흐름을 생각해 보겠습니다.

len = -1이므로 line 12에서는 buf = malloc(0)이 호출되고, 리눅스에서는 malloc의 인자가 0이라면 정상적인 힙 메모리가 반환됩니다. 이후 line 19에서 read(0, buf, -1)이 호출됩니다. 인자로 전달된 값은 int형 값 -1이고, read 함수의 세 번째 인자는 size_t 형이므로 묵시적 형 변환이 일어납니다. 따라서 read 함수를 호출할 때, 32비트 아키텍처라고 가정하면 read(0, buf, pow(2, 32) - 1)이 호출됩니다.

그러므로 지정된 크기의 버퍼를 넘는 데이터를 넣을 수 있어 힙 오버플로우가 발생합니다.


```cpp
// int-2.c
char *create_tbl(unsigned int width, unsigned int height, char *row) {
	unsigned int n;
	int i;
	char *buf;
	n = width * height;
	buf = (char *)malloc(n);
	
	if(!buf)
		return NULL;
	for(i = 0; i < height; i++)
		memcpy(&buf[i * width], row, width);
	return buf;
}
```
width, height, n이 전부 unsigned int형의 변수이기 때문에 width * height가 pow(2, 32)를 넘어가면 의도하지 않은 값이 들어가게 됩니다. width가 65536이고 height가 65537이라고 가정하겠습니다. 이 경우 width * height의 값은 65536 * 65537 = pow(2, 32) + 65536이므로 실제로 저장되는 값은 65536 * 65537이 아닌 65536이 됩니다.

그러나 memcpy 함수에서는 반복문을 순회하면서 메모리를 복사하기 때문에 버퍼 오버플로우가 발생하게 됩니다.

```cpp
char *read_data(int fd) {
	char *buf;
	int length = get_int(fd); // length는 사용자가 입력할 수 있는 값입니다.
	if(!(buf = (char *)malloc(MAX_SIZE))) // #define MAX_SIZE 0x8000
		exit(-1);
	if(length < 0 || length + 1 >= MAX_SIZE) {
		free(buf);
		exit(-1);
	}
	if(read(fd, buf, length) <= 0) {
		free(buf);
		exit(-1);
	}
	
	buf[length] = '\0';
	
	return buf;
}
```
0x7FFFFFFF