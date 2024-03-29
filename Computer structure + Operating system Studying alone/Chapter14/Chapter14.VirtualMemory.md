# Chapter14.VirtualMemory

---

# 연속 메모리 할당

## 스와핑

1. 입출력 작업의 요구로 대기 상태가 된 프로세스
2. 오랫동안 사용되지 않은 프로세스

이런 프로세스들을 임시로 보조기억장치 일부 영역으로 쫓아내고, 그렇게 해서 생긴 메모리 상의 빈 공간에 또 다른 프로세스를 적재하여 실행하는 방식

보조기억장치 일부 영역 -> **스왑 영역**
실행되지 않는 프로세스가 메모리에서 스왑 영역으로 옮겨지는 것 -> **스왑 아웃**
스왑영역에 있던 프로세스가 다시 메모리로 옮겨 오는 것 -> **스왑 인**

## 메모리 할당

비어 있는 메모리 공간에 프로세스를 연속적으로 할당하는 방식

1. 최초 적합 -> 최초로 발견한 적재 가능한 빈 공간에 프로세스를 배치하는 방식
2. 최적 적합 -> 프로세스가 적재될 수 있는 가장 작은 공간에 프로세스를 배치
3. 최악 적합 -> 프로세스가 적재될 수 있는 가장 큰 공간에 프로세스를 배치

프로세스를 메모리에 연속적으로 배치하는 연속 메모리 할당은 메모리를 효율적으로 사용하는 방법이 아님. 왜? **외부 단편화** 때문

## 외부 단편화

![Untitled](Chapter14%20VirtualMemory%2051a124707a454a13a76350b01087f89d/Untitled.png)

프로세스들이 메모리에 연속적으로 할당되는 환경에서는 프로세스들이 실행되고 종료되기를 반복하며 메모리 사이 사이에 빈 공간들이 생기는데, 프로세스 바깥에 생기는 이러한 빈 공간들은 분명 빈 공간이지만 그 공간보다 큰 프로세스를 적재하기 어려운 상황을 초래하고 결국 메모리 낭비로 이어진다. -> 외부 단편화

외부 단편화를 해결할 수 있는 대표적인 방안으로는 메모리를 압축하는 **조각모음**이 있다.

### 조각 모음

메모리 내에 저장된 프로세스를 적당히 재배치시켜 흩어져 있는 작은 빈 공간들을 하나의 큰 빈공간으로 만드는 방법.

**단점**

- 백업을 위해 시스템이 하던 일을 중지해야 함.
- 메모리에 있는 내용을 옮기는 작업은 많은 오버헤드를 야기
- 어떤 프로세스를 어떻게 움직여야 오버헤드를 최소화하며 압축할 수 있는지 명확한 방법을 결정하기 어렵다.

이를 해결하기 위해 또 다른 해결 방법인 **페이징 기법**이 등장한다.

# 페이징 기법을 통한 가상 메모리 관리

- 오늘날까지도 사용되는 가상 메모리 기법이다.
- 가상 메모리는 실행하고자 하는 프로그램을 일부만 메모리에 적재하여 실제 물리 메모리 크기보다 더 큰 프로세스를 실행할 수 있게 하는 기술.
- 가상 메모리는 크게 페이징과 세그멘테이션이 있지만, 이 책에서는 현대 대부분의 운영체제가 사용하는 페이징 기법만을 다룬다.
- 페이징 기법을 활용하면 물리 메모리보다 큰 프로세스를 실행할 수 있을 뿐더러 외부 단편화 문제도 해결 가능

## 페이징이란?

- 메모리의 물리 주소 공간을 **프레임** 단위로 자르고, 프로세스의 논리 주소 공간을 **페이지** 단위로 자른 뒤 각 페이지를 프레임에 할당하는 가상 메모리 관리 기법.
- 페이징에서도 스와핑 사용 가능. 프로세스 전체가 스왑 아웃/인 되는 것이 아닌 페이지 단위로 스왑됨.
    
    메모리에 적재될 필요없는 페이지들은 보조기억 장치로 스왑 아웃. -> **페이지 아웃**
    
    실행에 필요한 페이지들은 메모리로 스왑 인. -> **페이지 인**
    
