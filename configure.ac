dnl Process this file with autoconf to produce a configure script.

AC_INIT()
AC_PREREQ(2.52)

AC_MSG_NOTICE([Configuring ccache])

AC_CONFIG_HEADER(config.h)

AC_SUBST(extra_deps)
AC_SUBST(include_dev_mk)
AC_SUBST(test_suites)

dnl Checks for programs.
AC_PROG_CC
AC_PROG_CPP
AC_PROG_INSTALL
AC_PROG_RANLIB
AC_CHECK_TOOL(AR, ar)
if test -z "$AR"; then
    AC_MSG_ERROR(cannot find ar)
fi

m4_include(m4/feature_macros.m4)

# If GCC, turn on warnings.
if test "x$GCC" = "xyes"; then
    CFLAGS="$CFLAGS -Wall -W"
else
    CFLAGS="$CFLAGS -O"
fi

AC_HEADER_DIRENT
AC_HEADER_TIME
AC_HEADER_STDBOOL
AC_HEADER_SYS_WAIT

AC_CHECK_HEADERS(ctype.h pwd.h stdlib.h string.h strings.h sys/time.h sys/mman.h)
AC_CHECK_HEADERS(termios.h)

AC_CHECK_FUNCS(gethostname)
AC_CHECK_FUNCS(getopt_long)
AC_CHECK_FUNCS(getpwuid)
AC_CHECK_FUNCS(gettimeofday)
AC_CHECK_FUNCS(mkstemp)
AC_CHECK_FUNCS(realpath)
AC_CHECK_FUNCS(strndup)
AC_CHECK_FUNCS(strtok_r)
AC_CHECK_FUNCS(utimes)

AC_CACHE_CHECK([for compar_fn_t in stdlib.h],ccache_cv_COMPAR_FN_T, [
    AC_TRY_COMPILE(
        [#include <stdlib.h>],
        [void test_fn(void) { qsort(NULL, 0, 0, (__compar_fn_t)NULL); }],
        ccache_cv_COMPAR_FN_T=yes,
        ccache_cv_COMPAR_FN_T=no)])
if test x"$ccache_cv_COMPAR_FN_T" = x"yes"; then
   AC_DEFINE(HAVE_COMPAR_FN_T, 1,
             Define to 1 if you have the `__compar_fn_t' typedef.)
fi

dnl Replacements of snprintf and friends.
m4_include(m4/snprintf.m4)
HW_FUNC_VSNPRINTF
HW_FUNC_SNPRINTF
HW_FUNC_VASPRINTF
HW_FUNC_ASPRINTF

dnl Check if -lm is needed.
AC_SEARCH_LIBS(cos, m)

dnl Check for zlib.
AC_CACHE_CHECK(
    [for zlib >= 1.2.3],
    [ccache_cv_zlib_1_2_3],
    AC_TRY_COMPILE(
        [#include <zlib.h>],
        [
        #if (ZLIB_VERNUM >= 0x1230)
        #else
        #error "ZLIB_VERNUM < 0x1230"
        #endif
        ],
        [ccache_cv_zlib_1_2_3=yes],
        [ccache_cv_zlib_1_2_3=no]))
AC_CHECK_LIB(z, gzdopen, true)
if test $ccache_cv_zlib_1_2_3 = yes && test $ac_cv_lib_z_gzdopen = yes; then
    use_bundled_zlib=no
else
    use_bundled_zlib=yes
    extra_deps="zlib/libz.a"
    CPPFLAGS="$CPPFLAGS -I\$(srcdir)/zlib"
    LIBS="-Lzlib $LIBS"
    mkdir -p zlib
fi

AC_C_INLINE

dnl Check for "extern inline".
AC_CACHE_CHECK(
    for extern inline,
    ac_cv_c_extern_inline,
    [
    ac_cv_c_extern_inline=no
    AC_TRY_COMPILE(
        [
        extern $ac_cv_c_inline double foo(double x);
        extern $ac_cv_c_inline double foo(double x) { return x+1.0; };
        double foo (double x) { return x + 1.0; };
        ],
        [foo(1.0)],
        [ac_cv_c_extern_inline="yes"])])
if test "$ac_cv_c_extern_inline" != no ; then
    AC_DEFINE(HAVE_EXTERN_INLINE, 1,
              Define to 1 if your compiler supports extern inline)
fi

dnl Enable developer mode if dev.mk.in exists.
if test -f $srcdir/dev.mk.in && test "$RUN_FROM_BUILD_FARM" != yes; then
    AC_MSG_NOTICE(Enabling developer mode)
    AC_CONFIG_FILES([dev.mk])
    include_dev_mk='include dev.mk'
    version=`(git describe --dirty 2>/dev/null || echo vunknown) | sed -e 's/v//' -e 's/-/+/' -e 's/-/_/g'`
    echo "const char CCACHE_VERSION@<:@@:>@ = \"$version\";" >version.c
elif test ! -f $srcdir/version.c; then
    AC_MSG_WARN(unable to determine ccache version)
    echo "const char CCACHE_VERSION@<:@@:>@ = \"unknown\";" >version.c
fi

dnl Find test suite files.
test_suites=""
for x in $srcdir/test/test_*.c; do
    test_suites="$test_suites $x"
done

AC_CONFIG_FILES([Makefile])
AC_OUTPUT

cat <<EOF >config.h.tmp
#ifndef CCACHE_CONFIG_H
#define CCACHE_CONFIG_H
EOF
cat config.h >>config.h.tmp
echo '#endif' >>config.h.tmp
mv config.h.tmp config.h

mkdir -p .deps test

if test x$use_bundled_zlib = xyes; then
   AC_MSG_WARN(using bundled zlib)
fi

AC_MSG_NOTICE(now build ccache by running make)
