#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.65])
AC_INIT([toxic], [0.1.1], [http://tox.im/])
AC_CONFIG_AUX_DIR(configure_aux)
AC_CONFIG_SRCDIR([src/main.c])
AC_CONFIG_HEADERS([config.h])
AM_INIT_AUTOMAKE([1.10 -Wall])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])
AC_CONFIG_MACRO_DIR([m4])

if test "x${prefix}" = "xNONE"; then
    prefix="${ac_default_prefix}"
fi

DEPSEARCH=
LIBTOXCORE_SEARCH_HEADERS=
LIBTOXCORE_SEARCH_LIBS=
LIBSODIUM_SEARCH_HEADERS=
LIBSODIUM_SEARCH_LIBS=

AC_ARG_WITH(dependency-search,
    AC_HELP_STRING([--with-dependency-search=DIR],
                   [search for dependencies in DIR, i.e. look for libraries in
                    DIR/lib and for headers in DIR/include]),
    [
        DEPSEARCH="$withval"
    ]
)

if test -n "$DEPSEARCH"; then
    CFLAGS="$CFLAGS -I$DEPSEARCH/include"
    CPPFLAGS="$CPPFLAGS -I$DEPSEARCH/include"
    LDFLAGS="$LDFLAGS -L$DEPSEARCH/lib"
    export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:$DEPSEARCH/lib/pkgconfig
fi

AC_ARG_WITH(libtoxcore-headers,
        AC_HELP_STRING([--with-libtoxcore-headers=DIR],
                       [search for libtoxcore header files in DIR]),
        [
            LIBTOXCORE_SEARCH_HEADERS="$withval"
            AC_MSG_NOTICE([Will search for libtoxcore header files in $withval])
        ]
)

AC_ARG_WITH(libtoxcore-libs,
        AC_HELP_STRING([--with-libtoxcore-libs=DIR],
                       [search for libtoxcore libraries in DIR]),
        [
            LIBTOXCORE_SEARCH_LIBS="$withval"
            AC_MSG_NOTICE([Will search for libtoxcore libraries in $withval])
        ]
)

AC_ARG_WITH(libsodium-headers,
        AC_HELP_STRING([--with-libsodium-headers=DIR],
                       [search for libsodium header files in DIR]),
        [
            LIBSODIUM_SEARCH_HEADERS="$withval"
            AC_MSG_NOTICE([Will search for libsodium header files in $withval])
        ]
)

AC_ARG_WITH(libsodium-libs,
        AC_HELP_STRING([--with-libsodium-libs=DIR],
                       [search for libsodium libraries in DIR]),
        [
            LIBSODIUM_SEARCH_LIBS="$withval"
            AC_MSG_NOTICE([Will search for libsodium libraries in $withval])
        ]
)

WIN32=no
AC_CANONICAL_HOST
case $host_os in
    *mingw*)
        WIN32="yes"
    ;;
esac


# Checks for programs.
AC_PROG_CC
AM_PROG_CC_C_O

AC_CHECK_HEADERS(
    [limits.h locale.h stdint.h stdlib.h string.h unistd.h wchar.h wctype.h],
    [],
    [ AC_MSG_ERROR([required header is missing on your system]) ])

# Checks for typedefs, structures, and compiler characteristics.
AC_HEADER_STDBOOL
AC_TYPE_SIZE_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT64_T
AC_TYPE_UINT8_T

# Checks for library functions.
AC_FUNC_MALLOC
AC_CHECK_FUNCS(
    [iswprint memmove memset mkdir setlocale strchr strdup],
    [],
    [ AC_MSG_ERROR([required library function is missing on your system])])

# pkg-config based tests
PKG_PROG_PKG_CONFIG

NCURSES_WIDECHAR_SUPPORT="yes"
if test -n "$PKG_CONFIG"; then
    if test "$WIN32" != "xyes"; then
        PKG_CHECK_MODULES([NCURSES], [ncursesw],
            [],
            [
                NCURSES_WIDECHAR_SUPPORT="no"
                PKG_CHECK_MODULES([NCURSES], [ncurses],
                    [],
                    [
                        AC_MSG_ERROR([required library ncursesw was not found on your system: $NCURSES_PKG_ERRORS])
                    ])
            ])
    fi