- 이는 다르게 말하면 한 프로세스를 실행하기 위해 프로세스 전체가 메모리에 적재될 필요가 없다는 말과 같다.
- 프로세스를 이루는 페이지 중 실행에 필요한 일부 페이지만을 메모리에 적재하고, 당장 실행에 필요하지 않은 페이지들은 보조기억장치에 남겨둠으로써, 물리 메모리보다 더 큰 프로세스를 실행할 수 있다.

페이징 기법을 통해 프로세스가 메모리에 불연속 적으로 배치되어 있다면 CPU 입장에서 다음에 실행할 명령어의 위치를 찾기 어려우므로 순차적으로 실행할 수 없다.

이를 해결하기 위해 페이징 시스템은 프로세스가 비록 물리 주소에 불연속적으로 배치 되더라도 논리 주소에는 연속적으로 배치되도록 **페이지 테이블**을 이용한다.

## 페이지 테이블?

페이지 테이블은 페이지 번호와 프레임 번호를 짝지어 주는 일종의 이정표이다. CPU는 이를 보고 페이지 번호를 통해 해당 페이지가 적재된 프레임을 찾을 수 있다.

프로세스마다 각자의 테이블이 있다. 이와 같은 방식으로 프로세스들이 메모리에 분산되어 저장되어 있더라도 CPU는 논리 주소를 그저 순차적으로 실행하면 된다.

### 페이징은 외부 단편화를 해결 할 수 있지만, 내부 단편화라는 문제를 야기할 수 있다.

![Untitled](Chapter14%20VirtualMemory%2051a124707a454a13a76350b01087f89d/Untitled%201.png)

![Untitled](Chapter14%20VirtualMemory%2051a124707a454a13a76350b01087f89d/Untitled%202.png)

페이징은 논리 주소 공간을 페이지라는 일정한 크기의 단위로 자른다. 하지만 모든 프로세스가 페이지 크기에 딱 맞게 잘리는 것이 아니다.
ex) 페이지 크기가 10kb 인데 프로세스의 크기가 108kb라면 마지막 페이지는 2kb가 남는다.

이처럼 내부 단편화는 하나의 페이지 크기보다 작은 크기로 발생한다. 그렇기에 하나의 페이지 크기가 작다면 발생하는 내부 단편화의 크기도 작아질 것으로 기대할 수 있다.
하지만 하나의 페이지 크기를 너무 작게 설정하면 그만큼 페이지 테이블 크기도 커지기에 페이지 테이블이 차지하는 공간이 낭비된다.

내부 단편화를 적당히 방지하면서 너무 크지 않은 페이지 테이블이 만들어지도록 페이지의 크기를 조정하는 것이 중요!

프로세스마다 각자의 프로세스 테이블을 가지고 있고 각 프로세스의 페이지 테이블들은 메모리에 적재되어 있다. CPU 내의 **페이지 테이블 베이스 레지스터(PTBR)**는 각 페이지 테이블이 적재된 주소를 가리키고 있다.

ex) 프로세스 A가 실행될 때 PTBR은 프로세스 A의 페이지 테이블을 가리킨다.

각 프로세스들의 페이지 테이블 정보들은 각 프로세스의 PCB에 기록된다. 그리고 프로세스의 문맥 교환이 일어날 때 다른 레지스터와 마찬가지로 함께 변경된다.

페이지 테이블을 메모리에 두면 메모리 접근 시간이 두 배로 늘어난다는 단점이 있다.
페이지 테이블을 보기위해 한 번, 알게 된 프레임에 접근하기 위해 한 번.

이와 같은 문제를 해결하기 위해 CPU 곁에 **TLB(Translation Lookaside Buffer)**라는 페이지 테이블의 캐시 메모리를 둔다.

### TLB

TLB는 페이지 테이블의 캐시이기 때문에 페이지 테이블의 일부 내용을 저장한다. 참조 지역성에 근거해 주로 최근에 사용된 페이지 위주로 가져와 저장.

CPU가 발생한 논리 주소에 대한 페이지 번호가 TLB에 있을 경우 -> TLB 히트
없을 경우 -> TLB 미스

## 페이징에서의 주소 변환

