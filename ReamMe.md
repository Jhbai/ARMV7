# Setup Enviroment
## 1. Simulator for STM32H7 ARMV6
### $ sudo apt install qemu-system-arm
It's Quick EMUlator, this is a open source of a simulator and virtualization software.
It simluates a hardware system, including
1. A ARM Cortext-M7
2. RAM
3. Peripherals, such as UART, Timer, Interrupt Controller
So, it can also simulate qemu-system-x86_64, qemu-system-mips ...etc.

### sudo apt install gcc-arm-none-eabi binutils-arm-none-eabi gdb-multiarch
gcc: GNU Compiler Collection
arm-none-eabi: this means the gcc target, none means "Bare Machine", eabi means "Embedded Application Binary Interface".
binutils: compile a program not only transform files into binary file, but a lot of tools used to dealing with binary files. including "as: assembler", "ld: linker", "objcopy: sw that makes .elf to .bin", "objdump: deassembly"
gdb-multiarch: gdb is a debugger, multiarch means multi-architecture.

## 2. A Project Structure
### startup.s
A start-up code(Assembly), this is the first program MCU after powered, its tasks including
1. IVT Set up
2. Stack Pointer
3. Call the C main program
### app.c

### stm32h7.ld
