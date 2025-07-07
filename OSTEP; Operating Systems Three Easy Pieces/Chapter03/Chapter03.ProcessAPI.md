## Unix 시스템의 프로세스 생성
**fork(), exec() 시스템 콜 사용
생성한 프로세스의 종료를 기다릴 때 wait() 시스템 콜**

### fork() 시스템 콜
* Unix 시스템에서 프로세스를 생성할 때 사용하는 시스템 콜
* 호출한 프로세스를 복제해 새로운 프로세스 생성
* 원래의 프로세스를 부모 프로세스, 새로 생긴 프로세스를 자식 프로세스
* **fork() 호출 이후의 변화**
  * fork() 호출 직후에는 동일한 코드, 동일한 스택 상태, 동일한 힙/데이터/코드 영역을 가지는 두 개의 프로세스가 존재
  * 단, 서로 독립적인 주소 공간 (다른 PID)
  * 자식 프로세스는 fork()를 호출하면서부터 시작 (main 함수의 처음부터 실행하지 않음
  * 부모 프로세스 : fork()의 반환값으로 자식의 PID를 받음
  * 자식 프로세스 : fork()의 반환값으로 0을 받음
  * 반환값을 기준으로 if/else문 등을 통해 서로 다른 코드를 실행하도록 만들 수 있음
* **예시 코드**
  ```
  #include <stdio.h>
  #include <stdlib.h>
  #include <unistd.h>
  int main(int argc, char *argv[])
  {
  	printf("hello world (pid:%d)\n", (int)getpid());
  	int rc = fork();
  	if (rc < 0)
  	{
  		// fork 실패
      	fprintf(stderr, "fork failed\n");
      	exit(1);
  	}
  	else if (rc == 0)
  	{
  		// 자식 프로세스
      	printf("hello, I am child (pid:%d)\n", (int)getpid());
  	}
  	else
  	{
  		// 부모 프로세스
      	printf("hello, I am parent of %d (pid:%d)\n", rc, (int)getpid());
  	}
  	return 0;
  }
  ```
  * 출력 예시
  ```
  hello world (pid:29146)
  hello, I am parent of 29417 (pid:29416)
  hello, I am child (pid:29417)
  ```
  * 위의 출력 순서는 항상 같지 않음
  * CPU 스케줄러가 어떤 프로세스를 먼저 실행하느냐에 따라 달라짐
* **비결정성 (Nondeterminism)**
  * fork() 이후 부모와 자식 프로세스 중 어떤 프로세스가 먼저 실행될지는 보장되지 않음
  * CPU 스케줄러의 정책에 따라 변동
  * 이러한 비결정성은 멀티쓰레드 프로그램에서 경쟁 조건(Race Condition) 등 다양한 문제의 원인

### wait() 시스템 콜
* 부모 프로세스가 자식 프로세스의 종료를 기다릴 때 사용하는 시스템 콜
* 더 많은 기능을 가진 waitpid()도 있음
* 자식이 종료되면 wait()는 자식의 PID를 반환하며 리턴
* 자식이 없거나 오류 발생 시 -1 반환
* 자식 프로세스가 종료되기 전까지 부모는 블로킹(대기) 상태
  * waitpid()는 블로킹/논블로킹 선택 가능 (WNOHANG 옵션)
* **예시 코드**
  ```
  #include <stdio.h>
  #include <stdlib.h>
  #include <unistd.h>
  #include <sys/wait.h>
  int main(int args, char *argv[])
  {
  	printf("hello world (pid:%d)\n", (int)getpid());
  	int rc = fork();
  	if (rc < 0)
  	{
  		// fork 실패
      	fprintf(stderr, "fork failed\n");
      	exit(1);
  	}
  	else if (rc == 0)
  	{
  		// 자식 프로세스
      	printf("hello, I am child (pid:%d)\n", (int)getpid());
  	}
  	else
  	{
  		// 부모 프로세스
      	int wc = wait(NULL);
      	printf("hello, I am parent of %d (wc:%d) (pid:%d)\n", rc, wc, (int) getpid());
  	}
  	return 0;
  }
  ```
  * 출력 예시
  ```
  hello world (pid:29266)
  hello, I am child (pid:29267)
  hello, I am parent of 29267 (wc:29267) (pid:29266)
  ```
  * 항상 자식 프로세스의 출력이 먼저 나타남
  * 부모가 먼저 실행되더라도 wait() 호출로 인해 자식이 끝날 때까지 대기
* **사용 이유**
  * 부모가 wait() 없이 종료되면 자식은 고아 프로세스(Orphan)가 되어 init 프로세스가 대신 부모 역할 수행
    * init 프로세스 : 유닉스 운영체제의 부팅 과정에 생성되는 최초의 프로세스, 시스템 종료까지 계속 살아있는 데몬 프로세스
  * 반대로 자식이 종료되었는데 부모가 wait()를 호출하지 않으면 자식은 좀비 프로세스(Zombie) 상태로 남아 리소스 차지
    * 낭비되는 리소스 발생, PID/프로세스 종료 상태 등
  * wait()는 자식 프로세스가 종료된 후 해당 자원의 정리를 도와주는 호출
  
### exec() 시스템 콜
* 현재 실행 중인 프로세스를 완전히 다른 프로그램으로 대체
* 기존의 코드, 데이터, 힙, 스택은 모두 사라짐, 새로운 프로그램의 코드와 메모리로 덮어씌움
* 새로운 프로세스를 생성하지 않음, 기존 프로세스의 변경
* 성공 시 리턴하지 않음 (실패 시 -1 리턴)
* **예시 코드**
  ```
  #include <stdio.h>
  #include <stdlib.h>
  #include <unistd.h>
  #include <string.h>
  #include <sys.wait.h>
  int main(int argc, char *argv[])
  {
  	printf("hello world (pid:%d)\n", (int)getpid());
  	int rc = fork();
  	if (rc < 0)
  	{
  		// fork 실패
  		fprintf(stderr, "fork failed\n");
  		exit(1);
  	}
  	else if (rc == 0)
  	{
  		printf("hello, I am child (pid:%d)\n", (int)getpid());
  		char *myargs[3];
  		myargs[0] = strdup("wc");				// wc 프로그램 (단어 세기)
  		myargs[1] = strdup("p3.c");				// 인자 : 단어 파일
  		myargs[2] = NULL;						// 배열의 끝 표시
  		execvp(myargs[0], myargs);				// wc p3.c 실행
  		printf("this shouldn't print out");		// 실행 안 됨
  	}
  	else
  	{
  		// 부모 프로세스
  		int wc = wait(NULL);
  		printf("hello, I am parent of %d (wc:%d) (pid:%d)\n", rc, wc, (int)getpid());
  	}
  	return 0;
  }
  ```
  * 출력 예시
  ```
  hello world (pid:29383)
  hello, I am child (pid:29384)
      29    107    1030    p3.c
  hello, I am parent of 29384 (wc:29384) (pid:29383)
  ```
  * 실행 순서
    * 부모 프로세스가 fork()로 자식 생성
    * 자식 프로세스는 exec()로 wc 프로그램 실행
    * 기존 코드(p3)는 사라지고 새로운 프로그램(wc)이 실행
    * exec() 성공 시 리턴 안 함 (기존 프로그램 코드 실행 안 함)
    * 부모는 wait()로 자식의 종료 대기

### 위의 API 사용 이유
* fork()와 exec()는 의도적으로 분리된 설계
* 쉘 같은 프로그램이 exec() 전에 유용한 환경 설정 및 기능을 준비할 수 있게 하기 위함
* 출력 리디렉션, 파이프, 백그라운드 실행 등의 기능 구현 가능
* **쉘이 프로세스를 다루는 방식**
  1. 사용자 입력 대기 (ex wc p3.c)
  2. 입력을 분석해 실행할 프로그램과 인자 분리
  3. 자식 프로세스를 fork()로 생성
  4. 자식 프로세스에서 exec()의 변형 중 하나를 호출해 프로그램 실행
  5. 부모 프로세스는 wait()로 자식 종료 대기
  * 이 흐름에서 fork()와 exec()이 분리되어 있으므로 쉘이 exec() 전에 필요한 전처리 가능
* **예시 코드**
  * 출력 리디렉션
    ```
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <string.h>
    #include <fcntl.h>
    #include <sys/wait.h>
    int main(int argc, char *argv[])
    {
    	int rc = fork();
        if (rc < 0)
        {
        	// fork 실패
            fprintf(stderr, "fork failed\n");
            exit(1);
        }
        else if (rc == 0)
        {
        	// 자식 프로세스
            close(STDOUT_FILENO);		// 1번 파일 디스크립터(표준 출력) 닫기
            open("./p4.output", O_CREAT|OWRONLY|O_TRUNC, S_IRWXU);
            // STDOUT_FILENO가 닫혀 있으므로, open이 1번 디스크립터를 사용함
            
            // exec으로 wc p4.c 실행
            char *myargs[3];
            myargs[0] = strdup("wc");
            myargs[1] = strdup("p4.c");
            myargs[2] = NULL;
            execvp(myargs[0], myargs);
        }
        else
        {
        	// 부모 프로세스
            int wc = wait(NULL);		// 자식 프로세스 종료 대기
        }
        return 0;
    }
    ```
    ```
    prompt > wc p3.c | newfile.txt
    ```
    * wc p3.c를 실행해 화면(stdout) 대신 newfile.txt에 출력
    * fork()로 자식 생성
    * 자식에서 exec() 전에 표준 출력 파일을 닫고 open("newfile.txt) 호출
    * 이 과정에서 표준 출력이 파일로 변경
    * 이후 exec("wc", ["wc", "p3.c"]) 실행
    * fork()와 exec()의 분리로 exec 이전에 자식에서 환경 설정 가능
  * 파이프 명령어
    ```
    prompt > grep foo file.txt | wc -1
    ```
    * grep foo file.txt 명령어 출력이 wc -1 명령의 입력으로 전환
    * pipe() 시스템 콜로 파이프 연결 생성
    * fork() 두 번 (두 개의 자식 생성)
    * 첫번째 자식은 출력을 파이프로, 두번째 자식은 입력을 파이프로 설정

### 여타 다른 API들
**시스템 모니터링, 디버깅, 안정성 유지에 중요**
* kill() 시스템 콜
  * 프로세스에 시그널(Signal)을 보냄
    * 시그널 : OS가 프로세스에게 외부 이벤트를 전달하는 방법, 인터럽트와 비슷 (프로세스 실행 중단 및 특정 처리 진행)
  * 프로세스 종료 요청(SIGKILL, SIGTERM), 일시 중지(SIGSTOP). 계속 실행(SIGCONT)
  ```
  int kill(pid_t pid, int sig);
  int kill(1234, SIGKILL);		// PID가 1234인 프로세스 강제 종료
  ```
* ps 명령어
  * 현재 실행 중인 프로세스 목록 출력
* top 명령어
  * 시스템의 실시간 프로세스 정보 및 리소스 사용률 표시
  * CPU, 메모리 사용률 등 확인 가능
  * 여러 번 실행할 경우 자기 자신이 가장 많은 자원을 사용한다 지적
* CPU 부하 측정 도구
  * MenuMeter (Raging Menace Software)
  * 어느 때건 CPU 이용률 점검 가능
***
## 사용된 C 헤더파일
* **stdio.h**
  * Standard Input/Output Library 헤더
  * 콘솔 입출력 함수 제공
  * printf(), scanf() : 출력, 입력
  * fopen(), fread(), fwrite(), fclose() : 파일 입출력
* **stdlib.h**
  * Standard Library 헤더
  * 메모리 할당, 프로그램 종료, 문자열/숫자 변환
  * malloc(), free() : 동적 메모리 관리
  * exit() : 프로그램 종료
  * atoi(), atof(), strtol() : 문자열을 숫자로 변환
* **unistd.h**
  * POSIX 운영체제 API 제공
  * 시스템 호출 (System Call) 관련 함수
  * fork() : 프로세스 생성
  * exec() 계열 : 새로운 프로그램 실행
  * read(), write() : 저수준 입출력
  * pipe(), sleep(), getpid() 등 제공
* **string.h**
  * 문자열 처리 함수 제공
  * C 문자열 연산
  * strlen(), strcpy(), strcat(), strcmp() 등 제공
* **fcntl.h**
  * 파일 제어 (File Control) 옵션 제공
  * 주로 open() 같은 함수에 사용되는 상수 정의
  * O_RDONLY(읽기 전용), O_WRONLY(쓰기 전용), O_RDWR(읽기, 쓰기)
  * O_CREAT(파일이 없다면 새로 생성), O_TRUNC(기존 내용 전부 삭제)
* **sys/wait.h**
  * 자식 프로세스 상태 관리 헤더
  * wait(), waitpid() 함수 및 관련 매크로 제공
  * WIFEXITED(), WEXITSTATUS() : 자식 종료 상태 검사
  * WIFSIGNALED(), WTERMSIG() : 시그널로 종료되었는지 확인

## 디스크립터 (Descriptor)
* 자원을 식별하고 관리하기 위해 사용하는 정수형 ID
* 파일뿐만 아니라 파이프, 소켓, 터미널 등 모든 I/O 자원을 추상화
* **파일 디스크립터 (File Descriptor)**
  * 열려 있는 파일이나 입출력 자원을 관리하기 위해 부여하는 고유한 정수 값
  ```
  int fd = open("file.txt", O_RDONLY);
  ```
  * 위에서 fd는 file.txt를 가리키는 파일 디스크립터
  * 이 정수(fd) 통해 read/write 등의 시스템 콜 접근 가능
  * 파일 디스크립터의 번호 규칙
    * 표준 입력 (stdin) : 0
    * 표준 출력 (stdout) : 1
    * 표준 에러 (stderr) : 2
    * 추가로 열리는 파일이나 소켓은 보통 3부터 할당
* 디스크립터를 통해 파일 내용과 상관 없이 내부 테이블에 접근 및 관리
* 복잡한 파일 정보 대신 정수 하나만 기억하면 됨 (추상화된 자원 관리 방식)

***
_출처 : OSTEP (Operating Systems : Three Easy Pieces)
[unistd.h — Implementation-specific functions](https://www.ibm.com/docs/en/zos/3.1.0?topic=files-unistdh-implementation-specific-functions)
[fcntl.h](https://www.ibm.com/docs/ko/aix/7.2.0?topic=files-fcntlh-file)
[C stdlib (stdlib.h) Library](https://www.w3schools.com/c/c_ref_stdlib.php)
_