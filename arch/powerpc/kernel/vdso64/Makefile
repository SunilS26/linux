# SPDX-License-Identifier: GPL-2.0
# List of files in the vdso, has to be asm only for now

ARCH_REL_TYPE_ABS := R_PPC_JUMP_SLOT|R_PPC_GLOB_DAT|R_PPC_ADDR32|R_PPC_ADDR24|R_PPC_ADDR16|R_PPC_ADDR16_LO|R_PPC_ADDR16_HI|R_PPC_ADDR16_HA|R_PPC_ADDR14|R_PPC_ADDR14_BRTAKEN|R_PPC_ADDR14_BRNTAKEN
include $(srctree)/lib/vdso/Makefile

obj-vdso64 = sigtramp.o gettimeofday.o datapage.o cacheflush.o note.o getcpu.o

ifneq ($(c-gettimeofday-y),)
  CFLAGS_vgettimeofday.o += -include $(c-gettimeofday-y)
  CFLAGS_vgettimeofday.o += $(DISABLE_LATENT_ENTROPY_PLUGIN)
  CFLAGS_vgettimeofday.o += $(call cc-option, -fno-stack-protector)
  CFLAGS_vgettimeofday.o += -DDISABLE_BRANCH_PROFILING
  CFLAGS_vgettimeofday.o += -ffreestanding -fasynchronous-unwind-tables
  CFLAGS_REMOVE_vgettimeofday.o = $(CC_FLAGS_FTRACE)
endif

# Build rules

targets := $(obj-vdso64) vdso64.so.dbg
obj-vdso64 := $(addprefix $(obj)/, $(obj-vdso64))

GCOV_PROFILE := n
KCOV_INSTRUMENT := n
UBSAN_SANITIZE := n
KASAN_SANITIZE := n

ccflags-y := -shared -fno-common -fno-builtin -nostdlib \
	-Wl,-soname=linux-vdso64.so.1 -Wl,--hash-style=both
asflags-y := -D__VDSO64__ -s

obj-y += vdso64_wrapper.o
targets += vdso64.lds
CPPFLAGS_vdso64.lds += -P -C -U$(ARCH)

$(obj)/vgettimeofday.o: %.o: %.c FORCE

# Force dependency (incbin is bad)
$(obj)/vdso64_wrapper.o : $(obj)/vdso64.so.dbg

# link rule for the .so file, .lds has to be first
$(obj)/vdso64.so.dbg: $(src)/vdso64.lds $(obj-vdso64) $(obj)/vgettimeofday.o FORCE
	$(call if_changed,vdso64ld_and_check)

# Generate VDSO offsets using helper script
gen-vdsosym := $(srctree)/$(src)/gen_vdso_offsets.sh
quiet_cmd_vdsosym = VDSOSYM $@
      cmd_vdsosym = $(NM) $< | $(gen-vdsosym) | LC_ALL=C sort > $@

include/generated/vdso64-offsets.h: $(obj)/vdso64.so.dbg FORCE
	$(call if_changed,vdsosym)

# actual build commands
quiet_cmd_vdso64ld_and_check = VDSO64L $@
      cmd_vdso64ld_and_check = $(CC) $(c_flags) -o $@ -Wl,-T$(filter %.lds,$^) $(filter %.o,$^); $(cmd_vdso_check)
