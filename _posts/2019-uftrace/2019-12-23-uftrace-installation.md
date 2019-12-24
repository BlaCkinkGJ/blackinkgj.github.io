---
layout: post
title: "[uftrace] uftrace의 설치와 사용법"
date: 2019-12-23
excerpt: "uftrace를 설치하고 사용하도록 한다."
tag:
- uftrace
comments: true
typora-root-url: ../../
---

# uftrace의 설치와 사용법

uftrace는 C/C++ 프로그램에 대한 함수 호출 관계를 추적하는 도구로 김남형[^1]님이 제작하신 툴입니다. 최근에 KSC 2019[^2]에 참석하여 처음 이 툴을 알게 되었고, 개인적으로 사용해보니 괜찮은 것 같아서 이렇게 자료로 남기게 되었습니다.

uftrace는 기본적으로 Linux 환경에서 돌아가는 장치로 윈도우에서의 동작은 보장되어 있지 않습니다. 이 글은 Ubuntu 18.04 LTS 버전에서 돌린 것을 기반으로 하는 글임을 유념해주시길 바랍니다.

## 설치

 먼저 설치를 위해서 github의 uftrace 저장소에서 소스코드를 복제를 하도록 합니다.

```bash
git clone https://github.com/namhyung/uftrace.git
```

이 다음에 의존성 패키지를 설치하기 위해서 uftrace의 폴더의 misc에서 의존성 패키지 설치 쉘을 실행시켜주도록 합니다. 의존성 패키지 설치 쉘에 따르면 지원하는 배포판은 다음과 같습니다.

- ubuntu, debian, raspbian
- fedora
- rhel, centos
- arch, manjaro
- alpine

```bash
cd uftrace/misc
sudo ./install-deps.sh
```

의존성 설치가 완료되면 다시 uftrace 저장소의 루트 폴더로 이동하여 설치를 아래의 과정을 거쳐 진행하도록 합니다.

```bash
cd ..
./configure
make -j <number of cpu>
sudo make install
```

여기까지 문제 없이 수행하셨으면 아래의 명령을 수행하면 다음과 같이 나오게 됩니다. (버전 번호는 다를 수 있습니다.)

```bash
➜  uftrace --version
uftrace v0.9.3-189-gd9049 ( dwarf python luajit tui perf sched dynamic )
```

## 사용법

다음과 같은 간단한 프로그램을 제작해주도록 합니다. (이 글에서는 이 파일 이름을 `sample.c`라고 명명하겠습니다.)

```c
#include <stdio.h>

int factorial(int n)
{
        return (n <= 1 ? 1 : n * factorial(n - 1));
}

int main(void)
{
        int n = 0;
        scanf("%d", &n);
        printf("factorial(%d) = %d\n", n, factorial(n));
        return 0;
}

```

이렇게 만들어진 `sample.c` 파일을 다음과 같은 옵션을 주어 컴파일을 진행하도록 합니다.

```bash
gcc -pg sample.c
```

그리고 이제 uftrace를 사용하기 위해서 `uftrace ./a.out`을 실행하고 `5`를 입력하면 다음과 같은 결과가 나오게 됩니다.

```bash
# DURATION     TID     FUNCTION
   0.905 us [ 23720] | __monstartup();
   0.324 us [ 23720] | __cxa_atexit();
            [ 23720] | main() {
 740.578 ms [ 23720] |   __isoc99_scanf();
            [ 23720] |   factorial() {
            [ 23720] |     factorial() {
            [ 23720] |       factorial() {
            [ 23720] |         factorial() {
   0.217 us [ 23720] |           factorial();
   0.939 us [ 23720] |         } /* factorial */
   1.320 us [ 23720] |       } /* factorial */
   1.684 us [ 23720] |     } /* factorial */
   2.128 us [ 23720] |   } /* factorial */
  20.812 us [ 23720] |   printf();
 740.604 ms [ 23720] | } /* main */

(END)
```

함수가 호출된 순서 및 각각의 함수가 실행된 시간을 알 수 있습니다. 기본적인 사용은 이렇게 사용할 수 있습니다. 하지만 로그가 너무 많고 이를 기록하고 싶을 때에는 이렇게 사용하면 나중에 관리하기가 복잡합니다. 따라서 이를 쉽게 관리할 수 있도록 uftrace에서는 `record`라는 기능을 지원합니다.

