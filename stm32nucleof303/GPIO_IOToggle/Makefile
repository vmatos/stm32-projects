USER_HOME = $(shell echo $$HOME)

STM32CUBE = $(USER_HOME)/stm32/STM32CubeF3

DEVICE_FAMILY = STM32F3xx
DEVICE_TYPE = STM32F303xE
LDSCRIPT = STM32F303XE_FLASH.ld

CMSIS = $(STM32CUBE)/Drivers/CMSIS
CMSIS_DEVSUP = $(CMSIS)/Device/ST/$(DEVICE_FAMILY)
CMSIS_OPT = -D$(DEVICE_TYPE) -DUSE_HAL_DRIVER -DNUCLEO_F303RE -DSTM32F303RETx
OTHER_OPT = "-D__weak=__attribute__((weak))" "-D__packed=__attribute__((__packed__))" 
CPU = -mthumb -mcpu=cortex-m4 -mfloat-abi=softfp -mfpu=fpv4-sp-d16 -mlittle-endian -march=armv7e-m
PREFIX = arm-none-eabi

SRCDIR = Src
INCDIR = Inc

LIBSRC = $(STM32CUBE)/Drivers/$(DEVICE_FAMILY)_HAL_Driver/Src
LIBINC = -I$(CMSIS)/Include
LIBINC += -I$(CMSIS_DEVSUP)/Include
LIBINC += -I$(STM32CUBE)/Drivers/$(DEVICE_FAMILY)_HAL_Driver/Inc
LIBINC += -I$(STM32CUBE)/Drivers/BSP/STM32F3xx-Nucleo

#LIBS = $(CMSIS)/Lib/GCC/libarm_cortexM4lf_math.a
LIBS = -lc -lm -lnosys

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
CFLAGS  = $(CPU) $(CMSIS_OPT) $(OTHER_OPT) -Wall -fno-common -ffunction-sections -fdata-sections -MD -fno-strict-aliasing -O2 $(INCLUDES) -g -Wfatal-errors -g 
# ASFLAGS = $(CFLAGS) -x assembler-with-cpp
ASFLAGS = $(CFLAGS) 
LDFLAGS = -Wl,--gc-sections,-Map=$*.map,-cref -T $(LDSCRIPT) $(CPU)
ARFLAGS = cr
OBJCOPYFLAGS = -Obinary
OBJDUMPFLAGS = -S

STARTUP_OBJ = startup_stm32f303xe.o

OBJS = $(sort \
 $(patsubst %.c,%.o,$(wildcard $(SRCDIR)/*.c)) \
 $(patsubst %.s,%.o,$(wildcard $(SRCDIR)/*.s)) \
 $(patsubst %.c,%.o,$(wildcard $(LIBSRC)/*.c)) \
 $(STARTUP_OBJ))
 
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
	-rm -f $(OBJS) $(DEPS) stm32f3xx_it.d system_stm32f3xx.d
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
	@sudo st-flash write main.bin 0x08000000
