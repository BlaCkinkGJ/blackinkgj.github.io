---
layout: post
title: "[유닉스] fork와 exec"
date: 2019-3-12
excerpt: "fork와 exec에 관해서 알아보도록 한다."
tag:
- UNIX
comments: true
---
# fork와 exec

## 들어가기 전에

먼저 셸(shell)도 하나의 프로세스라는 것을 알아야 합니다.

유닉스 체계에서 `ps` 명령어를 치면 프로세스 리스트가 나옴을 알 수 있습니다. 그리고 `ps -al`을 치면 PID와 PPID가 나오는 데 각각은 Process ID와 Parent Process ID를 의미합니다. 이러한 `ps` 명령어의 위치를 확인하기 위해서 `whereis ps`를 쳐주시길 바랍니다. 그러면 다음과 같은 창이 나옴을 알 수 있습니다.

![where is ps](/assets/img/res/2019-UnixAdvanced/2/whereisps.PNG)

즉, 이런 `ps` 명령어도 실행을 되기 위해서는 `/ → bin → ps`의 디렉터리를 찾아가서 메모리에 불러오는 과정이 있어야 함을 알 수 있습니다.

그리고 이번에는 `cd /sbin` 이후에 `ls i*`을 한 후에 `init`이라는 프로그램을 찾아주십시오. 이 프로세스는 유닉스 시스템의 첫 번째 프로세스로서 이 프로세스는 죽으면 안 되기에 무한 루프로 만들어집니다.

