# 13강 교착 상태

## 식사하는 철학자 문제

동그란 원탁에 5명의 철학자, 식사 5개, 포크 5개

포크 2개 잡을 때만 식사 가능

모든 철학자가 동시에 왼쪽 포크 잡으면 어떤 철학자도 식사 X, 영원히 생각함

→모든 철학자는 다른 철학자가 포크를 내려놓을 때까지 기다림

## 교착 상태

일어나지 않을 사건을 기다리며 진행이 멈춰 버리는 현상

철학자: 프로세스, 스레드

포크: 자원, 한 번에 하나의 프로세스나 스레드만 접근 가능한 임계 구역

생각하는 행위: 자원을 기다림

## 교착 상태 발생 조건

- 상호 배제
    - 해당 자원을 한 번에 하나의 프로세스만 이용 가능
- 점유와 대기
    - 자원을 할당받은 상태에서 다른 자원을 할당받기를 기다림
- 비선점
    - 어떤 프로세스도 다른 프로세스의 자원을 강제로 빼앗지 못 함
- 원형 대기
    - 자원 할당 그래프가 원의 형태

---

## 교착 상태 예방

1. 자원의 상호 배제 없애기
    1. 모든 자원 공유 가능하게? → 무리
2. 점유와 대기 없애기
    1. 한 프로세스에 필요한 자원 몰아줌. 
    2. 단점: 자원을 활용률 낮아짐. 
    3. 당장 자원이 필요해도 기다릴 수 밖에 없거나 사용되지 않으면서 오랫동안 할당되는 자원도 생김.
    4. 많은 자원 사용하는 프로세스가 불리함 → 기아 현상(무한정 기다림)
3. 비선점 없애기
    1. 이용 중인 프로세스로부터 자원 빼앗기
    2. 모든 자원이 선점 가능하지는 않다. 
    3. 단점: 범용성 낮음
4. 원형 대기 없애기
    1. 모든 자원에 번호 붙이고 오름차순으로 자원 할당
    2. 원형 식탁이 아닌 사각형 식탁에서 일렬로 앉아서 식사
    3. 단점: 수많은 자원에 번호 붙이는 건 어렵고, 특정 자원의 활용률 떨어질 수 있음

## 교착 상태 회피

한정된 자원의 무분별한 할당으로 인해 발생하는 문제 해결

안전 상태를 유지할 수 있는 경우에만 자원을 할당

- 안전 상태: 교착 상태 발생하지 않고 모든 프로세스가 정상적으로 자원 할당받고 종료될 수 있는 상태
- 불안전 상태: 교착 상태 발생 가능, 안전 순서열 없는 상황
- 안전 순서열: 교착 상태 없이 안전하게 프로세스들에 자원 할당할 수 있는 순서
    - 예시) 웹 브라우저 → 메모장 → 게임

## 교착 상태 검출 후 회복

자원 요구하면 그때그때 모두 할당, 교착 상태 발생 여부를 주기적으로 검사. 

교착 상태 검출되면 회복

1. 선점을 통한 회복
    1. 교착 상태 해결될 때까지 한 프로세스씩 자원 몰아주기
2. 프로세스 강제 종료를 통한 회복
    1. 교착 상태 놓인 프로세스 모두 강제 종료
        1. 한 번에 해결 가능, 많은 프로세스가 작업 내용 잃음
    2. 해결될 때까지한 프로세스씩 강제 종료
        1. 작업 내역 잃는 건 최소화, 교착 상태 없어졌는지 확인하는 과정에서 오버헤드 야기함

타조 알고리즘: 교착 상태 무시하는 방법
