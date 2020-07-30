---
layout: post
title:  "Interface의 이해"
date:   2020-07-14 22:53:01 +0900
categories: oop
---

# 인터페이스

인터페이스는 한마디로 추상클래스중 public 메소드만 남긴것이라 할 수 있다.

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

사실 이렇게 개념적으로만 놓고보면 감이 잘 안잡힌다. 사용을 정리하자면 다음의 용도로 사용한다

- 다중상속을 사용하지못하는 언어에서의 다중상속 흉내 + 다중상속의 다이아몬드 문제 해결
- 다형성 구현 : 클래스 작성자에게 해당 메소드 구현을 강제
  - 추상클래스와 달리 해당 메소드를 기본적으로 구현을 하지않기때문에 모든 클래스의 구현이 각각 있는경우


최상위 부모클래스인 Person의 자식클래스인 Person1, Person2을 각각 상속 받는 경찰과 검찰은 체포 할 수있는 특별한 메소드를 가지고 있다. 이 경우에 메소드가 해당 클래스가 따로 가지고 있게 하는 것은 해당 메소드를 가진 객체를 사용해야할때 따로따로 처리해줘야하는 불편함이 있다. 이때 interface를 사용해서 해당 인터페이스를 가지는 자료구조를 만듦으로서 묶어서 처리할 수 있다.

```java
public interface IArrestable
{
    void arrest(Person person);
}
public class Person
{
    public abstract void register(PersonManager manager);
}

public class Prosecution extends Person1 implements IArrestable
{
    public void arrest(Person person){
        //...
    }
    public void register(PersonManager manager){
        //...
        manager.registerArrestablePerson(this);
    }
}

public class Police extends Person2 implements IArrestable
{
    public void arrest(Person person)
    {
        //...
    }
    public void register(PersonManager manager){
        //...
        manager.registerArrestablePerson(this);
    }
}

public class PersonManager
{
    ArrayList<Person> persons;
    ArrayList<IArrestable> arrestablePersons;
    //..
    public void registerPerson(Person person){
        this.persons.add(person);
        person.register(this);
    }
    public void registerArrestablePerson(IArrestable arrestablePerson){
        this.arrestablePersons.add(arrestablePerson);
    }
}
```
또한 이런방식으로 메소드를 연쇄적으로 부르면서 다운클래스를 함으로생기는 성능 저하를 막을 수 있다.