하나의 페이지 혹은 프레임은 여러 주소를 포괄하고 있다.

1. 어떤 페이지 혹은 프레임에 접근하고 싶은지.
2. 접근하려는 주소가 그 페이지 혹은 프레임으로부터 얼마나 떨어져 있는지.

페이징 시스템에서는 모든 논리 주소가 기본적으로 페이지 번호와 변위로 이루어져 있다.
논리 주소의 변위와 물리 주소의 변위 값은 같다.

- ex) 프로세스 A-2(페이지 번호)에서, 10(변위)만큼 떨어진 주소에 접근
- CPU [5, 2] 라는 논리 주소에 접근 요청 
-> 페이지 테이블 5번 페이지는 현재 1번 프레임(8번지)에 있음
-> CPU는 10번지에 접근함

### 페이지 테이블 엔트리

![Untitled](Chapter14%20VirtualMemory%2051a124707a454a13a76350b01087f89d/Untitled%203.png)

페이지 테이블의 각각의 행들을 페이지 테이블 엔트리 라고함.

페이지 테이블 엔트리에는 페이지 번호, 프레임 번호 외에 유효 비트, 보호 비트, 참조 비트, 수정 비트가 있음

**유효 비트**

해당 페이지에 접근 가능한지 여부를 알려준다.
프레임 번호 다음으로 중요한 정보.

일반적으로 페이징 스와핑으로 인해 모든 페이지가 메모리에 있지 않음.
유효 비트는 현재 페이지가 메모리에 적재되어 있는지 아니면 보조기억 장치에 있는지를 알려주는 비트.
있다면 비트가 1, 없다면 0

만약 유효 비트가 0인 메모리에 CPU가 접근하려고 하면 **페이지 폴트** 라는 예외가 발생함.

### 페이지 폴트 처리 과정

1. CPU는 기존 작업을 백업
2. 페이지 폴트 처리 루틴 실행
3. 페이지 처리 루틴은 원하는 페이지를 메모리로 가져 온 뒤 유효 비트를 1로 변경해 준다.
4. 페이지 폴트를 처리했다면 CPU는 해당 페이지 접근 가능.

**보호 비트**

페이지 보호 기능을 위해 존재하는 비트
해당 페이지가 읽고 쓰기가 모두 가능한 페이지인지, 아니면 둘 중 하나만 가능한 페이지인지를 나타낼 수 있다. 0일 경우 읽기만, 1이면 둘 다 가능.

읽기 전용 영역 -> 프로세스를 이루는 요소 중 코드 영역

읽기 전용 페이지에 쓰기를 시도하는 것을 막기 위함.

보호 비트는 읽기를 나타내는 r
쓰기를 나타내는 w
실행을 나타내는 x
세 개의 비트로 좀 더 복잡하게 구현 가능

110 은 읽고 쓰기 가능. 실행 불가
111은 모두 가능

**참조 비트**

CPU가 이 페이지에 접근한 적이 있는지 여부를 나타낸다.

적재 이후 CPU가 읽거나 쓴 페이지는 참조 비트가 1로 세팅, 한 번도 없었다면 0.

**수정 비트**

해당 페이지에 데이터를 쓴 적이 있는지 없는지 수정 여부를 알려준다.
더티 비트라고도 부름. 변경된 적이 있다면 1, 없다면 0.

수정 비트는 페이지가 메모리에서 사라질 때 보조기억장치에 쓰기 작업을 해야하는 지 판단하기 위해 존재.
한 번도 수정된 적 없는 페이지(접근한 적 없거나, 읽기만 한 페이지)가 스왑 아웃될 경우 아무런 추가 작업 없이 새로 적재된 페이지로 덮어쓰기만 하면 됨. 똑같은 페이지가 보조기억장치에 저장되어 있기 때문.

CPU가 쓰기 작업을 수행한 경우 보조기억장치에 저장된 페이지 내용과 메모리에 저장된 내용은 서로 다른 값을 갖게되기 때문에 스왑 아웃 될 경우
보조 기억 장치에 기록하는 작업이 추가 되어야 함. 이 작업이 필요한지 아닌지 판단하기 위해 수정 비트가 필요.

