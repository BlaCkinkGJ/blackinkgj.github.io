---
layout: post
title: "[운영체제] Concurrency : Semaphore"
date: 2018-6-14
excerpt: "세마포어에 관해서 알아보도록 한다."
tag:
- os
- Concurrency
comments: true
---
출처 : http://pages.cs.wisc.edu/~remzi/OSTEP/
# Concurrency
### Semaphore란 무엇인가?

세마포어(Semaphore)라는 용어는 깃발로 신호를 표현하는 세마포어 플래그 신호(Flag Semaphore)에서 가져왔습니다.[^1] 컴퓨터의 세마포어 또한 이와 비슷한 역할을 합니다. 이제 이것에 대해 본격적으로 알아보도록 하겠습니다. 세마포어는 초기화 과정과 동작에 필요한 2개의 과정(`sema_wait()`와 `sem_post()`)으로 구성됩니다. 


먼저 `sem_init`을 좀 더 살펴보도록 하겠습니다. POSIX 표준[^2]에 따르면 아래와 같은 선언을 가지고 있습니다.

`int sem_init(sem_t *sem, int pshared, unsigned int value)`

여기서 **sem**은 초기화할 세마포어의 포인터를 넣는 위치입니다. 그리고 **pshared** 값이 0인 경우에는 세마포어는 프로세스 안에서의 <u>쓰레드들끼리 공유</u>가 되나, 그 외 경우에는 <u>프로세스 간 공유</u>가 되게 됩니다. 마지막으로 **value**는 세마포어가 가지는 초기 값을 지칭합니다. 아래와 같이 사용하여 초기화를 수행하도록 합니다.

```C
#include<semaphore.h>
sem_t s;
sem_init(&s, 0, 1);
```

다음으로 `sem_wait(sem_t *s)`를 알아보도록 하겠습니다. 이는 `sem_t *s`에서 받은 세마포어 값이 1 이상이면 그 값을 감소시키고 즉시 함수를 빠져나가도록 합니다. 하지만 그 값이 0 이라면 이 값이 1 이상이 되어 감소를 수행할 수 있거나 인터럽트 호출이 있지 않는 이상 현재 지점에서 작업이 대기하도록 만듭니다.

그리고 `sem_post(sem_t *s)`는 `sem_t *s`로 받은 세마포어의 값을 증가시키도록 합니다. 만약 세마포어 값이 이로 인해 1 이상이 된다면 다른 `sem_wait()`에 의해 대기 중인 프로세스 혹은 쓰레드를 깨우도록 합니다.


### Binary Semaphore
이러한 세마포어의 활용 예제로 이진세마포어(binary semaphore)가 있습니다. 이러한 이진세마포어는 위에서 초기화 가정에서 <u>초기값을 1</u>로 주어 사용하는 것을 의미합니다. 이런 이진세마포어는 **락**과 동등한 역할을 합니다.

```C
// binary semaphore     // lock
// ==================== // ====================
sem_t m;                // lock_t m;
sem_init(&m, 0, 1);     //
                        //
sem_wait(&m);           // lock(&m)
// 임계 구역            // // 임계 구역
sem_post(&m);           // unlock(&m)
```

이와 비슷하게 활용하는 예제로 **조건 변수**와 같이 활용하는 방법이 있습니다. 이는 <u>초기값을 0</u>으로 주어 구현할 수 있습니다.
```C
sem_t s;

void *child(void *arg){
    printf("child\n");
    sem_post(&m);
    return NULL;
}

int main(int argc, char *argv[]){
    int X = 0
    sem_init(&s, 0, X); // X의 값이 0이면 조건 변수로 사용할 수 있습니다.
    printf("parent: begin\n");
    pthread_t c;
    pthread_create(c, NULL, child, NULL);
    sem_wait(&s);
    printf("parent: end\n");
    return 0;
}
```

