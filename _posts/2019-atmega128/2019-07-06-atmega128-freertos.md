---
layout: post
title: "[ATmega128] ATMega128에 FreeRTOS를 설치해보자"
date: 2018-7-6
excerpt: "ATMega128에 FreeRTOS를 설치해보자"
tag:
- FreeRTOS
comments: true
---

이 내용은 [이 내용](https://scienceprog.com/using-freertos-kernel-in-avr-projects/)를 참조하여 만들었습니다.

## FreeRTOS?

FreeRTOS는 Cross platform RTOS([Real-Time Operating System](https://en.wikipedia.org/wiki/Real-time_operating_system)) 커널로 업계에서 꽤 많이 사용되는 OS[^1]입니다.  이런 FreeRTOS는 아래의 특징을 가집니다.

- 전문적인 개발이 가능함
- 엄격한 품질 관리가 가능함
- 견고한 시스템을 개발 할 수 있음
- **상용으로 사용해도 상관없이 무료임**[^2]

이것을 사용하면 임베디드 환경에서의 동시성 프로그래밍을 쉽게 할 수 있습니다. 또한, 이런 동시성 프로그래밍 환경에서 태스크[^3]간의 데이터 교환이나 동기화 문제를 FreeRTOS를 사용하면 효과적으로 문제를 풀 수 있는 장점이 있습니다.

## ATMega128 + FreeRTOS

하지만 FreeRTOS가 ATmega128에서 포팅된 Demo가 공식 사이트에서는 존재하지 않습니다. 대신 ATMega323으로 포팅된 데모가 있는 데 그 데모를 기반으로 ATMega128에 포팅을 진행하도록 하겠습니다.

## 프로젝트 구성 과정

먼저 FreeRTOS 공식 사이트에서 FreeRTOS 파일을 [다운로드](https://sourceforge.net/projects/freertos/files/latest/download?source=files)하도록 합니다. 그리고 다운이 완료되면 적절한 위치에 다운로드 받은 파일을 실행시켜 압축을 설치하도록 합니다.

그런 다음에 [Atmel Studio 7.0](https://www.microchip.com/mplab/avr-support/atmel-studio-7)을 실행 시켜주도록 합니다. 화면이 뜨면 `File → new → project`를 클릭하고 `GCC C Executable Project`를 선택하여 적절한 이름을 지어주도록 합니다.

이러면 장치 선택 창이 뜰 것입니다. 여기서 반드시 ATMega128을 선택해주도록 합니다.

![Device Selection](/assets/img/res/2019-FreeRTOS/1/1.PNG)

제일 처음 할 일은 프로젝트의 **root(최상위 폴더)**에 `Source`라는 폴더를 만들어주도록 합니다.

다음으로 FreeRTOS를 앞서 설치한 위치의 `FreeRTOS/Source` 디렉토리를 찾아가도록 합니다.

```bash
drwxr-xr-x 1 sample 197609      0 5월  13 11:58 include/
drwxr-xr-x 1 sample 197609      0 5월  11 10:12 portable/
-rw-r--r-- 1 sample 197609  13177 5월  11 10:24 croutine.c
-rw-r--r-- 1 sample 197609  26791 5월  11 10:24 event_groups.c
-rw-r--r-- 1 sample 197609   8475 5월  11 10:24 list.c
-rw-r--r-- 1 sample 197609  96319 5월  11 10:24 queue.c
-rw-r--r-- 1 sample 197609    822 2월  18 02:38 readme.txt
-rw-r--r-- 1 sample 197609  43728 5월  11 10:24 stream_buffer.c
-rw-r--r-- 1 sample 197609 174827 5월  11 10:24 tasks.c
-rw-r--r-- 1 sample 197609  40685 5월  11 10:24 timers.c
```

위 디렉토리에서 `croutine.c ~ timers.c` 파일을 아까 **프로젝트 창**에서 만든 `Source` 폴더에 넣어주도록 합니다. 그리고 **프로젝트**의 `Source`의 하위 폴더로 `include`를 만들어주도록 합니다. 그리고 `FreeRTOS/Source` 안에 있는 `include` 폴더를 **프로젝트**의 `Source/include`에 넣어주도록 합니다.

그리고 **프로젝트**에 `Source`의 하위 폴더로 `portable` 폴더를 만들어주고, `FreeRTOS/Source/portable/GCC/ATMega323` 폴더 안의 내용은 `Source/portable/GCC/ATMega323`에 `FreeRTOS/Source/portable/MemMang`에 넣고 있는 **`heap_1.c`** 파일을 `Source/portable/MemMang`에 넣어주도록 합니다.

## 프로젝트 포팅 과정

마지막으로 `FreeRTOS/Demo/AVR_ATMega323_WinAVR/FreeRTOSConfig.h`를 루트 디렉토리에 넣어주도록 합니다. 해당 파일의 내용을 아래와 같이 만들어주도록 합니다.

```c
#ifndef FREERTOS_CONFIG_H
#define FREERTOS_CONFIG_H

#include <avr/io.h>

#define configUSE_PREEMPTION        1
#define configUSE_IDLE_HOOK         0
#define configUSE_TICK_HOOK         0
#define configCPU_CLOCK_HZ          ( ( unsigned long ) 7372800 )
#define configTICK_RATE_HZ          ( ( portTickType ) 1000 )
#define configMAX_PRIORITIES        ( /*( unsigned portBASE_TYPE )*/ 32 )
#define configMINIMAL_STACK_SIZE    ( ( unsigned short ) 85 )
#define configTOTAL_HEAP_SIZE       ( (size_t ) ( 3500 ) )
#define configMAX_TASK_NAME_LEN     ( 8 )
#define configUSE_TRACE_FACILITY    0
#define configUSE_16_BIT_TICKS      1
#define configIDLE_SHOULD_YIELD     1
#define configQUEUE_REGISTRY_SIZE   0

#define configUSE_CO_ROUTINES           0
#define configMAX_CO_ROUTINE_PRIORITIES ( 2 )

#define INCLUDE_vTaskPrioritySet        0
#define INCLUDE_uxTaskPriorityGet       0
#define INCLUDE_vTaskDelete             0
#define INCLUDE_vTaskCleanUpResources   0
#define INCLUDE_vTaskSuspend            0
#define INCLUDE_vTaskDelayUntil         1
#define INCLUDE_vTaskDelay              0


#endif /* FREERTOS_CONFIG_H */
```

정리하면 Solution은 아래와 같은 디렉토리 구조를 가져야만 합니다.

![solution directory](/assets/img/res/2019-FreeRTOS/1/2.PNG)

이제 프로젝트 설정을 하도록 하겠습니다. 

![find project properties](/assets/img/res/2019-FreeRTOS/1/3.PNG)

이때, 반드시  **`Release`** 설정이 되었는 지를 확인하도록 합니다.

> 이것을 반드시 해줘야 하는 이유는 이를 수행하지 않은 경우 높은 확률로 Segmentation Fault를 빌드 중에 발생시키기 때문입니다.

![Release](/assets/img/res/2019-FreeRTOS/1/3-1.PNG)

먼저 `Toolchain`에서 `symbol`에 `GCC_MEGA_AVR` 값을 아래와 같이 추가해주도록 합니다.

![add symbol](/assets/img/res/2019-FreeRTOS/1/4.PNG)

그리고 `Toolchain`의 `Directories`에서 Include Paths에 `Source/include`와 `Source/portable/GCC/ATMega323`과 **root 폴더(`..`으로 표기)**를  넣어주도록 합니다.

![set the library](/assets/img/res/2019-FreeRTOS/1/5.PNG)

다음으로 `Source/portable/GCC/ATMega323/port.c` 파일을 찾아가도록 합니다. 여기서 `SIG_OUTPUT_COMPARE1A`을 찾습니다. 이 방식은 오래된 방식이기 때문에 [문서](http://www.nongnu.org/avr-libc/user-manual/group__avr__interrupts.html)를 참고해서 최신 ATMega128에 맞는 규격인 `TIMER1_COMPA_vect`로 변경해주도록 합니다. 그러면 다음과 같은 파일이 될 것입니다.

```c
// port.c 파일
// (앞 부분 생략)
/*-----------------------------------------------------------*/

#if configUSE_PREEMPTION == 1
	// void SIG_OUTPUT_COMPARE1A( void ) __attribute__ ( ( signal, naked ) );
	// void SIG_OUTPUT_COMPARE1A( void )
	void TIMER1_COMPA_vect( void ) __attribute__ ( ( signal, naked ) );
	void TIMER1_COMPA_vect( void )
	{
		vPortYieldFromTick();
		asm volatile ( "reti" );
	}
#else
	// void SIG_OUTPUT_COMPARE1A( void ) __attribute__ ( ( signal ) );
	// void SIG_OUTPUT_COMPARE1A( void )
	void TIMER1_COMPA_vect( void ) __attribute__ ( ( signal ) );
	void TIMER1_COMPA_vect( void )
	{
		xTaskIncrementTick();
	}
#endif
```

여기까지 했으면 이제는 장치를 잡아주고

![set device](/assets/img/res/2019-FreeRTOS/1/6.PNG)

FreeRTOS를 가지고 [이것](https://github.com/BlaCkinkGJ/Atmega128-SimpleProject-FreeRTOS)과 같이 다양한 RTOS를 활용한 프로그램을 만드시면 됩니다.

## 실행 예제

2개의 LED를 PD0와 PD1을 사용하여 하나는 500ms 주기로, 다른 하나는 1s 주기로 껐다 켜는 예제입니다. 각각의 파일을 루트에 만들어주고 돌리면 됩니다.

### **LED.c**

```c
/*
 * LED.c
 *
 * Created: 2019-07-06 오후 2:55:28
 * Author : BlaCkinkGJ
 */ 
#include "FreeRTOS.h"
#include "task.h"
#include "LED.h"

void vLEDinit( volatile uint8_t *ddr, uint8_t bit ) {
    *ddr |= (1 << bit);
}

void vLEDset( volatile uint8_t *port, uint8_t bit, signed portBASE_TYPE xValue ) {
    if (xValue == pdTRUE) {
        *port &= ~(1 << bit);
    } else {
        *port |= (1 << bit);
    }
}

void vLEDtoggle( volatile uint8_t *port, uint8_t bit) {
    *port ^= (1 << bit);
}
```

### LED.h

```c
/*
 * LED.h
 *
 * Created: 2019-07-06 오후 2:55:28
 * Author : BlaCkinkGJ
 */ 
#ifndef LEDTEST_H
#define LEDTEST_H

#include <inttypes.h>

void vLEDinit( volatile uint8_t *ddr, uint8_t bit );
void vLEDset( volatile uint8_t *port, uint8_t bit, signed portBASE_TYPE xValue );
void vLEDtoggle( volatile uint8_t *port, uint8_t bit );

#endif

```

### main.c

```c
/*
 * main.c
 *
 * Created: 2019-07-06 오후 2:55:28
 * Author : BlaCkinkGJ
 */ 
#include <avr/io.h>
#include "FreeRTOS.h"
#include "task.h"
#include "LED.h"

#define mainLED_TASK_PRIORITY tskIDLE_PRIORITY

void vLEDFlashTask1(void *pvParameters) {
    portTickType xLastWakeTime;
    const portTickType xFrequency = 500;
    vLEDinit(&DDRD, 0);
    xLastWakeTime = xTaskGetTickCount();
    while(1) {
        vTaskDelayUntil(&xLastWakeTime, xFrequency);
        vLEDtoggle(&PORTD, 0);
    }
}

void vLEDFlashTask2(void *pvParameters) {
    portTickType xLastWakeTime;
    const portTickType xFrequency = 1000;
    xLastWakeTime = xTaskGetTickCount();
    vLEDinit(&DDRD, 1);
    while(1) {
        vTaskDelayUntil(&xLastWakeTime, xFrequency);
        vLEDtoggle(&PORTD, 1);
    }
}

portSHORT main(void){
    xTaskCreate(vLEDFlashTask1,
                (const char *)"LED1",
                configMINIMAL_STACK_SIZE,
                NULL,
                mainLED_TASK_PRIORITY,
                NULL);
    xTaskCreate(vLEDFlashTask2,
                (const char *)"LED2",
                configMINIMAL_STACK_SIZE,
                NULL,
                mainLED_TASK_PRIORITY,
                NULL);
    vTaskStartScheduler();
    
    while (1) { }
    return 0;
}
```

---

[^1]: https://m.eet.com/media/1246048/2017-embedded-market-study.pdf
[^2]: https://www.freertos.org/a00114.html
[^3]: 리눅스나 윈도우 운영체제에서 프로세스라고 불리는 것과 유사한 개념입니다.