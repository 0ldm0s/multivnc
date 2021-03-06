dnl Process this file with autoconf to produce a configure script.

# init
AC_INIT(multivnc, 0.6)
AM_INIT_AUTOMAKE([subdir-objects foreign])
AC_CONFIG_MACRO_DIR([m4])

# if this macro is defined, use it
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

# the file where results of system scan go, e.g. is there some header file or not
AM_CONFIG_HEADER(config.h)

dnl this one needs a prefix
AX_PREFIX_CONFIG_H([src/rfb/rfbconfig.h], LibVNCServer)


#
# this checks the host system OS
#
AC_CANONICAL_HOST
case $host_os in
     darwin*)
	HOST_IS_DARWIN="yes"
	;;  
     *mingw32*)
	HOST_IS_WINDOWS="yes"
	;;
     *linux*)
	HOST_IS_LINUX="yes"
	;;
     *)
	HOST_IS_DARWIN="no"
	HOST_IS_WINDOWS="no"
	;;
esac


# sets automake conditionals, those are used in the .am files
AM_CONDITIONAL(MINGW, [ test "$HOST_IS_WINDOWS" = "yes" ])
AM_CONDITIONAL(DARWIN, [ test "$HOST_IS_DARWIN" = "yes" ])
AM_CONDITIONAL(LINUX, [ test "$HOST_IS_LINUX" = "yes" ])



#
# wxWidgets check stuff
#
AM_OPTIONS_WXCONFIG
reqwx=2.8.0
AM_PATH_WXCONFIG($reqwx, wxWin=1)
if test "$wxWin" != 1; then
	AC_MSG_ERROR([
		wxWidgets must be installed on your system.
 
		Please check that wx-config is in path, the directory
		where wxWidgets libraries are installed (returned by
		'wx-config --libs' or 'wx-config --static --libs' command)
		is in LD_LIBRARY_PATH or equivalent variable and
		wxWidgets version is $reqwx or above.
		])
fi


dnl Check if we have enabled debug support.
AC_MSG_CHECKING(whether to enable debugging)
AC_ARG_ENABLE(debug, [  --enable-debug          enable debug build [[default=no]]],, enable_debug="no")
if test "x$enable_debug" = "xyes"; then
	CXXFLAGS="$CXXFLAGS -Wall -Wextra -Wno-unused-parameter"
	CFLAGS="$CFLAGS -DMULTICAST_DEBUG"
	AC_MSG_RESULT(yes)
else
	CFLAGS="$CFLAGS -O2"
	CXXFLAGS="$CXXFLAGS -Wall -O2 -fno-strict-aliasing "
	AC_MSG_RESULT(no)
fi





dnl gdb under windows does backtraces only with stabs format?
if test "$HOST_IS_WINDOWS" = "yes"; then
   CFLAGS="$CFLAGS -gstabs+"
   CXXFLAGS="$CXXFLAGS -gstabs+"
fi
 

#
# only when we have pkg-config: check for NetworkManager stuffy
#
if test "$HOST_IS_LINUX" = "yes"; then
   true # noop in case the m4 ifdef evals to nothing, the if body cannot be empty
   m4_ifdef([PKG_CHECK_MODULES], [PKG_CHECK_MODULES(GTK, gtk+-2.0)])
fi


 
AC_SUBST(LIBVNCSERVER_LIBS)

AC_ISC_POSIX
AC_PROG_CPP
AC_PROG_CC
AM_PROG_CC_C_O
AC_PROG_CXX
AC_HEADER_STDC
AM_PROG_LIBTOOL
PKG_PROG_PKG_CONFIG

AC_SUBST(MULTIVNC_CFLAGS)
AC_SUBST(MULTIVNC_LIBS)



dnl ***************************************************************************
dnl Internatinalization
dnl ***************************************************************************
#GETTEXT_PACKAGE=multivnc
#AC_SUBST(GETTEXT_PACKAGE)
#AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE,"$GETTEXT_PACKAGE", [GETTEXT package name])
#IT_PROG_INTLTOOL([0.35.0])


dnl ***************************************************************************
dnl libvncclient stuff, copied over from their configure.ac
dnl ***************************************************************************

