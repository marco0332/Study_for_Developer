# Overloading과 Overriding

## 메소드 오버로딩 (Method Overloading)

메소드 오버로딩이란 같은 이름의 메소드를 한 클래스에 여러 개 정의할 수 있는 기능을 의미. 오버로딩이 성립하기 위해서는 다음과 같은 규칙이 지켜져야 한다.

1. 파라미터의 타입이나 개수가 달라야 한다.

   ```java
   String getName(int id);
   String getName(int id, String userInfo);
   ```

2. 파라미터의 이름은 오버로딩 성립에 영향을 주지 않는다.

   ```java
   String getName(int id);
   String getName(int userId);
   ```

3. 반환 타입은 오버로딩 성립에 영향을 주지 않는다.

   ```java
   String getName(int id);
   char[] getName(int id);
   ```

4. 동일한 메소드 명을 사용해야 한다.

다음은 오버로딩의 예제이다.

```java
class ExamOfOverloading {
    int value;
    int userId;
    
    // 생성자
    public ExamOfOverloading() {
        this.value = 10000;
        this.userId = 1;
    }
    
    // 생성자 오버로딩
    public ExamOfOverloading(int value, int userId) {
        this.value = value;
        this.userId = userId;
    }
    
    public void printClassInfo() {
        System.out.println("value: " + value + ", userId: " + userId);
    }
}

class TestClass {
    /**
    * Overloading 테스트
    */
    public static void main(String[] args) {
        ExamOfOverloading eoo = new ExamOfOverloading();
        ExamOfOverloading eooFilled = new ExamOfOverloading(100000000, 150);
        eoo.printClassInfo();
        eooFilled.printClassInfo();
    }
}
```

> **실행 결과**
>
> value: 10000, userId: 1
>
> value: 100000000, userId: 150

## 오버라이딩 (Overriding)

오버라이딩이란 부모로부터 상속받은 메소드를 자식 클래스에서 자기 자신에 맞게끔 재정의하는 기능이다. 오버라이딩이 성립하려면 다음 규칙을 지켜야 한다.

1. 부모 클래스와 자식 클래스 사이에만 성립될 수 있다. 즉, 상속이 가능해야 오버라이딩이 가능한다.
2. static 메소드는 클래스에 속하는 메소드이기 때문에 상속이 불가능하다.
3. 접근 제한자가 private으로 정의된 메소드는 상속이 불가능하다.
4. interface를 구현해서 메소드를 오버라이딩할 때는 반드시 접근 제한자를 public으로 정의해야 한다.
5. 메소드의 인자의 개수와 타입이 완전히 일치해야 하고, 리턴 타입까지 일치해야 한다.
6. 부모 클래스의 메소드의 접근제한자보다 좁아질 수 없다.
7. 부모 클래스의 메소드보다 더 많은 예외를 던질 수 없다.
8. final 예악어가 지정된 메소드는 오버라이딩할 수 없다.

```java
class Parent {
    void printClass() {
        System.out.println("Parent Class");
    }
}

class Child extends Parent {
    @Overriding
    void printClass() {
        System.out.println("Child Class");
    }
}

class Test {
    /**
    * Overriding 테스트
    */
    public static void main(String[] args) {
        Parent parent = new Parent();
        Child child = new Child();
        parent.printClass();
        child.printClass();
    }
}
```

>**실행 결과**
>
>Parent Class
>
>Child Class



<br/>

## 작성자

<a href="https://github.com/marco0332"><img src="https://avatars2.githubusercontent.com/u/27988544?s=460&v=4" width="100" height="100" /></a>