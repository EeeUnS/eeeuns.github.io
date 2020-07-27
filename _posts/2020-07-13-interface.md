---
layout: post
title:  "Interface의 이해(미완)"
date:   2020-07-14 22:53:01 +0900
categories: oop
---

# 인터페이스

인터페이스는 한마디로 추상클래스중 메소드만 남긴것이라 할 수 있다.

java에서 두개는 사실상 같다.
```java
abstract class InterfaceEX1{
    public abstract void method1();
    public abstract void method2();
    ...
}

interface InterfaceEX2{
    void method1();
    void method2();
    ....
}
```

C++에서는 자체적으로 지원하지않기 때문에 첫번째 얘시처럼 abstract대신 vitual을 떡칠함으로써 흉내낼 수 있다. 

```c++
class InterfaceEX3{
public :
    virtual ~InterfaceEX3();
    virtual void method1();
    virtual void method2();
    ...
}
```

특징으로는
- 멤버 변수를 사용할수없고 
- 무조건 public 멤버함수이다.

# 실 사용 예시

사실 이렇게 개념적으로만 놓고보면 감이 잘 안잡힌다.

```JAVA
class {

}

interface Rider{

}

class personManage{
    ArrayList<person> persons;
    ArrayList<Rider> riders;
    void registerRider(Rider)
}

class racer extends person implements Rider{

}

class biker extends person implements Rider{
    
}

class perosnManage{

}

```
......