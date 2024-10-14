#
# Copyright 2021 xhr-git
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

MAKEFILE_FLAGS += -rR
MAKEFILE_FLAGS += --no-print-directory

ifeq ($(ROOT_DIR),)

BUILD_OUTPUT = build
# Application name
BUILD_EXE    = test.exe

# Add a search directory for header files other than the $(BUILD_SRC) directory
INCLUDES = -I$(ROOT_DIR)/include -I$(ROOT_DIR)/src
# Add a search directory for libraries
LD_DIRS = 
# Add required libraries
LD_LIBS = -lpthread


ROOT_DIR = $(shell pwd)
ifeq ($(BUILD_OUTPUT),)
  BUILDDIR = $(ROOT_DIR)
else
  BUILDDIR = $(ROOT_DIR)/$(BUILD_OUTPUT)
endif

CFLAGS = -g -O2

CPP       = $(CC) -E
CC        = $(CROSS_COMPILE)gcc
LD        = $(CROSS_COMPILE)ld
AR        = $(CROSS_COMPILE)ar
NM        = $(CROSS_COMPILE)nm
OBJCOPY   = $(CROSS_COMPILE)objcopy
OBJDUMP   = $(CROSS_COMPILE)objdump
READELF   = $(CROSS_COMPILE)readelf
STRIP     = $(CROSS_COMPILE)strip

export CPP CC LD AR NM OBJCOPY OBJDUMP READELF STRIP

BUILTIN_O = built-in.o


.PHONY: __all__ $(BUILD_EXE)
all: $(BUILD_EXE)
	@echo
	@echo ">>>  successful !!"
	@echo


$(BUILD_EXE):
	@$(MAKE) -C . $(MAKEFILE_FLAGS)
	@set -e; \
	 if [ ! -e $@ -o $@ -ot $(BUILDDIR)/$(BUILTIN_O) ]; then \
		echo "    LD  $@"; \
		$(CC) $(ALL_FLAGS) -o $@ $(BUILDDIR)/$(BUILTIN_O) $(LD_LIBS); \
	 fi

COMPILE_CFLAGS  := -std=gnu99 -Wall -Werror

INCLUDES_DIR = $(sort $(INCLUDES))
ALL_FLAGS = $(INCLUDES_DIR) $(CFLAGS) $(COMPILE_CFLAGS)
export COMPILE_CFLAGS CFLAGS ALL_FLAGS
export ROOT_DIR BUILDDIR BUILTIN_O INCLUDES_DIR LD_DIRS LD_LIBS

endif # ifeq ($(ROOT_DIR),)

############################################################
#                                                          #
#  The following is the second time to enter the Makefile  #
#                                                          #
############################################################

include BUILD.mk

ifeq ($(ROOT_DIR),$(CURDIR))
  CUR_RDIR = .
else
  CUR_RDIR = $(patsubst $(ROOT_DIR)/%,%,$(CURDIR))
endif

subdirs = $(patsubst %/,%,$(filter %/, $(obj-y)))

cur_objs = $(addprefix $(BUILDDIR)/$(CUR_RDIR)/,$(filter-out %/, $(obj-y)))

abs_subdir_objs = $(patsubst %,$(BUILDDIR)/$(CUR_RDIR)/%/$(BUILTIN_O),$(subdirs))

DEPENDS := $(cur_objs:.o=.d)

PHONY := __sub_dir_all__ $(subdirs)
__sub_dir_all__: $(cur_objs) $(subdirs)
	@set -e; \
	 NEW_BUILTIN=$(BUILDDIR)/$(CUR_RDIR)/$(BUILTIN_O); \
	 MD5_FILE=$${NEW_BUILTIN}.submd5.a; \
	 SUB_DIR_OBJS="$(cur_objs) $(abs_subdir_objs)"; \
	 for v in $${SUB_DIR_OBJS}; do \
		MD5VAL="$${MD5VAL}$$(md5sum $${v}|cut -f 1 -d ' ')"; \
	 done; \
	 if [ "$${MD5VAL}" != "$$(cat $${MD5_FILE} 2>/dev/null)" ]; then \
		echo "    LD  $(CUR_RDIR)/$(BUILTIN_O)"; \
		echo "$${MD5VAL}" > $${MD5_FILE}; \
		$(LD) -r -o $${NEW_BUILTIN} $${SUB_DIR_OBJS}; \
	 fi


$(subdirs):
	@$(MAKE) -C $@ -f $(ROOT_DIR)/Makefile $(MAKEFILE_FLAGS)

-include $(DEPENDS)

$(BUILDDIR)/$(CUR_RDIR)/%.o: $(CURDIR)/%.c
	@mkdir -p $(dir $@)
	@echo "    CC  $(CUR_RDIR)/$*.o"
	@$(CC) $(ALL_FLAGS) -c -o $@ -MMD -MP $<

PHONY += clean
clean:
	@-rm -f $(BUILD_EXE)
ifdef BUILD_OUTPUT
	@-rm -rf $(BUILD_OUTPUT)
else
	@-find $(ROOT_DIR) -name "*.o" | xargs rm -f
	@-find $(ROOT_DIR) -name "*.d" | xargs rm -f
	@-find $(ROOT_DIR) -name "*.a" | xargs rm -f
endif

PHONY += show_vars
show_vars:
	@echo "  ROOT_DIR = $(ROOT_DIR)"
	@echo "  BUILDDIR = $(BUILDDIR)"
	@echo "  subdirs = $(subdirs)"
	@echo "  CURDIR = $(CURDIR)"
	@echo "  CUR_RDIR = $(CUR_RDIR)"
	@echo "  cur_objs = $(cur_objs)"
	@echo "  INCLUDES_DIR = $(INCLUDES_DIR)"
	@echo "  DEPENDS = $(DEPENDS)"

# Declare the contents of the PHONY variable as phony.  We keep that
# information in a variable so we can use it in if_changed and friends.
.PHONY: $(PHONY)