else
    AC_MSG_WARN([pkg-config was not found on your sytem])
fi


if (test -z "$PKG_CONFIG") || (test "x$WIN32" = "xyes"); then
    AC_CHECK_HEADER([curses.h],
        [],
        [
            AC_MSG_ERROR([headers for the ncurses library were not found on your system])
        ]
    )

    if test "x$WIN32" = "xyes"; then
        dnl Check if pdcurses provides wide char support
        NCURSES_WIDECHAR_SUPPORT="no"
        AC_CHECK_LIB([pdcurses], [clear],
            [],
            [
                AC_MSG_ERROR([required library pdcurses was not found on your system])
            ]
        )

        AC_CHECK_LIB(ws2_32, main,
            [
                WINSOCK2_LIBS="-lws2_32"
                AC_SUBST(WINSOCK2_LIBS)
            ],
            [
                AC_MSG_ERROR([required library winsock2 was not found on the system, please check your MinGW installation])
            ]
        )
    else
        AC_CHECK_LIB([ncursesw], [get_wch],
            [],
            [
                unset ac_cv_lib_ncursesw_get_wch
                AC_CHECK_LIB([ncursesw], [get_wch],
                    [],
                    [
                        NCURSES_WIDECHAR_SUPPORT="no"
                        AC_CHECK_LIB([ncurses], [clear],
                            [],
                            [
                                unset ac_cv_lib_ncursesw_get_wch
                                AC_CHECK_LIB([ncurses], [clear],
                                    [],
                                    [
                                        AC_MSG_ERROR([required library ncurses was not found on your system])
                                    ],
                                    [
                                        -ltinfo
                                    ]
                                )
                            ]
                        )
                    ],
                    [
                        -ltinfo
                    ]
                )
            ]
        )
    fi
fi

# sodium is included by Tox headers so we kind of need to know where it is
LIBSODIUM_CFLAGS=
CFLAGS_SAVE="$CFLAGS"
CPPFLAGS_SAVE="$CPPFLAGS"
if test -n "$LIBSODIUM_SEARCH_HEADERS"; then
    CFLAGS="$CFLAGS -I$LIBSODIUM_SEARCH_HEADERS"
    CPPFLAGS="$CFLAGS -I$LIBSODIUM_SEARCH_HEADERS"
    AC_CHECK_HEADER(sodium.h,
        [
            LIBSODIUM_CFLAGS="-I$LIBSODIUM_SEARCH_HEADERS"
        ],
        [
            AC_MSG_ERROR([header files for required library libsodium was not found in requested location $LIBSODIUM_SEARCH_HEADERS])
        ]
    )
