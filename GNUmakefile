
OBJDIR := obj#define the obj directory

# Run 'make V=1' to turn on verbose commands, or 'make V=0' to turn them off.
# 如果编译时使用make V=1，则会输出详细编译信息，否则输出类似 “CC MAIN”的简单信息
V = @
ifeq ($(V),1)
override V =
endif
ifeq ($(V),0)
override V = @
endif

#these two files just define 3 variables:V LAB PACKAGEDATE, no big deal
#-include conf/lab.mk

#-include conf/env.mk#just another makefile

TOP = .

GCCPREFIX =
QEMU := qemu-system-i386 

# try to generate a unique GDB port
GDBPORT	:= $(shell expr `id -u` % 5000 + 25000)

CC	:= $(GCCPREFIX)gcc -pipe
AS	:= $(GCCPREFIX)as
AR	:= $(GCCPREFIX)ar
LD	:= $(GCCPREFIX)ld
OBJCOPY	:= $(GCCPREFIX)objcopy
OBJDUMP	:= $(GCCPREFIX)objdump
NM	:= $(GCCPREFIX)nm

# Native commands
PERL	:= perl

# Compiler flags
# -fno-builtin is required to avoid refs to undefined functions in the kernel.
# Only optimize to -O1 to discourage inlining, which complicates backtraces.
CFLAGS := $(CFLAGS) $(DEFS) $(LABDEFS) -O1 -fno-builtin -I$(TOP) -MD
CFLAGS += -fno-omit-frame-pointer
CFLAGS += -std=gnu99
CFLAGS += -static
CFLAGS += -Wall -Wno-format -Wno-unused -Werror -gstabs -m32
# -fno-tree-ch prevented gcc from sometimes reordering read_ebp() before
# mon_backtrace()'s function prologue on gcc version: (Debian 4.7.2-5) 4.7.2
CFLAGS += -fno-tree-ch

# Add -fno-stack-protector if the option exists.
CFLAGS += $(shell $(CC) -fno-stack-protector -E -x c /dev/null >/dev/null 2>&1 && echo -fno-stack-protector)

# Common linker flags
LDFLAGS := -m elf_i386

# Linker flags for JOS user programs ---------------not being used for now
ULDFLAGS := -T user/user.ld

GCC_LIB := $(shell $(CC) $(CFLAGS) -print-libgcc-file-name)

# Lists that the */Makefrag makefile fragments will add to
OBJDIRS :=

#-------------------------------------it seems 'all' is the cmd that make will run------------------
# Make sure that 'all' is the first target
all:

# Eliminate default suffix rules
.SUFFIXES:

# Delete target files if there is an error (or make is interrupted)
.DELETE_ON_ERROR:

#write .PRECIOUS here, so that no intermediate .o files are ever deleted
.PRECIOUS: %.o $(OBJDIR)/boot/%.o $(OBJDIR)/kern/%.o \
	   $(OBJDIR)/lib/%.o $(OBJDIR)/fs/%.o $(OBJDIR)/net/%.o \
	   $(OBJDIR)/user/%.o

KERN_CFLAGS := $(CFLAGS) -DJOS_KERNEL -gstabs
USER_CFLAGS := $(CFLAGS) -DJOS_USER -gstabs

# Update .vars.X if variable X has changed since the last make run.
#
# Rules that use variable X should depend on $(OBJDIR)/.vars.X.  If
# the variable's value has changed, this will update the vars file and
# force a rebuild of the rule that depends on it.
# .vars.X is located in obj/ directory, used by /kern/Makefrag for builting kernel
# output of $(info --$@-- ---$*---)
#--obj/.vars.KERN_CFLAGS-- ---KERN_CFLAGS---
#--obj/.vars.INIT_CFLAGS-- ---INIT_CFLAGS---
#--obj/.vars.KERN_LDFLAGS-- ---KERN_LDFLAGS---
$(OBJDIR)/.vars.%: FORCE
	$(V)echo "$($*)" | cmp -s $@ || echo "$($*)" > $@

.PRECIOUS: $(OBJDIR)/.vars.%
.PHONY: FORCE


# Include Makefrags for subdirectories
include boot/Makefrag
include kern/Makefrag


#---------------------------------------------------end of 'all'-----------------------

clean:
	rm -rf $(OBJDIR) .gdbinit jos.in qemu.log;


# This magic automatically generates makefile dependencies
# for header files included from C source files we compile,
# and keeps those dependencies up-to-date every time we recompile.
# See 'mergedep.pl' for more information.
# 
# the generated file  .deps  is empty, and it has not been used by Makefrag for now
# so ignore it 
# -------------------------------------------------------------------------------
$(OBJDIR)/.deps: $(foreach dir, $(OBJDIRS), $(wildcard $(OBJDIR)/$(dir)/*.d))
	@mkdir -p $(@D)
	@$(PERL) mergedep.pl $@ $^
-include $(OBJDIR)/.deps