### `record`, `replay` 옵션

위에서 컴파일한 프로그램을 `uftrace record ./a.out`이라 치고, `5`를 입력하면 프로그램이 문제없이 종료되는 것을 알 수 있습니다. 그리고 `uftrace replay`를 하면 위의 출력과 동일한 출력이 나오게 됩니다.

#### `-k`  옵션

만약 커널 내부에서 어떤 작업이 일어나는 지 알고 싶은 경우에는 `sudo uftrace record -k ./a.out`을 수행해주도록 합니다. 이 경우에 `uftrace replay`를 수행하면 아래와 같이 나오게 됩니다.

```bash
# DURATION     TID     FUNCTION
   0.865 us [ 26670] | __monstartup();
   0.294 us [ 26670] | __cxa_atexit();
            [ 26670] | main() {
            [ 26670] |   __isoc99_scanf() {
   4.039 us [ 26670] |     __x64_sys_newfstat();
  12.314 us [ 26670] |     __x64_sys_read();
  24.144 us [ 26670] |   } /* __isoc99_scanf */
            [ 26670] |   factorial() {
            [ 26670] |     factorial() {
            [ 26670] |       factorial() {
            [ 26670] |         factorial() {
   0.217 us [ 26670] |           factorial();
   0.896 us [ 26670] |         } /* factorial */
   1.260 us [ 26670] |       } /* factorial */
   1.547 us [ 26670] |     } /* factorial */
   1.904 us [ 26670] |   } /* factorial */
            [ 26670] |   printf() {
   2.766 us [ 26670] |     __x64_sys_newfstat();
  29.175 us [ 26670] |     __x64_sys_write();
  39.895 us [ 26670] |   } /* printf */
  67.459 us [ 26670] | } /* main */

(END)
```

아까와 달리 추가적으로 시스템 호출에 해당하는 함수도 같이 나오는 것을 알 수 있습니다.

#### `-D` 옵션

하지만 함수가 너무 깊은 경우를 보도록 하겠습니다. 위 프로그램을 `uftrace record ./a.out`을 하고, `50`을 치는 경우에는 결과도 잘못 나올 뿐더러 아주 깊게 들어가는 `factorial` 함수를 확인할 수 있습니다.

이 경우에 일정 이상 들어가는 함수를 제외한 결과가 보고 싶을 때가 있습니다. 이럴 때 쓰는 것이 `-D` 옵션으로 `uftrace replay -D 5`와 같이 사용하여 볼 수 있습니다.

그러면 적어도 사람이 볼 수 있는 내용으로 아래와 같이 나오는 것을 알 수 있습니다.

```bash
# DURATION     TID     FUNCTION
   0.985 us [ 28794] | __monstartup();
   0.268 us [ 28794] | __cxa_atexit();
            [ 28794] | main() {
  26.187  s [ 28794] |   __isoc99_scanf();
            [ 28794] |   factorial() {
            [ 28794] |     factorial() {
            [ 28794] |       factorial() {
   2.953 us [ 28794] |         factorial();
   3.658 us [ 28794] |       } /* factorial */
   4.056 us [ 28794] |     } /* factorial */
   4.724 us [ 28794] |   } /* factorial */
  21.621 us [ 28794] |   printf();
  26.187  s [ 28794] | } /* main */

(END)
```

#### `-F`, `-N`, `-C` 옵션

그리고 다시 `uftrace record ./a.out`하고, `5`를 입력해주도록 합시다.

만약에 `factorial` 함수만 보고 싶을 때에는 어떻게 해야 할까요? 이 경우를 위해서 `-F` 옵션이 있습니다. 이 경우에는 `uftrace replay -F factorial`라고 치면 됩니다.

하지만 이번에는 `factorial` 함수만 보기 싫은 경우에는 어떻게 해야 할까요? 이 경우에는 `uftrace replay -N factorial`이라고 하면 해당 함수만 제외한 결과를 볼 수 있습니다.