실제 페이지 테이블 엔트리에는 이외에도 다양한 정보가 있지만 많은 전공서는 여기까지만 설명함.

### 페이징의 이점 - 쓰기 시 복사

외부 단편화 문제 해결 이외에도 페이징이 제공하는 이점은 다양하다.

대표적인 것은 프로세스 간에 페이지를 공유할 수 있다는 점. 공유하는 사례는 다양하지만
대표적인 것이 쓰기 시 복사

![Untitled](Chapter14%20VirtualMemory%2051a124707a454a13a76350b01087f89d/Untitled%204.png)

유닉스나 리눅스와 같은 운영체제에서 fork 시스템을 호출하면 부모 프로세스의 복사본이 자식 프로세스로서 만들어지게 된다.

프로세스 간에는 자원을 공유하지 않으므로 새롭게 생성된 자식 프로세스의 코드 및 데이터 영역은 부모 프로세스가 적재된 메모리 공간과는 전혀 다른 메모리 공간이 생성된다. 이 복사 작업은 프로세스 생성 시간을 늦출 뿐만 아니라 불필요한 메모리 낭비를 야기한다.

![Untitled](Chapter14%20VirtualMemory%2051a124707a454a13a76350b01087f89d/Untitled%205.png)

부모 프로세스와 동일한 자식 프로세스가 생성되면 자식 프로세스로 하여금 부모 프로세스와 동일한 프레임을 가리킨다. 이로써 굳이 메모리 공간을 복사하지 않고도 그저 읽기 작업만 이어 나간다면 이 상태가 지속된다.

프로세스 간에는 자원을 공유하지 않으므로 부모 프로세스 혹은 자식 프로세스 둘 중 하나가 이 페이지에 쓰기 작업을 하면 그 순간 해당 페이지가 별도의 공간으로 복제된다. 각 프로세스는 자신의 고유한 페이지가 할당된 프레임을 가리킨다. 이것이 쓰기 시 복사.

이러한 쓰기 시 복사를  통해 프로세스 생성 시간을 줄이고, 메모리 공간 절약도 가능하다.

### 계층적 페이징

![Untitled](Chapter14%20VirtualMemory%2051a124707a454a13a76350b01087f89d/Untitled%206.png)

- 페이지 테이블의 크기는 생각보다 작지 않다. 프로세스의 크기가 커지면 비례해서 커짐.
프로세스를 이루는 모든 페이지를 테이블 엔트리를 메모리에 두는 것은 큰 메모리 낭비.
이를 해결하는 방법이 계층적 페이징
- 페이지 테이블을 페이징하여 여러 단계의 페이지를 두는 방식. 프로세스의 페이지 테이블을 여러 개의 페이지로 자르고, 바깥쪽에 페이지 테이블을 하나 더 두어 잘린 페이지 테이블의 페이지들을 가리키게 하는 방식.
- 이렇게 계층적으로 구성하면 모든 페이지 테이블을 항상 메모리에 유지할 필요가 없음.
다만 CPU와 가장 가까이 위치한 페이지테이블(Outer Page Table)은 항상 메모리에 유지해야됨.
- 계층적 페이징을 사용하는 경우 논리 주소도 달라진다.
    
    사용하지 않을 경우 - 변위, 페이지 번호.
    사용할 경우 - 바깥 페이지 번호, 안쪽 페이지 번호, 변위
    

**접근 방식**

1. 바깥 페이지 번호를 통해 페이지 테이블의 페이지를 찾기
2. 페이지 테이블의 페이지를 통해 프레임 번호를 찾고 변위를 더함으로서 물리 주소 얻기

다만 페이지 테이블의 계층이 늘어날수록 페이지 폴트가 발생했을 경우 메모리 참조 횟수가 많아지므로 반드시 좋다고 볼 수는 없다.

## 페이지 교체와 프레임 할당

- 가상 메모리를 통해 작은 물리 메모리보다 큰 프로세스도 실행할 수 있다고 하지만,
그럼에도 불구하고 여전히 물리 메모리의 크기는 한정되어 있다.
- 운영체제는 프로세스들이 한정된 메모리를 효율적으로 이용할 수 있도록 기존에 메모리에 적재된 불필요한 페이지를 선별하여 보조기억장치로 내보낼 수 있어야 하고(스왑 아웃), 프로세스들에 적절한 수의 프레임을 할당하여 페이지를 할당할 수 있게 해야한다.

