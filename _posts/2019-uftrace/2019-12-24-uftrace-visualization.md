---
layout: post
title: "[uftrace] uftrace를 사용한 시각화"
date: 2019-12-24
excerpt: "uftrace의 tui를 사용해보도록 하자"
tag:
- uftrace
comments: true
typora-root-url: ../../
---

# uftrace를 사용한 시각화

## 지난 글

1. [[uftrace] uftrace의 설치와 사용법](https://blackinkgj.github.io/uftrace-installation/)
2. [[uftrace] uftrace의 tui를 사용해보도록 하자.](https://blackinkgj.github.io/uftrace-tui/)

## tracing?

chromium을 기반으로 하는 웹 브라우저는 tracing[^1]이라고 불리는 프로그램을 일반적으로 동봉하고 있습니다. 이것은 event profiling 하는 툴로 웹 브라우저의 동작을 디버깅하기 위한 것이라고 보면 됩니다.

![tracing](http://www.gamesetwatch.com/120823-aboutracing.jpg)

여기서도 나오듯이 `chrome://tracing`이라고 치면 해당 프로그램을 실행할 수 있습니다. 그리고 이 프로그램에서 나온 출력 값을 저장하거나 불러올 수 있습니다. 이때, 규격은 JSON[^2] 규격으로 다음과 같은 방식으로 구성되게 됩니다.

```json
{"traceEvents":[{"pid":14908,"tid":19960,"ts":102734724620,"ph":"X","cat":"input","name":"InputRouterImpl::MouseEventHandled","dur":22,"tdur":21,"tts":804647304,"args":{"type":"MouseMove","ack":"CONSUMED"}},
{"pid":14908,"tid":19960,"ts":102734724664,"ph":"E","cat":"sequence_manager","name":"ui_thread_tq","tts":804647348,"args":{}},
...
}
```

앞서 언급했듯이 이 저장된 파일을 `LOAD` 버튼을 눌러서 가져올 수 있다고 말했습니다. 다시 말해, `chrome://tracing`에서 요구되는 규격만 맞다면 어떠한 내용이든지 불러올 수 있어서 비단 웹 내용만 아니라 프로세스 트레이싱 결과와 같은 것도 가져올 수 있음을 의미합니다.

### dump --chrome

먼저, 터미널 환경이라면 윈도우에 xming또는 x window 포워딩이 가능한 SSH-client(e.g. [Putty](https://www.putty.org/), [MobaXterm](https://mobaxterm.mobatek.net/))를 설치해주도록 합니다. 그리고 터미널에서 `~/.bashrc` 또는 `~/.zshrc`의 설정을 변경하여 x window의 포워딩이 가능하게 만듭니다. 그런 다음 아래의 명령을 수행하여 chromium을 설치하도록 합니다.

```bash
sudo apt install chromium-browser
```

여기까지 했으면 기본 세팅이 완료되었습니다. 이제 다시 임의의 폴더에 들어가셔서 아래의 프로그램을 작성해주시길 바라며 파일 이름을 `hanoi.c`로 가정하겠습니다.

```c
#include <stdio.h>

void hanoi(int n, int ox, int tx, int mx)
{
        if (n < 1) {
                printf("Error: n >= 1\n");
        } else if (n == 1) {
                printf("%d --> %d\n", ox, tx);
        } else {
                hanoi(n - 1, ox, mx, tx);
                hanoi(1, ox, tx, mx);
                hanoi(n - 1, mx, tx, ox);
        }
}

int main(void)
{
        int n;

        printf("Enter the height of the tower\n");
        scanf("%d", &n);

        hanoi(n, 1, 3, 2);

        return 0;
}
```

이렇게 만들어진 파일을 `gcc -pg -g hanoi.c`로 컴파일을 해주시길 바랍니다.

이렇게 만들어진 내용을 recording하기 위해서 `sudo uftrace record -k ./a.out`을 수행해주시고, 입력 값으로 `5`를 넣어주도록 합니다. 그러면 약간의 시간이 걸리고 프로그램은 종료되게 됩니다.

이 다음에 아래의 명령어를 쳐서 `chrome://tracing`을 위한 JSON 덤프 파일을 만들어주도록 합니다.

```bash
uftrace dump --chrome > hanoi.json
```

그리고 아까 포워딩 설정을 해놓았으므로 터미널에 `chromium-browser`를 쳐서 chromium을 실행시켜주고, [chrome://tracing](chrome://tracing)을 쳐서 링크에 들어가도록 합니다. 다음에 `Load` 버튼을 눌러서 파일 탐색기에서 `hanoi.json` 파일을 찾아 불러오도록 합니다. 그러면 다음과 같은 결과가 나오게 됩니다.

![image-20191224141740229](/assets/img/res/2019-uftrace/gui-1.png)

여기서 만약에 궁금한 부분이 있다면 우측 상단에 있는 `?`를 눌러서 사용법을 익히고, 어떤 함수가 얼마나 불렸는지를 눈으로 확인해보도록 합니다. 만약 이 내용을 배포하고 싶은 경우에는 [trace2html](https://github.com/catapult-project/catapult/blob/master/tracing/bin/trace2html)을 사용하여 html로 만들어 배포하도록 합니다.

## FlameGraph?

flamegraph[^3]는 앞서 언급한 `chrome://tracing`과 유사한 것으로 가장 많이 불리는 코드 경로를 쉽게 확인할 수 있는 그래프입니다. 일반적으로 아래와 같은 모습을 띕니다.

![flame graph sample](https://camo.githubusercontent.com/789f18134b375f4ef0ce667012aa7992bef365d5/687474703a2f2f7777772e6272656e64616e67726567672e636f6d2f466c616d654772617068732f6370752d626173682d666c616d6567726170682e737667)

### dump --flame-graph

flamegraph를 dump하기 위해서는 먼저 [framegraph.pl](https://github.com/brendangregg/FlameGraph/blob/master/flamegraph.pl)을 받으셔야 합니다. 아래와 같은 명령을 통해 받도록 합니다.

```bash
wget https://raw.githubusercontent.com/brendangregg/FlameGraph/master/flamegraph.pl
```

만약에 다운이 되지 않는 경우에는 `wget`을 `sudo apt install wget`을 통해서 설치해주도록 합니다. 이렇게 받은 `flamegraph.pl`을 `chmod 755 flamegraph.pl`로 실행할 수 있도록 만들어줍니다.

그리고 앞서 chrome 덤프를 만들때 만든 recording한 결과가 있는 폴더로 가서 아래의 명령을 실행해주도록 합니다.

```bash
uftrace dump --flame-graph | ./flamegraph.pl > hanoi.svg
```

`hanoi.svg` 파일을 서버에서 FTP를 통해 다운 받은 후에 적절한 `svg` 파일을 열 수 있는 프로그램으로 확인하면 다음과 같이 뜨게 됩니다.

![image-20191224144311973](/assets/img/res/2019-uftrace/gui-2.png)

그리고 `svg`는 interactive하므로 원하는 내용에 커서를 올리면 해당하는 코드의 스택 프레임을 확인할 수 있습니다. **추가로, flamegraph는 `chrome://tracing`과 달리 시간 순서가 아니라 얼마나 많이 호출되어 있는가가 중점인 특징이 있습니다.**

## 다음 글

1. [[uftrace] 기존 바이너리 fio 트레이싱](https://blackinkgj.github.io/uftrace-fio/)



참고자료
----

[^1]: https://www.chromium.org/developers/how-tos/trace-event-profiling-tool
[^2]: https://www.json.org/json-ko.html
[^3]: http://www.brendangregg.com/flamegraphs.html