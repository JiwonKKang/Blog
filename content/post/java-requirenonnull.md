---
title: "[Java] Objects.requireNonNull()"
date: 2023-09-17T02:22:08+09:00
draft: false
pin: false
summary: Objects.requireNonNull()은 왜 사용해야 할까?
tags:
  - java
---

> # Objects.requireNonNull()은 왜 사용해야 할까?

intelliJ로 코딩을 하는데 자꾸 노란줄이 뜨면서 

Objects.requireNonNull()을 권장하길래 뭐하는 메서드인지 한번 찾아보았다.

이름에서 알 수 있듯 이 메서드는 non-null을 표시하는 역할을 합니다.

```java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}
```

requireNonNull메서드의 내부는 다소 충격적이다. 왜냐하면 너무나도 **당연한** 코드이기 때문입니다.

**_왜 사용하는걸까??_**

참조 객체가 null일 때 NPE가 나는 것은 당연한 것이기 때문에 잘 와닿지 않을 수 있습니다.

이를 사용하는 이유는 크게 다음과 같습니다.

- explicity (명시성)
- fail fast (빠른 실패)

<br>

> # explicit

다음과 같이 A를 참조하는 B클래스가 있을 때

```java
public class A {
	String name;
}
```

```java
public class B {
    A a;

    public B(A a) {
        this.a = Objects.requireNonNull(a);
    }
}
```

코드 상에서 A가 null이 아니어야 함을 명시적으로 표현할 수 있습니다.  
따라서 과거에 짠 코드가 미래에 사용될 때 해당 객체가 null이면 안된다는 것을 개발자가 **명시적으로 알 수 있습니다.**

<br>

> # fail-fast

`fail-fast`란 장애가 발생한 시점에서 즉시 (혹은 최대한 빠르게) 파악할 수 있는 것을 뜻합니다.  
위의 B클래스는 다음과 같이 객체 생성 시점에 바로 익셉션이 발생할 것입니다.

```java
A a = null;
B b = new B(a);     // 생성 시점에 바로 NPE 발생
```

---

반면 다음과 같이 requireNonNull을 사용하지 않은 경우는 어떨까?

```java
public class C{

    A a;

    public C(A a) {
        this.a = a;     //Objects.requireNonNull 사용x
    }

    //...getter
}
```

바로 익셉션을 발생하지 않고 이후에 해당 객체가 사용될 때 알 수 있게 됩니다.

```java
A a = null;
C c = new C(a);
c.getA();      // 객체 생성 이후에 늦게 NPE 발생
```

이는 시스템이 복잡해 질 수록 장애를 발견하기 어렵게 만들 수 있습니다.

<br>

> # 기타 장점

- **디버깅이 용이**해지고 안정성이 높아 집니다.
    
- 항상 같은 시점에 익셉션을 발생시키는 것은 시스템의 **일관성**을 높이고. 개발자가 나머지 부분에 더 신경 쓸 수 있게 해줍니다.
    
- NPE를 명시적으로 던지는 것이 JVM이 발견해서 발생시키는 것 보다 **성능상의 이점**이 있다고 합니다.

<br>

> # vs Optional

### 목적

- Optional은 **null일지도 모르는 값을 처리하는데** 초점이 맞춰져 있지만
- requireNonNull은 해당 참조가 **null일 경우 즉시 개발자에게 알리는 것**이 목적입니다.

### 사용 형태

- Optional은 리턴타입에 사용되는 것이 권장됩니다.
- requireNonNull은 메서드 내부에서 큰 제약 없이 사용될 수 있습니다.

## requireNonNullElseGet

자바9버전 부터는 다음과 같은 메서드가 추가되면서 Optional과 비슷하게 사용이 가능합니다.

```java
requireNonNullElseGet(T obj, Supplier<? extends T> supplier)
```

<br>

> # vs @Nonnull

- @Nonnull은 주어진 인수가 NULL이 아니라고 컴파일러에게 알리는 것
- requireNonNull은 주어진 객체가 NULL인지 확인하는 방어 프로그래밍


<br><br>

---

>_출처_

https://velog.io/@rockpago/Objects.requireNonNull