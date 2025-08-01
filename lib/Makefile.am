#***************************************************************************
#                                  _   _ ____  _
#  Project                     ___| | | |  _ \| |
#                             / __| | | | |_) | |
#                            | (__| |_| |  _ <| |___
#                             \___|\___/|_| \_\_____|
#
# Copyright (C) Daniel Stenberg, <daniel@haxx.se>, et al.
#
# This software is licensed as described in the file COPYING, which
# you should have received as part of this distribution. The terms
# are also available at https://curl.se/docs/copyright.html.
#
# You may opt to use, copy, modify, merge, publish, distribute and/or sell
# copies of the Software, and permit persons to whom the Software is
# furnished to do so, under the terms of the COPYING file.
#
# This software is distributed on an "AS IS" basis, WITHOUT WARRANTY OF ANY
# KIND, either express or implied.
#
# SPDX-License-Identifier: curl
#
###########################################################################
AUTOMAKE_OPTIONS = foreign nostdinc

# Get CSOURCES, HHEADERS, LIB_RCFILES variables
include Makefile.inc

CMAKE_DIST = CMakeLists.txt curl_config.h.cmake

CHECKSRC_DIST = .checksrc \
  curlx/.checksrc vauth/.checksrc vquic/.checksrc vssh/.checksrc vtls/.checksrc

EXTRA_DIST = config-mac.h config-os400.h config-plan9.h config-riscos.h \
  config-win32.h curl_config.h.in $(LIB_RCFILES) libcurl.def            \
  $(CMAKE_DIST) Makefile.soname optiontable.pl $(CHECKSRC_DIST)

lib_LTLIBRARIES = libcurl.la

if BUILD_UNITTESTS
noinst_LTLIBRARIES = libcurlu.la

# generate a file with "private" prototypes for unit testing
UNITPROTOS = unitprotos.h

else
noinst_LTLIBRARIES =
endif

# This might hold -Werror
CFLAGS += @CURL_CFLAG_EXTRAS@

# Specify our include paths here, and do it relative to $(top_srcdir) and
# $(top_builddir), to ensure that these paths which belong to the library
# being currently built and tested are searched before the library which
# might possibly already be installed in the system.
#
# $(top_srcdir)/include is for libcurl's external include files
# $(top_builddir)/lib is for libcurl's generated lib/curl_config.h file
# $(srcdir) for the generated unity source to find included sources

AM_CPPFLAGS = -I$(top_srcdir)/include   \
              -I$(top_builddir)/lib     \
              -I$(srcdir)

# Prevent LIBS from being used for all link targets
LIBS = $(BLANK_AT_MAKETIME)

AM_LDFLAGS =
AM_CFLAGS =
if DEBUGBUILD
AM_CPPFLAGS += -DDEBUGBUILD
endif
if CURLDEBUG
AM_CPPFLAGS += -DCURLDEBUG
endif
AM_CPPFLAGS += -DBUILDING_LIBCURL

if DOING_NATIVE_WINDOWS
CSOURCES += dllmain.c
endif

if USE_UNITY
libcurl_unity.c: $(top_srcdir)/scripts/mk-unity.pl $(CSOURCES)
	@PERL@ $(top_srcdir)/scripts/mk-unity.pl --include $(CSOURCES) > libcurl_unity.c

nodist_libcurl_la_SOURCES = libcurl_unity.c
libcurl_la_SOURCES =
nodist_libcurlu_la_SOURCES = libcurl_unity.c
libcurlu_la_SOURCES =
CLEANFILES = libcurl_unity.c
else
libcurl_la_SOURCES = $(CSOURCES) $(HHEADERS)
libcurlu_la_SOURCES = $(CSOURCES) $(HHEADERS)
CLEANFILES =
endif

CLEANFILES += $(UNITPROTOS)

libcurl_la_CPPFLAGS_EXTRA =
libcurl_la_LDFLAGS_EXTRA =
libcurl_la_CFLAGS_EXTRA =

if CURL_LT_SHLIB_USE_VERSION_INFO
# Get VERSIONCHANGE, VERSIONADD, VERSIONDEL, VERSIONINFO variables
include Makefile.soname
libcurl_la_LDFLAGS_EXTRA += $(VERSIONINFO)
endif

if CURL_LT_SHLIB_USE_NO_UNDEFINED
libcurl_la_LDFLAGS_EXTRA += -no-undefined
endif

