# ################################################################
# Copyright (c) 2015-2020, Yann Collet, Facebook, Inc.
# All rights reserved.
#
# This source code is licensed under both the BSD-style license (found in the
# LICENSE file in the root directory of this source tree) and the GPLv2 (found
# in the COPYING file in the root directory of this source tree).
# You may select, at your option, one of the above-listed licenses.
# ################################################################

Q = $(if $(filter 1,$(V) $(VERBOSE)),,@)

# When cross-compiling from linux to windows,
# you might need to specify this as "Windows."
# Fedora build fails without it.
#
# Note: mingw-w64 build from linux to windows
# does not fail on other tested distros (ubuntu, debian)
# even without manually specifying the TARGET_SYSTEM.
TARGET_SYSTEM ?= $(OS)

# Version numbers
LIBVER_MAJOR_SCRIPT:=`sed -n '/define ZSTD_VERSION_MAJOR/s/.*[[:blank:]]\([0-9][0-9]*\).*/\1/p' < ./zstd.h`
LIBVER_MINOR_SCRIPT:=`sed -n '/define ZSTD_VERSION_MINOR/s/.*[[:blank:]]\([0-9][0-9]*\).*/\1/p' < ./zstd.h`
LIBVER_PATCH_SCRIPT:=`sed -n '/define ZSTD_VERSION_RELEASE/s/.*[[:blank:]]\([0-9][0-9]*\).*/\1/p' < ./zstd.h`
LIBVER_SCRIPT:= $(LIBVER_MAJOR_SCRIPT).$(LIBVER_MINOR_SCRIPT).$(LIBVER_PATCH_SCRIPT)
LIBVER_MAJOR := $(shell echo $(LIBVER_MAJOR_SCRIPT))
LIBVER_MINOR := $(shell echo $(LIBVER_MINOR_SCRIPT))
LIBVER_PATCH := $(shell echo $(LIBVER_PATCH_SCRIPT))
LIBVER := $(shell echo $(LIBVER_SCRIPT))
VERSION?= $(LIBVER)
CCVER := $(shell $(CC) --version)

# This is a helper variable that configures a bunch of other variables to new,
# space-optimized defaults.
ZSTD_LIB_MINIFY ?= 0
ifneq ($(ZSTD_LIB_MINIFY), 0)
  HAVE_CC_OZ ?= $(shell echo "" | $(CC) -Oz -x c -c - -o /dev/null 2> /dev/null && echo 1 || echo 0)
  ZSTD_LEGACY_SUPPORT ?= 0
  ZSTD_LIB_DEPRECATED ?= 0
  HUF_FORCE_DECOMPRESS_X1 ?= 1
  ZSTD_FORCE_DECOMPRESS_SHORT ?= 1
  ZSTD_NO_INLINE ?= 1
  ZSTD_STRIP_ERROR_STRINGS ?= 1
ifneq ($(HAVE_CC_OZ), 0)
    # Some compilers (clang) support an even more space-optimized setting.
    CFLAGS += -Oz
else
    CFLAGS += -Os
endif
  CFLAGS += -fno-stack-protector -fomit-frame-pointer -fno-ident \
            -DDYNAMIC_BMI2=0 -DNDEBUG
else
  CFLAGS += -O3
endif

DEBUGLEVEL ?= 0
CPPFLAGS += -DXXH_NAMESPACE=ZSTD_ -DDEBUGLEVEL=$(DEBUGLEVEL)
ifeq ($(TARGET_SYSTEM),Windows_NT)   # MinGW assumed
  CPPFLAGS += -D__USE_MINGW_ANSI_STDIO   # compatibility with %zu formatting
endif
DEBUGFLAGS= -Wall -Wextra -Wcast-qual -Wcast-align -Wshadow \
            -Wstrict-aliasing=1 -Wswitch-enum -Wdeclaration-after-statement \
            -Wstrict-prototypes -Wundef -Wpointer-arith \
            -Wvla -Wformat=2 -Winit-self -Wfloat-equal -Wwrite-strings \
            -Wredundant-decls -Wmissing-prototypes -Wc++-compat
CFLAGS   += $(DEBUGFLAGS) $(MOREFLAGS)
FLAGS     = $(CPPFLAGS) $(CFLAGS)

HAVE_COLORNEVER = $(shell echo a | grep --color=never a > /dev/null 2> /dev/null && echo 1 || echo 0)
GREP_OPTIONS ?=
ifeq ($HAVE_COLORNEVER, 1)
  GREP_OPTIONS += --color=never
