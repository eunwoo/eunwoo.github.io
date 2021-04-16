---
title: "semihosting.h"
categories: eclipse embedded
---

semihosting.h
```
static inline int
__attribute__ ((always_inline))
call_host (int reason, void* arg)
{
  int value;
  asm volatile (

      " mov r0, %[rsn]  \n"
      " mov r1, %[arg]  \n"
#if defined(OS_DEBUG_SEMIHOSTING_FAULTS)
      " " AngelSWITestFault " \n"
#else
      " " AngelSWIInsn " %[swi] \n"
#endif
      " mov %[val], r0"

      : [val] "=r" (value) /* Outputs */
      : [rsn] "r" (reason), [arg] "r" (arg), [swi] "i" (AngelSWI) /* Inputs */
      : "r0", "r1", "r2", "r3", "ip", "lr", "memory", "cc"
      // Clobbers r0 and r1, and lr if in supervisor mode
  );

  // Accordingly to page 13-77 of ARM DUI 0040D other registers
  // can also be clobbered. Some memory positions may also be
  // changed by a system call, so they should not be kept in
  // registers. Note: we are assuming the manual is right and
  // Angel is respecting the APCS.
  return value;
}
```

trace-impl.o.lst
```
  55              		.loc 2 99 1 view .LVU6
  56              	.LBB9:
 100:../system/include/arm/semihosting.h **** {
 101:../system/include/arm/semihosting.h ****   int value;
  57              		.loc 2 101 3 view .LVU7
 102:../system/include/arm/semihosting.h ****   asm volatile (
  58              		.loc 2 102 3 view .LVU8
  59 000e 0425     		movs	r5, #4
  60              		.syntax unified
  61              	@ 102 "../system/include/arm/semihosting.h" 1
  62 0010 2846     		 mov r0, r5  
  63 0012 2146     	 mov r1, r4  
  64 0014 ABBE     	 bkpt #171 
  65 0016 0446     	 mov r4, r0
  66              	@ 0 "" 2
  67              	.LVL2:
```


  
