---
layout: post
title: "[유닉스] 유닉스 시스템의 개요"
date: 2019-3-11
excerpt: "유닉스 시스템의 개요에 관해서 알아보도록 한다."
tag:
- UNIX
comments: true
---
# 유닉스 시스템의 개요

## 유닉스 시스템 구조

유닉스 시스템의 기본적인 구조는 아래와 같이 됩니다.

![UNIX STRUCT](/assets/img/res/2019-UnixAdvanced/1/UnixStruct.PNG)

그리고 이러한 구조는 크게 'system-call interfacae to the kernel' 위의 공간에 해당하는 사용자 주소 공간(user address space)에 해당하는 부분과 밑의 공간에 해당하는 커널 주소 공간(kernel address space)으로 이루어집니다.

여기서 가장 괄목할만한 것은 'kernel interface to the HW '부분으로 이는 API 또는 function들을 의미하며 다른 말로는 **<u>HAL(Hardware Abstraction Layer)</u>**라고 합니다. 여기서 abstraction은 common thing이라고도 쓰입니다. 이러한 abstraction은 내부의 구현을 감추고 인터페이스만을 제공하는 것이라고 할 수 있습니다.

예를 들어, 안드로이드 휴대폰을 만드는 상황이라 가정하도록 하겠습니다. 이 경우에 구글은 안드로이드라고 불리는 운영체제 즉, 하드웨어 인터페이스를 제공을 해줍니다. 이러한 인터페이스의 제공을 통해서 삼성하고 LG는 서로 다른 하드웨어에 인터페이스에 대한 구현만 하는 것을 통해서 안드로이드라는 운영체제를 사용할 수 있게 됩니다.

리눅스를 포함하는 유닉스 계열의 운영체제에서는 이러한 HAL은 **함수 포인터(function pointer)**로 구성되게 됩니다. 이러한 방법을 통해서 가정하길 `read()`라는 안드로이드 인터페이스는 삼성이 만든 `Samsung.read()`에 매핑을 할 수 있게 됩니다. 결과적으로, 이 방법을 통해서 동적인 바인딩(dynamic binding)이 가능해지게 됩니다.

## 사용자 영역과 커널 영역

유닉스 계열 운영체제에서는 메모리에서 사용자 영역과 커널 영역을 나누도록 하여 서로에 접근을 할 수 없도록 만듭니다. 이렇게 하는 이유는 유닉스는 C로 개발이 되었고 이러한 C 언어는 포인터라는 강력한 기능을 통해서 메모리에 직접적인 접근을 가능하게 만들어줍니다. 따라서 보안 문제가 발생할 수가 있고, 그렇기에 그 둘을 철저하게 나누어 접근을 못하게 할 필요가 있어지게 되어 나누게 되었습니다.

만약에 사용자 영역의 프로그램이 **잘못된** 위치(커널)에접근하려고 하면 운영체제는 프로세스를 죽여버리도록 합니다. 이렇게 죽여버려서 커널은 생존하게 되고, **견고하면서(robust) 신뢰성이 높은(reliable) 프로그램**을 만들 수 있게 됩니다.

## 메모리 레이아웃

프로그램은 구문(statement)들의 집합이고 저장 매체에 저장되게 됩니다. 어떤 임의의 프로그램을 만든다고 하겠습니다.

```c
#include<stdio.h>
#include<stdlib.h>

int main(void){
    int i; //-> local variables
    char *c;
    i = 2; //-> code
    c = (char *)malloc(sizeof(char)*10);
    free(c);
    return 0;
}
```

이 프로그램을 컴파일을 하게 되면(적어도 gcc 하에서는) **메모리 레이아웃**을 만들게 됩니다. 일반적으로, 유명한 메모리 레이아웃으로는 COFF나 ELF[^1], a.out 등이 있습니다. 해당 내용에는 운영체제가 몇 비트인지를 확인하고 엔디언을 확인하는 등의 각종 작업을 하는 내용도 같이 포함된니다.

일반적으로, 이러한 메모리 레이아웃은 4GB를 할당을 받게 됩니다. 왜냐하면 32비트 운영체제 하에서의 1개의 프로세서는 4GB를 최대로 할당 받을 수 있게 됩니다. 하지만 실제 리눅스에서의 할당 내용을 보면 `0x0000 0000 ~ 0xffff ffff`까지 사용하지 않습니다. 일반적으로, `0x4000 0000`(1GB)만큼은 커널을 위해서 남겨두고 나머지 `0x0000 0000 ~ 0xbfff ffff`(3GB)가 사용자 영역을 위한 공간이기 때문입니다.

하지만 우리가 여기서 알 수 있듯이 4GB의 물리적 메모리에 프로세스의 메모리 4GB가 통째로 들어간다면 프로세스가 2개만 들어가도 메모리가 터질 것입니다. 따라서 프로세스가 메모리 상에 올라갈 때에는 부분적으로 필요한 부분만 올라가게 됩니다. 하지만 이렇게 부분적으로 올라간 것들이 너무 많아 모든 메모리를 다 사용하고 있는 경우가 발생할 수 있습니다. 이 경우에는 가상 메모리(Virtual Memory)로 운영체제가 스왑(swap)을 실시하도록 하게 만듭니다. 이런 것을 보고 가상 메모리 관리(Virtual Memory Management)라고 아야기 합니다.



[^1]: https://elinux.org/Executable_and_Linkable_Format_(ELF)