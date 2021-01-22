---
layout: post
title: "[Linux] C 언어에서 호출 스택 백트레이스(backtrace)하는 방법"
date: 2021-01-22
excerpt: "C 언어에서 호출 스택을 트레이싱 하는 방법을 알아보도록 한다."
tag:
- Linux
- C
- backtrace
comments: true
typora-root-url: ../../
---

# 백트레이스(backtrace)란?

백트레이스(backtrace)는 시스템에 문제를 발생했을 때, 디버그를 좀 더 편리하게 도와주는 것입니다. 이를 테면, 파이썬 언어에서 오류가 발생하면 다음과 같은 화면이 뜨는 것을 확인할 수 있습니다.

![python backtrace](http://anh.cs.luc.edu/handsonPythonTutorial/_images/tracebackAtError.png)

이미지에서 볼 수 있듯이 에러가 발생한 시점까지 오기까지 어느 과정을 거쳤는지를 그리고 어떤 문제가 발생했는지를 보여줍니다. 이렇듯, 백트레이스를 잘 활용하면 좀 더 수월한 디버깅을 수행할 수 있는 특징이 있습니다.

하지만 C의 경우에는 이를 명시적으로 지원해주지는 않으므로, 백트레이스를 지원하게 하기 위해서는 특별한 함수들을 사용하여 백트레이스 기능을 만들어주어야 합니다.

# C에서의 백트레이스

이제 Linux에서 백트레이스가 어떻게 지원되는지를 알아보도록 하겠습니다.

>  주의! 윈도우의 경우 해당 기법이 동작하지 않을 수 있습니다.

먼저, backtrace에 대한 기본적인 설명은 [링크](https://man7.org/linux/man-pages/man3/backtrace.3.html)에서 확인하실 수 있으십니다. 먼저, 다음과 같은 코드를 작성해서 `backtrace.c`로 저장하도록 합니다.

```c
// backtrace.c

#include <stdio.h>
#include <stdlib.h>
#include <execinfo.h>

void callstack_dump(void)
{
	void *callstack[128];
	int i, nr_frames;
	char **strs;

	nr_frames = backtrace(callstack, sizeof(callstack)/sizeof(void *));
	strs = backtrace_symbols(callstack, nr_frames);
	for (i = 0; i < nr_frames; i++) {
		printf("%s\n", strs[i]);
	}
	free(strs);
}

void child(void)
{
	callstack_dump();
}

void parent(void)
{
	child();
}

int main(void)
{
	parent();
	return 0;
}
```

코드에서의 `main`, `parent`, `child`는 호출 과정을 명시적으로 확인하기 위한 테스트 함수에 해당합니다. 실제로 호출 과정을 출력하는 백트레이 기능은 `callstack_dump` 함수에서 구현되게 됩니다.

`callstack_dump` 함수를 간단히 알아보도록 하겠습니다. 사용되는 변수는 호출 스택들의 포인터를 담을 포인터 변수 `callstack` 배열과, 순회문 탐색을 위한 `i`, 백트레이스 결과로 탐색된 호출 스택의 갯수를 담는 `nr_frames`, 포인터로 표현된 호출 스택 값의 문자열화 된 배열을 담는 `strs`로 구성됩니다.

이제 동작 과정을 따라가도록 하겠습니다. 먼저, `backtrace`  함수를 통해서 호출된 함수를 최대 `sizeof(callstack)/sizeof(void*)`까지 가져오도록 합니다. 해당 식은 배열의 크기를 반환하는 식에 해당합니다.

> 만약에 최대 N개 까지만 가져오고 싶은 경우에는 `callstack` 배열의 크기를 넘지 않는 선에서 `backtrace(callstack, N)`과 같이 작성하여 수행해주도록 합니다.

다음으로 `backtrace_symbols`를 통해서 `callstack`에 있는 호출 스택들의 포인터를 심볼 문자열 값으로 변경을 해주도록 합니다. 이때, `backtrace_symbols`의 반환값은 문자열 배열이 되면서 크기는 `nr_frames`만큼 가져오도록 합니다.

이렇게 반환된 문자열들을 `printf`를 통해 출력하고, 마지막으로 **반드시 strs는 향후 `free`를 통해 메모리를 해제해주어야 합니다.** 

이를, `gcc -g -rdynamic backtrace.c`를 통해서 컴파일을 해주도록 합니다. 그리고 나온 프로그램을 `./a.out`으로 출력하면 다음과 같은 결과가 나타나게 됩니다.

> `-g` 옵션은 디버깅 관련 정보를 포함해서 컴파일을 하라는 것에 해당하며, `-rdynamic`은 링커가 사용되는 것들 뿐만 아니라 모든 심볼들을 동적 심볼 테이블에 추가하도록  만들어주는 것입니다. `backtrace`나 몇몇 `dlopen` 사용에 필요합니다.

```bash
./a.out(callstack_dump+0x32) [0x55a7f167e1fb]
./a.out(child+0xd) [0x55a7f167e298]
./a.out(parent+0xd) [0x55a7f167e2a8]
./a.out(main+0xd) [0x55a7f167e2b8]
/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0xf3) [0x7f06451250b3]
./a.out(_start+0x2e) [0x55a7f167e10e]
```

앞선 python에서 보는 출력과 유사한 내용이 출력되는 것을 확인할 수 있습니다. 이를 잘 사용하면 여러 디버깅 상황에 대해서 효율적으로 대처할 수 있을 것으로 생각됩니다.