if CURL_LT_SHLIB_USE_MIMPURE_TEXT
libcurl_la_LDFLAGS_EXTRA += -mimpure-text
endif

if CURL_LT_SHLIB_USE_VERSIONED_SYMBOLS
libcurl_la_LDFLAGS_EXTRA += -Wl,--version-script=libcurl.vers
else
# if symbol-hiding is enabled, hide them!
if DOING_CURL_SYMBOL_HIDING
libcurl_la_LDFLAGS_EXTRA += -export-symbols-regex '^curl_.*'
endif
endif

if USE_CPPFLAG_CURL_STATICLIB
libcurl_la_CPPFLAGS_EXTRA += -DCURL_STATICLIB
else
if HAVE_WINDRES
libcurl_la_SOURCES += $(LIB_RCFILES)
$(LIB_RCFILES): $(top_srcdir)/include/curl/curlver.h
endif
endif

if DOING_CURL_SYMBOL_HIDING
libcurl_la_CPPFLAGS_EXTRA += -DCURL_HIDDEN_SYMBOLS
libcurl_la_CFLAGS_EXTRA += $(CFLAG_CURL_SYMBOL_HIDING)
endif

libcurl_la_CPPFLAGS = $(AM_CPPFLAGS) $(libcurl_la_CPPFLAGS_EXTRA)
libcurl_la_LDFLAGS = $(AM_LDFLAGS) $(libcurl_la_LDFLAGS_EXTRA) $(CURL_LDFLAGS_LIB) $(LIBCURL_PC_LIBS_PRIVATE)
libcurl_la_CFLAGS = $(AM_CFLAGS) $(libcurl_la_CFLAGS_EXTRA)

libcurlu_la_CPPFLAGS = $(AM_CPPFLAGS) -DCURL_STATICLIB -DUNITTESTS
libcurlu_la_LDFLAGS = $(AM_LDFLAGS) -static $(LIBCURL_PC_LIBS_PRIVATE)
libcurlu_la_CFLAGS = $(AM_CFLAGS)

CHECKSRC = $(CS_$(V))
CS_0 = @echo "  RUN     " $@;
CS_1 =
CS_ = $(CS_0)

checksrc:
	$(CHECKSRC)(@PERL@ $(top_srcdir)/scripts/checksrc.pl -D$(srcdir) \
	$(CSOURCES) $(HHEADERS))

if NOT_CURL_CI
if DEBUGBUILD
# for debug builds, we scan the sources on all regular make invokes
CHECKSOURCES = checksrc
endif
endif

all-local: $(CHECKSOURCES) $(UNITPROTOS)

UNIT_V = $(UNITV_$(V))
UNITV_0 = @echo "  UNITPR  " $@;
UNITV_1 =
UNITV_ = $(UNITV_0)

# UNITPROTOS depends on every C file in the lib/ dir
$(UNITPROTOS): $(CSOURCES)
	$(UNIT_V)(cd $(srcdir) && @PERL@ ../scripts/extract-unit-protos $(CSOURCES) > $(top_builddir)/lib/$(UNITPROTOS))

# disable the tests that are mostly causing false positives
TIDYFLAGS := -checks=-clang-analyzer-security.insecureAPI.bzero,-clang-analyzer-security.insecureAPI.strcpy,-clang-analyzer-optin.performance.Padding,-clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling -quiet
if CURL_WERROR
TIDYFLAGS += --warnings-as-errors=*
endif

TIDY := clang-tidy

tidy:
	(_csources=`echo ' $(CSOURCES)' | sed -E -e 's/ +$$//' -e 's/ +/ /g' -e 's| | $(srcdir)/|g'`; \
	$(TIDY) $$_csources $(TIDYFLAGS) $(CURL_CLANG_TIDYFLAGS) -- $(AM_CPPFLAGS) $(CPPFLAGS) -DHAVE_CONFIG_H)

optiontable:
	@PERL@ $(srcdir)/optiontable.pl < $(top_srcdir)/include/curl/curl.h > $(srcdir)/easyoptions.c

if HAVE_WINDRES
.rc.lo:
	$(LIBTOOL) --tag=RC --mode=compile $(RC) -I$(top_srcdir)/include $(RCFLAGS) -i $< -o $@
endif
