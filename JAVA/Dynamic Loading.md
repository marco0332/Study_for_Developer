# Dynamic Loading

Java 프로그램은 한 개 혹은 그 이상의 클래스들의 조합으로 실행된다. 그리고 실행 시 모든 클래스 파일들이 한 번에 JVM 메모리에 로딩되지 않고 요청되는 순간 로딩된다. Java의 **클래스 로더**가 이 역할을 수행한다. 여기서 클래스 로더는 '.class' 바이트 코드를 읽어 들여 class 객체를 생성하는 역할을 담당한다. 즉, 클래스 로더는 클래스가 요청될 때 파일로부터 읽어 메모리로 로딩하는 역할을 하며 JVM의 중요한 요소 중 하나이다.

자바의 클래스 로딩은 세부적으로 로딩, 링크, 초기화라는 세 단계 과정을 거친다.

- 로딩 : 클래스 파일을 바이트 코드로 읽어 메모리로 가져오는 과정
- 링크 : 읽어온 바이트 코드가 자바 규칙을 따르는지 검증, 클래스에 정의된 필드, 메소드, 인터페이스들을 나타내는 데이터 구조를 준비, 그 클래스가 참조하는 다른 클래스를 로딩.
- 초기화 : super 클래스 및 정적 필드를 초기화.

여기서는 로딩에 대해서만 알어보도록 하자.

