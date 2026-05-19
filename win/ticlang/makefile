MSPM0_SDK_INSTALL_DIR ?= D:/path/to/mspm0_sdk_x_xx_xx_xx

include $(MSPM0_SDK_INSTALL_DIR)/imports.mak

CC = "$(TICLANG_ARMCOMPILER)/bin/tiarmclang"
LNK = "$(TICLANG_ARMCOMPILER)/bin/tiarmclang"

SYSCONFIG_GUI_TOOL = $(dir $(SYSCONFIG_TOOL))sysconfig_gui$(suffix $(SYSCONFIG_TOOL))
SYSCFG_CMD_STUB = $(SYSCONFIG_TOOL) --compiler ticlang --product $(MSPM0_SDK_INSTALL_DIR)/.metadata/product.json
SYSCFG_GUI_CMD_STUB = $(SYSCONFIG_GUI_TOOL) --compiler ticlang --product $(MSPM0_SDK_INSTALL_DIR)/.metadata/product.json
SYSCFG_FILE := $(firstword $(wildcard ../*.syscfg))

NAME := $(basename $(notdir $(SYSCFG_FILE)))

BUILD_DIR = output

SYSCFG_FILES := $(shell $(SYSCFG_CMD_STUB) --listGeneratedFiles --listReferencedFiles --output . $(SYSCFG_FILE))

SYSCFG_C_FILES = $(filter %.c,$(SYSCFG_FILES))
SYSCFG_H_FILES = $(filter %.h,$(SYSCFG_FILES))
SYSCFG_OPT_FILES = $(filter %.opt,$(SYSCFG_FILES))

# Enable verbose output by setting VERBOSE=1
V := @
ifeq ($(VERBOSE), 1)
  V :=
endif

USER_C_FILES := $(wildcard ../*.c) $(wildcard ../user/Src/*.c)
USER_OBJ_FILES := $(addprefix $(BUILD_DIR)/,$(notdir $(USER_C_FILES:.c=.obj)))
SYSCFG_OBJ_FILES := $(addprefix $(BUILD_DIR)/,$(patsubst %.c,%.obj,$(notdir $(SYSCFG_C_FILES))))
OBJECTS = $(USER_OBJ_FILES) $(SYSCFG_OBJ_FILES)


CFLAGS += -MD \
	    -I.. \
    -I. \
    -I../user/Inc \
    $(addprefix @,$(SYSCFG_OPT_FILES)) \
    -O2 \
    @device.opt \
    "-I$(MSPM0_SDK_INSTALL_DIR)/source/third_party/CMSIS/Core/Include" \
    "-I$(MSPM0_SDK_INSTALL_DIR)/source" \
    -gdwarf-3 \
    -mcpu=cortex-m0plus \
    -march=thumbv6m \
    -mfloat-abi=soft \
    -mthumb \
    -Wall

LFLAGS += -ldevice.cmd.genlibs \
    "-L$(MSPM0_SDK_INSTALL_DIR)/source" \
    -L.. \
    ../ticlang/device_linker.cmd \
    "-Wl,-m,$(BUILD_DIR)/$(NAME).map" \
    -Wl,--rom_model \
    -Wl,--warn_sections \
    "-L$(TICLANG_ARMCOMPILER)/lib" \
    -llibc.a

all: $(BUILD_DIR)/$(NAME).out

$(BUILD_DIR):
	@mkdir -p $@

.INTERMEDIATE: syscfg
$(SYSCFG_FILES): syscfg
	@ echo generation complete

syscfg: $(SYSCFG_FILE)
	@ echo Generating configuration files...
	$(V) $(SYSCFG_CMD_STUB) --output $(@D) $<


# Helpful hint that the user needs to use a standalone SysConfig installation
$(SYSCONFIG_GUI_TOOL):
	$(error $(dir $(SYSCONFIG_TOOL)) does not contain the GUI framework \
        necessary to launch the SysConfig GUI.  Please set SYSCONFIG_TOOL \
        (in your SDK's imports.mak) to a standalone SysConfig installation \
        rather than one inside CCS)

syscfg-gui: $(SYSCFG_FILE) $(SYSCONFIG_GUI_TOOL)
	@ echo Opening SysConfig GUI
	$(V) $(SYSCFG_GUI_CMD_STUB) $<


define C_RULE
$(BUILD_DIR)/$(basename $(notdir $(1))).obj: $(1) $(SYSCFG_H_FILES) | $(BUILD_DIR)
	@ echo Building $$@
	$(V) $(CC) $(CFLAGS) -c $$< -o $$@
endef

$(foreach c_file,$(SYSCFG_C_FILES),$(eval $(call C_RULE,$(c_file))))

VPATH = .. ../user/Src

-include $(OBJECTS:.obj=.d)

$(BUILD_DIR)/%.obj: %.c $(SYSCFG_H_FILES) | $(BUILD_DIR)
	@ echo Building $@
	$(V) $(CC) $(CFLAGS) -c $< -o $@

$(BUILD_DIR)/$(NAME).out: $(OBJECTS) | $(BUILD_DIR)
	@ echo linking $@
	$(V) $(LNK) -Wl,-u,_c_int00 $(OBJECTS) $(LFLAGS) -o $@

clean:
	@ echo Cleaning...
	$(V) $(RM) -rf $(BUILD_DIR)