endif
GREP = grep $(GREP_OPTIONS)
SED_ERE_OPT ?= -E

ZSTDCOMMON_FILES := $(sort $(wildcard common/*.c))
ZSTDCOMP_FILES := $(sort $(wildcard compress/*.c))
ZSTDDECOMP_FILES := $(sort $(wildcard decompress/*.c))
ZDICT_FILES := $(sort $(wildcard dictBuilder/*.c))
ZDEPR_FILES := $(sort $(wildcard deprecated/*.c))
ZSTD_FILES := $(ZSTDCOMMON_FILES)

ifeq ($(findstring GCC,$(CCVER)),GCC)
decompress/zstd_decompress_block.o : CFLAGS+=-fno-tree-vectorize
endif

# Modules
ZSTD_LIB_COMPRESSION ?= 1
ZSTD_LIB_DECOMPRESSION ?= 1
ZSTD_LIB_DICTBUILDER ?= 1
ZSTD_LIB_DEPRECATED ?= 1

# Legacy support
ZSTD_LEGACY_SUPPORT ?= 5
ZSTD_LEGACY_MULTITHREADED_API ?= 0

# Build size optimizations
HUF_FORCE_DECOMPRESS_X1 ?= 0
HUF_FORCE_DECOMPRESS_X2 ?= 0
ZSTD_FORCE_DECOMPRESS_SHORT ?= 0
ZSTD_FORCE_DECOMPRESS_LONG ?= 0
ZSTD_NO_INLINE ?= 0
ZSTD_STRIP_ERROR_STRINGS ?= 0

ifeq ($(ZSTD_LIB_COMPRESSION), 0)
  ZSTD_LIB_DICTBUILDER = 0
  ZSTD_LIB_DEPRECATED = 0
endif

ifeq ($(ZSTD_LIB_DECOMPRESSION), 0)
  ZSTD_LEGACY_SUPPORT = 0
  ZSTD_LIB_DEPRECATED = 0
endif

ifneq ($(ZSTD_LIB_COMPRESSION), 0)
  ZSTD_FILES += $(ZSTDCOMP_FILES)
endif

ifneq ($(ZSTD_LIB_DECOMPRESSION), 0)
  ZSTD_FILES += $(ZSTDDECOMP_FILES)
endif

ifneq ($(ZSTD_LIB_DEPRECATED), 0)
  ZSTD_FILES += $(ZDEPR_FILES)
endif

ifneq ($(ZSTD_LIB_DICTBUILDER), 0)
  ZSTD_FILES += $(ZDICT_FILES)
endif

ifneq ($(HUF_FORCE_DECOMPRESS_X1), 0)
  CFLAGS += -DHUF_FORCE_DECOMPRESS_X1
endif

ifneq ($(HUF_FORCE_DECOMPRESS_X2), 0)
  CFLAGS += -DHUF_FORCE_DECOMPRESS_X2
endif

ifneq ($(ZSTD_FORCE_DECOMPRESS_SHORT), 0)
  CFLAGS += -DZSTD_FORCE_DECOMPRESS_SHORT
endif

ifneq ($(ZSTD_FORCE_DECOMPRESS_LONG), 0)
  CFLAGS += -DZSTD_FORCE_DECOMPRESS_LONG
endif

ifneq ($(ZSTD_NO_INLINE), 0)
  CFLAGS += -DZSTD_NO_INLINE
endif

ifneq ($(ZSTD_STRIP_ERROR_STRINGS), 0)
  CFLAGS += -DZSTD_STRIP_ERROR_STRINGS
endif

ifneq ($(ZSTD_LEGACY_MULTITHREADED_API), 0)
  CFLAGS += -DZSTD_LEGACY_MULTITHREADED_API
endif

ifneq ($(ZSTD_LEGACY_SUPPORT), 0)
ifeq ($(shell test $(ZSTD_LEGACY_SUPPORT) -lt 8; echo $$?), 0)
  ZSTD_FILES += $(shell ls legacy/*.c | $(GREP) 'v0[$(ZSTD_LEGACY_SUPPORT)-7]')
endif
endif
CPPFLAGS  += -DZSTD_LEGACY_SUPPORT=$(ZSTD_LEGACY_SUPPORT)

ZSTD_LOCAL_SRC := $(notdir $(ZSTD_FILES))
ZSTD_LOCAL_OBJ := $(ZSTD_LOCAL_SRC:.c=.o)

ZSTD_SUBDIR := common compress decompress dictBuilder legacy deprecated
vpath %.c $(ZSTD_SUBDIR)

UNAME := $(shell uname)
ifeq ($(UNAME), Darwin)
  HASH ?= md5
endif
ifeq ($(UNAME), FreeBSD)
  HASH ?= gmd5sum
endif
ifeq ($(UNAME), OpenBSD)
  HASH ?= md5
endif
HASH ?= md5sum

HAVE_HASH := $(shell echo 1 | $(HASH) > /dev/null && echo 1 || echo 0)
ifeq ($(HAVE_HASH), 1)
  HASH_VALUE := $(shell echo $(CC) $(CPPFLAGS) $(CFLAGS) $(LDFLAGS) $(ZSTD_FILES) | $(HASH) | head -c 16)
  HASH_DIR := conf_$(HASH_VALUE)
else
  $(info warning : could not find hash function to differentiate builds with different flags)
  HASH_DIR := 0
endif


BUILD_DIR ?= obj/$(HASH_DIR)
ZSTD_DYNLIB_DIR := $(BUILD_DIR)/dynlib
ZSTD_DYNLIB_OBJ := $(addprefix $(ZSTD_DYNLIB_DIR)/, $(ZSTD_LOCAL_OBJ))
ZSTD_STATLIB_DIR := $(BUILD_DIR)/statlib
ZSTD_STATLIB_OBJ := $(addprefix $(ZSTD_STATLIB_DIR)/, $(ZSTD_LOCAL_OBJ))

# macOS linker doesn't support -soname, and use different extension
# see : https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/DynamicLibraryDesignGuidelines.html
ifeq ($(UNAME), Darwin)
  SHARED_EXT = dylib
  SHARED_EXT_MAJOR = $(LIBVER_MAJOR).$(SHARED_EXT)
  SHARED_EXT_VER = $(LIBVER).$(SHARED_EXT)
  SONAME_FLAGS = -install_name $(LIBDIR)/libzstd.$(SHARED_EXT_MAJOR) -compatibility_version $(LIBVER_MAJOR) -current_version $(LIBVER)
else
  SONAME_FLAGS = -Wl,-soname=libzstd.$(SHARED_EXT).$(LIBVER_MAJOR)
  SHARED_EXT = so
  SHARED_EXT_MAJOR = $(SHARED_EXT).$(LIBVER_MAJOR)
  SHARED_EXT_VER = $(SHARED_EXT).$(LIBVER)
endif


.PHONY: default lib-all all clean install uninstall

default: lib-release

# alias
lib-all: all

all: lib

$(ZSTD_STATLIB_DIR)/libzstd.a: ARFLAGS = rcs
$(ZSTD_STATLIB_DIR)/libzstd.a: | $(ZSTD_STATLIB_DIR)
$(ZSTD_STATLIB_DIR)/libzstd.a: $(ZSTD_STATLIB_OBJ)
	@echo compiling static library
	$(Q)$(AR) $(ARFLAGS) $@ $^

.PHONY: libzstd.a  # must be run every time
libzstd.a: $(ZSTD_STATLIB_DIR)/libzstd.a
	$(Q)ln -sf $< $@

ifneq (,$(filter Windows%,$(TARGET_SYSTEM)))

LIBZSTD = dll\libzstd.dll
$(LIBZSTD):
$(LIBZSTD): $(ZSTD_FILES)
	@echo compiling dynamic library $(LIBVER)
	$(CC) $(FLAGS) -DZSTD_DLL_EXPORT=1 -Wl,--out-implib,dll\libzstd.dll.a -shared $^ -o $@

else

LIBZSTD = libzstd.$(SHARED_EXT_VER)
$(ZSTD_DYNLIB_DIR)/$(LIBZSTD): CFLAGS += -fPIC
$(ZSTD_DYNLIB_DIR)/$(LIBZSTD): LDFLAGS += -shared -fvisibility=hidden
$(ZSTD_DYNLIB_DIR)/$(LIBZSTD): | $(ZSTD_DYNLIB_DIR)
$(ZSTD_DYNLIB_DIR)/$(LIBZSTD): $(ZSTD_DYNLIB_OBJ)
	@echo compiling dynamic library $(LIBVER)
	$(Q)$(CC) $(FLAGS) $^ $(LDFLAGS) $(SONAME_FLAGS) -o $@
	@echo creating versioned links
	$(Q)ln -sf $@ libzstd.$(SHARED_EXT_MAJOR)
	$(Q)ln -sf $@ libzstd.$(SHARED_EXT)

.PHONY: $(LIBZSTD)  # must be run every time
$(LIBZSTD): $(ZSTD_DYNLIB_DIR)/$(LIBZSTD)
	$(Q)ln -sf $< $@

endif

.PHONY: libzstd
libzstd : $(LIBZSTD)

.PHONY: lib
lib : libzstd.a libzstd


# note : do not define lib-mt or lib-release as .PHONY
# make does not consider implicit pattern rule for .PHONY target

%-mt : CPPFLAGS += -DZSTD_MULTITHREAD
%-mt : LDFLAGS  += -pthread
%-mt : %
	@echo multi-threading build completed

%-release : DEBUGFLAGS :=
%-release : %
	@echo release build completed


# Generate .h dependencies automatically

DEPFLAGS = -MT $@ -MMD -MP -MF

$(ZSTD_DYNLIB_DIR)/%.o : %.c $(ZSTD_DYNLIB_DIR)/%.d | $(ZSTD_DYNLIB_DIR)
	@echo $@
	$(Q)$(COMPILE.c) $(DEPFLAGS) $(ZSTD_DYNLIB_DIR)/$*.d $(OUTPUT_OPTION) $<

$(ZSTD_STATLIB_DIR)/%.o : %.c $(ZSTD_STATLIB_DIR)/%.d | $(ZSTD_STATLIB_DIR)
	@echo $@
	$(Q)$(COMPILE.c) $(DEPFLAGS) $(ZSTD_STATLIB_DIR)/$*.d $(OUTPUT_OPTION) $<

MKDIR ?= mkdir
$(ZSTD_DYNLIB_DIR) $(ZSTD_STATLIB_DIR):
	$(Q)$(MKDIR) -p $@

DEPFILES := $(ZSTD_DYNLIB_OBJ:.o=.d) $(ZSTD_STATLIB_OBJ:.o=.d)
$(DEPFILES):

include $(wildcard $(DEPFILES))


# Special case : building library in single-thread mode _and_ without zstdmt_compress.c
ZSTDMT_FILES = compress/zstdmt_compress.c
ZSTD_NOMT_FILES = $(filter-out $(ZSTDMT_FILES),$(ZSTD_FILES))
libzstd-nomt: LDFLAGS += -shared -fPIC -fvisibility=hidden
libzstd-nomt: $(ZSTD_NOMT_FILES)
	@echo compiling single-thread dynamic library $(LIBVER)
	@echo files : $(ZSTD_NOMT_FILES)
	$(Q)$(CC) $(FLAGS) $^ $(LDFLAGS) $(SONAME_FLAGS) -o $@

clean:
	$(Q)$(RM) -r *.dSYM   # macOS-specific
	$(Q)$(RM) core *.o *.a *.gcda *.$(SHARED_EXT) *.$(SHARED_EXT).* libzstd.pc
	$(Q)$(RM) dll/libzstd.dll dll/libzstd.lib libzstd-nomt*
	$(Q)$(RM) -r obj/*
	@echo Cleaning library completed

#-----------------------------------------------------------------------------
# make install is validated only for below listed environments
#-----------------------------------------------------------------------------
ifneq (,$(filter $(UNAME),Linux Darwin GNU/kFreeBSD GNU OpenBSD FreeBSD NetBSD DragonFly SunOS Haiku))

all: libzstd.pc

DESTDIR     ?=
# directory variables : GNU conventions prefer lowercase
# see https://www.gnu.org/prep/standards/html_node/Makefile-Conventions.html
# support both lower and uppercase (BSD), use uppercase in script
prefix      ?= /usr/local
PREFIX      ?= $(prefix)
exec_prefix ?= $(PREFIX)
EXEC_PREFIX ?= $(exec_prefix)
libdir      ?= $(EXEC_PREFIX)/lib
LIBDIR      ?= $(libdir)
includedir  ?= $(PREFIX)/include
INCLUDEDIR  ?= $(includedir)

PCLIBDIR ?= $(shell echo "$(LIBDIR)" | sed -n $(SED_ERE_OPT) -e "s@^$(EXEC_PREFIX)(/|$$)@@p")
PCINCDIR ?= $(shell echo "$(INCLUDEDIR)" | sed -n $(SED_ERE_OPT) -e "s@^$(PREFIX)(/|$$)@@p")

ifeq (,$(PCLIBDIR))
# Additional prefix check is required, since the empty string is technically a
# valid PCLIBDIR
ifeq (,$(shell echo "$(LIBDIR)" | sed -n $(SED_ERE_OPT) -e "\\@^$(EXEC_PREFIX)(/|$$)@ p"))
$(error configured libdir ($(LIBDIR)) is outside of prefix ($(EXEC_PREFIX)), can't generate pkg-config file)
endif
endif

ifeq (,$(PCINCDIR))
# Additional prefix check is required, since the empty string is technically a
# valid PCINCDIR
ifeq (,$(shell echo "$(INCLUDEDIR)" | sed -n $(SED_ERE_OPT) -e "\\@^$(PREFIX)(/|$$)@ p"))
$(error configured includedir ($(INCLUDEDIR)) is outside of exec_prefix ($(PREFIX)), can't generate pkg-config file)
endif
endif

ifneq (,$(filter $(UNAME),FreeBSD NetBSD DragonFly))
  PKGCONFIGDIR ?= $(PREFIX)/libdata/pkgconfig
else
  PKGCONFIGDIR ?= $(LIBDIR)/pkgconfig
endif

ifneq (,$(filter $(UNAME),SunOS))
  INSTALL ?= ginstall
else
  INSTALL ?= install
endif

INSTALL_PROGRAM ?= $(INSTALL)
INSTALL_DATA    ?= $(INSTALL) -m 644


libzstd.pc:
libzstd.pc: libzstd.pc.in
	@echo creating pkgconfig
	$(Q)@sed $(SED_ERE_OPT) -e 's|@PREFIX@|$(PREFIX)|' \
          -e 's|@LIBDIR@|$(PCLIBDIR)|' \
          -e 's|@INCLUDEDIR@|$(PCINCDIR)|' \
          -e 's|@VERSION@|$(VERSION)|' \
          $< >$@

install: install-pc install-static install-shared install-includes
	@echo zstd static and shared library installed

install-pc: libzstd.pc
	$(Q)$(INSTALL) -d -m 755 $(DESTDIR)$(PKGCONFIGDIR)/
	$(Q)$(INSTALL_DATA) libzstd.pc $(DESTDIR)$(PKGCONFIGDIR)/

install-static: libzstd.a
	@echo Installing static library
	$(Q)$(INSTALL) -d -m 755 $(DESTDIR)$(LIBDIR)/
	$(Q)$(INSTALL_DATA) libzstd.a $(DESTDIR)$(LIBDIR)

install-shared: libzstd
	@echo Installing shared library
	$(Q)$(INSTALL) -d -m 755 $(DESTDIR)$(LIBDIR)/
	$(Q)$(INSTALL_PROGRAM) $(LIBZSTD) $(DESTDIR)$(LIBDIR)
	$(Q)ln -sf $(LIBZSTD) $(DESTDIR)$(LIBDIR)/libzstd.$(SHARED_EXT_MAJOR)
	$(Q)ln -sf $(LIBZSTD) $(DESTDIR)$(LIBDIR)/libzstd.$(SHARED_EXT)

install-includes:
	@echo Installing includes
	$(Q)$(INSTALL) -d -m 755 $(DESTDIR)$(INCLUDEDIR)/
	$(Q)$(INSTALL_DATA) zstd.h $(DESTDIR)$(INCLUDEDIR)
	$(Q)$(INSTALL_DATA) common/zstd_errors.h $(DESTDIR)$(INCLUDEDIR)
	$(Q)$(INSTALL_DATA) dictBuilder/zdict.h $(DESTDIR)$(INCLUDEDIR)

uninstall:
	$(Q)$(RM) $(DESTDIR)$(LIBDIR)/libzstd.a
	$(Q)$(RM) $(DESTDIR)$(LIBDIR)/libzstd.$(SHARED_EXT)
	$(Q)$(RM) $(DESTDIR)$(LIBDIR)/libzstd.$(SHARED_EXT_MAJOR)
	$(Q)$(RM) $(DESTDIR)$(LIBDIR)/$(LIBZSTD)
	$(Q)$(RM) $(DESTDIR)$(PKGCONFIGDIR)/libzstd.pc
	$(Q)$(RM) $(DESTDIR)$(INCLUDEDIR)/zstd.h
	$(Q)$(RM) $(DESTDIR)$(INCLUDEDIR)/zstd_errors.h
	$(Q)$(RM) $(DESTDIR)$(INCLUDEDIR)/zdict.h
	@echo zstd libraries successfully uninstalled

endif
