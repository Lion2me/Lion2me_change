---

layout: post

title: Thread에 대하여

date: 2021-09-03 21:05:23 +0900

categories: Basic

---

Thread에 대해서
---

Thread 관련 내용을 긴장한채로 질문 받았을 때 제대로 답변을 하지 못했기에 다시 정리하기위해 정리합니다.

가장 먼저 Thread가 어떤 것인지 알아보겠습니다.

### Process

Thread를 알기 위해서는 Process에 대한 지식이 선행되어야 합니다.

프로그램을 실행하기 위해서는 프로그램을 메모리에 적재 할 필요가 있습니다.
이유는 I/O 속도 때문입니다!
조금만 더 제대로 설명드리면 프로그램이 실행되기 위해서는 해당 프로그램의 데이터를 불러와야 합니다. 컴퓨터에는 데이터를 저장 할 수 있는 공간은 크게 4개로 나뉩니다.

**레지스터 / 캐시 / 메모리(RAM) / 디스크** 이렇게 크게 4가지로 왼쪽에 있는 것부터 점차 느려지는 성격이 있으며 동시에 용량이 커지는 성격이 있다고 생각하시면 됩니다.

디스크에 2TB의 파일을 저장하지만, 속도가 너무 느리기 때문에 프로그램을 실행시키기에는 성능상의 문제가 극심합니다.
반대로 캐시에 저장하고 프로그램을 실행시키기에는 nMB 정도의 작은 공간에 담기에는 프로그램이 너무 큽니다.

중간 지점이 **메모리**입니다.

요즘 메모리(RAM)은 16GB가 넘는 기기가 출시 될 정도로 용량이 충분하며 속도도 디스크보다 훨씬 빠릅니다. 이러한 이유로 우리는 프로그램을 실행하기 위해 메모리에 적재하는 방식을 사용합니다.

16GB도 최근 프로그램의 높은 용량을 실행시킬 수 없다고 생각하실수도 있지만, 이 내용은 이후에 포스팅하도록 하겠습니다.

즉 Process는 메모리에 적재 된 프로그램의 인스턴스 정도로 생각 할 수 있습니다.

### Thread

Process가 CPU를 할당받고 실행됩니다. 그리고 **Process 내의 Thread가 프로그램이 원하는 작업을 수행**합니다. 하지만 하나의 작업에 모든 CPU자원을 사용한다면 CPU가 얼마나 사용될까요?

CPU는 생각보다 높은 성능을 가지고 있기 때문에 특정 작업이 아닌 대부분의 작업에 사용하는 되는 동안 CPU 활용도는 정말 낮습니다. 어차피 노는 CPU를 최대한 효율적으로 사용하고자 두 가지 방법을 생각해냅니다.

**Multi Process** 와 **Multi Thread**입니다.

#### Multi Process

Multi Process는 한 프로그램 실행에 대해서 2개 이상의 프로세스를 이용하는 방식입니다. 정확히 이 방식을 자주 사용하지는 않습니다만, 제가 가끔하는 게임인 **리그오브레전드**의 로비 창 프로세스와 게임 플레이 프로세스의 구분이 이러한 Multi Process로 추정됩니다.

Process의 메모리 구조는 다음의 [블로그](https://lion2me.github.io/basic/2021/07/12/메모리-영역에-대하여.html)를 통해 정리했습니다.

프로세스별로 각 메모리 공간을 할당받기 때문에 메모리의 낭비가 클 수 있습니다. 주로 이런 문제로 우리는 **Multi Thread** 방식을 사용합니다.

#### Multi Thread

Thread는 Process 내에서 동작을 수행하는 단위이므로, Process에 포함 된 개념이라고 생각 할 수 있습니다. 그리고 여러 개의 Thread를 사용하는 Multi Thread는 결국 하나의 Process가 실행하면서 여러 개의 Thread를 사용하여 CPU의 활용을 최대화한다. 정도로 이해 할 수 있을 것 같습니다.

Multi Thread의 특징으로는
**1. 한 프로세스의 스택을 제외 한 모든 영역을 공유해서 사용합니다.**
**2. 그렇기 때문에 공유하는 영역에 접근 할 때 여러 Thread가 동시에 접근 할 수 있는 문제가 발생합니다.**
**3. 그럼에도 메모리가 효율적이고 여러 동작을 동시에 수행 할 수 있게 하기에 자주 사용됩니다.**

여러 작업을 동시에 수행 할 수 있는 만큼 Multi Thread를 사용 할 때 주의 할 점이 있습니다.

**1. 동기화에 주의해야 한다.**
**2. 동기화에 주의해야 한다.**
**3. 동기화에 주의해야 한다.**

중요해서 3번 적었습니다. 발생 할 수 있는 문제 중 심각한 문제를 꼽자면
**한 자원에 두 개 이상의 Thread가 동시에 접근함으로써 원치 않은 결과가 발생하는 문제** 정도로 말 할 수 있습니다.

예를 들면 두 Thread가 각각 **5를 더한다**/**10을 뺀다** 의 동작을 수행 할 때 **20 의 값을 가지고 동작을 수행한다면** 결과는 **15**가 나와야 합니다.

하지만 두 Thread가 동시에 **Read 20**을 수행하고 **A Thread가 5를 더하고, B Thread가 10을 빼버린다면? 결과는 25아니면 10이 나올 수 있겠네요.** 즉 접근하는 시점과 동작 수행 시점이 겹칠 수 있습니다.

이런 문제가 Multi Thread에서 일어 날 수 있는 문제입니다. 이 문제를 해결 할 수 있는 방법으로는 강조했던 **동기화**가 필요합니다.

이렇게 작업에 대해 동기화가 잘 이뤄지는 것이 바로 **Thread가 Safe한 상태**라고 합니다.

### 참고
<https://velog.io/@cateto/Java-Thread-Safe란>