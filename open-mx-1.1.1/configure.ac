# Open-MX
# Copyright © INRIA, CNRS 2007-2009 (see AUTHORS file)
#
# The development of this software has been funded by Myricom, Inc.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or (at
# your option) any later version.
#
# This program is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
#
# See the GNU General Public License in COPYING.GPL for more details.

AC_INIT([Open-MX],
        1.1.1,
        [http://open-mx.org],
        open-mx)
AC_CONFIG_SRCDIR(common/omx_io.h)
AC_PREREQ(2.60)
AC_CANONICAL_SYSTEM
AC_PREFIX_DEFAULT(/opt/open-mx)
AC_PRESERVE_HELP_ORDER

################################################################
# help the makefile guess whether it has been configured or not

AC_SUBST(OMX_CONFIGURED, 1)

######################
# usual config checks

AC_PROG_CPP
AC_PROG_CC
AC_PROG_MKDIR_P
AC_PROG_SED
AC_PROG_LN_S
AC_PROG_INSTALL

AC_CHECK_TOOL(AR, ar, false)
if test $AR = false; then
  AC_MSG_ERROR(ar program not found in \$PATH)
fi

########################
# save the build config

OMX_BUILD_STR="`id -run 2> /dev/null`@`uname -n`:`cd ${srcdir} && pwd` `date`"
AC_DEFINE_UNQUOTED(OMX_BUILD_STR, "$OMX_BUILD_STR", [Define to the build version string])

#####################
# build flags config

AC_SUBST(CPPFLAGS)
AC_SUBST(CFLAGS)

if test -z "$DBGCFLAGS" ; then
	DBGCFLAGS="-g -O0"
fi
AC_SUBST(DBGCFLAGS)

#####################
# what to build?

libopen_mx_dir="libopen-mx/"
driver_dir="driver/"

AC_MSG_CHECKING(whether driver source is available)
if test -d "$srcdir/$driver_dir" ; then
  AC_MSG_RESULT(yes)
  build_driver=yes
else
  AC_MSG_RESULT(no)
  build_driver=no
fi

AC_MSG_CHECKING(whether the driver has to be built)
AC_ARG_ENABLE(driver-build,
	[AS_HELP_STRING([--disable-driver-build], [do not build the driver])],
	build_driver=$enableval)
AC_MSG_RESULT($build_driver)

if test x$build_driver = xyes ; then
	driver_ac_output="driver/linux/Makefile driver/linux/Kbuild"
	AC_SUBST(OMX_BUILD_DRIVER, 1)
else
	AC_SUBST(OMX_BUILD_DRIVER, 0)
fi

AC_MSG_CHECKING(whether the library source is available)
if test -d "$srcdir/$libopen_mx_dir" ; then
  AC_MSG_RESULT(yes)
  build_lib=yes
else
  AC_MSG_RESULT(no)
  build_lib=no
fi

AC_MSG_CHECKING(whether the library and user-space tools have to be built)
AC_ARG_ENABLE(library-build,
	[AS_HELP_STRING([--disable-library-build], [do not build the library and user-space tools])],
	build_lib=$enableval)
AC_MSG_RESULT($build_lib)

if test x$build_lib = xyes ; then
	lib_ac_output="libopen-mx/Makefile libopen-mx/Makefile.build tools/Makefile tests/Makefile tests/mx/Makefile doc/man/Makefile"
	AC_SUBST(OMX_BUILD_LIB, 1)
else
	AC_SUBST(OMX_BUILD_LIB, 0)
fi

########################
# Kernel related config

AC_ARG_VAR(KCC, [Compiler for building the Open-MX kernel module])

AC_SUBST(ARCH)

if test x$build_driver = xyes ; then

    if test -n "$KCC" ; then
	AC_CHECK_PROG(KCC, gcc, $CC)
	AC_MSG_CHECKING(which compiler to use for kernel modules)
	AC_MSG_RESULT($KCC)
	KCC_VERSION=`$KCC --version 2>/dev/null | sed -ne 's/gcc.* (GCC) \(.\..\).*/\1/p'`
    else
	KCC_VERSION=`$CC --version 2>/dev/null | sed -ne 's/gcc.* (GCC) \(.\..\).*/\1/p'`
    fi

    LINUX_GCC_VERSION=`sed -e 's/.*gcc version \(.\..\).*/\1/' /proc/version`

    AC_ARG_WITH(linux-release,
	[AS_HELP_STRING([--with-linux-release=<version>], [release number of the target kernel])],
	FORCE_LINUX_RELEASE="$withval"; LINUX_GCC_VERSION="")
    if test -z "$FORCE_LINUX_RELEASE"; then
	LINUX_SRC="/lib/modules/"`uname -r`"/source"
	LINUX_BUILD="/lib/modules/"`uname -r`"/build"
    else
	LINUX_SRC="/lib/modules/$FORCE_LINUX_RELEASE/source"
	LINUX_BUILD="/lib/modules/$FORCE_LINUX_RELEASE/build"
    fi
    # If the user changes the source, use it as build by default
    # If it changes the build, do not use it as the source by default
    AC_ARG_WITH(linux,
	[AS_HELP_STRING([--with-linux=<dir>], [location of the Linux kernel source])],
	LINUX_SRC="$withval"; LINUX_BUILD="$withval"; LINUX_GCC_VERSION="")
    AC_ARG_WITH(linux-build,
	[AS_HELP_STRING([--with-linux-build=<dir>], [location of the Linux kernel build tree])],
	LINUX_BUILD="$withval"; LINUX_GCC_VERSION="")
    AC_SUBST(LINUX_SRC)
    AC_SUBST(LINUX_BUILD)

    # Issue a warning if the kernel compiler seems to be different
    if test -n "$KCC_VERSION" && test -n "$LINUX_GCC_VERSION" ; then
      if test "$KCC_VERSION" != "$LINUX_GCC_VERSION" ; then
        AC_MSG_NOTICE([kernel built with gcc $LINUX_GCC_VERSION and Open-MX with $KCC_VERSION? KCC might need to be set])
      fi
    else
      if test -n  "$KCC" ; then
        AC_MSG_NOTICE([cannot check kernel module compiler version, assuming KCC is set correctly ($KCC)])
      else
        AC_MSG_NOTICE([cannot check kernel module compiler version, assuming KCC is unset on purpose])
      fi
    fi

    # Find where the headers are (to avoid grepping at both places).
    # Do not check for autoconf.h or version.h since these are in
    # both the source and the build directory.
    AC_MSG_CHECKING(include/linux/kernel.h)
    LINUX_HDR=
    if test -f ${LINUX_SRC}/include/linux/kernel.h ; then
      LINUX_HDR=$LINUX_SRC
    else if test -f ${LINUX_BUILD}/include/linux/kernel.h ; then
      LINUX_HDR=$LINUX_BUILD
    fi fi
    if test -z ${LINUX_HDR} ; then
      AC_MSG_ERROR(not found)
    fi
    AC_MSG_RESULT(found in ${LINUX_HDR})
    AC_SUBST(LINUX_HDR)

    # Find where the autoconf header is (to avoid grepping at both places).
    # The one in the build dir is preferred since this is where the configured
    # tree should be
    AC_MSG_CHECKING(include/linux/autoconf.h)
    LINUX_AUTOCONF_DIR=
    if test -f ${LINUX_BUILD}/include/linux/autoconf.h ; then
      LINUX_AUTOCONF_DIR=${LINUX_BUILD}
    else if test -f ${LINUX_SRC}/include/linux/autoconf.h ; then
      LINUX_AUTOCONF_DIR=${LINUX_SRC}
    fi fi
    if test -z ${LINUX_AUTOCONF_DIR} ; then
      AC_MSG_ERROR(not found)
    fi
    LINUX_AUTOCONF_H=${LINUX_AUTOCONF_DIR}/include/linux/autoconf.h
    AC_MSG_RESULT(found in ${LINUX_AUTOCONF_DIR})

    # Find where the kbuild Makefile is
    AC_MSG_CHECKING(kernel Makefile)
    if ! test -f ${LINUX_BUILD}/Makefile ; then
      AC_MSG_ERROR(not found in ${LINUX_BUILD})
    fi
    AC_MSG_RESULT(found in ${LINUX_BUILD})

    # Check the kernel release
    REAL_LINUX_RELEASE=$(make kernelrelease -C ${LINUX_BUILD} | grep ^2.6.)
    AC_MSG_CHECKING(kernel release)
    if test x${REAL_LINUX_RELEASE} = x; then
      AC_MSG_ERROR(not found)
    fi
    AC_MSG_RESULT(${REAL_LINUX_RELEASE})
    if test -z "$FORCE_LINUX_RELEASE"; then
	LINUX_RELEASE="$REAL_LINUX_RELEASE"
    else
	if test "x$FORCE_LINUX_RELEASE" != "x$REAL_LINUX_RELEASE" ; then
	    AC_MSG_WARN(kernel build reports $REAL_LINUX_RELEASE instead of $FORCE_LINUX_RELEASE)
	fi
	LINUX_RELEASE="$FORCE_LINUX_RELEASE"
    fi
    AC_SUBST(LINUX_RELEASE)

fi # end of $build_driver = yes

#######################################
# options that change multilib configs

AC_ARG_ENABLE(debug,
	[AS_HELP_STRING([--enable-debug], [enable debugging features in the driver and main library])],
	enable_debug=$enableval, enable_debug=optional)
AC_MSG_CHECKING(if debug is enabled in the driver and main library)
AC_MSG_RESULT($enable_debug)

AC_ARG_ENABLE(multilib,
	[AS_HELP_STRING([--enable-multilib], [build both 32bits and 64bits libraries])],
	enable_multilib=$enableval, enable_multilib=no)
AC_MSG_CHECKING(if multilib is enabled)
AC_MSG_RESULT($enable_multilib)

##################
# multilib builds

if test x$build_lib = xyes ; then
    # find the default pointer size of the compiler
    AC_MSG_CHECKING(checking compiler default pointer size)
    AC_COMPILE_IFELSE([int a[[sizeof(void *)-7]];], defaultptrsize=64, defaultptrsize=32)
    AC_MSG_RESULT($defaultptrsize)

    # setup the creation of a lib symlink to compiler-default-pointer-size library
    AC_MSG_CHECKING(default lib symlink)
    INSTALL_LIB_LINKTARGET=lib${defaultptrsize}
    AC_MSG_RESULT(lib${defaultptrsize})
fi
AC_SUBST(INSTALL_LIB_LINKTARGET)

if test x$enable_multilib = xyes; then
  libopen_mx_default_builddir="${libopen_mx_dir}build${defaultptrsize}/"
else
  libopen_mx_default_builddir="${libopen_mx_dir}build/"
fi
libopen_mx_name="libopen-mx"
LIB_TARGETS=""

generate_lib_build() {
	dir=$1; shift
	name=$1; shift
	ptrsize=$1; shift
	debug=$1; shift
	installsuffix=$1; shift

	if test x$build_lib != xyes ; then return ; fi

	mkdir -p ${libopen_mx_dir}${dir} || exit -1

	cat >${libopen_mx_dir}${dir}/Makefile <<EOF
omx_build_lib_debug = $debug
omx_build_lib_ptrsize = $ptrsize
omx_build_lib_name = $name
omx_build_lib_dir = ${libopen_mx_dir}${dir}
libdirsuffix = ${installsuffix}
include ../Makefile.build
EOF
	test $? -eq 0 || exit -1

	AC_MSG_NOTICE(building the $name library in ${libopen_mx_dir}${dir})
	LIB_TARGETS="$LIB_TARGETS $dir"
}

if test x$enable_debug = xyes; then
  # only build the debug lib lib
  AC_DEFINE(OMX_DRIVER_DEBUG, [1], [Define to enable debug features in the driver])
  AC_SUBST(omx_debug_tests, 1)
  if test x$enable_multilib = xyes; then
    generate_lib_build build32 32bits-debug-only 32 1 32
    generate_lib_build build64 64bits-debug-only 64 1 64
  else
    generate_lib_build build debug-only "" 1 "${defaultptrsize}"
  fi
else if test x$enable_debug = xno; then
  # only build the non-debug lib
  AC_SUBST(omx_debug_tests, 0)
  if test x$enable_multilib = xyes; then
    generate_lib_build build32 32bits-normal 32 0 32
    generate_lib_build build64 64bits-normal 64 0 64
  else
    generate_lib_build build normal "" 0 "${defaultptrsize}"
  fi
else
  # build both by default
  if test x$enable_multilib = xyes; then
    generate_lib_build build32 32bits-normal 32 0 32
    generate_lib_build build64 64bits-normal 64 0 64
    generate_lib_build build32-dbg 32bits-debug 32 1 "32/debug"
    generate_lib_build build64-dbg 64bits-debug 64 1 "64/debug"
  else
    generate_lib_build build normal "" 0 "${defaultptrsize}"
    generate_lib_build build-dbg debug "" 1 "${defaultptrsize}/debug"
  fi
  AC_SUBST(omx_debug_tests, 0)
fi fi

AC_SUBST(LIB_TARGETS)

#######################################
# debug, profiling and valgrind config

AC_ARG_ENABLE(debug-requests,
	[AS_HELP_STRING([--enable-debug-requests], [enable request debugging in the library])],
	enable_debug_requests=$enableval, enable_debug_requests=no)
AC_MSG_CHECKING(if request debugging is enabled in the library)
if test x$enable_debug_requests = xyes; then
  if test x$enable_debug = xyes ; then
    AC_MSG_RESULT(yes)
  else
    AC_MSG_RESULT(yes in the debug library)
  fi
  AC_DEFINE(OMX_DEBUG_REQUESTS, [1], [Define to enable request debugging features in the library])
  AC_SUBST(OMX_DEBUG_REQUESTS, 1)
else
  AC_MSG_RESULT(no)
  AC_SUBST(OMX_DEBUG_REQUESTS, 0)
fi

AC_ARG_ENABLE(counters,
	[AS_HELP_STRING([--disable-counters], [disable counters support in the driver])],
	enable_counters=$enableval, enable_counters=yes)
AC_MSG_CHECKING(if counter support is enabled in the driver)
if test x$enable_counters = xyes; then
  AC_MSG_RESULT(yes)
  AC_DEFINE(OMX_DRIVER_COUNTERS, [1], [Define to enable countering in the driver])
  AC_SUBST(OMX_DRIVER_COUNTERS, 1)
else
  AC_MSG_RESULT(no)
  AC_SUBST(OMX_DRIVER_COUNTERS, 0)
fi

AC_ARG_ENABLE(profiling,
	[AS_HELP_STRING([--enable-profiling], [enable profiling support in the driver])],
	enable_profiling=$enableval, enable_profiling=no)
AC_MSG_CHECKING(if profiling support is enabled in the driver)
if test x$enable_profiling = xyes; then
  AC_MSG_RESULT(yes)
  AC_DEFINE(OMX_DRIVER_PROFILING, [1], [Define to enable profiling in the driver])
  AC_SUBST(OMX_DRIVER_PROFILING, 1)
else
  AC_MSG_RESULT(no)
  AC_SUBST(OMX_DRIVER_PROFILING, 0)
fi

AC_MSG_CHECKING(if valgrind is available)
AC_PREPROC_IFELSE([
#include <valgrind/memcheck.h>
#ifndef VALGRIND_MAKE_MEM_NOACCESS
#error VALGRIND_MAKE_MEM_NOACCESS not defined
#endif
], valgrind_available=yes)
if test x$valgrind_available = xyes ; then
  AC_MSG_RESULT(yes)
else
  AC_MSG_RESULT(no)
fi

AC_ARG_ENABLE(valgrind,
	[AS_HELP_STRING([--enable-valgrind], [enable Valgrind hooks in the debug or non-debug libraries])],
	enable_valgrind=$enableval, enable_valgrind=maybe)
AC_MSG_CHECKING(if valgrind hooks are enabled in the non-debug library)
if test x$valgrind_available = xyes -a x$enable_valgrind = xyes ; then
  AC_MSG_RESULT(yes)
  AC_DEFINE(OMX_LIB_VALGRIND_NONDEBUG, 1, [Enable memory allocation debugging with Valgrind in the non-debug library])
else
  AC_MSG_RESULT(no)
fi
AC_MSG_CHECKING(if valgrind hooks are enabled in the debug library)
if test x$valgrind_available = xyes -a x$enable_valgrind != xno ; then
  AC_MSG_RESULT(yes)
  AC_DEFINE(OMX_LIB_VALGRIND_DEBUG, 1, [Enable memory allocation debugging with Valgrind in the debug library])
else
  AC_MSG_RESULT(no)
fi

AC_ARG_ENABLE(recvcopy,
	[AS_HELP_STRING([--disable-recvcopy], [do not copy data on the receive side, pass random data to user-space])],
	enable_recvcopy=$enableval, enable_recvcopy=yes)
if test x$enable_recvcopy = xno ; then
  AC_MSG_CHECKING(if data is copied by the driver on the receive side)
  AC_MSG_RESULT(no)
  AC_DEFINE_UNQUOTED(OMX_NORECVCOPY, 1, [Enable to remove the driver copy on the receive side and pass random data to user-space])
# else
  # print nothing in the regular case, this is a high-debug options
fi

#######################################
# self and shared communication config

AC_ARG_ENABLE(self,
	[AS_HELP_STRING([--disable-self], [disable communication to myself])],
	enable_self=$enableval, enable_self=yes)
AC_MSG_CHECKING(if self communication is enabled)
if test x$enable_self = xyes; then
  AC_MSG_RESULT(yes)
else
  AC_MSG_RESULT(no)
  AC_DEFINE(OMX_DISABLE_SELF, [1], [Define to disable communication to myself])
fi

AC_ARG_ENABLE(shared,
	[AS_HELP_STRING([--disable-shared], [disable shared communication between endpoints on the same node])],
	enable_shared=$enableval, enable_shared=yes)
AC_MSG_CHECKING(if shared communication is enabled)
if test x$enable_shared = xyes; then
  AC_MSG_RESULT(yes)
  AC_SUBST(OMX_DISABLE_SHARED, 0)
else
  AC_MSG_RESULT(no)
  AC_DEFINE(OMX_DISABLE_SHARED, [1], [Define to disable shared communication between endpoints on the same node])
  AC_SUBST(OMX_DISABLE_SHARED, 1)
fi

############################
# MX compat and wire config

AC_ARG_ENABLE(mx-abi,
	[AS_HELP_STRING([--disable-mx-abi], [disable MX binary compatibility])],
	enable_mx_abi=$enableval, enable_mx_abi=yes)
AC_MSG_CHECKING(if MX ABI compatibility is enabled)
if test x$enable_mx_abi = xyes; then
  AC_MSG_RESULT(yes)
  AC_DEFINE(OMX_MX_ABI_COMPAT, 1, [Define to enable MX ABI compatibility])
  AC_SUBST(OMX_MX_ABI_COMPAT, 1)
else
  AC_MSG_RESULT(no)
  AC_SUBST(OMX_MX_ABI_COMPAT, 0)
fi

AC_ARG_ENABLE(mx-wire,
	[AS_HELP_STRING([--enable-mx-wire], [enable MX wire compatibility])],
	enable_mx_wire=$enableval, enable_mx_wire=no)
AC_MSG_CHECKING(if MX wire compatibility is enabled)
if test x$enable_mx_wire = xyes; then
  AC_MSG_RESULT(yes)
  AC_DEFINE(OMX_MX_WIRE_COMPAT, 1, [Define to enable MX wire compatibility])
else
  AC_MSG_RESULT(no)
fi

AC_ARG_WITH(mtu,
	[AS_HELP_STRING([--with-mtu=<n>], [change MTU (default is 9000 in non-MX-wire-compatible mode)])],
	with_mtu=$withval)
if test x$enable_mx_wire = xno; then
  AC_MSG_CHECKING(MTU)
  if test x$with_mtu != x; then
    AC_MSG_RESULT($with_mtu)
  else
    AC_MSG_RESULT([default (9000)])
    with_mtu=9000
  fi
  AC_DEFINE_UNQUOTED(OMX_MTU, $with_mtu, [Define to change the MTU])
else
  if test x$with_mtu != x; then
    AC_MSG_ERROR([Changing the MTU requires MX-wire compatibility to be disabled (--disable-mx-wire)])
  fi
fi

AC_ARG_WITH(medium-frag-length,
	[AS_HELP_STRING([--with-medium-frag-length=<n>], [change medium fragment maximum payload size (default depends on the MTU and wire compatibility)])],
	with_medium_frag_length=$withval)
if test x$with_medium_frag_length != x; then
  if test x$enable_mx_wire = xno; then
    AC_MSG_CHECKING(medium frag length)
    AC_MSG_RESULT($with_medium_frag_length)
    AC_DEFINE_UNQUOTED(OMX_MEDIUM_FRAG_LENGTH_MAX, $with_medium_frag_length, [Define to change the medium frag length max])
  else
    AC_MSG_ERROR([Changing the packet size requires MX-wire compatibility to be disabled (--disable-mx-wire)])
  fi
fi

AC_ARG_WITH(pull-reply-length,
	[AS_HELP_STRING([--with-pull-reply-length=<n>], [change pull reply maximum payload size (default depends on the MTU and wire compatibility)])],
	with_pull_reply_length=$withval)
if test x$with_pull_reply_length != x; then
  if test x$enable_mx_wire = xno; then
    AC_MSG_CHECKING(pull reply length)
    AC_MSG_RESULT($with_pull_reply_length)
    AC_DEFINE_UNQUOTED(OMX_PULL_REPLY_LENGTH_MAX, $with_pull_reply_length, [Define to change the pull reply length max])
  else
    AC_MSG_ERROR([Changing the packet size requires MX-wire compatibility to be disabled (--disable-mx-wire)])
  fi
fi

AC_ARG_WITH(pull-block-replies,
	[AS_HELP_STRING([--with-pull-block-replies=<n>], [change the number of pull replies per block (default is 32 in non-MX-wire-compatible mode)])],
	with_pull_block_replies=$withval)
if test x$with_pull_block_replies != x; then
  if test x$enable_mx_wire = xno; then
    AC_MSG_CHECKING(pull block replies)
    AC_MSG_RESULT($with_pull_block_replies)
    AC_DEFINE_UNQUOTED(OMX_PULL_REPLY_PER_BLOCK, $with_pull_block_replies, [Define to change the number of pull replies per block])
  else
    AC_MSG_ERROR([Changing the number of pull replies per block requires MX-wire compatibility to be disabled (--disable-mx-wire)])
  fi
fi

AC_ARG_WITH(ethertype,
	[AS_HELP_STRING([--with-ethertype=<value>], [change Ethertype (default is 0x86DF)])],
	ethertype=$withval, ethertype=default)
AC_MSG_CHECKING(the Ethertype to use)
if test x$ethertype = xdefault; then
  AC_MSG_RESULT([default (0x86df)])
else
  AC_MSG_RESULT($ethertype)
  AC_DEFINE_UNQUOTED(ETH_P_OMX, $ethertype, [Define to change the Ethertype])
fi

AC_ARG_ENABLE(endian,
	[AS_HELP_STRING([--disable-endian], [disable endian-independent networking])],
	enable_endian=$enableval, enable_endian=yes)
AC_MSG_CHECKING(if endian-independent networking is enabled)
if test x$enable_endian = xyes; then
  AC_MSG_RESULT(yes)
  AC_DEFINE(OMX_ENDIAN_COMPAT, 1, [Define to enable endian-independent networking])
else
  AC_MSG_RESULT(no)
  if test x$enable_mx_wire = xyes; then
    AC_MSG_WARN([endian-independent networking is required to talk to native MX peers])
  fi
fi

############################
# misc config

AC_ARG_ENABLE(threads,
	[AS_HELP_STRING([--disable-threads], [disable thread safety in the library])],
	enable_lib_threads=$enableval, enable_lib_threads=yes)
AC_MSG_CHECKING(if thread safety is enabled in the library)
if test x$enable_lib_threads = xyes; then
  AC_MSG_RESULT(yes)
  AC_DEFINE(OMX_LIB_THREAD_SAFETY, 1, [Define to enable thread safety support in the library])
  AC_SUBST(OMX_LIB_THREAD_SAFETY, 1)
else
  AC_MSG_RESULT(no)
fi

AC_ARG_ENABLE(dynamic-tools,
	[AS_HELP_STRING([--enable-dynamic-tools], [links tools and tests dynamically])],
	enable_dynamic_tools=$enableval, enable_dynamic_tools=no)
AC_MSG_CHECKING(if tools and tests are statically linked)
if test x$enable_dynamic_tools = xyes; then
  AC_MSG_RESULT(yes)
  AC_SUBST(OMX_DYNAMIC_TOOLS, 1)
else
  AC_MSG_RESULT(no)
  AC_SUBST(OMX_DYNAMIC_TOOLS, 0)
fi

#################
# peer discovery

OMX_PEER_DISCOVERY=omxoed

AC_ARG_ENABLE(fma,
	[AS_HELP_STRING([--disable-fma], [never use FMA])],
	enable_fma=$enableval, enable_fma=yes)

AC_MSG_CHECKING([whether FMA is enabled])
if test "$enable_fma" = no ; then
  AC_MSG_RESULT([no])
else
  AC_MSG_RESULT([yes])
  AC_MSG_CHECKING([whether the FMA source is available])
  if test -d "$srcdir/fma/" ; then
    AC_MSG_RESULT([yes])
    if test "$enable_mx_abi" = yes ; then
      AC_MSG_NOTICE([using FMA for dynamic peer discovery])
      OMX_PEER_DISCOVERY=fma
    else
      AC_MSG_WARN([Building FMA requires the MX API compatibility])
    fi
  else
    AC_MSG_RESULT([no])
  fi
fi

if test $OMX_PEER_DISCOVERY = fma ; then
  AC_SUBST(OMX_PEER_DISCOVERY, fma)
  AC_CONFIG_SUBDIRS(fma)
else
  AC_SUBST(OMX_PEER_DISCOVERY, omxoed)
  AC_MSG_NOTICE([using omxoed for dynamic peer discovery])
  if test "$enable_mx_wire" = yes -a "$enable_fma" = yes ; then
    AC_MSG_ERROR([FMA is recommended for dynamic discovery of native MX peers, use --disable-fma to force])
  fi
fi

AC_ARG_ENABLE(static-peers,
	[AS_HELP_STRING([--enable-static-peers], [use static peer instead of dynamic discovery])],
	static_peers=$enableval, static_peers=no)
AC_MSG_CHECKING(how peer mapping is performed)
if test "$static_peers" = yes; then
  AC_MSG_RESULT(static)
  AC_SUBST(OMX_STATIC_PEERS, 1)
else
  AC_MSG_RESULT(dynamic)
  AC_SUBST(OMX_STATIC_PEERS, 0)
fi

AC_ARG_WITH(peers-file,
	[AS_HELP_STRING([--with-peers-file=<path>], [use argument as a static peers file])],
	with_peers_file=$withval, with_peers_file=/etc/open-mx/peers)
AC_MSG_NOTICE(using static peers file in $with_peers_file)
AC_SUBST(OMX_PEERS_FILE, $with_peers_file)

######################
# FMS specific config

AC_ARG_WITH(fms-run,
	[AS_HELP_STRING([--with-fms-run=<dir>], [directory where fms data is to be stored])],
	FMS_RUN=$withval, FMS_RUN=/var/run/fms)
AC_ARG_WITH(fms-server,
	[AS_HELP_STRING([--with-fms-server=<fms_server>], [node on which fms process will run])],
	FMS_SERVER=$withval, FMS_SERVER="")
AC_SUBST(FMS_RUN)
AC_SUBST(FMS_SERVER)

######################
# installation config

AC_ARG_WITH(device,
	[AS_HELP_STRING([--with-device=<path>:<group>:<mode>],
	[change main device node name, group and/or mode])],
	main_device_attrs=$withval, main_device_attrs="")

AC_MSG_CHECKING(main device node name)
OMX_MAIN_DEVICE_NAME=open-mx
OMX_MAIN_DEVICE_GROUP=root
OMX_MAIN_DEVICE_MODE=0666
main_device_path=$(echo $main_device_attrs | cut -d: -f1)
main_device_group=$(echo $main_device_attrs | cut -d: -f2 -s)
main_device_mode=$(echo $main_device_attrs | cut -d: -f3 -s)
if ! test -z "$main_device_path"; then
  main_device_path=$(echo "$main_device_path" | sed -r -e 's@//+@/@g')
  main_device_name=${main_device_path##/dev/}
  if test "$main_device_name" = "$main_device_path"; then
    AC_MSG_ERROR(device path must start with /dev/)
  fi
  if test -z "$main_device_name" ; then
    AC_MSG_ERROR(cannot use empty device name within /dev/)
  fi
  OMX_MAIN_DEVICE_NAME="$main_device_name"
fi
if ! test -z "$main_device_group"; then
  OMX_MAIN_DEVICE_GROUP=$main_device_group
fi
if ! test -z "$main_device_mode"; then
  OMX_MAIN_DEVICE_MODE=$main_device_mode
fi
AC_MSG_RESULT($OMX_MAIN_DEVICE_NAME (group=$OMX_MAIN_DEVICE_GROUP, mode=$OMX_MAIN_DEVICE_MODE))
AC_DEFINE_UNQUOTED(OMX_MAIN_DEVICE_NAME, "$OMX_MAIN_DEVICE_NAME", [Define to change the main device node name])
AC_SUBST(OMX_MAIN_DEVICE_NAME)
AC_SUBST(OMX_MAIN_DEVICE_GROUP)
AC_SUBST(OMX_MAIN_DEVICE_MODE)

AC_ARG_WITH(raw-device,
	[AS_HELP_STRING([--with-raw-device=<path>:<group>:<mode>],
	[change raw device node name, group and/or mode])],
	raw_device_attrs=$withval, raw_device_attrs="")

AC_MSG_CHECKING(raw device node name)
OMX_RAW_DEVICE_NAME=open-mx-raw
OMX_RAW_DEVICE_GROUP=root
OMX_RAW_DEVICE_MODE=0660
raw_device_path=$(echo $raw_device_attrs | cut -d: -f1)
raw_device_group=$(echo $raw_device_attrs | cut -d: -f2 -s)
raw_device_mode=$(echo $raw_device_attrs | cut -d: -f3 -s)
if ! test -z "$raw_device_path"; then
  raw_device_path=$(echo "$raw_device_path" | sed -r -e 's@//+@/@g')
  raw_device_name=${raw_device_path##/dev/}
  if test "$raw_device_name" = "$raw_device_path"; then
    AC_MSG_ERROR(raw device path must start with /dev/)
  fi
  if test -z "$raw_device_name" ; then
    AC_MSG_ERROR(cannot use empty raw device name within /dev/)
  fi
  OMX_RAW_DEVICE_NAME="$raw_device_name"
fi
if ! test -z "$raw_device_group"; then
  OMX_RAW_DEVICE_GROUP=$raw_device_group
fi
if ! test -z "$raw_device_mode"; then
  OMX_RAW_DEVICE_MODE=$raw_device_mode
fi
AC_MSG_RESULT($OMX_RAW_DEVICE_NAME (group=$OMX_RAW_DEVICE_GROUP, mode=$OMX_RAW_DEVICE_MODE))
AC_DEFINE_UNQUOTED(OMX_RAW_DEVICE_NAME, "$OMX_RAW_DEVICE_NAME", [Define to change the raw device node name])
AC_SUBST(OMX_RAW_DEVICE_NAME)
AC_SUBST(OMX_RAW_DEVICE_GROUP)
AC_SUBST(OMX_RAW_DEVICE_MODE)

##########
# Globals

AC_SUBST(OPEN_MX_LIB_STATIC, ${libopen_mx_name}.a)
AC_SUBST(OPEN_MX_LIB_DYNAMIC, ${libopen_mx_name}.so)
AC_SUBST(OPEN_MX_LIB_DYNAMIC_LINK_OPT, ${libopen_mx_name/lib/-l})
AC_SUBST(MX_LIB_DYNAMIC_LINK_OPT, -lmyriexpress)
AC_SUBST(OPEN_MX_DEFAULT_LIB_DIR, ${libopen_mx_default_builddir})
AC_SUBST(OPEN_MX_DEFAULT_LIB_STATIC_PATH, ${libopen_mx_default_builddir}/${libopen_mx_name}.a)
AC_SUBST(OPEN_MX_DEFAULT_LIB_DYNAMIC_PATH, ${libopen_mx_default_builddir}/${libopen_mx_name}.so)

if test x$build_driver = xyes ; then
    orig_builddir=$(pwd)
    mkdir -p driver/linux/ ; cd driver/linux

    # Check kernel headers early
    AC_MSG_NOTICE([checking kernel headers ...])
    ${orig_builddir}/${srcdir}/driver/linux/check_kernel_headers.sh --force omx_checks.h "$LINUX_BUILD" "$LINUX_HDR" "$LINUX_RELEASE"

    if ( cd ${orig_builddir} && test ! "$srcdir" -ef . ) then
	# Symlink kernel sources into the build tree if needed
	AC_MSG_NOTICE([creating symlinks to kernel sources in driver/linux/ ...])
	for file in ${orig_builddir}/${srcdir}/driver/linux/*.[[ch]] ; do
	    # don't symlink omx_checks.h, it has been generated in builddir above.
	    # this will avoid problems if it has been generated in srcdir as well.
	    if test "$(basename $file)" != omx_checks.h ; then
		rm -f $(basename $file) && ln -s $file
	    fi
	done
    fi

    cd ${orig_builddir} > /dev/null
fi

AC_CONFIG_HEADER(common/omx_config.h)
AC_OUTPUT(Makefile $driver_ac_output $lib_ac_output)
