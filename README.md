# Setup Enviroment
## 1. Simulator for STM32H7 ARMV6
### $ sudo apt install qemu-system-arm
It's Quick EMUlator, this is a open source of a simulator and virtualization software.
It simluates a hardware system, including
1. A ARM Cortext-M7
2. RAM
3. Peripherals, such as UART, Timer, Interrupt Controller
So, it can also simulate qemu-system-x86_64, qemu-system-mips ...etc.

### $ sudo apt install gcc-arm-none-eabi binutils-arm-none-eabi gdb-multiarch
1. gcc: GNU Compiler Collection
2. arm-none-eabi: this means the gcc target, none means "Bare Machine", eabi means "Embedded Application Binary Interface".
3. binutils: compile a program not only transform files into binary file, but a lot of tools used to dealing with binary files. including (a) "as: assembler", (b) "ld: linker", (c) "objcopy: sw that makes .elf to .bin", (d) "objdump: deassembly"
4. gdb-multiarch: gdb is a debugger, multiarch means multi-architecture.

## 2. A Project Structure
### startup.s
    @ startup.s
    .syntax unified
    .cpu cortex-m7
    .thumb
    .global Reset_Handler
    
    @ The Vector Table
    .section .isr_vector
    .word   0x20020000      @ Initial Stack Pointer Value (Gave it a bit more space)
    .word   Reset_Handler   @ 1. Reset Handler
    .word   NMI_Handler     @ 2. NMI Handler
    .word   HardFault_Handler @ 3. HardFault Handler
    .word   0               @ 4. MemManage Handler
    .word   0               @ 5. BusFault Handler
    .word   0               @ 6. UsageFault Handler
    .word   0               @ ... reserved
    .word   0
    .word   0
    .word   0
    .word   0               @ 11. SVCall Handler
    .word   0               @ 12. Debug Monitor Handler
    .word   0               @ 13. reserved
    .word   0               @ 14. PendSV Handler
    .word   0               @ 15. SysTick Handler
    .word   UART0_Handler   @ 16. IRQ0: UART0 Interrupt
    
    
    .section .text
    .thumb_func
    Reset_Handler:
        @ Call the main function in C
        bl      main
    
    @ After main returns (which it shouldn't), loop forever
    Loop:
        b       Loop
    
    @ A default handler for interrupts
    .thumb_func
    Default_Handler:
        b       .
    
    @ Use the .weak directive to create default handlers.
    @ If a function with the same name is defined in C,
    @ the linker will use that one instead.
    .weak NMI_Handler
    .weak HardFault_Handler
    .weak UART0_Handler
    
    NMI_Handler:
    HardFault_Handler:
    UART0_Handler:
        b Default_Handler


A start-up code(Assembly), this is the first program MCU after powered, its tasks including
1. IVT Set up
2. Stack Pointer
3. Call the C main program
### app.c
    volatile unsigned int *const UART_DR = (unsigned int *)0x40004000;
    void print_str(const char *s){
	    while(*s != '\0'){
		    *UART_DR = *s;
		    s++;
	    }
    }

    int main(void){
	    print_str("This is the test code, print string via UART");
	    return 0;
    }

    void UART0_Handler(void){
	    print_str("--- UART Interrupted Occured! --- \n");
    }


### stm32h7.ld
    ENTRY(Reset_Handler)
    
    MEMORY
    {
    	FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 128*1024
    	RAM (rwx) : ORIGIN = 0x20000000, LENGTH = 128*1024
    }
    
    SECTIONS
    {
    	.isr_vector :
    	{
    		KEEP(*(.isr_vector))
    	} > FLASH
    	
    	.text :
    	{
    		*(.text*)
    	} > FLASH
    	
    	.data :
    	{
    		*(.data*)
    	} > RAM AT > FLASH
    
    	.bss :
    	{
    		*(.bss*)
    	} > RAM
    }
Notice that linker script use "space" to identify elements, so "ORIGIN = 0x08000000" each blanks is necessary.