그런데 `-F` 옵션을 사용하는 경우에는 `factorial` 함수만 찾아주고, 이 함수를 누가 불렀는지는 전혀 알 수 없는 문제가 있습니다. 이때 사용하는 옵션으로 `-C` 옵션이 있습니다. `-C` 옵션은 `uftrace replay -C factorial`과 같이 사용합니다. 이렇게 사용하면 `factorial` 함수 뿐만 아니라 `factorial`을 부른 함수까지 다 나타나게 됩니다.

 #### `-t` 옵션

이번에 다시 `uftrace record ./a.out`을 하고, `50`을 입력해주도록 합니다.

그러면 이전과 동일하게 `uftrace replay`를 하게 되면 엄청난 양의 로그가 나오게 됩니다. 이를 좀 더 뭉쳐서 보기 위해서는 `-t` 옵션을 부착해주도록 합니다. `uftrace replay -t 15us`를 수행하게 되면, 어마어마한 로그가 나오는 것이 아니라 **`10us` 내에 끝난 함수는 출력이 안되게 됩니다.**

```bash

# DURATION     TID     FUNCTION
            [  1606] | main() {
   2.714  s [  1606] |   __isoc99_scanf();
            [  1606] |   factorial() {
            [  1606] |     factorial() {
  15.214 us [  1606] |       factorial();
  15.491 us [  1606] |     } /* factorial */
  16.059 us [  1606] |   } /* factorial */
  56.207 us [  1606] |   printf();
   2.714  s [  1606] | } /* main */

```

### `-a`, `-R`, `-A` 옵션

함수에 들어가는 매개 변수 값과 반환 값을 알고 싶은 경우에는 먼저 프로그램을 컴파일을 할 때에 다음과 같이 진행해주셔야 합니다. 

```bash
gcc -pg -g sample.c
```

그런 다음에 `record`를 하는 경우에 다음과 같이 하여 실행시켜주도록 합니다.

```bash
uftrace record -a ./a.out
```

그리고 `10` 값을 넣어주면 `uftrace replay`를 하면 다음과 같은 결과가 나옵니다.

```bash
# DURATION     TID     FUNCTION
   1.230 us [ 22216] | __monstartup();
   0.221 us [ 22216] | __cxa_atexit();
            [ 22216] | main() {
   2.386  s [ 22216] |   __isoc99_scanf();
            [ 22216] |   factorial(10) {
            [ 22216] |     factorial(9) {
            [ 22216] |       factorial(8) {
            [ 22216] |         factorial(7) {
            [ 22216] |           factorial(6) {
            [ 22216] |             factorial(5) {
            [ 22216] |               factorial(4) {
            [ 22216] |                 factorial(3) {
            [ 22216] |                   factorial(2) {
   0.178 us [ 22216] |                     factorial(1) = 1;
   1.394 us [ 22216] |                   } = 2; /* factorial */
   4.398 us [ 22216] |                 } = 6; /* factorial */
   4.713 us [ 22216] |               } = 24; /* factorial */
   5.032 us [ 22216] |             } = 120; /* factorial */
   5.356 us [ 22216] |           } = 720; /* factorial */
  12.193 us [ 22216] |         } = 5040; /* factorial */
  12.523 us [ 22216] |       } = 40320; /* factorial */
  12.890 us [ 22216] |     } = 0x58980; /* factorial */
  14.566 us [ 22216] |   } = 0x375f00; /* factorial */
 133.326 us [ 22216] |   printf("factorial(%d) = %d\n") = 24;
   2.386  s [ 22216] | } = 0; /* main */
```

`-a` 옵션의 경우에는 보이는 바와 같이 uftrace가 파악해서 적절한 값으로 변환을 합니다.

만약 `factorial`의 반환 값을 10진법이나 16진법을 선택해서 보고 싶은 경우에는 다음과 같이 하면 됩니다.

```bash
uftrace record -R factorial@retval/i32 ./a.out
```

