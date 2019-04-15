---
layout: post
title: "[유닉스] semaphore"
date: 2019-4-13
excerpt: "semaphore를 사용해보자."
tag:
- UNIX
comments: true
---

# Semaphore

쓰레드<sub>thread</sub>를 사용하는 환경에서는 뮤텍스<sub>mutex</sub>만으로 문제를 해결할 수 있을 것입니다. 하지만 만약 IPC를 사용하는 경우에는 뮤텍스의 사용은 꽤나 곤란할 것입니다. 왜냐하면 **뮤텍스**는 공유된 자원의 데이터를 <u>여러 쓰레드가 접근</u>하는 것을 막는 것이기 때문입니다.

따라서 공유된 자원의 데이터를 <u>여러 프로세스</u>가 접근하는 것을 막는 것을 만들 필요가 있었고, 그러한 요구로 만들어진 것이 **세마포어<sub>semaphore</sub>**입니다. 세마포어는 이 [게시물](https://blackinkgj.github.io/Semaphore/)에서는 잘 나와 있지만 원래 깃발로 신호를 표현하는 방법에 해당합니다. 

이런 세마포어에서 주요 용어로는 **P, V 명령**이 있고, 이들은 원자적으로 실행됩니다. 그리고 각각의 정의는 아래와 같습니다.

- **P 연산** : 세마포어의 값을 $$S​$$라고 할 때에 $$S = S - 1​$$을 실행을 합니다. 그리고 $$S-1 \geq 0​$$이면 **계속 실행** 하고, 만약 $$S &lt; 0$$이면 프로세스를 **대기** 상태로 넣어주도록 합니다.
- **V 연산** : $$S = S + 1$$을 해주도록 합니다. 만약 $$S > 0$$이면 **계속 실행** 하도록 하고, $$S \leq 0$$이면 대기 상태 프로세스를 **깨우고** 현재 프로세스를 계속 실행합니다.

그리고 세마포어 값이 0, 1만 가지는 것을 보고 이진 세마포어<sub>binary semaphore</sub>라고 합니다. 그리고 그 이상의 값을 가질 수 있는 세마포어를 계수 세마포어<sub>counting semaphore</sub>라고 합니다.

유닉스 체계에서 이러한 세마포어는 2 가지 유형(POSIX, System V)을 가집니다. POSIX 버전의 경우에는 사용이 쉬운 장점이 있습니다. 하지만 세밀한 조정이 힘든 단점이 있습니다. System V 버전의 경우에는 POSIX와 반대로 세밀한 조정이 가능한 장점이 있으나, 사용이 복잡한 단점이 있습니다. 그리고 각각을 사용하기 위해서는 라이브러리를 `-lpthread` 옵션을 주어 연결해줘야 합니다.

## POSIX

POSIX 버전의 경우에는 `#include <semaphore.h>`를 넣어줍니다. 아래의 예제([출처](https://gist.github.com/junfenglx/7412986))로 확인해보도록 합시다.

```c
#include <stdio.h>          /* printf()                 */
#include <stdlib.h>         /* exit(), malloc(), free() */
#include <unistd.h>
#include <sys/types.h>      /* key_t, sem_t, pid_t      */
#include <sys/wait.h>
#include <sys/shm.h>        /* shmat(), IPC_RMID        */
#include <errno.h>          /* errno, ECHILD            */
#include <semaphore.h>      /* sem_open(), sem_destroy(), sem_wait().. */
#include <fcntl.h>          /* O_CREAT, O_EXEC          */


int main (int argc, char **argv){
    int i;                        /*      loop variables          */
    key_t shmkey;                 /*      shared memory key       */
    int shmid;                    /*      shared memory id        */
    sem_t *sem;                   /*      synch semaphore         *//*shared */
    pid_t pid;                    /*      fork pid                */
    int *p;                       /*      shared variable         *//*shared */
    unsigned int n;               /*      fork count              */
    unsigned int value;           /*      semaphore value         */

    /* initialize a shared variable in shared memory */
    shmkey = ftok ("/dev/null", 5);       /* valid directory name and a number */
    printf ("shmkey for p = %d\n", shmkey);
    shmid = shmget (shmkey, sizeof (int), 0644 | IPC_CREAT);
    if (shmid < 0){                           /* shared memory error check */
        perror ("shmget\n");
        exit (1);
    }

    p = (int *) shmat (shmid, NULL, 0);   /* attach p to shared memory */
    *p = 0;
    printf ("p=%d is allocated in shared memory.\n\n", *p);

    /********************************************************/

    printf ("How many children do you want to fork?\n");
    printf ("Fork count: ");
    scanf ("%u", &n);

    printf ("What do you want the semaphore value to be?\n");
    printf ("Semaphore value: ");
    scanf ("%u", &value);

    /* initialize semaphores for shared processes */
    sem = sem_open ("pSem", O_CREAT | O_EXCL, 0644, value); 
    /* name of semaphore is "pSem", semaphore is reached using this name */
    sem_unlink ("pSem");      
    /* unlink prevents the semaphore existing forever */
    /* if a crash occurs during the execution         */
    printf ("semaphores initialized.\n\n");


    /* fork child processes */
    for (i = 0; i < n; i++){
        pid = fork ();
        if (pid < 0)              /* check for error      */
            printf ("Fork error.\n");
        else if (pid == 0)
            break;                  /* child processes */
    }


    /******************************************************/
    /******************   PARENT PROCESS   ****************/
    /******************************************************/
    if (pid != 0){
        /* wait for all children to exit */
        while ((pid = waitpid (-1, NULL, 0)) > 0){
           if (errno == ECHILD)
                break;
        }

        printf ("\nParent: All children have exited.\n");

        /* shared memory detach */
        shmdt (p);
        shmctl (shmid, IPC_RMID, 0);

        /* cleanup semaphores */
        sem_unlink("pSem");
        sem_close(sem);

        exit (0);
    }

    /******************************************************/
    /******************   CHILD PROCESS   *****************/
    /******************************************************/
    else{
        sem_wait (sem);           /* P operation */
        printf ("  Child(%d) is in critical section.\n", i);
        sleep (1);
        *p += i % 3;              /* increment *p by 0, 1 or 2 based on i */
        printf ("  Child(%d) new value of *p=%d.\n", i, *p);
        sem_post (sem);           /* V operation */
        exit (0);
    }
}
```

각각의 함수에 관해서 알아보기 전에 POSIX 세마포어는 *익명 세마포어*와 *이름이 있는 세마포어*의 2 종류가 있습니다. 익명 세마포어는 `int sem_init(sem_t *sem, int pshared, unsigned int value)`로 만들어주도록 합니다. 해당 함수를 분석하면 아래와 같습니다.

-  `sem`은 초기화 할 세마포어의 포인터를 의미합니다.
- `pshared`의 값이 0이면 프로세스 하나의 쓰레드 간에 공유를 할 수 있게 합니다. 
- `value`의 값은 세마포어의 초깃값으로 1을 주면 이진 세마포어, 그 이외는 계수 세마포어가 됩니다.

그리고 이름이 있는 세마포어는 `sem_open`으로 만들어 줍니다. 

```c
sem_t* sem_open(const char *name, int oflag, /*optional*/ mode_t mode , /*optional*/ unsigned int value);
```

이런 구성을 가지는 이유는 이름이 있는 세마포어는 파일로 만들어지기 때문입니다. 만약 `O_CREAT | O_EXCL`이 `oflag`에 들어간다면 세마포어가 존재하면 오류를 반환(`O_EXCL`)하고,  그렇지 않으면 생성(`O_CREAT`)하도록 합니다. 그리고 `mode`와 `value`는 `O_CREAT`가 특정되는 경우 줘야할 것으로, `mode`는 파일을 여는 방식을 설정을 하고  `value` 값은 새로운 세마포어의 값을 지정합니다. 하지만 만약 `O_CREAT`이 설정이 되었는데, 지금 부여하는 이름을 가진 세마포어가 이미 존재하면 `mode`와 `value`는 무시되게 됩니다.

다음으로 `int sem_unlink(const char *name)`가 있습니다. 이는 이름이 있는 세마포어를 삭제하는 데 사용을 합니다.

이러한 POSIX 세마포어에서 **P 연산**에 해당하는 명령어로는 아래와 같습니다.

```c
sem_wait(sem_t *sem);
sem_trywait(sem_t *sem);
sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
```

`sem_wait`는 만약 세마포어 값이 0보다 작으면 대기를 하도록 합니다. 이에 반해, `sem_trywait`는 세마포어 값이 0보다 작으면 획득을 못했다는 오류를 띄우면서 대기를 하지 않고 빠져나옵니다. 마지막으로 `sem_timewait`는 대기 상태에 들어간 후에 `abs_timeout` 안에 획득을 하지 못하면 오류를 띄웁니다. 세마포어를 반환하는 **V 연산**은 `sem_post(sem_t *sem)`를 통해서 해줍니다.

그리고 이러한 세마포어를 다 사용한 경우에는 **이름이 있는 세마포어**는 `sem_close(sem_t *sem);`를 **익명 세마포어**는 `sem_destroy(sem_t *sem);`을 사용해주시길 바랍니다.

## SystemV

이번 버전은 `sys/types.h, sys/ipc.h, sys/sem.h`를 넣어 사용합니다. 이는 POSIX와 사용은 비슷하나 좀 더 복잡합니다. 아래의 주석을 잘 따라가 주시길 바랍니다.

```c
#ifndef __SYS_V_SEM__
#define __SYS_V_SEM__
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>

#define SEM_RESOURCE_MAX 1 // 이진 세마포어를 의미합니다.

/**
 * 아래를 이해하기 위해서는 다음을 이해해야 합니다.
 * struct sembuf { short sem_num; short sem_op; short sem_flg; };
 * - sem_num은 세마포어의 수로 여러 개의 세마포어를 사용하는 경우 사용합니다. 하나의 경우 0이 됩니다.
 * - sem_op에 값을 넣어주면 그 값을 세마포어 값에 더하거나 빼주도록 합니다.
 * - sem_flg는 세마포어 옵션으로 크게 2가지가 있습니다.
 *   - IPC_NOWAIT: POSIX에서 sem_trywait와 유사한 역할을 합니다.
 *   - SEM_UNDO: 프로세스가 세마포어를 돌려주지 않고 종료할 경우 커널이 알아서 증가시켜줍니다.
 */
#define SEM_LOCK {0, -1, SEM_UNDO} // P 연산에 해당합니다.
#define SEM_UNLOCK {0, 1, SEM_UNDO} // V 연상에 해당합니다.

union semun { /***** <<<<< UNION임을 유의하도록 합니다. >>>>> *****/
    int val;                   // SETVAL의 값을 지칭합니다.
    struct semid_ds *buf;      // IPC_SET, IPC_STAT을 위한 버퍼입니다.
    unsigned short int *array; // GETALL, SETALL을 위한 배열에 해당합니다.
};

static inline int
sem_init(int *semid, key_t key) {
    union semun semopts;
    // 1은 세마포어의 크기를 의미합니다. 
    if((*semid = semget(key, 1, IPC_CREAT|IPC_EXCL|0666)) == -1) {
        return -1;
    }
    semopts.val = SEM_RESOURCE_MAX;
    /**
     * - 1st arg: 세마포어 아이디
     * - 2nd arg: 여러 개의 세마포어를 사용하는 경우 사용합니다. 하나의 경우 0이 됩니다.
     * - 3rd arg: 명령이 들어가는 곳으로 semun을 설정하도록 합니다.
     * - 4th arg: 설정이 될 semun에 해당합니다.
     */
    semctl(*semid, 0, SETVAL, semopts);
    return 0;
}

static inline int
sem_open(int *semid, key_t key) {
   // 세마포어의 접근만 하는 경우에는 0을 사용하도록 합니다.
   if((*semid = semget(key, 0, 0666)) == -1)  {
       perror("Semaphore doesn't exist");
       return -1;
   }
   return 0;
}

static inline void
sem_wait(int *semid) {
    struct sembuf sem_lock = SEM_LOCK;
    /**
     * int semop(int semid,. struct sembuf *sops, unsigned nsops);
     * 세마포어 식별자와 어떤 연산을 이루어 질 것인지와 관련된 명령입니다.
     * 여기서 nsops는 sops가 가리키는 배열의 수행 연산을 선택합니다.
     * SEM_LOCKS = {0, -1, SEM_UNDO} 이므로, 아래의 명령은 해당 배열에서 '-1'을 지칭하라는 것입니다.
     */
    semop(*semid, &sem_lock, 1);
}

static inline void
sem_post(int *semid) {
    struct sembuf sem_unlock = SEM_UNLOCK;
    semop(*semid, &sem_unlock, 1);
}

static inline void
sem_destroy(int *semid) {
    // 세마포어를 삭제하도록 합니다. 삭제이므로 semun은 0으로 아무것도 넘기지 않습니다. 
    semctl(*semid, 0, IPC_RMID, 0);
}
#endif
```

만약 세마포어가 여러 개가 있는 경우에는 `semctl`의 2 번째 인자를 적절히 잘 사용을 해야하고, `wait` 및 `post`도 `sem_num`  설정을 해당하는 번호로 해주어야 합니다.

## 결론

간단한 세마포어의 사용에는 POSIX 세마포어가 좀 더 세밀한 조정에는 System V 세마포어가 적절합니다.