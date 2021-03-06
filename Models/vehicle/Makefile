# Makefile for 20-sim FMU 2.0 export

FMU_NAME=vehicle
FMU=$(FMU_NAME).fmu

FMU_DIR=project/fmu
FMU_SRC_DIR   = $(FMU_DIR)/sources
FMU_DOC_DIR   = $(FMU_DIR)/documentation

BUILD = project/gcc
SRC   = src

XXSIM_SOURCES = EulerAngles.c \
	MotionProfiles.c \
	xxfuncs.c \
	xxinteg.c \
	xxinverse.c \
	xxmatrix.c \
	xxmodel.c \
	xxsubmod.c \
	xxTable2D.c

FMU_SOURCES = fmi2Functions.c $(XXSIM_SOURCES)

INCLUDES=
OBJS = $(addprefix $(BUILD)/, $(addsuffix .o, $(basename $(FMU_SOURCES))))
DEPS = $(addprefix $(BUILD)/, $(addsuffix .d, $(basename $(FMU_SOURCES))))

CXX?=g++
CC?=gcc

current_dir = $(shell pwd)
DEFINES=-DNO_FUNCTION_PREFIX

# OS detection
ifeq ($(OS),Windows_NT)
	# Windows
	WINDOWS=1
	DEFINES+= -DWIN32
	# For now, assume 32-bit
	# TODO detect MinGW 64
	OS_BITS=32
	DLL_EXT=dll
	# -shared to build a DLL, -static to prevent a dependency on libgcc_s_*.dll (gcc runtime library)
	BUILD_DLL=-shared -static
else
	# Other OS (Linux/MacOSX)
	ifeq ($(shell uname), Linux)
		LINUX=1
		DEFINES+= -DLINUX
		DLL_EXT=so
		BUILD_DLL=-shared
	endif
	ifeq ($(shell uname), Darwin)
		MACOSX=1
		DEFINES+= -DMACOSX
		DLL_EXT=so
		BUILD_DLL=-dynamiclib
	endif
	# Detect 32-bit / 64-bit OS
	OS_BITS=$(shell getconf LONG_BIT)
endif

# Set the OS specific settings
ifeq ($(WINDOWS),1)
	FMU_BIN32_DIR = $(FMU_DIR)/binaries/win32
	FMU_BIN64_DIR = $(FMU_DIR)/binaries/win64
	TEMPLATE_DIR=$(subst :,,$(subst \,/,/C:\Program Files (x86)\20-sim 4.6\fmi-export-20sim-master\StandaloneFMU))
	ZIP_COMMAND="$(TEMPLATE_DIR)/bin/7z.exe" a -tzip -xr!.svn "$(FMU)" *
	POS_INDEP_CODE=
endif
ifeq ($(LINUX),1)
	FMU_BIN32_DIR = $(FMU_DIR)/binaries/linux32
	FMU_BIN64_DIR = $(FMU_DIR)/binaries/linux64
	ZIP_COMMAND=zip -r $(FMU) *
	POS_INDEP_CODE=-fPIC
endif
ifeq ($(MACOSX),1)
	FMU_BIN32_DIR = $(FMU_DIR)/binaries/darwin32
	FMU_BIN64_DIR = $(FMU_DIR)/binaries/darwin64
	ZIP_COMMAND=zip -r $(FMU) *
	POS_INDEP_CODE=-fPIC
endif

ifeq ($(MAKECMDGOALS),debug)
  CFLAGS=-g -DDEBUG $(INCLUDES) $(DEFINES) $(POS_INDEP_CODE)
  DEBUG=1
else
  CFLAGS=-O2 -Wall $(INCLUDES) $(DEFINES) $(POS_INDEP_CODE)
  DEBUG=0
endif

debug: fmu

default: fmu

.PHONY: clean

clean:
	@rm -rf $(BUILD)
	@rm -f $(FMU)
	@rm -rf $(FMU_BIN32_DIR)
	@rm -rf $(FMU_BIN64_DIR)
	@rm -rf $(FMU_SRC_DIR)
	@rm -f $(FMU_DOC_DIR)/*

welcome:
	@echo ------------------------------------------------------------
	@echo 20-sim standalone co-simulation FMU export for:
	@echo '$(FMU_NAME)'
	@echo ------------------------------------------------------------

$(BUILD):
	@mkdir -p $(BUILD)

$(BUILD)/%.o: $(SRC)/%.c Makefile $(BUILD)
	@echo "[$(CC)] $<"
	@$(CC) -MD -MF $(patsubst %.o,%.d,$@) -c $(CFLAGS) $< -o $@

dll: welcome $(OBJS)
	@$(CC) $(CFLAGS) -o $(BUILD)/$(FMU_NAME).$(DLL_EXT) $(BUILD_DLL) $(OBJS) -lm
	@[ $(DEBUG)==0 ] && strip $(BUILD)/$(FMU_NAME).$(DLL_EXT)

createfmudir:
	@echo Creating an empty FMU
	@mkdir -p $(FMU_DIR)
	@mkdir -p $(FMU_SRC_DIR)

fmu: dll createfmudir
	@if [ $(OS_BITS) -eq 32 ]; then \
		mkdir -p $(FMU_BIN32_DIR); \
		echo Copy the compiled 32-bit $(DLL_EXT) $(BUILD)/$(FMU_NAME).$(DLL_EXT) to $(FMU_BIN32_DIR); \
		cp $(BUILD)/$(FMU_NAME).$(DLL_EXT) $(FMU_BIN32_DIR); \
	else \
		mkdir -p $(FMU_BIN64_DIR); \
		echo Copy the compiled 64-bit $(DLL_EXT) $(BUILD)/$(FMU_NAME).$(DLL_EXT) to $(FMU_BIN64_DIR); \
		cp $(BUILD)/$(FMU_NAME).$(DLL_EXT) $(FMU_BIN64_DIR); \
	fi

	@echo Copy the generated sources to $(FMU_SRC_DIR)
	@cp $(SRC)/*.c $(FMU_SRC_DIR)
	@cp $(SRC)/*.h $(FMU_SRC_DIR)
	@cp $(SRC)/defines.def $(FMU_SRC_DIR)

	@echo Create the FMU: $(FMU)
	@cd $(FMU_DIR) && $(ZIP_COMMAND) && mv $(FMU) $(current_dir) && echo Ready!