else
    AC_CHECK_HEADER(sodium.h,
        [],
        [
            AC_MSG_ERROR([header files for required library libsodium was not found on your system, please check http://download.libsodium.org/libsodium/releases/])
        ]
    )
fi
CFLAGS="$CFLAGS_SAVE"
CPPFLAGS="$CPPFLAGS_SAVE"
AC_SUBST(LIBSODIUM_CFLAGS)


LIBSODIUM_LIBS=
LIBSODIUM_LDFLAGS=
LDFLAGS_SAVE="$LDFLAGS"
if test -n "$LIBSODIUM_SEARCH_LIBS"; then
    LDFLAGS="$LDFLAGS -L$LIBSODIUM_SEARCH_LIBS"
    AC_CHECK_LIB(sodium, randombytes_random,
        [
            LIBSODIUM_LDFLAGS="-L$LIBSODIUM_SEARCH_LIBS"
            LIBSODIUM_LIBS="-lsodium"
        ],
        [
            AC_MSG_ERROR([required library libsodium was not found in requested location $LIBSODIUM_SEARCH_LIBS])
        ]
    )
else
    AC_CHECK_LIB(sodium, randombytes_random,
        [],
        [
            AC_MSG_ERROR([required library libsodium was not found on your system, please check http://download.libsodium.org/libsodium/releases/])
        ]
    )
fi

LDFLAGS="$LDFLAGS_SAVE"
AC_SUBST(LIBSODIUM_LIBS)
AC_SUBST(LIBSODIUM_LDFLAGS)



LIBTOXCORE_CFLAGS=
CFLAGS_SAVE="$CFLAGS"
CPPFLAGS_SAVE="$CPPFLAGS"

if test -n "$LIBTOXCORE_SEARCH_HEADERS"; then
    CFLAGS="$CFLAGS -I$LIBTOXCORE_SEARCH_HEADERS $LIBSODIUM_CFLAGS"
    CPPFLAGS="$CPPFLAGS -I$LIBTOXCORE_SEARCH_HEADERS $LIBSODIUM_CFLAGS"
    AC_CHECK_HEADER([Messenger.h],
        [
            LIBTOXCORE_CFLAGS="-I$LIBTOXCORE_SEARCH_HEADERS"
        ],
        [
            AC_MSG_ERROR([headers for the toxcore library were not found on your system])
        ]
    )
else
    CFLAGS="$CFLAGS $LIBSODIUM_CFLAGS"
    CPPFLAGS="$CPPFLAGS $LIBSODIUM_CFLAGS"
    AC_CHECK_HEADER([Messenger.h],
        [],
        [
            AC_MSG_ERROR([headers for the toxcore library were not found on your system])
        ]
    )
fi
CFLAGS="$CFLAGS_SAVE"
CPPFLAGS="$CPPFLAGS_SAVE"
AC_SUBST(LIBTOXCORE_CFLAGS)

LIBTOXCORE_LIBS=
LIBTOXCORE_LDFLAGS=
LDFLAGS_SAVE="$LDFLAGS"
if test -n "$LIBTOXCORE_SEARCH_LIBS"; then
    LDFLAGS="$LDFLAGS $LIBSODIUM_LDFLAGS -L$LIBTOXCORE_SEARCH_LIBS $LIBSODIUM_LIBS"
    AC_CHECK_LIB([toxcore], [initMessenger],
        [
            LIBTOXCORE_LDFLAGS="-L$LIBTOXCORE_SEARCH_LIBS"
            LIBTOXCORE_LIBS="-ltoxcore"
        ],
        [
            AC_MSG_ERROR([required library toxcore was not found on your system])
        ]
    )
else
    LDFLAGS="$LDFLAGS $LIBSODIUM_LDFLAGS $LIBSODIUM_LIBS"
    AC_CHECK_LIB([toxcore], [initMessenger],
        [],
        [
            AC_MSG_ERROR([required library toxcore was not found on your system])
        ]
    )
fi
LDFLAGS="$LDFLAGS_SAVE"
AC_SUBST(LIBTOXCORE_LIBS)
AC_SUBST(LIBTOXCORE_LDFLAGS)



TOXIC_VERSION="$PACKAGE_VERSION"
AC_PATH_PROG([GIT], [git], [no])
if test "x$GIT" != "xno"; then
    if test -d ${srcdir}/.git; then
        TOXIC_VERSION="${TOXIC_VERSION}_r`${GIT} rev-list HEAD --count`"
    fi
fi
AC_SUBST(TOXIC_VERSION)

eval PACKAGE_DATADIR="${datadir}/${PACKAGE}"
eval PACKAGE_DATADIR="${PACKAGE_DATADIR}"
AC_DEFINE_UNQUOTED(PACKAGE_DATADIR, "$PACKAGE_DATADIR", [toxic data directory])

if test "x$NCURSES_WIDECHAR_SUPPORT" = "xyes"; then
    AC_DEFINE([HAVE_WIDECHAR], [1], [ncurses wide char support available])
    AC_DEFINE([_XOPEN_SOURCE_EXTENDED], [1],
              [enable X/Open Portability Guide functionality])
fi
 
AC_CONFIG_FILES([Makefile
                 misc/Makefile
                 build/Makefile])
AC_OUTPUT
