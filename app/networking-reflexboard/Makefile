#
#	This makefile produces a simple binary that tests the STM32 peripherals
#

CROSS_COMPILE=arm-none-eabi-
QEMU_STM32 ?= ../../../RTOS/qemu_stm32/arm-softmmu/qemu-system-arm
#QEMU_STM32 ?= qemu-system-arm

ARCH=CM3
VENDOR=ST
PLAT=STM32F4x7
#CMSIS_LIB=../../rtos/CMSIS/$(ARCH)
CMSIS_LIB=../../rtos/CMSIS/STM32F4x7
#STM32_LIB=../../rtos/STM32F10x_StdPeriph_Driver
STM32_LIB=../../rtos/STM32F4xx_StdPeriph_Driver

CMSIS_PLAT_SRC = $(CMSIS_LIB)/Device/ST/STM32F4xx/Source/Templates

FREERTOS_SRC = ../../rtos/FreeRTOS
FREERTOS_INC = $(FREERTOS_SRC)/include/                                       
FREERTOS_PORT_INC = $(FREERTOS_SRC)/portable/GCC/ARM_$(ARCH)/

all: app

app: src/main.c
	$(CROSS_COMPILE)gcc \
		-Wl,-Tsrc/stm32f427.ld -nostartfiles \
		-I. \
		-Iheaders\
		-I$(FREERTOS_INC) -I$(FREERTOS_PORT_INC) \
		-I$(CMSIS_LIB)/Include \
		-I$(CMSIS_LIB)/Device/ST/STM32F4xx/Include \
		-I../../rtos/STM32F4xx_StdPeriph_Driver/inc \
		-fno-common -O0 \
		-gdwarf-2 -g3 \
		-mcpu=cortex-m3 -mthumb -std=c99 \
		-Wl,-Map=main.map,--cref \
		-o main.elf \
		\
		$(CMSIS_PLAT_SRC)/system_stm32f4xx.c \
		$(CMSIS_PLAT_SRC)/gcc_ride7/startup_stm32f4xx.s \
		$(STM32_LIB)/src/stm32f4xx_rcc.c \
		$(STM32_LIB)/src/stm32f4xx_gpio.c \
		$(STM32_LIB)/src/stm32f4xx_usart.c \
		$(STM32_LIB)/src/stm32f4xx_exti.c \
		$(STM32_LIB)/src/misc.c \
		\
		$(FREERTOS_SRC)/croutine.c \
		$(FREERTOS_SRC)/list.c \
		$(FREERTOS_SRC)/queue.c \
		$(FREERTOS_SRC)/tasks.c \
		$(FREERTOS_SRC)/portable/GCC/ARM_CM3/port.c \
		$(FREERTOS_SRC)/portable/MemMang/heap_1.c \
		\
		src/enet.c \
		src/stubs.c \
		src/console.c \
		src/main.c
	$(CROSS_COMPILE)objcopy -Obinary main.elf main.bin
	$(CROSS_COMPILE)objdump -S main.elf > main.list

qemu: main.bin $(QEMU_STM32)
	$(QEMU_STM32) -M stm32-p103 -kernel main.bin

qemudbg: main.bin $(QEMU_STM32)
	$(QEMU_STM32) -M stm32-p103 \
		-gdb tcp::3333 -S \
		-kernel main.bin

gdb-qemu: main.bin $(QEMU_STM32)
	$(QEMU_STM32) -M stm32-p103 \
		-gdb tcp::3333 -S \
		-kernel main.elf

qemuauto: main.bin
	bash emulate.sh main.bin

emu: main.bin
	bash emulate.sh main.bin

clean:
	rm -f *.elf *.bin *.list *.map

OPENOCD=../../tools/openocd/bin/openocd
program: app
	$(OPENOCD) -f ../../tools/olimex-arm-usb-tiny-h.cfg -f ../../tools/stm32.cfg -c "init; halt; flash write_image erase main.bin 0x8000000; verify_image main.bin 0x8000000; sleep 100; reset run; sleep 100; shutdown"

dump_flash:
	$(OPENOCD) -f ../../tools/olimex-arm-usb-tiny-h.cfg -f ../../tools/stm32.cfg -c "init; halt; dump_image main.bin.dump 0x8000000 0x1000; reset run; shutdown"

gdb_server: app
	$(OPENOCD) -f ../../tools/olimex-arm-usb-tiny-h.cfg -f ../../tools/stm32.cfg -c "init; halt"
	
gdb: app
	$(CROSS_COMPILE)gdb main.elf -x ../../tools/gdb_init_commands

reset:
	$(OPENOCD) -f ../../tools/olimex-arm-usb-tiny-h.cfg -f ../../tools/stm32.cfg -c "init; halt; reset run; shutdown"
