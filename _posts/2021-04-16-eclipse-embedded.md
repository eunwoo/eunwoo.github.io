---
title: "semihosting.h"
categories: eclipse embedded
---
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