이렇게 하면 32비트 정수형으로 `uftrace replay` 수행 시에 아래와 같이 나오게 됩니다. (더 많은 옵션은 [링크](https://github.com/namhyung/uftrace/blob/master/doc/uftrace-record.md#arguments)를 참고)

```bash
# DURATION     TID     FUNCTION
   1.286 us [ 32534] | __monstartup();
   0.254 us [ 32534] | __cxa_atexit();
            [ 32534] | main() {
   2.001  s [ 32534] |   __isoc99_scanf();
            [ 32534] |   factorial() {
            [ 32534] |     factorial() {
            [ 32534] |       factorial() {
            [ 32534] |         factorial() {
            [ 32534] |           factorial() {
            [ 32534] |             factorial() {
            [ 32534] |               factorial() {
            [ 32534] |                 factorial() {
            [ 32534] |                   factorial() {
   0.224 us [ 32534] |                     factorial() = 1;
  15.725 us [ 32534] |                   } = 2; /* factorial */
  16.282 us [ 32534] |                 } = 6; /* factorial */
  16.717 us [ 32534] |               } = 24; /* factorial */
  21.150 us [ 32534] |             } = 120; /* factorial */
  21.587 us [ 32534] |           } = 720; /* factorial */
  22.008 us [ 32534] |         } = 5040; /* factorial */
  22.436 us [ 32534] |       } = 40320; /* factorial */
  22.873 us [ 32534] |     } = 362880; /* factorial */
  23.656 us [ 32534] |   } = 3628800; /* factorial */
  37.140 us [ 32534] |   printf();
   2.001  s [ 32534] | } /* main */

(END)
```

만약에 `factorial`의 매개 변수 값을 10진법이나 16진법으로 선택해서 보고 싶은 경우에는 다음과 같이 하면 됩니다.

```bash
uftrace record -A factorial@arg1/x ./a.out
```

이렇게 하면 16진법으로 `uftrace replay`를 수행 시 아래와 같이 매개 변수 값이 나오게 됩니다.

```bash
# DURATION     TID     FUNCTION
   1.031 us [  8375] | __monstartup();
   0.332 us [  8375] | __cxa_atexit();
            [  8375] | main() {
   1.563  s [  8375] |   __isoc99_scanf();
            [  8375] |   factorial(0xa) {
            [  8375] |     factorial(0x9) {
            [  8375] |       factorial(0x8) {
            [  8375] |         factorial(0x7) {
            [  8375] |           factorial(0x6) {
            [  8375] |             factorial(0x5) {
            [  8375] |               factorial(0x4) {
            [  8375] |                 factorial(0x3) {
            [  8375] |                   factorial(0x2) {
   0.267 us [  8375] |                     factorial(0x1);
   1.563 us [  8375] |                   } /* factorial */
   5.816 us [  8375] |                 } /* factorial */
   6.217 us [  8375] |               } /* factorial */
   6.598 us [  8375] |             } /* factorial */
   6.985 us [  8375] |           } /* factorial */
  16.262 us [  8375] |         } /* factorial */
  16.660 us [  8375] |       } /* factorial */
  17.094 us [  8375] |     } /* factorial */
  18.972 us [  8375] |   } /* factorial */
  21.762 us [  8375] |   printf();
   1.563  s [  8375] | } /* main */

(END)
```

### report 옵션

`uftrace record ./a.out`을 하고, `10`을 입력해주도록 합니다.

그리고 이전처럼 `replay`를 하는 것이 아니라 **`uftrace report`**를 수행하면 다음과 같은 결과가 나오게 됩니다.

```bash
  Total time   Self time       Calls  Function
  ==========  ==========  ==========  ====================
    2.714  s    3.367 us           1  main
    2.714  s    2.714  s           1  __isoc99_scanf
   56.207 us   56.207 us           1  printf
   16.059 us   16.059 us          50  factorial
    1.039 us    1.039 us           1  __monstartup
    0.364 us    0.364 us           1  __cxa_atexit
(END)
```

Total time은 서브루틴에서 걸린 시간을 합친 시간이고, Self time은 서브루틴을 제외한 시간을 합친 시간을 의미합니다. 그리고 Calls는 해당 함수가 불린 횟수로 우리가 `factorial(50)`을 했으므로, 50번 불린 것을 알 수 있습니다.

만약 각각 항목에 따라서 정렬을 하고 싶은 경우에는 `-s total`, `-s self`, `-s call`을 `uftrace report`의 옵션으로 넣어주면 됩니다.

## 예정 작성 자료

1. uftrace tui 사용법
2. uftrace를 사용한 시각화
3. 기존 바이너리인 `fio` tracing


출처
---

[^1]: https://github.com/namhyung

[^2]: http://www.kiise.or.kr/conference/KSC/2019/