이를 잘 활용을 하게 되면 이전에 문제가 있었던 유한 버퍼 문제를 잘 풀 수 있습니다. 이전에 유한 버퍼 문제를 풀 때에는 락 변수 하나와 조건 변수 2개로 이 문제를 풀었지만 여기서는 세마포어 변수들 만으로 풀 수 있습니다.
```C
int buffer[MAX];
int fill = 0;
int use = 0;

void put(int value){
    buffer[fill] = value;
    fill = (fill + 1) % MAX;
}

int get(){
    int temp = buffer[use];
    use = (use + 1) % MAX;
    return temp;
}

sem_t empty;
sem_t full;

void *producer(void *arg){
    int i;
    for(i = 0; i < loops; i++){
        sem_wait(&empty);
        put(i);
        sem_post(&full);
    }
}

void *consumer(void *arg){
    int i, temp = 0;
    while(temp != -1){
        sem_wait(&full);
        temp = get();
        sem_post(&empty);
        printf("%d\n", temp);
    }
}

int main(int argc, char *argv[]){
    // ...
    sem_init(&empty, 0, MAX);
    sem_init(&fill, 0, 0);
    // ...
}
```

이러한 해결 방법은 버퍼 최댓값이 1인 경우에는 완벽하게 동작하지만 2 이상의 경우에는 경쟁 상태에 들어가므로 과거의 데이터가 덮어씌워지는 문제가 발생할 수 있습니다. 다시 말해, 상호 배제를 고려하지 않은 상태라는 것을 의미합니다. 그렇다면 여기서 임계 구역은 `put()` 및 `get()`을 수행하는 부분이 그 대상이 됩니다. 이를 고려해서 수정을 하면 아래와 같이 하게됩니다.

```C
sem_t empty; // cond_t empty 
sem_t full;  // cond_t full
sem_t mutex; // mutex_t mutex

void *producer(void *arg){
    int i;
    for(i = 0; i < loops; i++){
        sem_wait(&mutex); // pthread_mutex_lock(&mutex);
        sem_wait(&empty); // pthread_cond_wait(&empty, &mutex);
        put(i);
        sem_post(&full); // pthread_cond_signal(&full);
        sem_post(&mutex); // pthread_mutex_lock(&mutex);
    }
}

void *consumer(void *arg){
    int i;
    for(i = 0; i < loops; i++){
        sem_wait(&mutex);
        sem_wait(&empty);
        int temp = get(i);
        sem_post(&full);
        sem_post(&mutex);
        printf("%d\n", temp);
    }
}

int main(int argc, char *argv[]){
    // ...
    sem_init(&empty, 0, MAX);
    sem_init(&fill, 0, 0);
    sem_init(&mutex, 0, 1);
    // ...
}
```

하지만 이 방법은 답이 될 수 없습니다. 왜냐하면 만약 소비자(consumer)가 `mutex`를 획득을 하고, `sem_wait()`를 세마포어가 가득찬 상태에서 진행한다고 가정하겠습니다. 그 후에 소비자가 대기 상태에 들어가고 CPU에 의해 생산자(producer)가 깨어나게 됩니다. 하지만 앞에서 소비자가 `mutex`를 가지고 있기 때문에 생산자는 `sem_wait()` 상태에 들어가게 됩니다. 이러면 소비자가 다시 시간이 지나 깨어난다고 해도 `empty` 세마포어가 조건을 만족하지 못하므로 대기 상태에 들어가게 됩니다. 그렇게 되면 교착상태에 걸리게 됩니다. 이것의 해결 방법은 그냥 락과 조건 변수의 위치를 서로 반대로 해주면 바로 해결됩니다. 이는 아래와 같이 각 내용을 변경을 하면 됩니다.

```C
sem_wait(&empty);
sem_wait(&mutex);
// Critical Section
sem_post(&mutex);
sem_post(&full)
```



[^1]: http://www.anbg.gov.au/flags/semaphore.html
[^2]: http://man7.org/linux/man-pages/man3/sem_init.3.html