이러한 배경 지식을 이야기 하는 이유는 유닉스의 모든 프로세스는 `init` 프로그램을 기반으로 하여 `fork()` 및 `exec()`이 발생하기 때문입니다.

 ![init process의 동작](http://teaching.csse.uwa.edu.au/units/CITS2002/lectures/lecture10/images/proctree.jpg)

이것이 바로 유닉스 시스템이 안정적으로 관리될 수 있는 핵심 사항이라고 할 수 있습니다.

## exec()

먼저 `exec`의 동작에 관해서 알아보도록 하겠습니다. 아래의 예제 코드를 작성하여 `gcc`로 컴파일을 해주시길 바랍니다.

```c
#include<stdio.h>
#include<unistd.h>

int main(int argc, char* argv[]){
    printf("argc: %d\n", argc);
    printf("file name: %d\n", argv[0]);
    printf("%s %s\n", argv[1], argv[2]);
    execl("/bin/ls", "/bin/ls", "-al", "/tmp", NULL);
}
```

이러한 프로그램에 `a.out a b`라고 입력을 주는 경우에는 아래와 같은 결과를 보여줄 것입니다.

```sh
argc: 3
file name: a.out
a b
# 이후는 /tmp 디렉토리에서의 ls -al의 결과가 나옴
```

여기서 `argc`는 매개변수의 개수를 반환을 해주게 됩니다. 다음으로 `argv[0]`는 현재 실행 중인 파일을 이야기하고 `argv[1]`과 `argv[2]`는 각각 `a.out` 뒤에 온 인자들에 해당합니다.

그리고 `execl`이라고 있습니다. 이에 관해서 좀 더 알아보자면 `execl`은  `int execl(const char *path, const char *arg, ...);`로 선언됩니다. 따라서 실행할 파일의 경로(`/bin/ls`)와 `arg[0]`부터 `NULL`을 만날 때까지를 넣어주는 데, 여기서 `argv[0]`는 위에서도 보이듯이 실행 파일이 들어가야 하므로 `/bin/ls` 또는 `ls`를 넣어주도록 합니다. 이후에는 우리가 쓰는 그대로 쓰게 됩니다. 결과적으로는 `ls -al /tmp`를 셸에서 수행한 값이 나오게 됩니다.

결과적으로, 위와 같은 결론이 나옴을 알 수 있습니다.

이번에는 `execl` 의 위치를 조금 움직여보도록 하겠습니다. 다음과 같은 코드를 짜주시길 바랍니다.

```c
#include<stdio.h>
#include<unistd.h>

int main(int argc, char* argv[]){
    execl("/bin/ls", "/bin/ls", "-al", "/tmp", NULL);
    printf("argc: %d\n", argc);
    printf("file name: %d\n", argv[0]);
    printf("%s %s\n", argv[1], argv[2]);
}
```

이것을 위와 동일한 입력을 주고 실행한 결과는 아래와 같이 나오게 됩니다.

```sh
# /tmp 디렉토리에서의 ls -al의 결과가 나옴
```

여기서 놀라운 사실은 기존에 뜨던 `printf`의 내용이 무시된다는 점입니다. 이러한 `exec`의 동작을 보면 그림과 같이 진행됩니다. 

![where is ps](/assets/img/res/2019-UnixAdvanced/2/exec.PNG)

`exec()`을 하게 되면 기존의 `a.out` 프로그램은 새롭게 부른 `/bin/ls`에 의해서 덮어지게 됩니다. 따라서 위의 코드는 `/bin/ls` 코드로 `execl` 을 부른 시점부터 바뀌게 되므로 이후의 `printf`가 출력이 되지 않는 상황이 벌어지는 것입니다.

## fork()

하지만 `exec`을 하게 되면 기존 프로세스를 덮어버리게 됩니다. 오로지 유닉스 체계에 `exec`만 있다고 가정을 하면  부팅 후에 생성된 `init` 프로세스는 다른 사용자 프로그램에 덮어지게 되고, 이러면 커널이 붕괴되는 것은 자명한 사실입니다. 그러한 연유로 유닉스에서는 `fork()`라는 명령어가 있습니다.

이러한 `fork()`는 현재 프로세스(이하 부모)를 메모리상에서 복사한 프로세스(이하 자식)를 만듭니다. 이러한 자식은 부모의 **변수, 스택, 힙**의 내용을 복사하게 됩니다. 이때, **PCB(Process Control Block)**도 함께 복사됩니다. 이러한 PCB는 프로세스를 지원하고 관리하기 위한 정보들이 담긴 데이터 구조체로 리눅스에서는 `task_struct`와 형태로 구현됩니다.

이런 PCB가 자식에 복사되기 때문에 `fork()`를 실행 후에 자식은 부모가 시작한 위치에서 시작을 하게 됩니다. 즉, 임의의 코드에서 아래와 같이 `fork()`를 실행하게 되면

![fork-prev](http://www.csl.mtu.edu/cs4411.ck/www/NOTES/process/fork/fork-1.jpg)

PCB의 복사로 인해 부모 자식은 동일한 위치에서 시작을 하게 됩니다.

![fork-after](http://www.csl.mtu.edu/cs4411.ck/www/NOTES/process/fork/fork-2.jpg)

이러한 `fork()`된 자식 프로세스는 **부모 프로세스가 종료되거나 부모 프로세스에서 `wait`나 `waitpid` 함수를 부르게 되면 종료**가 되게 됩니다.

실질적으로, `fork()`의 동작을 보기 위해서는 먼저 아래와 같은 프로그램을 작성해주시길 바랍니다.

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

int main(void){
    pid_t pid;
    pid = fork();
    printf("my fork test");
    sleep(10); // sleep 10 sec
    exit(0);
}
```

이 프로그램을 실행한 후에 `gcc -o myfork myfork.c`를 해준 후에 해당 프로그램을 실행하고 10초가 되기 전에 `Ctrl+Z`를 눌러서 프로그램을 중지 시켜 줍니다. 그러면 `ps `을 치면 아래와 같은 창이 뜨게 됩니다.

![fork execution](/assets/img/res/2019-UnixAdvanced/2/fork.PNG)

`fork`가 잘 되는 것을 알 수 있습니다.

유닉스 체계에서 `init` 프로세스가 실행이 되면 `init`을 `fork`를 하고 `fork`된 프로세스를 `exec`으로 덮어씌우도록 하여 프로세스를 계속 생성하는 방식이라고 볼 수 있습니다.

**여담으로 이러한 `fork`는 자식도 부를 수 있을까요?** 가능합니다. 만약 아래와 같은 코드를 짜신다면 무한히 늘어나는 자식을 볼 수 있습니다. 단, 아래의 코드를 수행했을 때 벌어지는 일에 관해서 책임을 제가 지지 않습니다.

```c
#include<stdio.h>
#include<unistd.h>

int main(void){
    while(1){
        fork();
        sleep(10); // This line uses to protect the system from quickly die.
    }
}
```

**다음으로 exec을 하고 난 후에 프로세스의 PID는 변경이 될까요?** 프로세스가 본디 가지고 있는 템플릿을 그대로 사용하기 때문에 PID는 변경되지 않습니다.

이런 포크를 활용해서 프로그램[^1]을 만들면 아래와 같이 됩니다.

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<sys/wait.h>

#define MAX_BUF_SIZE 256

int main(void){
    char    buf[MAX_BUF_SIZE];
    pid_t   pid;
    int     status;

    fprintf(stdout, "%% ");
    while(fgets(buf, MAX_BUF_SIZE, stdin) != NULL){
        if(buf[strlen(buf) - 1] == '\n')
            buf[strlen(buf) - 1] = 0;

        if((pid = fork()) < 0){
            fprintf(stderr,"fork error");
        }
        else if(pid == 0){ // 자식의 경우
            execlp(buf, buf, (char *)0);
            fprintf(stderr,"couldn't execute: %s", buf);
            exit(127);
        }
		// 부모의 경우
        if((pid = waitpid(pid, &status, 0)) < 0)
            fprintf(stderr, "waitpid error");
        fprintf(stdout, "%% ");
    }
    exit(0);
}
```

이 프로그램은 가장 간단한 셸 프로그램으로 `ls`와 같은 명령어를 치면 그 결과 값을 받아볼 수 있는 프로그램입니다. 여기서 괄목할만한 사항은 자식의 경우 pid 값은 0이고, 부모의 경우에는 그 값이 0 이상을 가진다는 점입니다.

그리고 이렇게 하나의 자식만을 만드는 것이 아니라 상황에 따라서 수많은 자식도 만들 수 있습니다. 이를 테면, 시스템을 다운을 시키고 싶은 경우에 아래와 같은 프로그램을 만들면 무수히 많은 자식이 생겨남에 따라 시스템이 죽게 됩니다.

```c
while(1) {
    fork();
}
```

이러면 부모가 자식을 부르고 자식이 또 부모가 되어 자식을 부르고 그 부모의 부모는 자식을 부르고 하면서 기하급수적으로 그 값이 늘어나게 됩니다.

결과적으로, 리눅스라는 시스템은 `init` 프로세스를 중점으로 하여 이 프로세스가 `fork`가 되고, 다음에 `exec`을 불러서 시스템이 동작하는 것이라고 알 수 있습니다.



[^1]: 출처) Stevens, W. R., & Rago, S. A. (2014). *Advanced programming in the UNIX environment*. Upper Saddle River, NJ: Addison-Wesley.

