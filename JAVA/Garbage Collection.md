# GC (Garbage Collection)

C나 C++에서는 OS레벨의 메모리에 직접 접근하기 때문에 free() 메소드를 호출해서 메모리를 명시적으로 해제해야 한다. 그렇지 않으면 memory leak이 발생하게 된다.

반면 자바는 OS의 메모리 영역에 직접적으로 접근하지 않고 JVM이라는 가상머신을 이용해서 간접적으로 접근한다. JVM은 C로 쓰여진 또 다른 프로그램인데, 오브젝트가 필요해지지 않는 시점에서 알아서 free()를 수행하여 메모리를 확보한다. 이렇게 가상머신을 사용함으로써 운영체제로부터 독립적이라는 장접외에도 OS 레벨에서의 memory leak은 불가능하게 된다는 장점이 있다.

자바가 메모리 누수현상을 방지하는 또 다른 방법이 GC이다.

<b> - Garbage Collection</b>

> 핵심은 Heap 영역의 객체 중 stack에서 도달 불가능한(Unreachable) 객체들은 GC의 대상이다.

![img](https://s3.ap-northeast-2.amazonaws.com/yaboong-blog-static-resources/java/java-memory-management_gc-6.png)

Heap은 <b>Young Generation, Old Generation</b>으로 크게 두개의 영역으로 나누어지고, Young Generation은 또다시 <b>Eden, Survivor Space 0, 1</b>으로 세분화 되어진다.

1. 새로운 객체는 Eden 영역에 할당.

2. Eden이 가득차면 MinorGC가 발생.

3. MinorGC가 발생하면 Reachable 객체들은 S0으로 옮겨짐. Unreachable 객체들은 Eden 영역이 클리어 될 때 함께 메모리에서 사라짐.

4. 다음 MinorGC가 발생할 때, Eden 영역에는 3번 과정 발생. 이 때 기존에 S0에 있었던 Reachable 객체들은 S1으로 옮겨지는데, age값이 증가된다. 살아남은 모든 객체들이 S1으로 옮겨지면 S0와 Eden은 클리어된다. <b>Survivor Space에서 Survivor Space로의 이동은 age값이 증가한다.</b>

5. 다음 MinorGC가 발생하면 4번 과정 반복. S1이 가득차 있으면 S1에서 살아남은 객체들은 S0으로 옮겨지면서 Eden과 S1은 클리어된다. 마찬가지로 age값 증가.

6. Young Generation에서 계속해서 살아남으며 age값이 증가하는 객체들은 age값이 특정값 이상이 되면 Old Generation으로 옮겨지는데 이 단계를 <b>Promotion</b>이라고 한다.

7. MinorGC가 계속해서 반복되면 Promotion작업도 꾸준히 발생한다.

8. Promotion작업이 계속해서 반복되면서 Old Generation이 가득차게 되면 MajorGC가 발생한다.

   이 때 발생하는 MajorGC는 STW(stop-the-world)에 걸리는 Serial한 Full GC를 사용하게 되는데, 이럴 경우 정상적인 애플리케이션이 필요로하는 응답시간을 넘길 가능성이 높다.

어떤 GC를 사용할지를 결정하는 것은 결국 <b>stop-the-world 시간을 줄일 수 있는가</b>이다. 그래서 Java9 부터는 안정화를 거쳐 기본 GC가 G1 GC이다.

<b>- G1 GC</b>

G1(Garbage First) GC는 CMS GC나 Parallel Old GC에 비해서 스레드 정지가 예측 가능한 시간 안에 이루어지는 점진적으로 처리되는 병렬 compacting GC이다. 병렬성과 동시성, 다중화된 masking cycle로 인해 G1 GC는 최악의 경우라도 괜찮은 정도 수준의 스레드 정지가 발생하기에 더 큰 Heap을 다룰 수 있게 되었다. G1 GC의 가장 기본 아이디어는 heap 메모리의 범위와 현실적인 목표 스레드 정지시간을 설정하고 GC가 작업을 할 수 있도록 만들어주는 것이다.

G1 GC는 기존의 HotSpot에서 사용하는 전통적인 방식인 자바 heap 메모리를 young 영역과 old 영역 둘로 나누는 개념을 응용해야 한다. G1 GC는 <b>Region</b>이라는 개념을 새로 도입했다. 큰 자바 heap 공간을 고정된 크기의 region들로 나눈다. 이 region들을 free한 region들의 리스트 형태로 관리한다. 메모리 공간이 필요로 해지면, free region은 young영역이나 old 영역으로 할당한다. 이 region의 크기는 전체 heap 사이즈 용량이 2048개의 region으로 나누어 질 수 있도록 하는 범위 내에서 결정된다. region이 비어지면 이 region은 다시 free region 리스트로 돌아간다. <b>G1 GC의 기본 원리는 자바 heap의 메모리를 회수할 때 최대한 살아 있는 객체가 적게 들어있는 region을 수집하는 것이다</b> (목표 정지시간에 최대한 부합하도록). 가장 살아있는 객체가 적을 수록 쓰레기란 의미이고 따라서 GC의 이름도 쓰레기 우선 수집(Garbage First)라는 이름이 붙게되었다.

![fig2largeB 1](https://user-images.githubusercontent.com/27988544/68945958-04f88800-07f5-11ea-8dfd-6a8e43127858.jpg)
Garbage First GC Layout
{:.figure}

G1 GC에서 중요한 점은 Young이나 Old 영역이 인접해 있지 않다는 점이다. 이는 영역의 사이즈가 필요에 따라서 동적으로 바뀔 수 있다는 점에서 편리하다. 다만 HotSpot의 기본적인 개념(메모리 할당, eden, survivor, old generation 이동)은 활용한다.

<br/>

## 작성자

<a href="https://github.com/marco0332"><img src="https://avatars2.githubusercontent.com/u/27988544?s=460&v=4" width="100" height="100" /></a>