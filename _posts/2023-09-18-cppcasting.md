---
layout: post
title:  "cpp 캐스팅"
date:   2023-09-18 19:26:01 +0900
tag: c++
---


C의 캐스팅 vs c++스타일의 캐스팅

- c++ 캐스팅 중 하나를 함 -> c++에 비해 명확하지 못함
- 명백한 실수를 컴파일러가 캐치하지 못함


# static_cast

1. 값
    - 두 숫자 형 간의 변환
      - 값을 유지
      - 이진수 표기는 달라질 수 있음


```cpp
int num = 3
long long number = static_cast<long long>(number); // 값 유지

float num = 3.f;
int number = static_cast<int>(num);
```

```cpp
Animal* myPet = new Cat(2, "coco");

Cat* myCat = static_cast<Cat*>(myPet); //OK

Dog* myDog = static_cast<Cat*>(myPet); //컴파일 가능 그러나 크래시 위험
myDog->GetDogHouseName();
```



# reinterpret_cast

- 연관없은 두 포인터 형 사이의 변환을 허용
  - static_cast에서는 cat* -> house* 형변환이 컴파일 에러이나 reinterpret에서는 허용
- 포인터와 포인터 아닌 변수 사이의 형변환을 허용
  - 포인터를 int로 변환할때만 예외적으로 사용
  - 주소 오프셋 등을 저장하기 위해서 사용
  - Cat* <-> unsigned int
- 이진수 표기는 달라지지 않음
- cpp중에 가장 위험한 캐스팅


# const_cast
- 하지말아야 할 캐스팅
- 형 변환 불가
- const 또는 volatile 어트리뷰트를 제거할 때 사요ㅕㅇ

```cpp
Animal* myPet = new Cat(2, "coco");
const Animal* petPtr = myPet;

Animal* myAnimal1 = (Animal*)petPtr;
Cat* myCat1 = (Cat*)petPtr;

Animal* myAnimal2 = const_cast<Animal*>(petPtr);
Cat* myCat2 = const_cast<Cat*>petPtr;
```

# dynamic_cast

- 실행 중에 형을 판단
- 포인터 또는 참조 형을 캐스팅할 때만 쓸 수 있음
- 호환되지 않는 자식형으로 캐스팅하려 하면 NULL을 반환
    - 따라서 dynamic_cast가 static_cast보다 안전
- RTTI옵션을 켜야함


```cpp
c style

Animal *myPet = new Cat();

// Compiles ok
Dog* myDog = (Dog*)myPet;

// Compiles ok
myDog->GetHouseName();
```

```cpp
c++ style
Animal *myPet = new Cat();

// Compiles and returns NULL
Dog* myDog = dynamic_cast<Dog*>myPet;

if (myDog)
    myDog->GetHouseName();
```


# 캐스팅 규칙

규칙 (제일 안전한 것 -> 가장 위험한 것)

1. 기본적으로 static_cast 사용
  - reinterpret_cast<Cat*> 대신 static_cast<Cat*> 사용
    - cat이 animal이아니면 컴파일 에러
2. reinterpret_cast 사용
  - 포인터와 비포인터 사이의 변환
  - 서로 연관이 없는 포인터 사이의 변환은 그 데이터형이 맞다고 확실할때만
3. 내가 변경권한이 없는 외부 라이브러리를 호출할 때만 const_cast사용
    