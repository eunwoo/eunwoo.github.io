uos++
==
[https://github.com/micro-os-plus/micro-os-plus-iii-cortexm/blob/117e1bb2783b08b27d6bc14c520d21a7e0b26a0b/src/rtos/port/os-core.cpp](https://github.com/micro-os-plus/micro-os-plus-iii-cortexm/blob/117e1bb2783b08b27d6bc14c520d21a7e0b26a0b/src/rtos/port/os-core.cpp)  

FreeRTOS  
==
ARM Cortex-M7  
port.c
```

[https://mcuoneclipse.com/2016/08/28/arm-cortex-m-interrupts-and-freertos-part-3/](https://mcuoneclipse.com/2016/08/28/arm-cortex-m-interrupts-and-freertos-part-3/)  

void xPortPendSVHandler( void )
{
	/* This is a naked function. */

    __asm volatile
    (
    "   mrs r0, psp                         \n"
    "   isb                                 \n"
    "                                       \n"
    "   ldr r3, pxCurrentTCBConst           \n" /* Get the location of the current TCB. */
    "   ldr r2, [r3]                        \n"
    "                                       \n"
    "   tst r14, #0x10                      \n" /* Is the task using the FPU context?  If so, push high vfp registers. */
    "   it eq                               \n"
    "   vstmdbeq r0!, {s16-s31}             \n"
    "                                       \n"
    "   stmdb r0!, {r4-r11, r14}            \n" /* Save the core registers. */
    "   str r0, [r2]                        \n" /* Save the new top of stack into the first member of the TCB. */
    "                                       \n"
    "   stmdb sp!, {r0, r3}                 \n"
    "   mov r0, %0                          \n"
    "   msr basepri, r0                     \n"
    "   dsb                                 \n"
    "   isb                                 \n"
    "   bl vTaskSwitchContext               \n"
    "   mov r0, #0                          \n"
    "   msr basepri, r0                     \n"
    "   ldmia sp!, {r0, r3}                 \n"
    "                                       \n"
    "   ldr r1, [r3]                        \n" /* The first item in pxCurrentTCB is the task top of stack. */
    "   ldr r0, [r1]                        \n"
    "                                       \n"
    "   ldmia r0!, {r4-r11, r14}            \n" /* Pop the core registers. */
    "                                       \n"
    "   tst r14, #0x10                      \n" /* Is the task using the FPU context?  If so, pop the high vfp registers too. */
    "   it eq                               \n"
    "   vldmiaeq r0!, {s16-s31}             \n"
    "                                       \n"
    "   msr psp, r0                         \n"
    "   isb                                 \n"
    "                                       \n"
    #ifdef WORKAROUND_PMU_CM001 /* XMC4000 specific errata workaround. */
        #if WORKAROUND_PMU_CM001 == 1
    "           push { r14 }                \n"
    "           pop { pc }                  \n"
        #endif
    #endif
    "                                       \n"
    "   bx r14                              \n"
    "                                       \n"
    "   .align 4                            \n"
    "pxCurrentTCBConst: .word pxCurrentTCB  \n"
    ::"i"(configMAX_SYSCALL_INTERRUPT_PRIORITY)
    );
}
```

```
08003f00 <PendSV_Handler>:
 8003f00:	f3ef 8009 	mrs	r0, PSP
 8003f04:	f3bf 8f6f 	isb	sy
 8003f08:	4b15      	ldr	r3, [pc, #84]	; (8003f60 <pxCurrentTCBConst>)
 8003f0a:	681a      	ldr	r2, [r3, #0]
 8003f0c:	f01e 0f10 	tst.w	lr, #16
 8003f10:	bf08      	it	eq
 8003f12:	ed20 8a10 	vstmdbeq	r0!, {s16-s31}
 8003f16:	e920 4ff0 	stmdb	r0!, {r4, r5, r6, r7, r8, r9, sl, fp, lr}
 8003f1a:	6010      	str	r0, [r2, #0]
 8003f1c:	e92d 0009 	stmdb	sp!, {r0, r3}
 8003f20:	f04f 0050 	mov.w	r0, #80	; 0x50
 8003f24:	f380 8811 	msr	BASEPRI, r0
 8003f28:	f3bf 8f4f 	dsb	sy
 8003f2c:	f3bf 8f6f 	isb	sy
 8003f30:	f7ff fb7e 	bl	8003630 <vTaskSwitchContext>
 8003f34:	f04f 0000 	mov.w	r0, #0
 8003f38:	f380 8811 	msr	BASEPRI, r0
 8003f3c:	bc09      	pop	{r0, r3}
 8003f3e:	6819      	ldr	r1, [r3, #0]
 8003f40:	6808      	ldr	r0, [r1, #0]
 8003f42:	e8b0 4ff0 	ldmia.w	r0!, {r4, r5, r6, r7, r8, r9, sl, fp, lr}
 8003f46:	f01e 0f10 	tst.w	lr, #16
 8003f4a:	bf08      	it	eq
 8003f4c:	ecb0 8a10 	vldmiaeq	r0!, {s16-s31}
 8003f50:	f380 8809 	msr	PSP, r0
 8003f54:	f3bf 8f6f 	isb	sy
 8003f58:	4770      	bx	lr
 8003f5a:	bf00      	nop
 8003f5c:	f3af 8000 	nop.w
```

mrs r0, psp
==
mrs: move to (general) register from special register  
r0 = psp

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


it eq  
==
p.236  
IfThen 명령  
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

vstmdbeq	r0!, {s16-s31}  
==
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