AC_ARG_WITH(jpeg,
[  --without-jpeg          disable support for jpeg]
[  --with-jpeg=DIR         use jpeg include/library files in DIR],,)

if test "x$with_jpeg" != "xno"; then
	if test ! -z "$with_jpeg" -a "x$with_jpeg" != "xyes"; then
		# add user supplied directory to flags:
		saved_CPPFLAGS="$CPPFLAGS"
		saved_LDFLAGS="$LDFLAGS"
		CPPFLAGS="$CPPFLAGS -I$with_jpeg/include"
		LDFLAGS="$LDFLAGS -L$with_jpeg/lib"
		if test "x$ld_minus_R" = "xno"; then
			:
		elif test "x$GCC" = "xyes"; then
			# this is not complete... in general a rat's nest.
			LDFLAGS="$LDFLAGS -Xlinker -R$with_jpeg/lib"
		else
			LDFLAGS="$LDFLAGS -R$with_jpeg/lib"
		fi
	fi
	AC_CHECK_HEADER(jpeglib.h, HAVE_JPEGLIB_H="true")
	if test "x$HAVE_JPEGLIB_H" = "xtrue"; then
		AC_CHECK_LIB(jpeg, jpeg_CreateCompress, , HAVE_JPEGLIB_H="")
	fi
	if test ! -z "$with_jpeg" -a "x$with_jpeg" != "xyes"; then
		if test "x$HAVE_JPEGLIB_H" != "xtrue"; then
			# restore old flags on failure:
			CPPFLAGS="$saved_CPPFLAGS"
			LDFLAGS="$saved_LDFLAGS"
		fi
	fi
	if test "$build_x11vnc" = "yes"; then
		if test "x$HAVE_JPEGLIB_H" != "xtrue"; then
			AC_MSG_WARN([
==========================================================================
*** The libjpeg compression library was not found. ***
This may lead to reduced performance, especially over slow links.
If libjpeg is in a non-standard location use --with-jpeg=DIR to
indicate the header file is in DIR/include/jpeglib.h and the library
in DIR/lib/libjpeg.a.  A copy of libjpeg may be obtained from:
ftp://ftp.uu.net/graphics/jpeg/
==========================================================================
])
			sleep 5
		fi
	fi
fi

AM_CONDITIONAL(HAVE_LIBJPEG, test ! -z "$HAVE_JPEGLIB_H")



AC_ARG_WITH(libz,
[  --without-libz          disable support for deflate],,)
AC_ARG_WITH(zlib,
[  --without-zlib          disable support for deflate]
[  --with-zlib=DIR         use zlib include/library files in DIR],,)

if test "x$with_zlib" != "xno" -a "x$with_libz" != "xno"; then
	if test ! -z "$with_zlib" -a "x$with_zlib" != "xyes"; then
		saved_CPPFLAGS="$CPPFLAGS"
		saved_LDFLAGS="$LDFLAGS"
		CPPFLAGS="$CPPFLAGS -I$with_zlib/include"
		LDFLAGS="$LDFLAGS -L$with_zlib/lib"
		if test "x$ld_minus_R" = "xno"; then
			:
		elif test "x$GCC" = "xyes"; then
			LDFLAGS="$LDFLAGS -Xlinker -R$with_zlib/lib"
		else
			LDFLAGS="$LDFLAGS -R$with_zlib/lib"
		fi
	fi
	AC_CHECK_HEADER(zlib.h, HAVE_ZLIB_H="true")
	if test "x$HAVE_ZLIB_H" = "xtrue"; then
		AC_CHECK_LIB(z, deflate, , HAVE_ZLIB_H="")
	fi
	if test ! -z "$with_zlib" -a "x$with_zlib" != "xyes"; then
		if test "x$HAVE_ZLIB_H" != "xtrue"; then
			CPPFLAGS="$saved_CPPFLAGS"
			LDFLAGS="$saved_LDFLAGS"
		fi
	fi
	if test "$build_x11vnc" = "yes"; then
		if test "x$HAVE_ZLIB_H" != "xtrue"; then
			AC_MSG_WARN([
==========================================================================
*** The libz compression library was not found. ***
This may lead to reduced performance, especially over slow links.
If libz is in a non-standard location use --with-zlib=DIR to indicate the
header file is in DIR/include/zlib.h and the library in DIR/lib/libz.a.
A copy of libz may be obtained from: http://www.gzip.org/zlib/
==========================================================================
])
			sleep 5
		fi
	fi
