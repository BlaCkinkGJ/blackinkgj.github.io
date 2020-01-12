---
layout: post
title: "[uftrace] 기존 바이너리 fio 트레이싱"
date: 2019-12-24
excerpt: "uftrace를 사용해서 기존 바이너리인 fio를 트레이싱 해보자."
tag:
- uftrace
comments: true
typora-root-url: ../../
---

# uftrace를 사용한 시각화

# 기존 바이너리 fio 트레이싱

## 지난 글

1. [[uftrace] uftrace의 설치와 사용법](https://blackinkgj.github.io/uftrace-installation/)
2. [[uftrace] uftrace의 tui를 사용해보도록 하자.](https://blackinkgj.github.io/uftrace-tui/)
3. [[uftrace] uftrace를 사용한 시각화](https://blackinkgj.github.io/uftrace-visualization/)

## fio?

fio[^1]는 Flexible I/O tester로 디스크 벤치 마크 프로그램의 일종으로 사용은 이전에 작성한 [자료](https://blackinkgj.github.io/installation-qemu-nvme/)의 **fio 설치** 부분을 참고해주시길 바랍니다.

## --force

만약 `-pg -g` 옵션을 부여하지 않은 프로그램을 tracing을 하고자 하는 경우에는 `--force` 명령을 사용해서 recording을 실시하면 됩니다.

이를 위해 먼저 [자료](https://blackinkgj.github.io/installation-qemu-nvme/)에서 언급하는 jobfile을 테스트를 하는 폴더에 만들어주도록 합니다. 이때, 유의사항으로 **`directory`를 반드시 현재 위치인 `./`로 `direct=0`, `size=1gb`로 변경해주도록 합니다.**(이를 하는 이유는 안그러면 너무 느리거나 원치 않은 결과를 볼 수 있기 때문입니다.)

그리고 아래의 명령을 쳐서 recording을 해주도록 합니다. (`-K 10`은 kernel function depth를 10으로 함을 의미합니다. 자세한 내용은 `uftrace -h`를 쳐서 확인해주시길 바랍니다.)

```bash
sudo uftrace record --force -a -k -K 50 fio jobfile
```

이 다음에 내용을 확인하기 위해서 다음과 같이 치도록 합니다.

```bash
uftrace tui
```

그러면 `-pg` 옵션을 부여하지 않았음에도 불구하고 프로그램의 동작이 어떡게 이루어졌는 지를 확인할 수 있습니다.

유의사항으로는 `replay`의 경우에는 `uftrace replay -t 1us`과 같이 사용하는 경우에는 로그가 너무 많이 남아서 제대로된 결과가 안나올 수 있습니다.

따라서, 특정 함수를 필터링 하는 `-F` 옵션을 `uftrace replay -F read -t 10ms`와 같이 사용하는 것을 권고드립니다. 



참고자료
----

[^1]: https://fio.readthedocs.io/en/latest/fio_doc.html