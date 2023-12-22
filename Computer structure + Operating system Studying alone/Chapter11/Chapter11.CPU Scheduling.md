CPU 스케줄링
=============
  CPU 스케줄링 개요
  -------------
  
  CPU 스케줄링 : 모든 프로세스가 필요로 하는 CPU 자원의 사용을 배분하는 일
    - 간단하게 말하자면, 프로세스의 우선 순위를 정하는 일.

  프로세스 우선순위
    - IO Bound Process : 입출력 작업(IO Burst)의 비중이 높은 프로세스
    - CPU Bound Process : CPU 작업(CPU Burst)의 비중이 높은 프로세스
    - CPU Bound Process는 CPU 작업량, 즉 점유율이 높아 병목현상을 일으킬 수 있음.
    - 그렇기에 IO Bound Process가 CPU Bound Process보다 우선순위를 가짐.
    - 우선순위는 운영체제가 결정하고, PCB에 명시함.
    - ※ Bound: certain or extremely likely to happen

  스케줄링 큐
    - 프로세스의 메모리 적재 요청 순서에 따라 생성된 큐
      - 메모리 적재 순서의 큐
    - Ready Queue : 준비 상태의 프로세스가 대기하는 큐
    - Blocked Queue : 대기 상태의 프로세스가 대기하는 큐

  선점형 & 비선점형 스케줄링
    - 선점형 스케줄링 : 타이머 인터럽트가 발생 전 까지 프로세스가 메모리를 점유
    - 비선점형 스케줄링 : 하나의 프로세스가 종료되기 전까지 해당 프로세스가 메모리를 점유

  CPU 스케줄링 알고리즘
  -------------
    
  스케줄링 알고리즘 종류
    - FCFS(First Come First Scheduling) 스케줄링 : Ready Queue에 삽입된 순서대로 프로세스 처리.
      - 대기시간이 길어질 수 있는 단점이 존재 (호위 효과 (Convoy Effect))
      
    - SJF(Shortest Job First) 스케줄링 : 메모리 점유 시간이 가장 짧은 프로세스부터 우선적으로 처리.
      
    - Round Robin 스케줄링 : FCFS 스케줄링에 타임 슬라이스 개념을 삽입한 스케줄링 방법.
      - 타임 슬라이스란 프로세스가 메모리를 점유할 수 있는 최대 시간을 의미함.
      
    - SRT(Shortest Remaining Time) 스케줄링 : 정해진 타임 슬라이스동안 프로세스의 처리가 이루어 진 뒤, 이후 잔여 시간이 가장 짧은 프로세스를 처리.

    - Priority 스케줄링 : 프로세스에 우선순위를 부여하고, 해당 순위대로 처리.
        - SJF, SRT는 모두 Priority 스케줄링의 종류임.
        - 해당 스케줄링 방법은 하나의 프로세스가 처리되지 않고 장기간 대기하는 기아 현상이 발생할 수 있음
          - 이를 해결하기 위해 대기할 수록 우선순위를 조정하는 에이징 기법이 사용됨.
      
    - Multilevel Queue 스케줄링 : Priority 스케줄링에서 각 우선순위 별로 별도의 큐를 생성하여 처리.
        - 큐마다 다른 규칙, 다른 알고리즘이 적용가능.

    - Multilevel Feedback Queue 스케줄링 : Multilevel Queue 스케줄링에 에이징 기법을 적용하여 처리하는 방법.
    
