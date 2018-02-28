-include $(TOPDIR)/.config
-include $(TOPDIR)/Make.defs
include $(APPDIR)/Make.defs

CFLAGS := $(filter-out -Wundef,$(CFLAGS))

CROSS_COMPILE ?= $(CROSSDEV)
VPATH := :$(SRCDIR)

-include mpconfigport.mk
include ../../py/mkenv.mk

# qstr definitions (must come before including py.mk)
QSTR_DEFS = $(SRCDIR)/qstrdefsport.h

# include py core make definitions
include $(TOP)/py/py.mk

INC +=  -I.
INC +=  -I$(TOP)
INC += -I$(BUILD)

ifeq ($(MICROPY_USE_READLINE),1)
INC +=  -I$(TOP)/lib/mp-readline
CFLAGS_MOD += -DMICROPY_USE_READLINE=1
LIB_SRC_C_EXTRA += mp-readline/readline_.c
endif
ifeq ($(MICROPY_PY_TERMIOS),1)
CFLAGS_MOD += -DMICROPY_PY_TERMIOS=1
SRC_MOD += modtermios.c
endif
ifeq ($(MICROPY_PY_SOCKET),1)
CFLAGS_MOD += -DMICROPY_PY_SOCKET=1
SRC_MOD += modusocket.c
endif
ifeq ($(MICROPY_PY_THREAD),1)
CFLAGS_MOD += -DMICROPY_PY_THREAD=1 -DMICROPY_PY_THREAD_GIL=0
endif

# compiler settings
CFLAGS += $(INC) $(CFLAGS_MOD)

# source files
SRC_C = \
	mp_main.c \
	gccollect.c \
	unix_mphal.c \
	mpthreadport.c \
	input.c \
	file.c \
	modmachine.c \
	modos.c \
	moduos_vfs.c \
	modtime.c \
	moduselect.c \
	alloc.c \
	coverage.c \
	fatfs_port.c \
	$(SRC_MOD)

LIB_SRC_C = $(addprefix lib/,\
	$(LIB_SRC_C_EXTRA) \
	timeutils/timeutils.c \
	)

# FatFS VFS support
LIB_SRC_C += $(addprefix lib/,\
	oofatfs/ff.c \
	oofatfs/option/unicode.c \
	)

OBJ = $(PY_O)
OBJ += $(addprefix $(BUILD)/, $(SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(LIB_SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(STMHAL_SRC_C:.c=.o))

# List of sources for qstr extraction
SRC_QSTR += $(SRC_C) $(LIB_SRC_C)
# Append any auto-generated sources that are needed by sources listed in
# SRC_QSTR
SRC_QSTR_AUTO_DEPS +=

CONFIG_INTERPRETERS_MICROPYTHON_APPNAME ?= micropython
CONFIG_INTERPRETERS_MICROPYTHON_STACKSIZE ?= CONFIG_USERMAIN_STACKSIZE
CONFIG_INTERPRETERS_MICROPYTHON_PRIORITY ?= SCHED_PRIORITY_DEFAULT

APPNAME = $(CONFIG_INTERPRETERS_MICROPYTHON_APPNAME)
STACKSIZE = $(CONFIG_INTERPRETERS_MICROPYTHON_STACKSIZE)
PRIORITY = $(CONFIG_INTERPRETERS_MICROPYTHON_PRIORITY)

CONFIG_INTERPRETERS_MICROPYTHON_PROGNAME ?= micropython$(EXEEXT)
PROGNAME = $(CONFIG_INTERPRETERS_MICROPYTHON_PROGNAME)

MAINSRC = mp_main.c

ifeq ($(CONFIG_WINDOWS_NATIVE),y)
  BIN = $(OUTDIR)\$(CONFIG_APPS_DIR)\libapps$(LIBEXT)
else
ifeq ($(WINTOOL),y)
  BIN = $(OUTDIR)\\$(CONFIG_APPS_DIR)\\libapps$(LIBEXT)
else
  BIN = $(OUTDIR)/$(CONFIG_APPS_DIR)/libapps$(LIBEXT)
endif
endif

ifeq ($(WINTOOL),y)
  INSTALL_DIR = "${shell cygpath -w $(BIN_DIR)}"
else
  INSTALL_DIR = $(BIN_DIR)
endif

ROOTDEPPATH = --dep-path .
VPATH := :$(SRCDIR)

# Common build

all: .built
.PHONY: install context clean distclean preconfig
.PRECIOUS: ../../libapps$(LIBEXT)

.built: $(MICROPYTHON_UNPACKNAME) $(OBJ)
	$(call ARCHIVE, $(BIN), $(OBJ))
	@touch $@

ifeq ($(CONFIG_BUILD_KERNEL),y)
$(BIN_DIR)$(DELIM)$(PROGNAME): $(MICROPYTHON_UNPACKNAME) $(OBJ) $(MAINOBJ)
	@echo "LD: $(PROGNAME)"
	$(Q) $(LD) $(LDELFFLAGS) $(LDLIBPATH) -o $(INSTALL_DIR)$(DELIM)$(PROGNAME) $(ARCHCRT0OBJ) $(MAINOBJ) $(LDLIBS)
	$(Q) $(NM) -u $(INSTALL_DIR)$(DELIM)$(PROGNAME)

install: $(BIN_DIR)$(DELIM)$(PROGNAME)
else
install:
endif

ifeq ($(CONFIG_NSH_BUILTIN_APPS),y)
$(BUILTIN_REGISTRY)$(DELIM)$(APPNAME)_main.bdat: $(DEPCONFIG) Makefile
	$(call REGISTER,$(APPNAME),$(PRIORITY),$(STACKSIZE),micropython_main)

context: $(MICROPYTHON_UNPACKNAME) $(BUILTIN_REGISTRY)$(DELIM)$(APPNAME)_main.bdat
else
context:
endif

.depend: Makefile $(SRCS)
	@$(MKDEP) $(ROOTDEPPATH) "$(CC)" -- $(CFLAGS) -- $^ >Make.dep
	@touch $@

depend: .depend

clean::
	$(call DELDIR, build)
	$(call DELFILE, .built)
	$(call CLEAN)

distclean: clean
	$(call DELFILE, Make.dep)
	$(call DELFILE, .depend)

preconfig:

include $(TOP)/py/mkrules.mk
-include Make.dep

