
타겟보드
==
NUCLEO144 STM32H743ZI2  
https://www.st.com/en/evaluation-tools/nucleo-h743zi.html  
타겟보드는 STM32CubeIDE를 사용하여 다운로드와 디버깅이 가능한데, Eclipse로도 컴파일/다운로드/디버깅이 가능한지 확인해봄.  

LED Port 번호  
==
Mbed Studio에서는 NUCLEO144 STM32H743ZI2를 지원한다. 소스를 보면 아래와 같은 부분이 있다.  
- {Project Root}/targets/TARGET_STM/TARGET_STM32H7/TARGET_STM32H743xI/TARGET_NUCLEO_H743ZI2/PinNames.h
```
    // Generic signals namings
    LED1        = PB_0,  // LD1 = GREEN
    LED2        = PE_1,  // Yellow
    LED3        = PB_14, // Red
```
User Input
==
```
    // Standardized button names
    USER_BUTTON = PC_13,
    BUTTON1 = USER_BUTTON,
```

Eclipse 버전
==
![Image Alt 텍스트]({{site.url}}/assets/img/eclipse.png )


링커스크립트 수정
==
STM32CubeIDE에서 사용한 ld파일을 참고하여, mem.ld파일을 수정  

프로젝트 생성시  
```
MEMORY
{
  FLASH (rx) : ORIGIN = 0x00000000, LENGTH = 2048K
  RAM (xrw) : ORIGIN = 0x20000000, LENGTH = 1024K

  /*
   * Optional sections; define the origin and length to match
   * the the specific requirements of your hardware. The zero
   * length prevents inadvertent allocation.
   */
  CCMRAM (xrw) : ORIGIN = 0x10000000, LENGTH = 0
  FLASHB1 (rx) : ORIGIN = 0x00000000, LENGTH = 0
  EXTMEMB0 (rx) : ORIGIN = 0x00000000, LENGTH = 0
  EXTMEMB1 (rx) : ORIGIN = 0x00000000, LENGTH = 0
  EXTMEMB2 (rx) : ORIGIN = 0x00000000, LENGTH = 0
  EXTMEMB3 (rx) : ORIGIN = 0x00000000, LENGTH = 0
}
```
FLASH 주소를 0x08000000으로 수정
RAM 주소를 세분화하여 수정

```
MEMORY
{
  FLASH (rx)     : ORIGIN = 0x08000000, LENGTH = 2048K
  RAM (xrw)      : ORIGIN = 0x20000000, LENGTH = 128K
  RAM_D1 (xrw)   : ORIGIN = 0x24000000, LENGTH = 512K
  RAM_D2 (xrw)   : ORIGIN = 0x30000000, LENGTH = 288K
  RAM_D3 (xrw)   : ORIGIN = 0x38000000, LENGTH = 64K
  ITCMRAM (xrw)  : ORIGIN = 0x00000000, LENGTH = 64K

  /*
   * Optional sections; define the origin and length to match
   * the the specific requirements of your hardware. The zero
   * length prevents inadvertent allocation.
   */
  CCMRAM (xrw) : ORIGIN = 0x10000000, LENGTH = 0
  FLASHB1 (rx) : ORIGIN = 0x00000000, LENGTH = 0
  EXTMEMB0 (rx) : ORIGIN = 0x00000000, LENGTH = 0
  EXTMEMB1 (rx) : ORIGIN = 0x00000000, LENGTH = 0
  EXTMEMB2 (rx) : ORIGIN = 0x00000000, LENGTH = 0
  EXTMEMB3 (rx) : ORIGIN = 0x00000000, LENGTH = 0
}
```
아래 메모리맵은 743이 아니고 742에 대한 메모리 맵임. 743은 RAM크기가 다름.  
![Image Alt 텍스트]({{site.url}}/assets/img/stm32h742 memory map.png )  

742, 743 메모리맵 비교  
![Image Alt 텍스트]({{site.url}}/assets/img/stm32h742,743_memory_map.png )  


