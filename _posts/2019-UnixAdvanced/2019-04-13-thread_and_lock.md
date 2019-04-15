---
layout: post
title: "[유닉스] thread and lock"
date: 2019-4-13
excerpt: "thread와 lock을 사용해보자."
tag:
- UNIX
comments: true
---

# Thread & Lock

여태 저희는 `fork`를 활용하여 동시성<sub>concurrent</sub> 프로그래밍을 달성할 수 있게 되었습니다. 하지만 `fork`는 개별 프로세스로 나누어짐에 따라서 공유 자원에 대한 접근이 복잡한 것을 알 수 있습니다. 그렇기에 이런 공유 자원에 대한 접근이 복잡한 경우에 사용할 방법을 강구할 필요가 있게 되었습니다.

그러한 해결책이 바로 쓰레드입니다. 이런 쓰레드에 대한 기본 설명은 이 [게시물](https://blackinkgj.github.io/Threads/)에 설명이 되어 있으니 이를 확인해주시길 바랍니다. 해당 게시물을 통해서 이론적 배경이 생성이 되었다면 아래와 같은 코드를 보도록 하겠습니다.

```c
#include <stdio.h>
#include <string.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>

pthread_t tid[2];
int       count;

void* doSomething(void *arg) {

    unsigned long i = 0;
    count += 1;
    printf("\nJob %d started\n", count);

    for(i = 0; i < (0xFFFFFFFF); i++);
    printf("\nJob %d finished\n", count);

    return NULL;
}

int main(void) { // main도 쓰레드의 일종입니다.
    int i = 0;
    int err;

    while (i < 2) {
        err = pthread_create(&tid[i], NULL, &doSomething, NULL);
        if (err != 0){
            printf("\ncan not create thread: [%s]\n", strerror(err));
        }
        i++;
    }

    return 0;
}
```

`pthread_create`는 다음과 같이 선언이 됩니다. 

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                   void *(*start_routine)(void *), void *arg);
```

`pthread_t` 구조체를 받는 포인터와 쓰레드의 특성을 지정하기 위한 `attr`(NULL 은 기본 쓰레드 특성),  `start_routine`은 실행시킬 쓰레드 함수이고, `arg`는 `start_routine`에 들어가는 함수에 들어갈 인자에 해당합니다.

여기서 `void *(*start_routine)`이 이해가 잘 안될 수 있는 데, 이는 함수 포인터로 `void* (*start_routine)`으로 보는 것이 좀 더 명확합니다. `void *` 형식을 반환하면서 `(*start_routine)`에 함수를 받는 함수 포인터라고 생각하시면 됩니다.

위의 코드에서 높은 확률로 `main`이 `doSomething`의 실행이 얼마 지나지 않아 죽는 것을 알 수 있습니다. 여기서 알 수 있는 사실은 **<u>`main`도 쓰레드의 일종이라는 것</u>**입니다.

그리고 심지어 어쩌다 `doSomething`에 들어가도 우리가 생각한 덧셈이 일어나지 않는 것을 알 수 있습니다. 이런 2개의 쓰레드가 들어가서 문제를 발생 시킬 수 있는 영역을 임계 영역<sub>critical section</sub>이라고 합니다.

되도록이면 위와 같이 작성하지 않는 것을 권고 합니다. 이에 따라, 수정된 코드를 보여드리면 아래와 같이 됩니다.

```c
#include <stdio.h>
#include <string.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>

pthread_t       tid[2];
int             count;
pthread_mutex_t lock; // mutex는 mutual exclusive를 의미합니다.

void* doSomething(void *arg) {
    unsigned long i = 0;
    pthread_mutex_lock(&lock); // thread 1이 lock이면 thread 2는 못 들어갑니다.
    count += 1;
    printf("\nJob %d started\n", count);

    for(i = 0; i < (0xFFFFFFFF); i++);
    printf("\nJob %d finished\n", count);
    pthread_mutex_unlock(&lock); // thread 1이 unlock하면 thread 2는 그제서야 들어갑니다.
    return NULL;
}

int main(void) { // main도 쓰레드의 일종입니다.
    int i = 0;
    int err;

    while (i < 2) {
        err = pthread_create(&tid[i], NULL, &doSomething, NULL);
        if (err != 0){
            printf("\ncan not create thread: [%s]\n", strerror(err));
        }
        i++;
    }
    pthread_join(tid[0], NULL); // main 쓰레드가 먼저 끝나는 것을 방지합니다.
    pthread_join(tid[1], NULL);

    return 0;
}
```

일단 `main` 쓰레드가 먼저 죽는 것을 방지하기 위해서 `pthread_join`을 사용합니다. 이를 사용하게 되면 혹여  `main` 쓰레드가 먼저 죽어 발생할 불상사를 막을 수 있습니다. 그리고 `pthread_mutex_lock`이 있는 데, 이는 `pthread_mutex_t lock` 변수를 사용하여 다른 쓰레드가 임계 영역에 들어가는 것을 방지합니다. 위와 같이 짜면 좀 더 안전하게 쓰레드를 사용할 수 있게 됩니다.

결과적으로, 위와 같은 방식을 사용해서 쓰레드를 사용해주시길 바랍니다.