![img](https://t1.daumcdn.net/cfile/tistory/99CC283D5C57619618)

먼저 Loading은 저장소(하드디스크)에서 메모리로 데이터를 옮기는 작업이다. 프로그램을 실행시키면 .exe에 있는 파일이 메모리에 올라가야 실행이 되는데, 그것을 메모리에 적재한다 해서 로딩이라고 한다. 아래에서 더 자세하게 살펴보자.

### Process Address Space

메모리 주소에 저장된 프로세스 형태는 다음과 같다.

![img](https://t1.daumcdn.net/cfile/tistory/99CA543C5C57625C16)

 제일 아래 영역부터 Text Segment, Data Segment, Heap 으로 구성되어 있고, Stack은 위에서부터 밑으로 영역을 차지한다.

- **Text Segment & Data Segment**

  프로그래밍한 코드와 변수 정보들이 들어있는 메모리 영역. 프로그램이 실행되기 위해 항상 존재해야 하는 내용들이 저장된다. 보통 exe파일 안에 들어있는 내용이 이 영역에 들어간다. 이 영역은 JVM에서 실행되고 있는 모든 쓰레드에 의해 공유된다. JVM은 여러 개의 쓰레드가 메소드를 정상적으로 사용하기 위한 동기화 기법을 제공한다.

  이 곳에서 파일에 있는 것을 읽어서 메모리 영역에 Write하는 것을 Loading이라고 한다.

  참고 _ <a href=" https://whereisusb.tistory.com/10 ">프로세스의 주소공간</a>

- **Heap & Stack**

  **Stack 영역에는 지역변수와 매개변수가 저장된다.** 즉, 프로그램의 실행과정에서 '임시로 할당'되고, 끝나면 바로 소멸되는 데이터들이 저장된다. 자세하게 설명하면 수행되는 메소드 정보, 로컬 변수, 매개변수, 연산 중 발생하는 임시데이터 등이 저장되며, 메소드 수행이 끝나면 삭제된다. 이 때 Heap 영역에 생성된 Object 타입의 경우 데이터의 참조값이 할당되며, Primitive 타입의 경우 값과 함께 데이터가 할당된다.

  **Heap 영역에는 코드에서 'new' 명령을 통해 생성된 인스턴스 변수가 할당된다.** 스택 영역에 저장되는 로컬변수, 매개변수와 달리 힙 영역에 보관되는 메모리는 메소드 호출이 끝나도 사라지지 않고 GC가 지우거나 JVM이 종료될 때 까지 유지된다.

  이 영역도 유일한 공간으로 여러 쓰레드가 공유한다.
  
  참고 _ <a href=" https://yaboong.github.io/java/2018/05/26/java-memory-management/ ">Java에서 Stack과 Heap</a>

### Dynamic Loading

그럼 Dynamic Loading이 그냥 Loading과의 차이점은 쉽게 말해 **루틴이 호출될 때 까지 메모리에 적재되지 않는다**는 것이다.

- 장점
  1. 일부 클래스가 변경되어도 전체 어플리케이션을 다시 컴파일하지 않아도 된다.
  2. 변경사항이 발생해도 비교적 적은 작업만으로도 처리할 수 있는 유연한 어플리케이션을 작성할 수 있다.
  3. C언어와 다르게 코드 블럭이 여러 곳으로 분산되어 위치해도 된다.
  4. 다형성과 같은 객체지향 개념이 적용가능해진다.
- 단점
  1. 실행 시 연결된 부분에 대한 판단을 해야하므로 속도 측면에서는 불리하다.

Dynamic Loading의 종류로는 **로드 타임 동적 로딩(load-time dynamic loading)**과 **런타임 동적 로딩(run-time dynamic loading)**이 있다.

#### 1. 로드타임 동적 로딩

```java
public class LoadTimeExample {
    public static void main(String[] args) {
        System.out.println("로드 타임 동적 로딩 테스트");
    }
}
```

LoadTimeExample 클래스를 실행할 경우 JVM이 시작되고, 모든 클래스가 상속받고 있는 Object 클래스를 읽어온다. 그 후에 LoadTimeExample.class 파일을 읽고, 이 클래스를 로딩하는 과정에서 필요한 java.lang.String, java.lang.System 클래스를 가져온다. 즉, LoadTimeExample 클래스를 읽어오는 과정에서(로드 타임)에 로딩된다. 이처럼, 하나의 클래스를 로딩하는 과정에서 동적으로 다른 클래스를 로딩하는 것을 **로드타임 동적 로딩**이라고 한다.

#### 2. 런타임 동적 로딩

```java
public class RunTimeExample1 implements Runnable {
    public void run() {
        System.out.println("런타임 동적 로딩 테스트 11111");
    }
}

public class RunTimeExample2 implements Runnable {
    public void run() {
        System.out.println("런타임 동적 로딩 테스트 22222");
    }
}
```

이 클래스는 Runnable 인터페이스를 구현한 간단한 클래스이다. 실제로 런타임 동적 로딩이 일어나는 클래스를 만들어보자.

```java
public class RunTimeTest {
    public static void main(String[] args) {
        try {
            if (args.length < 1) {
                System.out.println("사용법: java RunTimeTest [클래스 이름]");
                System.exit(1);
            }
            Class inputClass = Class.forName(args[0]);
            Object obj = inputClass.newInstance();
            Runnable r = (Runnable) obj;
            r.run();
        } catch(Exception exception) {
            exception.printStackTrace();
        }
    }
}
```

위 코드에서 Class.forName(className)은 파라미터로 받은 className에 해당하는 클래스를 로딩한 후에, 그 클래스에 해당하는 Class 인스턴스(로딩한 클래스의 인스턴스 아님을 주의!)를 반환한다. Class 클래스의 newInstance() 메소드는 Class가 나타내는 클래스의 인스턴스를 생성한다. 예를 들어, 다음과 같이 한다면 java.lang.String 클래스 객체가 생성된다.

```java
Class inputClass = Class.forNAme("java.lang.String");
Object obj = inputClass.newInstance(); // java.lang.String 객체
```

따라서 Class.forName() 메소드가 실행되기 전까지는 RunTimeTest 클래스에서 어떤 클래스를 참조하는지 알 수 없다. 다시 말해, RunTimeTest 클래스를 로딩할 때는 어떤 클래스도 읽지 않고, main() 메소드가 실행되고 Class.forName(args[0])를 호출하는 순간에 args[0]에 해당하는 클래스를 읽어온다. 이처럼 클래스를 로딩할 때가 아니라 코드를 실행하는 순간에 클래스를 로딩하는 것을 **런타임 동적 로딩**이라고 한다.

다음은 RunTimeTest 클래스를 명령행에서 실행한 결과를 보여준다.

```
$ java.RunTimeTest RunTimeExample2
런타임 동적 로딩 테스트 22222
```

<br/>

## 작성자

<a href="https://github.com/marco0332"><img src="https://avatars2.githubusercontent.com/u/27988544?s=460&v=4" width="100" height="100" /></a>