fi

AM_CONDITIONAL(HAVE_LIBZ, test ! -z "$HAVE_ZLIB_H")

# Check for OpenSSL
AH_TEMPLATE(HAVE_LIBCRYPT, [libcrypt library present])
AC_ARG_WITH(crypt,
[  --without-crypt         disable support for libcrypt],,)
if test "x$with_crypt" != "xno"; then
	AC_CHECK_FUNCS([crypt], HAVE_LIBC_CRYPT="true")
	if test -z "$HAVE_LIBC_CRYPT"; then
		AC_CHECK_LIB(crypt, crypt,
			CRYPT_LIBS="-lcrypt"
			[AC_DEFINE(HAVE_LIBCRYPT)], ,)
	fi
fi
AC_SUBST(CRYPT_LIBS)

# some OS's need both -lssl and -lcrypto on link line:
AH_TEMPLATE(HAVE_LIBCRYPTO, [openssl libcrypto library present])
AC_ARG_WITH(crypto,
[  --without-crypto        disable support for openssl libcrypto],,)

AH_TEMPLATE(HAVE_LIBSSL, [openssl libssl library present])
AC_ARG_WITH(ssl,
[  --without-ssl           disable support for openssl libssl]
[  --with-ssl=DIR          use openssl include/library files in DIR],,)

if test "x$with_crypto" != "xno" -a "x$with_ssl" != "xno"; then
	if test ! -z "$with_ssl" -a "x$with_ssl" != "xyes"; then
		saved_CPPFLAGS="$CPPFLAGS"
		saved_LDFLAGS="$LDFLAGS"
		CPPFLAGS="$CPPFLAGS -I$with_ssl/include"
		LDFLAGS="$LDFLAGS -L$with_ssl/lib"
		if test "x$ld_minus_R" = "xno"; then
			:
		elif test "x$GCC" = "xyes"; then
			LDFLAGS="$LDFLAGS -Xlinker -R$with_ssl/lib"
		else
			LDFLAGS="$LDFLAGS -R$with_ssl/lib"
		fi
	fi
	AC_CHECK_LIB(crypto, RAND_file_name,
		[AC_DEFINE(HAVE_LIBCRYPTO) HAVE_LIBCRYPTO="true"], ,)
	if test ! -z "$with_ssl" -a "x$with_ssl" != "xyes"; then
		if test "x$HAVE_LIBCRYPTO" != "xtrue"; then
			CPPFLAGS="$saved_CPPFLAGS"
			LDFLAGS="$saved_LDFLAGS"
		fi
	fi
fi

AH_TEMPLATE(HAVE_X509_PRINT_EX_FP, [open ssl X509_print_ex_fp available])
if test "x$with_ssl" != "xno"; then
	if test "x$HAVE_LIBCRYPTO" = "xtrue"; then
		AC_CHECK_LIB(ssl, SSL_library_init,
			SSL_LIBS="-lssl -lcrypto"
			[AC_DEFINE(HAVE_LIBSSL) HAVE_LIBSSL="true"], ,
			-lcrypto)
	else
		AC_CHECK_LIB(ssl, SSL_library_init,
			SSL_LIBS="-lssl"
			[AC_DEFINE(HAVE_LIBSSL) HAVE_LIBSSL="true"], ,)
	fi
fi
AC_SUBST(SSL_LIBS)
AM_CONDITIONAL(HAVE_LIBSSL, test ! -z "$SSL_LIBS")



# Check for libgcrypt
AH_TEMPLATE(WITH_CLIENT_GCRYPT, [Enable support for libgcrypt in libvncclient])
AC_ARG_WITH(gcrypt,
[  --without-gcrypt        disable support for gcrypt],,)
AC_ARG_WITH(client-gcrypt,
[  --without-client-gcrypt disable support for gcrypt in libvncclient],,)