### 요구 페이징

페이지가 필요할 때에만 메모리에 적재하는 기법.

1. CPU가 특정 페이지에 접근 하는 명령어를 실행.
2. 해당 페이지가 현재 메모리에 있을 경우(유효 비트 1) 페이지가 적재된 프레임에 접근
3. 해당 페이지가 현재 메모리가 없을 경우(유효 비트 0) -> 페이지 폴트 발생
4. 페이지 폴트 처리 루틴은 해당 페이지를 메모리로 적재하고 유효 비트를 1로 설정
5. 다시 1번부터 수행 반복

**순수 요구 페이징 기법**
아무런 페이지도 메모리에 적재하지 않은 채 무작정 실행부터 할 수 있다. 이 경우 실행하는 순간부터 페이지 폴트가 계속 발생하고
실행에 필요한 페이지가 어느정도 적재된 이후부터는 페이지 폴트 발생이 떨어진다.

요구 페이징 시스템이 안정적으로 작동하려면 페이지 교체와 프레임 할당이 안정적으로 작동하게 해야됨.

요구 페이징 기법으로 페이지들을 적재하다보면 언젠가 메모리가 가득 차게된다.
이 때는 당장 실행에 필요한 페이지를 적재하기 위해 메모리에 적재된 페이지를 보조기억장치로 내보내야 한다.
쫓아낼 페이지를 결정하는 방법이 **페이지 교체 알고리즘.**

### 페이지 교체 알고리즘

- 좋은 페이지 교체 알고리즘이란? 일반적으로 페이지 폴트를 가장 적게 일으키는 알고리즘
페이지 폴트가 발생하면 느려짐. -> 컴퓨터 성능 저하 -> 이를 방지해야 함.
- 페이지 교체 알고리즘을 제대로 이해하려면 페이지 폴트 횟수를 알 수 있어야 한다.
페이지 폴트 횟수는 페이지 참조열을 통해 알 수 있다.

**페이지 참조열이란?**
CPU가 참조하는 페이지들 중 연속된 페이지를 생략한 페이지열을 의미.
2 2 2 3 5 5 5 3 3 7 -> 2 3 5 3 7

연속된 페이지를 생략하는 이유 -> 페이지 폴트를 발생 시키지 않음.

### FIFO 페이지 교체 알고리즘

![Untitled](Chapter14%20VirtualMemory%2051a124707a454a13a76350b01087f89d/Untitled%207.png)

메모리에 가장 먼저 올라온 페이지부터 내쫓는 방식. "오래 머물렀다면 나가라."

아이디어와 구현이 간단하지만, 마냥 좋은 것은 아님.

프로그램 실행 내내 사용될 내용을 포함한 페이지의 경우, 쫓아내면 안됨.

이를 조금 개선하기 위해 2차 기회 페이지 교체 알고리즘도 있음.
교체 알고리즘을 통해 가장 오래된 페이지에 접근 했을 때 참조 비트가 1이라면 0으로 변경을 해 한 번 더 기회를 준 페이지로 판별
이후 비트가 0인 상태인 채로 다시 접근하면 내쫓음.

### 최적 페이지 교체 알고리즘

CPU에 참조되는 횟수를 고려하는 페이지 교체 알고리즘.
메모리에 오랫동안 남아야하는 페이지는 자주 사용되는 페이지.
앞으로의 사용 빈도가 가장 낮을 것 같은 페이지를 예측해 교체하는 알고리즘.

가장 낮은 페이지 폴트율을 보장하는 알고리즘.
실제 구현이 어렵다. 왜? 사용 빈도 예측 알고리즘 구현이 어렵기 때문.
따라서 운영체제에서 사용하기 보다는, 주로 다른 페이지 교체 알고리즘의 이론상 성능을 평가하기 위한 목적으로 사용됨.

### LRU 페이지 교체 알고리즘

