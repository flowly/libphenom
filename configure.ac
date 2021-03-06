AC_INIT([phenom], [0.1], [], [phenom])
AC_CANONICAL_TARGET

case "$target_os-$target_cpu" in
  solaris*i386)
    dnl force us to consider this a 64-bit system.
    dnl also: you'd have to be insane to run 32-bit intel solaris.
    target_cpu=x86_64
    LDFLAGS="$LDFLAGS -lumem"
    ;;
esac
case "$target_cpu" in
  x86_64|amd64)
    CFLAGS="$CFLAGS -m64 -fno-omit-frame-pointer"
    ;;
	i*86|pentium*|athlon*|k5|k6|k6_2|k6_3)
    target_cpu=i686
    CFLAGS="$CFLAGS -m32 -march=i686 -msse -msse2"
    ;;
esac

dnl We always want large file support in our implementation.
dnl The functions we provide explicitly use 64-bit types and
dnl thus we do not force this requirement on our clients
CFLAGS="$CFLAGS -D_LARGEFILE_SOURCE -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64"

AM_INIT_AUTOMAKE([dist-bzip2 subdir-objects])

AC_PROG_CC
AC_C_BIGENDIAN
AC_C_INLINE
AC_PROG_RANLIB

AM_PROG_CC_C_O
AM_PROG_AS

lenient=no
AC_ARG_ENABLE(lenient, [
  --enable-lenient  Turn off more pedantic levels of warnings
                    and compilation checks
],[
   lenient=$enableval
])

rtm=no
AC_ARG_ENABLE(rtm, [
  --enable-rtm Turn on support for Intel TSX extensions
],[
   rtm=$enableval
])

if test "$rtm" == "yes" ; then
    RTM_ENABLE=CK_MD_RTM_ENABLE
else
    RTM_ENABLE=CK_MD_RTM_DISABLE
fi

stack_protect=no
AC_ARG_ENABLE(stack-protector, [
  --enable-stack-protector  Enable stack protection in the same
                            way that rpmbuild does on some systems.
],[
   stack_protect=$enableval
])

dnl flags we'll export to the pkg-config file
PC_CFLAGS=""
PC_LIBS=""

AC_SEARCH_LIBS([pthread_create], [pthread])
AC_SEARCH_LIBS([socket], [socket])
AC_SEARCH_LIBS([clock_gettime], [rt])
AC_SEARCH_LIBS([inet_pton], [nsl])

PC_LIBS="$LIBS"

AC_PATH_PROG(PKG_CONFIG, pkg-config, false)

if ! $PKG_CONFIG --exists --print-errors libcares ; then
  AC_MSG_ERROR([c-ares is required])
fi

ares_rpath=""
case "$target_os" in
  solaris*|darwin*)
    ;;
  *)
    dnl GNU LD specific; Solaris wants s/L/R/ but moving fast for now
    ares_rpath=`$PKG_CONFIG --libs-only-L libcares | sed -e 's/L/Wl,-rpath=/'`
    ;;
esac
LIBS="$LIBS `$PKG_CONFIG --libs libcares` $ares_rpath"
CFLAGS="$CFLAGS `$PKG_CONFIG --cflags libcares`"
AC_CHECK_FUNCS(ares_init)

AC_PATH_PROG(PHP, php, true)
AC_SUBST(PHP)

gimli=no
AC_ARG_WITH(gimli, [
  --with-gimli    Enable support for the gimli process monitor
                  https://bitbucket.org/wez/gimli/
],[
   gimli=$withval
])
if test "x$gimli" != "xno" ; then
  AC_DEFINE([USE_GIMLI], 1, [Use gimli])

  prefixes="$gimli /usr/local /usr"
  mylibdir="lib"
  rpath="-R"
  case $target_os-$target_cpu in
    linux-*64)
      mylibdir=lib64
      rpath="-Wl,-rpath="
      ;;
    linux*)
      rpath="-Wl,-rpath="
      ;;
  esac
  save_flags="$LDFLAGS"
  save_libs="$LIBS"
  save_cppflags="$CPPFLAGS"
  gimli_dir=""
  for dir in $prefixes ; do
    LDFLAGS="$save_flags $rpath$dir/$mylibdir -L$dir/$mylibdir"
    LIBS="$save_libs -lgimli"
    CPPFLAGS="$save_cppflags -I$dir/include"
    AC_TRY_LINK([
#include <libgimli.h>
],[gimli_heartbeat_attach();],
      [gimli_dir=$dir]
    )
    if test -n "$gimli_dir" ; then
      AC_CHECK_HEADERS(libgimli.h)
      save_flags="$save_flags $rpath$dir/$mylibdir -L$dir/$mylibdir"
      save_libs="$save_libs -lgimli"
      save_cppflags="$save_cppflags -I$dir/include"
      break
    fi
  done
  LDFLAGS="$save_flags"
  LIBS="$save_libs"
  CPPFLAGS="$save_cppflags"
fi

dnl for comparitive benchmarking
AC_ARG_ENABLE(libevent-comparison, [
  --enable-libevent-comparison  Build iopipes.c with libevent support
],[
  SAVE_LIBS="$LIBS"
  AC_SEARCH_LIBS([event_add], [event],[
    HAVE_LIBEVENT=yes
    AC_DEFINE(HAVE_LIBEVENT, 1, have libevent)
  ])
  LIBS="$SAVE_LIBS"
])
AM_CONDITIONAL(HAVE_LIBEVENT, test "x$HAVE_LIBEVENT" = xyes)

have_thread=no
AC_TRY_COMPILE([__thread int tls;],[tls;],
               [have_thread=yes])