if test "x$with_gcrypt" != "xno"; then
       AM_PATH_LIBGCRYPT(1.4.0, , with_client_gcrypt=no)
       CFLAGS="$CFLAGS $LIBGCRYPT_CFLAGS"
       LIBS="$LIBS $LIBGCRYPT_LIBS"
       if test "x$with_client_gcrypt" != "xno"; then
               AC_DEFINE(WITH_CLIENT_GCRYPT)
       fi
fi


# Checks for GnuTLS
AH_TEMPLATE(HAVE_GNUTLS, [GnuTLS library present])
AC_ARG_WITH(gnutls,
[  --without-gnutls        disable support for gnutls]
[  --with-gnutls=DIR       use gnutls include/library files in DIR],,)

if test "x$with_gnutls" != "xno"; then
	PKG_CHECK_MODULES(GNUTLS, gnutls >= 2.4.0)
	CFLAGS="$CFLAGS $GNUTLS_CFLAGS"
	LIBS="$LIBS $GNUTLS_LIBS"
fi
AM_CONDITIONAL(HAVE_GNUTLS, test ! -z "$GNUTLS_LIBS")
if test ! -z "$GNUTLS_LIBS" ; then
  AC_DEFINE(HAVE_GNUTLS)
fi


# warn if neither GnuTLS nor OpenSSL are available
if test -z "$SSL_LIBS" -a -z "$GNUTLS_LIBS"; then
		AC_MSG_WARN([
==========================================================================
*** No encryption library could be found. ***
A libvncserver/libvncclient built this way will not support SSL encryption.
To enable SSL install the necessary development packages (perhaps it is named
something like libssl-dev or gnutls-dev) and run configure again.
==========================================================================
])
	sleep 5
fi


# IPv6
AH_TEMPLATE(IPv6, [Enable IPv6 support])
AC_ARG_WITH(ipv6,
[  --without-ipv6          disable IPv6 support],,)
if test "x$with_ipv6" != "xno"; then
	AC_CHECK_FUNC(getaddrinfo, AC_DEFINE(IPv6,1),
		AC_CHECK_LIB(socket, getaddrinfo, AC_DEFINE(IPv6,1), [
			AC_MSG_CHECKING([for getaddrinfo in -lws2_32])
		        LIBS="$LIBS -lws2_32"
		        AC_TRY_LINK([#include <ws2tcpip.h>], [getaddrinfo(0, 0, 0, 0);], [
				      AC_DEFINE(IPv6,1)
		                      AC_MSG_RESULT([yes])
				      ],
				    AC_MSG_RESULT([no]))
                   ]))
fi


# Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS([arpa/inet.h fcntl.h netdb.h netinet/in.h stdlib.h string.h sys/socket.h sys/time.h sys/timeb.h syslog.h unistd.h ws2tcpip.h])


# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_C_INLINE
AC_C_BIGENDIAN
AC_TYPE_SIZE_T
AC_HEADER_TIME
AC_HEADER_SYS_WAIT
AC_TYPE_SOCKLEN_T
AC_CREATE_STDINT_H(src/rfb/rfbint.h)
AC_CACHE_CHECK([for in_addr_t],
        vnc_cv_inaddrt, [
                AC_TRY_COMPILE([#include <sys/types.h>
			        #include <netinet/in.h>],
                        [in_addr_t foo; return 0;],
                                        [inaddrt=yes],
                                        [inaddrt=no]),
        ])
AH_TEMPLATE(NEED_INADDR_T, [Need a typedef for in_addr_t])
if test $inaddrt = no ; then
  AC_DEFINE(NEED_INADDR_T)
fi

# Checks for library functions.
AC_FUNC_MEMCMP
AC_FUNC_STAT
AC_FUNC_STRFTIME
AC_FUNC_VPRINTF
AC_FUNC_FORK
AC_CHECK_LIB(nsl,gethostbyname)
AC_CHECK_LIB(socket,socket)




# ***************************************************************************
# finally
# ***************************************************************************
AC_OUTPUT([
Makefile
src/Makefile
src/wxServDisc/src/Makefile
src/libvncclient/Makefile
src/gui/Makefile
])
