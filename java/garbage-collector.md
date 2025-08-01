# Garbage Collector

---

![Understanding Java Memory Model. Understanding Java Memory Model is an… |  by Thilina Ashen Gamage | Platform Engineer | Medium](https://miro.medium.com/v2/resize:fit:1400/0*rzQ6-DyP-2gjiua7)

- JDK7부터 도입된 **_G1(Garbage First)_** 를 제외한 나머지는 힙 영역 공간을 관리
- 데몬 프로세스로 백그라운드에서 실행되며, 사용하지 않는 객체를 식별하여 제거하여 힙 영역을 관리
- 사용하지 않는지 확인하는 방법은 해당 객체가 다른 객체에 의해 참조되고 있는지를 판단
- 참조 여부로 가비지 컬렉션 대상인지를 판단하기 때문에 만약 더이상 사용하지 않는 객체의 경우에는 가비지 컬렉션 대상이 되도록 설정하는 것이 좋음
  - 사용하지 않는 객체가 할당된 참조 변수를 null로 할당
  - 참조 변수에 새로운 객체를 할당하여 기존 객체의 참조를 끊음
  - 객체를 메소드 내에서 생성하여(stack 영역에서 참조하게 하여) 메소드 종료와 함께 가비지 컬렉션 대상이 되도록 함
  - 어떤 참조 변수도 해당 객체를 참조하지 않는 방식으로 사용
- **stop-the-world**
  - GC를 실행하기 위해서 GC를 실행하는 스레드를 제외한 나머지 스레드 작업을 모두 멈추는 것
  - GC 튜닝은 **stop-the-world** 시간을 줄이는 것
- GC를 실행하기 위해 `System.gc()`를 명시적으로 실행할 수는 있으나 이는 애플리케이션 성능에 매우 큰 영향을 미침
- GC 프로세스의 전제 조건
  - 대부분의 객체는 금방 접근 불가능(unreachage) 됨
  - 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재
- 위의 가설을 바탕으로 HotSpot VM에서 힙 영역을 2가지 영역으로 나눔

  - Young
  - Old

- **Permanent Generation 영역(Perm 영역)**이 **Method Area**

  ![JavaGarbage1](https://d2.naver.com/content/images/2015/06/helloworld-1329-1.png)

  - 객체나 억류(intern)된 문자열 정보가 저장되는 곳
  - Old 영역에서 살아남은 객체가 영원히 남아 있는 곳은 절대 아님
  - 이 영역에서도 GC가 발생할 수 있으며, 이 영역에서 발생한 GC도 Major GC 횟수에 포함

- GC 프로세스

  ![JavaGarbage3](https://d2.naver.com/content/images/2015/06/helloworld-1329-3.png)

---

## Young

- 생성된지 얼마 되지 않은 객체들이 존재

- 대부분의 객체가 금방 접근 불가능 상태가 되기 때문에 매우 많은 객체가 이 영역에서 생성되었다가 사라짐.

- **Eden**

  - 객체를 생성하자마자 저장되는 장소

- **Survivor**

  - Eden 영역에서 살아있는 객체가 이동하는 영역

- **_Minor GC_**

  - Young 영역의 메모리를 관리하는 가비지 컬렉션

  - 프로세스

    1. **Eden** 영역에 객체가 생성

    2. **Eden** 영역이 생성된 객체로 인해 가득차게 되면, 살아있는 객체만 **Survivor** 영역으로 복사되고, **Eden** 영역을 다시 채움

    3. **Survivor** 영역이 가득차게 되면 살아남은 객체만을 **다른 Survivor** 영역으로 복사, 이 과정에서 **Eden** 영역에 있는 객체들 중 살아있는 객체들도 **다른 Survivor** 영역으로 가게 됨. 즉 **Survivor** 영역은 항상 둘 중 하나를 비어놓음

       -> 두 영역 모두 비어있거나 두 영역 모두 가득 차 있다면 정상적이지 않은 상황

    4. 오래 살아 남은 객체는 **Old** 영역으로 이동

       - 내부에 살아남은 횟수를 카운팅하며, 임계점을 넘어 살아남은 객체를 Old 영역으로 이동
       - survivor space 크기가 너무 작아 특정 객체를 가지고 있을 수 없을 경우 해당 객체는 임계점 초과 여부와 관계 없이 Old 영역으로 이동

## Old

- 오래 살아 남은 객체가 Young 영역에서 이동되는 곳

- 접근 불가능 상태로 되지 않아 Young 영역에서 살아남은 객체가 존재

- Old 영역에 존재하는 객체는 Young 영역에 존재하는 객체를 참조하지 않는다는 가정에 예외 사항을 위해 Old 영역에 512바이트 chunk 카드 테이블이 존재

  - Young 영역 마이너 GC 발생 시 Old 영역에 있는 카드 테이블을 뒤져 GC 대상인지 식별
  - write barrier(마이너 GC를 빠르게 할 수 있도록 하는 장치)를 사용하여 관리하며, 약간의 오버헤드는 발생하지만 전반적인 GC 시간은 줄어듬

  ![JavaGarbage2](https://d2.naver.com/content/images/2015/06/helloworld-1329-2.png)

- 기본적으로 Old 영역에 대한 GC는 데이터가 가득 차면 실행

- **Major GC**

  - Old 영역이 꽉차게 될 경우 진행하는 가비지 컬렉션으로 **Full GC**로 불리기도 함

- 일반적으로 **Minor GC** 가 **Major GC** 보다 더 작은 공간에 할당되고 개게들을 처리하는 방식도 다르기 때문에 더 빠르게 진행

- GC 종류

  - Serial GC (-XX:+UseSerialGC)

    - 기본적으로 **mark-sweep-compact** 알고리즘 사용
      1. Old 영역에 살아 있는 객체를 식별 **(mark)**
      2. 힙 영역의 앞 부분부터 확인하여 살아 있는 것만 남김 **(sweep)**
      3. 각 객체들이 연속되게 쌓이도록 힙의 가장 앞 부분부터 채워서 객체가 존재하는 부분과 객체가 없는 부분으로 나눔 **(compaction)**
    - 적은 메모리와 CPU 코어 개수를 사용하는 머신에서 적합

  - Parallel GC (-XX:+UseParallelGC)

    ![JavaGarbage4](https://d2.naver.com/content/images/2015/06/helloworld-1329-4.png)

    - Serial GC와 기본적인 알고리즘은 동일
    - GC를 처리하는 스레드가 여러개
    - 메모리가 충분하고, 코어 개수가 많을 때 유리
    - Throughput GC

  - Parallel OLD GC (-XX:+UseParallelOldGC)

    - JDK5 update 6부터 제공하는 방식
    - Parallel GC에서 Old 영역에 대한 GC 알고리즘만 다름
      - mark-sweep-compact 방식이 아니라 **mark-summary-compact** 사용
      - **summary** 단계에서는 GC를 수행한 영역에서 살아남은 객체를 제거**(sweep)** 하는게 아니라 가야할 위치를 식별하여 **compact** 단계에서 **summary** 단계에서 판단한 위치로 이동시키며, **sweep** 단계를 건너뜀

  - CMS GC (-XX:+UseConcMarkSweepGC)

    ![JavaGarbage5](https://d2.naver.com/content/images/2015/06/helloworld-1329-5.png)

    - **Initial Mark** 단계에서 클래스 로더에서 가장 가까운 객체 중 살아 있는 객체만을 찾음

      - GC Root에서 시작하여 직접 참조되는 객체들을 mark하는 것으로 GC Root에 클래스 로더가 포함

    - **Concurrent Mark** 단계에서 살아 있다고 확인한 객체에서 참조하고 있는 객체들을 따라가면서 확인. 다른 스레드가 실행 중인 상태에서 동시에 진행

    - **Remark** 단계에서 Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인

    - **Concurrent Sweep** 단계에서 다른 스레드가 실행되고 있는 상황에서 쓰레기를 정리

    - stop-the-world 시간을 최대한 단축하기 때문에 애플리케이션의 전체 스레드가 GC를 위해 멈추는 시간이 짧아 애플리케이션의 응답 속도가 매우 중요한 상황에서 사용

    - 단점

      - 다른 GC 방식보다 메모리와 CPU 사용량이 많음

      - **Compaction** 단계가 기본적으로 제공되지 않음

        - 조각난 메모리가 많아 **Compaction** 작업을 실행할 때 다른 GC 보다 애플리케이션 전체 스레드를 멈춰야 하는 stop-the-world 시간이 더 길게 나타남

        - CMS GC를 사용하기 위해서는 **Compaction** 작업이 얼마나 자주, 오랫동안 수행되는지 확인 필요