if test "$have_thread" == "yes" ; then
  case "$target_os" in
    solaris*)
      dnl I get linker errors with this enabled; seems broken
      ;;
    *)
      AC_DEFINE(HAVE___THREAD, [], [Have __thread TLS])
      ;;
  esac
fi

AC_CHECK_HEADERS(\
alloca.h \
inttypes.h \
locale.h \
port.h \
pthread.h \
stdbool.h \
stddef.h \
stdint.h \
string.h \
sys/inotify.h \
sys/epoll.h \
sys/event.h \
sys/eventfd.h \
sys/param.h \
sys/processor.h \
sys/procset.h \
sys/resource.h \
sys/timerfd.h \
)

dnl needs NBBY on FreeBSD
AC_CHECK_HEADERS([sys/cpuset.h],[],[],[[
#ifdef HAVE_SYS_PARAM_H
#include <sys/param.h>
#endif
]])

AC_CHECK_HEADERS([pthread_np.h],[],[],[[
#include <pthread.h>
]])

AC_CHECK_FUNCS(\
accept4 \
asprintf \
backtrace \
backtrace_symbols \
backtrace_symbols_fd \
clock_gettime \
cpuset_setaffinity \
epoll_create \
epoll_create1 \
getpagesize \
inotify_init \
kqueue \
localeconv \
pipe2 \
port_create \
processor_bind \
pthread_set_name_np \
pthread_setname_np \
pthread_setaffinity_np \
pthread_mach_thread_np \
strerror_r \
strtoll \
sysctlbyname \
thread_policy_set \
timerfd_create \
)

# Virtual memory address size in bits; CK uses this for
# pointer-packing tricks
VMA=unknown
case "$target_cpu" in
  x86_64|amd64)
    MM=CK_MD_TSO
    CPU=X86_64

    case "$target_os" in
      darwin)
        VMA=`sysctl -n machdep.cpu.address_bits.virtual`
        ;;
      linux)
        VMA=`awk '/address sizes/ {print $7;exit}' /proc/cpuinfo`
        ;;
    esac
    ;;
  i686)
    MM=CK_MD_TSO
    CPU=X86
    ;;
  *)
    AC_ERROR([too lazy to port CK for this platform $target_os-$target_cpu])
    ;;
esac
if test "$VMA" == "unknown" ; then
  VMA_BITS="CK_MD_VMA_BITS_UNKNOWN"
  VMA_BITS_VALUE=1
  POINTER_PACK_ENABLE="CK_MD_POINTER_PACK_DISABLE"
else
  VMA_BITS="CK_MD_VMA_BITS"
  VMA_BITS_VALUE="${VMA}ULL"
  POINTER_PACK_ENABLE="CK_MD_POINTER_PACK_ENABLE"
fi

case "$CPU" in
  X86_64)
    PC_CFLAGS="-D__x86_64__=1"
    ;;
  X86)
    PC_CFLAGS="-D__x86__=1"
    ;;
esac
CFLAGS="$CFLAGS $PC_CFLAGS"

AC_SUBST(MM)
AC_SUBST(VMA_BITS)
AC_SUBST(VMA_BITS_VALUE)
AC_SUBST(POINTER_PACK_ENABLE)
AC_SUBST(RTM_ENABLE)

# Do this after we've looked for functions
if test -n "$GCC" ; then
  GCC_VERSION=`$CC -dumpversion`
  GCC_VERSION_MAJOR=`echo $GCC_VERSION | cut -d. -f1`
  GCC_VERSION_MINOR=`echo $GCC_VERSION | cut -d. -f2`

  # Need to select c1x if we want to use _Static_assert
  if expr \( $GCC_VERSION_MAJOR \* 10000 \) \+ \( $GCC_VERSION_MINOR \* 100 \) \>= 40600 >/dev/null ; then
    GCC_STD=gnu1x
  else
    GCC_STD=gnu99
  fi
  # export the std setting so that clients can use ph_static_assert
  PC_CFLAGS="$PC_CFLAGS -std=$GCC_STD"
  CFLAGS="$CFLAGS -std=$GCC_STD \
    -pedantic -Wall -Wextra -Wno-declaration-after-statement \
    -W -Wundef -Wendif-labels -Wshadow -Wpointer-arith -Wcast-align \
    -Wwrite-strings -Wstrict-prototypes -Wmissing-prototypes \
    -Wno-nested-externs -Winline -Wdisabled-optimization \
    -fstrict-aliasing -pipe -Wno-parentheses"
fi
AC_SUBST(PC_CFLAGS)
AC_SUBST(PC_LIBS)

if test -n "$GCC" -a "$stack_protect" == "yes" ; then
  CFLAGS="$CFLAGS -Wp,-D_FORTIFY_SOURCE=2 -fexceptions \
    -fstack-protector --param=ssp-buffer-size=4"
fi

dnl CK needs optimization enabled otherwise it breaks.
dnl So turn it on unless an optimization level was explicitly
dnl specified in CFLAGS
case "$CFLAGS" in
  *-O*)
    ;;
  *)
    CFLAGS="$CFLAGS -O2"
    ;;
esac
dnl always include debug info in what we build
CFLAGS="$CFLAGS -g -gdwarf-2"

IRONMANCFLAGS=""
if test -n "$GCC" -a "$lenient" == "no" ; then
  IRONMANCFLAGS="-Werror"
fi
AC_SUBST(IRONMANCFLAGS)

AC_CONFIG_HEADER([include/phenom_build_config.h])
AC_CONFIG_FILES([Makefile thirdparty/ck/include/ck_md.h libphenom.pc])
AC_OUTPUT

dnl vim:ts=2:sw=2:

