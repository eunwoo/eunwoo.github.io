---
title: "Scheduler in Embedded RTOS"
categories: IDE
---

uos++
==
[https://github.com/micro-os-plus/micro-os-plus-iii-cortexm/blob/117e1bb2783b08b27d6bc14c520d21a7e0b26a0b/src/rtos/port/os-core.cpp](https://github.com/micro-os-plus/micro-os-plus-iii-cortexm/blob/117e1bb2783b08b27d6bc14c520d21a7e0b26a0b/src/rtos/port/os-core.cpp)  

FreeRTOS  
==
ARM Cortex-M7을 사용하는 STM32H743ZI device를 가진 NUCLEO보드에서 STM32CubeIDE를 사용하여 FreeRTOS를 MiddleWare로 추가했을 때 자동생성되는 port.c파일을 분석  

PendSV 인터럽트를 알아야 함  
[https://mcuoneclipse.com/2016/08/28/arm-cortex-m-interrupts-and-freertos-part-3/](https://mcuoneclipse.com/2016/08/28/arm-cortex-m-interrupts-and-freertos-part-3/)  

아래 링크에 스케쥴러 코드에 대해 설명이 나온다. isb나 dsb가 코드에서 불필요하게 사용되었다는 내용도 있다.    
[https://interrupt.memfault.com/blog/cortex-m-rtos-context-switching](https://interrupt.memfault.com/blog/cortex-m-rtos-context-switching)  


- port.c  
```
01void xPortPendSVHandler( void )
02{
03	/* This is a naked function. */
04
05    __asm volatile
06    (
07    "   mrs r0, psp                         \n"
08    "   isb                                 \n"
09    "                                       \n"
10    "   ldr r3, pxCurrentTCBConst           \n" /* Get the location of the current TCB. */
11    "   ldr r2, [r3]                        \n"
12    "                                       \n"
13    "   tst r14, #0x10                      \n" /* Is the task using the FPU context?  If so, push high vfp registers. */
14    "   it eq                               \n"
15    "   vstmdbeq r0!, {s16-s31}             \n"
16    "                                       \n"
17    "   stmdb r0!, {r4-r11, r14}            \n" /* Save the core registers. */
18    "   str r0, [r2]                        \n" /* Save the new top of stack into the first member of the TCB. */
19    "                                       \n"
20    "   stmdb sp!, {r0, r3}                 \n" /* sp is msp(main stack pointer), not psp(process stack pointer), r0는 psp, r3은  */
21    "   mov r0, %0                          \n"
22    "   msr basepri, r0                     \n"
23    "   dsb                                 \n"
24    "   isb                                 \n"
25    "   bl vTaskSwitchContext               \n"
26    "   mov r0, #0                          \n"
27    "   msr basepri, r0                     \n"
28    "   ldmia sp!, {r0, r3}                 \n"
29    "                                       \n"
30    "   ldr r1, [r3]                        \n" /* The first item in pxCurrentTCB is the task top of stack. */
31    "   ldr r0, [r1]                        \n"
32    "                                       \n"
33    "   ldmia r0!, {r4-r11, r14}            \n" /* Pop the core registers. */
34    "                                       \n"
35    "   tst r14, #0x10                      \n" /* Is the task using the FPU context?  If so, pop the high vfp registers too. */
36    "   it eq                               \n"
37    "   vldmiaeq r0!, {s16-s31}             \n"
38    "                                       \n"
39    "   msr psp, r0                         \n"
40    "   isb                                 \n"
41    "                                       \n"
42    #ifdef WORKAROUND_PMU_CM001 /* XMC4000 specific errata workaround. */
43        #if WORKAROUND_PMU_CM001 == 1
44    "           push { r14 }                \n"
45    "           pop { pc }                  \n"
46        #endif
47    #endif
48    "                                       \n"
49    "   bx r14                              \n"
50    "                                       \n"
51    "   .align 4                            \n"
52    "pxCurrentTCBConst: .word pxCurrentTCB  \n"
53    ::"i"(configMAX_SYSCALL_INTERRUPT_PRIORITY)
54    );
55}
```

07 ~ 18 : context save and update stack bottom address (컨텍스트 저장 및 스택 최하위 주소 업데이트)
==
```
07    "   mrs r0, psp                         \n"
08    "   isb                                 \n"
09    "                                       \n"
10    "   ldr r3, pxCurrentTCBConst           \n" /* Get the location of the current TCB. */
11    "   ldr r2, [r3]                        \n"
12    "                                       \n"
13    "   tst r14, #0x10                      \n" /* Is the task using the FPU context?  If so, push high vfp registers. */
14    "   it eq                               \n"
15    "   vstmdbeq r0!, {s16-s31}             \n"
16    "                                       \n"
17    "   stmdb r0!, {r4-r11, r14}            \n" /* Save the core registers. */
18    "   str r0, [r2]                        \n" /* Save the new top of stack into the first member of the TCB. */
```
mrs: move to (general) register from special register  
r0 := psp
psp의 값을 r0에 대입. psp는 process stack이며, thread mode에서 동작할 때 사용되는 스택이다.  
cortex-m의 동작모드는 thread mode와 handler mode가 있다. thread mode는 사용자 프로그램이 동작하는 모드이며  
handler mode는 operating system이 동작하는 모드이다.  
r0에 psp를 읽어 오는 이유는 사용자 프로그램의 컨텍스트인 레지스터를 저장하기 위해서다.  
인터럽트나 예외가 발생했을 때, cortex-m은 r0~r3까지 스택에 저장하나 r4부터는 스택에 저장하지 않기 때문에  
저장하는 코드를 추가해 주어야 한다.  
![Image]({{site.url}}/assets/img/contextsave1.png )  
인터럽트 서비스 루틴(Interrupt Service Routine, ISR 또는 Interrupt Handler)에서는 ISR에서 사용하는 레지스터만 스택에 저장하고 리턴하기 전에 복구하면 되지만 컨텍스트 저장인 경우에는 일반적으로 Thread에서 모든 레지스터를 사용하므로 모든 레지스터를 쓰레드 스택에 저장해 놓은 후에 쓰레드가 다시 시작될 때 복구해 주어야 한다. 아래 사이트에 Q&A 참고.  
[https://community.arm.com/developer/tools-software/tools/f/keil-forum/25373/cortex-saving-registers-during-interrupts](https://community.arm.com/developer/tools-software/tools/f/keil-forum/25373/cortex-saving-registers-during-interrupts)  

it eq  
--
p.236  
<span style="color:red">*I*</span>f<span style="color:red">*T*</span>hen 명령  
- it 명령어 다음에 오는 최대 4개까지의 명령어를 조건부로 실행되도록 함   
- it 블록: it명령어 다음에 오는 조건부 수행 명령어를 it 블록이라고 함  
- it 블록 내의 명령어로 분기할 수 없음. 즉, 다른 명령어를 수행 중 IT블록으로 분기하는 것은 안됨. 단, 예외에서 복귀하는 것은 가능  
- CMP, CMN, TST 외의 다른 명령어는 조건플래그를 설정하지 않음.  
IT{x{y{z}}}\<q\> \<firstcond\>  
x: 두번째 명령어의 조건을 지정  
y: 세번째 명령어의 조건을 지정  
z: 네번째 명령어의 조건을 지정  
\<x\>, \<y\>, \<z\>는 T 또는 E 가능. T=Then, E=Else를 의미.  
T \<firstcond\> 조건을 만족하면 실행.  
E \<firstcond\> 조건을 만족하지 않으면 실행.  
<q>  
	.N narrow, 16-bit 명령어 생성  
	.W wide, 32-bit 명령어 생성  
둘 다 지정되지 않으면, 16-bit 명령어 생성  

eq 조건은 PSR(Program Status Register)의 Z flag가 0일 때 참이 된다.  tst r14, #0x10 명령은 r15를 0x10과 AND연산했을 때,  
결과가 0이면 Z flag가 0이 된다.  따라서 r14의 bit4가 0이면 vstmdbeq 가 수행된다.  
r14의 bit4가 0이면 floating point register가 사용되었다는 것을 나타내기 때문에 레지스터를 저장한다.  

vstmdbeq	r0!, {s16-s31}  
--
p.499  
VSTM{\<mode\>}{\<c\>}{\<q\>}{.\<size\>} \<Rn\>{!}, \<list\>  
\<mode\>  
  IA  Increment After  
  DB  Decrement Before. Stack에 저장시 주소가 감소해야 하므로 이 설정 사용
\<Rn\>  base register  
!  이 명령을 실행시 \<Rn\>값이 변하도록 함. \<mode\>가 DB인 경우, 필요  
\<c\>    condition flag. 생략하면 always (AL)을 나타냄.  

ed20 8a10 	vstmdbeq	r0!, {s16-s31}  
P = 1, U = 0, D = 0, W = 1  
Vd = 0x08, imm8 = 0x10  
d = UInt(Vd:D) = UInt(0x08:0) = 0x10 = 16  저장할 레지스터 목록의 첫번째 레지스터 번호
regs = UInt(imm8) = 0x10 = 16 저장할 레지스터의 개수  
imm32 = ZeroExtend(imm8:'00', 32) =  0x40 = 64 하위에 2비트를 0을 추가한 후, 32비트로 확장, 어드레스 감소값  
저장할 레지스터의 개수가 16개이고, 각각이 4바이트이므로, 어드레스를 64 감소함.  
![Image Alt Exception return]({{site.url}}/assets/img/VSTM1.png )  	
![Image Alt Exception return]({{site.url}}/assets/img/VSTM2.png )  	

[https://developer.arm.com/documentation/dui0646/c/the-cortex-m7-processor/exception-model/exception-entry-and-return](https://developer.arm.com/documentation/dui0646/c/the-cortex-m7-processor/exception-model/exception-entry-and-return)  
![Image Alt Exception return]({{site.url}}/assets/img/exception return4.png )  

s0~s15는 자동으로 저장되므로, s16~s31을 저장한다.  

20줄
==
```
stmdb sp!, {r0, r3}  
```
r0와 r3를 main stack에 저장한다. sp는 process stack이 아니라 main stack를 나타낸다.  
process stack은 


52줄 pxCurrentTCBConst: .word pxCurrentTCB  
==
pxCurrentTCBConst 심볼은 08005bd0 주소를 나타내며, 이 주소에 저장된 값은 .word로 지정되어 있기 때문에 4바이트이고 pxCurrentTCB 변수의 주소를 나타낸다.  
이는 당연한데 pxCurrentTCB는 변수이기 때문에 프로그램 실행 중 계속 변하는 값이며, 컴파일시에 pxCurrentTCB의 값은 0으로 초기화되어 있으며,  
필요한 값은 프로그램 실행 중에 변하는 pxCurrentTCB의 값이기 때문에 pxCurrentTCB의 주소가 필요하기 때문이다.  
주소를 알면 주소를 적절한 어셈블리 명령을 사용하여 해당 변수의 값을 읽어 올 수 있다.  
map파일을 보면 pxCurrentTCB 변수(심볼)의 주소는 0x200007ac이며, list파일을 보면 이 값이 pxCurrentTCBConst 심볼에 저장되어 있음을 볼 수 있다.  
c언어로 표현하면 pxCurrentTCBConst = &pxCurrentTCB; 로 나타낼 수 있다.  
ldr r2, \[r3\]는 r2 = \*pxCurrentTCBConst;이고 r2 = pxCurrentTCB가 된다.  
r0에 새로운 stack top 주소를 계산한 후, str r0, \[r2\]를 수행하여 \*pxCurrentTCB = r0 가 되어,
r0를 pxCurrentTCB->pxTopOfStack에 저장한다.  

```
typedef struct tskTaskControlBlock 			/* The old naming convention is used to prevent breaking kernel aware debuggers. */
{
	volatile StackType_t	*pxTopOfStack;	/*< Points to the location of the last item placed on the tasks stack.  THIS MUST BE THE FIRST MEMBER OF THE TCB STRUCT. */

	#if ( portUSING_MPU_WRAPPERS == 1 )
		xMPU_SETTINGS	xMPUSettings;		/*< The MPU settings are defined as part of the port layer.  THIS MUST BE THE SECOND MEMBER OF THE TCB STRUCT. */
	#endif

	ListItem_t			xStateListItem;	/*< The list that the state list item of a task is reference from denotes the state of that task (Ready, Blocked, Suspended ). */
	ListItem_t			xEventListItem;		/*< Used to reference a task from an event list. */
	UBaseType_t			uxPriority;			/*< The priority of the task.  0 is the lowest priority. */
	StackType_t			*pxStack;			/*< Points to the start of the stack. */
	char				pcTaskName[ configMAX_TASK_NAME_LEN ];/*< Descriptive name given to the task when created.  Facilitates debugging only. */ /*lint !e971 Unqualified char types are allowed for strings and single characters only. */

	#if ( ( portSTACK_GROWTH > 0 ) || ( configRECORD_STACK_HIGH_ADDRESS == 1 ) )
		StackType_t		*pxEndOfStack;		/*< Points to the highest valid address for the stack. */
	#endif

	#if ( portCRITICAL_NESTING_IN_TCB == 1 )
		UBaseType_t		uxCriticalNesting;	/*< Holds the critical section nesting depth for ports that do not maintain their own count in the port layer. */
	#endif

	#if ( configUSE_TRACE_FACILITY == 1 )
		UBaseType_t		uxTCBNumber;		/*< Stores a number that increments each time a TCB is created.  It allows debuggers to determine when a task has been deleted and then recreated. */
		UBaseType_t		uxTaskNumber;		/*< Stores a number specifically for use by third party trace code. */
	#endif

	#if ( configUSE_MUTEXES == 1 )
		UBaseType_t		uxBasePriority;		/*< The priority last assigned to the task - used by the priority inheritance mechanism. */
		UBaseType_t		uxMutexesHeld;
	#endif

	#if ( configUSE_APPLICATION_TASK_TAG == 1 )
		TaskHookFunction_t pxTaskTag;
	#endif

	#if( configNUM_THREAD_LOCAL_STORAGE_POINTERS > 0 )
		void			*pvThreadLocalStoragePointers[ configNUM_THREAD_LOCAL_STORAGE_POINTERS ];
	#endif

	#if( configGENERATE_RUN_TIME_STATS == 1 )
		uint32_t		ulRunTimeCounter;	/*< Stores the amount of time the task has spent in the Running state. */
	#endif

	#if ( configUSE_NEWLIB_REENTRANT == 1 )
		/* Allocate a Newlib reent structure that is specific to this task.
		Note Newlib support has been included by popular demand, but is not
		used by the FreeRTOS maintainers themselves.  FreeRTOS is not
		responsible for resulting newlib operation.  User must be familiar with
		newlib and must provide system-wide implementations of the necessary
		stubs. Be warned that (at the time of writing) the current newlib design
		implements a system-wide malloc() that must be provided with locks.

		See the third party link http://www.nadler.com/embedded/newlibAndFreeRTOS.html
		for additional information. */
		struct	_reent xNewLib_reent;
	#endif

	#if( configUSE_TASK_NOTIFICATIONS == 1 )
		volatile uint32_t ulNotifiedValue;
		volatile uint8_t ucNotifyState;
	#endif

	/* See the comments in FreeRTOS.h with the definition of
	tskSTATIC_AND_DYNAMIC_ALLOCATION_POSSIBLE. */
	#if( tskSTATIC_AND_DYNAMIC_ALLOCATION_POSSIBLE != 0 ) /*lint !e731 !e9029 Macro has been consolidated for readability reasons. */
		uint8_t	ucStaticallyAllocated; 		/*< Set to pdTRUE if the task is a statically allocated to ensure no attempt is made to free the memory. */
	#endif

	#if( INCLUDE_xTaskAbortDelay == 1 )
		uint8_t ucDelayAborted;
	#endif

	#if( configUSE_POSIX_ERRNO == 1 )
		int iTaskErrno;
	#endif

} tskTCB;

typedef tskTCB TCB_t;

PRIVILEGED_DATA TCB_t * volatile pxCurrentTCB = NULL;
```

-led_test.map  
```
.bss.pxCurrentTCB
                0x00000000200007ac        0x4 Middlewares/Third_Party/FreeRTOS/Source/tasks.o
                0x00000000200007ac                pxCurrentTCB
```

-led_test.list  
```
08005b70 <PendSV_Handler>:

void xPortPendSVHandler( void )
{
	/* This is a naked function. */

	__asm volatile
 8005b70:	f3ef 8009 	mrs	r0, PSP
 8005b74:	f3bf 8f6f 	isb	sy
 8005b78:	4b15      	ldr	r3, [pc, #84]	; (8005bd0 <pxCurrentTCBConst>)
 8005b7a:	681a      	ldr	r2, [r3, #0]
 8005b7c:	f01e 0f10 	tst.w	lr, #16
 8005b80:	bf08      	it	eq
 8005b82:	ed20 8a10 	vstmdbeq	r0!, {s16-s31}
 8005b86:	e920 4ff0 	stmdb	r0!, {r4, r5, r6, r7, r8, r9, sl, fp, lr}
 8005b8a:	6010      	str	r0, [r2, #0]
 8005b8c:	e92d 0009 	stmdb	sp!, {r0, r3}
 8005b90:	f04f 0050 	mov.w	r0, #80	; 0x50
 8005b94:	f380 8811 	msr	BASEPRI, r0
 8005b98:	f3bf 8f4f 	dsb	sy
 8005b9c:	f3bf 8f6f 	isb	sy
 8005ba0:	f7fe ffc4 	bl	8004b2c <vTaskSwitchContext>
 8005ba4:	f04f 0000 	mov.w	r0, #0
 8005ba8:	f380 8811 	msr	BASEPRI, r0
 8005bac:	bc09      	pop	{r0, r3}
 8005bae:	6819      	ldr	r1, [r3, #0]
 8005bb0:	6808      	ldr	r0, [r1, #0]
 8005bb2:	e8b0 4ff0 	ldmia.w	r0!, {r4, r5, r6, r7, r8, r9, sl, fp, lr}
 8005bb6:	f01e 0f10 	tst.w	lr, #16
 8005bba:	bf08      	it	eq
 8005bbc:	ecb0 8a10 	vldmiaeq	r0!, {s16-s31}
 8005bc0:	f380 8809 	msr	PSP, r0
 8005bc4:	f3bf 8f6f 	isb	sy
 8005bc8:	4770      	bx	lr
 8005bca:	bf00      	nop
 8005bcc:	f3af 8000 	nop.w

08005bd0 <pxCurrentTCBConst>:
 8005bd0:	200007ac 	.word	0x200007ac
	"										\n"
	"	.align 4							\n"
	"pxCurrentTCBConst: .word pxCurrentTCB	\n"
	::"i"(configMAX_SYSCALL_INTERRUPT_PRIORITY)
	);
}
```




psp: process stack  

reference manual p.516  
![Image Alt 텍스트]({{site.url}}/assets/img/sp register.png )  
![Image Alt 텍스트]({{site.url}}/assets/img/AAPCS1.png )  

Thread mode  

RAZ/WI  
SBZP  

p.530 B1.5.5 Reset behavior
p.531 B1.5.6 Exception entry behavior
![Image Alt Exception entry]({{site.url}}/assets/img/exception entry1.png )  
p.539 B1.5.8 Exception return behavior  
![Image Alt Exception return]({{site.url}}/assets/img/exception return1.png )  
![Image Alt Exception return]({{site.url}}/assets/img/exception return2.png )  
![Image Alt Exception return]({{site.url}}/assets/img/exception return3.png )  

ISB instruction
==
p.59 Instruction Synchronization Barrier  

![Image Alt Exception return]({{site.url}}/assets/img/ISB1.png )  


	
[Some Link]({% post_url 2021-04-16-eclipse-embedded-nucleo144-stm32h743zi2 %})


```
void vPortSVCHandler( void )
{
    __asm volatile (
                    "   ldr r3, pxCurrentTCBConst2      \n" /* Restore the context. */
                    "   ldr r1, [r3]                    \n" /* Use pxCurrentTCBConst to get the pxCurrentTCB address. */
                    "   ldr r0, [r1]                    \n" /* The first item in pxCurrentTCB is the task top of stack. */
                    "   ldmia r0!, {r4-r11, r14}        \n" /* Pop the registers that are not automatically saved on exception entry and the critical nesting count. */
                    "   msr psp, r0                     \n" /* Restore the task stack pointer. */
                    "   isb                             \n"
                    "   mov r0, #0                      \n"
                    "   msr basepri, r0                 \n"
                    "   bx r14                          \n"
                    "                                   \n"
                    "   .align 4                        \n"
                    "pxCurrentTCBConst2: .word pxCurrentTCB             \n"
                );
}
```


```
#define portNVIC_INT_CTRL_REG		( * ( ( volatile uint32_t * ) 0xe000ed04 ) )
#define portNVIC_PENDSVSET_BIT		( 1UL << 28UL )

void xPortSysTickHandler( void )
{
	/* The SysTick runs at the lowest interrupt priority, so when this interrupt
	executes all interrupts must be unmasked.  There is therefore no need to
	save and then restore the interrupt mask value as its value is already
	known. */
	portDISABLE_INTERRUPTS();
	{
		/* Increment the RTOS tick. */
		if( xTaskIncrementTick() != pdFALSE )
		{
			/* A context switch is required.  Context switching is performed in
			the PendSV interrupt.  Pend the PendSV interrupt. */
			portNVIC_INT_CTRL_REG = portNVIC_PENDSVSET_BIT;
		}
	}
	portENABLE_INTERRUPTS();
}
```
![Image]({{site.url}}/assets/img/scheduler-freertos-arm-icsr.png )  


