# Copyright (C) 2017 GreenWaves Technologies
# All rights reserved.

# This software may be modified and distributed under the terms
# of the BSD license.  See the LICENSE file for details.

RM=rm -f
CNN_AT_PATH = $(TILER_GENERATOR_PATH)/CNN

APP = Cifar10
APP_SRCS += Cifar10.c Cifar10Kernels.c \
            $(CNN_AT_PATH)/CNN_BiasReLULinear_BasicKernels.c \
            $(CNN_AT_PATH)/CNN_Conv_BasicKernels.c \
            $(CNN_AT_PATH)/CNN_Pooling_BasicKernels.c
APP_INC += $(TILER_INC) $(CNN_AT_PATH)

APP_CFLAGS += -O2 -mno-memcpy -fno-tree-loop-distribute-patterns  -fdata-sections -ffunction-sections
# The generated code outputs a maybe-uninitialized error which is rather difficult to suppress
# in a clean way.
APP_CFLAGS += -Wall -Wno-maybe-uninitialized -Wno-unused-but-set-variable
#APP_CFLAGS += -I$(TILER_INC) -I$(CNN_AT_PATH)

LDFLAGS +=  -flto -Wl,--gc-sections

PLPBRIDGE_FLAGS += -jtag

# Files to copy in flash
FILE_BIAS0   = coef/Cifar10_Bias0.dat
FILE_BIAS1   = coef/Cifar10_Bias1.dat
FILE_BIAS2   = coef/Cifar10_Bias2.dat
FILE_FILTER0 = coef/Cifar10_Filter0.dat
FILE_FILTER1 = coef/Cifar10_Filter1.dat
FILE_FILTER2 = coef/Cifar10_Filter2.dat

FLASH_FILES = $(FILE_BIAS0) $(FILE_BIAS1) $(FILE_BIAS2) $(FILE_FILTER0) $(FILE_FILTER1) $(FILE_FILTER2)

PLPBRIDGE_FLAGS += -f $(FLASH_FILES)

# For GVSOC
override runner_args += --config-opt=flash/fs/files=$(CURDIR)/$(FILE_BIAS0) \
                        --config-opt=flash/fs/files=$(CURDIR)/$(FILE_BIAS1) \
                        --config-opt=flash/fs/files=$(CURDIR)/$(FILE_BIAS2) \
                        --config-opt=flash/fs/files=$(CURDIR)/$(FILE_FILTER0) \
                        --config-opt=flash/fs/files=$(CURDIR)/$(FILE_FILTER1) \
                        --config-opt=flash/fs/files=$(CURDIR)/$(FILE_FILTER2)

#PMSIS_OS ?= freertos

ifeq ('$(BOARD_NAME)', 'gapuino')
APP_CFLAGS += -DGAPUINO
else ifeq ('$(BOARD_NAME)', 'gapoc_a')
PLPBRIDGE_FLAGS += -ftdi
APP_CFLAGS += -DGAPOC_A
endif

# The double colon allows us to force this to occur before the imported all target
# Link model generation to all step
all:: model

# Build the code generator
GenTile:
	gcc -o GenCifar10 -I$(TILER_INC) -I$(CNN_AT_PATH) Cifar10Model.c $(CNN_AT_PATH)/CNN_Generators.c $(TILER_LIB)

# Run the code generator
Cifar10Kernels.c: GenTile
	./GenCifar10

model: Cifar10Kernels.c

clean::
	$(RM) GenCifar10 Cifar10Kernels.*

.PHONY: model clean

include $(GAP_SDK_HOME)/tools/rules/pmsis_rules.mk

