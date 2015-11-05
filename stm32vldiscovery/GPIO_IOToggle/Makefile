STM32CUBE = /home/vmatos/stm32/STM32CubeF1

DEVICE_FAMILY = STM32F1xx
DEVICE_TYPE = STM32F100xB
LDSCRIPT = STM32F100XB_FLASH.ld

CMSIS = $(STM32CUBE)/Drivers/CMSIS
CMSIS_DEVSUP = $(CMSIS)/Device/ST/$(DEVICE_FAMILY)
CMSIS_OPT = -D$(DEVICE_TYPE) -DUSE_HAL_DRIVER
CPU = -mthumb -mcpu=cortex-m3 -msoft-float -MD
PREFIX = arm-none-eabi

SRCDIR = Src
INCDIR = Inc

LIBSRC = $(STM32CUBE)/Drivers/STM32F1xx_HAL_Driver/Src
LIBINC = -I$(CMSIS)/Include
LIBINC += -I$(CMSIS)/Device/ST/STM32F1xx/Include
LIBINC += -I$(STM32CUBE)/Drivers/STM32F1xx_HAL_Driver/Inc
LIBINC += -I$(STM32CUBE)/Drivers/BSP/STM32VL-Discovery/

LIBS = $(CMSIS)/Lib/GCC/libarm_cortexM3l_math.a
LIBS += -lm

CC      = $(PREFIX)-gcc
CCDEP   = $(PREFIX)-gcc
LD      = $(PREFIX)-gcc
AR      = $(PREFIX)-ar
AS      = $(PREFIX)-gcc
OBJCOPY = $(PREFIX)-objcopy
OBJDUMP	= $(PREFIX)-objdump
GDB			= $(PREFIX)-gdb
SIZE		= $(PREFIX)-size

INCLUDES = -I$(INCDIR) $(LIBINC)
CFLAGS  = $(CPU) $(CMSIS_OPT) $(OTHER_OPT) -Wall -fno-common -MD -fno-strict-aliasing -Os $(INCLUDES) -g -Wfatal-errors -g 
ASFLAGS = $(CFLAGS) -x assembler-with-cpp
LDFLAGS = -Wl,--gc-sections,-Map=$*.map,-cref -T $(LDSCRIPT) $(CPU)
ARFLAGS = cr
OBJCOPYFLAGS = -Obinary
OBJDUMPFLAGS = -S

# not needed, in Src
#SYSTEM_OBJ = $(CMSIS_DEVSUP)/Source/Templates/system_stm32f1xx.o
STARTUP_OBJ = startup_stm32f100xb.o

OBJS = $(sort \
 $(patsubst %.c,%.o,$(wildcard $(SRCDIR)/*.c)) \
 $(patsubst %.s,%.o,$(wildcard $(SRCDIR)/*.s)) \
 $(patsubst %.c,%.o,$(wildcard $(LIBSRC)/*.c)) \
 $(STARTUP_OBJ) \
 $(SYSTEM_OBJ))
 
DEPS = $(OBJS:.o=.d)


all: main.bin
	@echo 'All done'
	
main.bin: main.elf
	$(OBJCOPY) $(OBJCOPYFLAGS) main.elf main.bin
	$(OBJDUMP) $(OBJDUMPFLAGS) main.elf > main.list
	$(SIZE) main.elf
	@echo Make finished



main.elf: $(OBJS)
	$(LD) $(LDFLAGS) -o $@ $(OBJS) $(LIBS)

clean:
	-rm -f $(OBJS) $(DEPS)
	-rm -f main.list main.elf main.hex main.map main.bin .depend main.d

depend dep: .depend

include .depend

.depend: Src/*.c
	$(CCDEP) $(CFLAGS) -MM $^ | sed -e 's@.*.o:@Src/&@' > .depend 

.c.o:
	@echo cc $<
	@$(CC) $(CFLAGS) -c -o $@ $<

.s.o:
	@echo as $<
	@$(AS) $(ASFLAGS) -c -o $@ $<
  
flash: main.bin
	@sudo st-flash write v1 main.bin 0x8000000