![Untitled](Chapter14%20VirtualMemory%2051a124707a454a13a76350b01087f89d/Untitled%208.png)

가장 오랫동안 사용되지 않은 페이지를 교체하는 알고리즘.

최근에 사용되지 않은 페이지는 앞으로도 사용되지 않을 것이라는 아이디어를 토대로 만들어진 알고리즘.
페이지마다 마지막으로 사용한 시간을 토대로 최근에 가장 사용이 적었던 페이지를 교체한다.

이외에도 페이지 교체 알고리즘은 다양하다. 앞서 설명한 것은 대표적인 것들. 특히 LRU는 파생이 다양하다.

### 스래싱과 프레임 할당

페이지 폴트가 자주 발생하는 이유에는 나쁜 페이지 교체 알고리즘의 사용만 있는 것은 아니다.
프로세스가 사용할 수 있는 프레임 수가 적어도 페이지 폴트는 자주 발생한다. -> 더 근본적인 이유

CPU가 쉴새 없이 프로세스를 실행해야 컴퓨터의 전체 생산성도 올라갈텐데, 페이지 교체에 너무 많은 시간을  쏟으면 당연히 성능에도 큰 악영향이 초래된다.

 

프로세스가 실제 실행되는 시간 < 페이징에 소요되는 시간 → 성능이 저해되는 문제 

이를 **스레싱**이라고 한다.

## 스레싱

지나치게 빈번한 페이지 교체로 인해 CPU 이용률이 낮아지는 문제. -> 스레싱

동시에 실행되는 프로세스의 수를 늘린다고 해서 CPU 이용률이 그에 비례해서 증가하는 것은 아님.

- 동시에 실행되는 프로세스 수가 어느정도 증가함 -> CPU 이용률이 높아지긴 함.
- 대신 필요 이상으로 늘리면 각 프로세스들이 사용할 수 있는 프레임 수가 적어짐.
- 페이지 폴트가 지나치게 빈번하게 발생.
- 다시 CPU 이용률이 떨어지게 됨.

아무리 CPU 성능이 뛰어나도 동시에 실행되는 프로세를 수용할 물리 메모리가 너무 작다면 전체 컴퓨터 성능이 안좋아지는 이유.

스레싱이 발생하는 근본적인 원인?
각 프로세스가 필요로하는 최소한의 프레임 수가 보장되지 않기 때문이다.

그렇기에 운영체제는 각 프로세스들이 무리없이 실행 하기 위한 최소한의 프레임 수를 파악하고 적절한 수 만큼 프레임을 할당해 줄 수 있어야 함.

### 프레임 할당 방식

**균등할당**
모든 프로세스에 균등하게 프레임을 제공하는 방식
-> 실행되는 프로세스들의 크기는 각기 다름. 비 합리적.

**비례할당**
프로세스의 크기가 크면 많이 할당하고, 적으면 적게 할당함.
-> 크기가 크더라도 막상 실행해보니 많은 프레임을 필요로 하지 않는 경우도 있다. 그 반대의 경우도 있음.

결국 실행을 해봐야 아는 경우가 많다.

프로세스를 실행하는 과정에서 배분할 프레임을 결정하는 방식에는 크게 **작업 집합 모델**을 사용하는 방식과 **페이지 폴트 빈도**를 사용하는 방식이 있다.

### 작업 집합 모델 기반 프레임 할당 방식

프로세스가 일정 기간 동안 참조한 페이지 집합을 기억하여 빈번한 페이지 교체를 방지.

실행 중인 프로세스가 일정 시간 동안 참조한 페이지의 집합을 작업 집합이라고 한다.
CPU가 과거에 주로 참조한 페이지를 작업 집합에 포함한다면 운영체제는 작업 집합의 크기만큼만 프레임을 할당해주면 된다.

### 페이지 폴트 빈도를 기반 한 프레임 할당 방식

1. 페이지 폴트율이 너무 높으면 그 프로세스는 너무 적은 프레임을 갖고 있다.
2. 페이지 폴트율이 너무 낮으면 그 프로세스는 너무 많은 프레임을 갖고 있다.

페이지 폴트율에 상한선과 하향선을 정하고, 이 범위 안에서만 프레임을 재할당하는 방식.