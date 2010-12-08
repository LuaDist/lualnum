# Copyright (C) 2007-2009 LuaDist.
# Submitted by David Manura (based on CMakeLists.txt for Lua 5.1.4 package).
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.

PROJECT ( lualnum C )
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
INCLUDE(dist.cmake )

# BEGIN END LNUM PATCH

# Apply LNUM patch and remove luaconf.h.
FILE(READ src/lua.h LUAHSTR)
STRING(REGEX MATCH LUA_LNUM LUAHSTR_LNUM ${LUAHSTR})
IF (NOT LUAHSTR_LNUM)
  EXECUTE_PROCESS(
    COMMAND patch -p 1 -i lua514-lnum-20090417-custom.patch
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  )
ENDIF (NOT LUAHSTR_LNUM)
FILE(REMOVE ${CMAKE_SOURCE_DIR}/src/luaconf.h)

# Select number features.
SET(LNUM_DOUBLE 1)
#SET(LNUM_FLOAT 1)
#SET(LNUM_LDOUBLE 1)
SET(LNUM_INT32 1)
#SET(LNUM_INT64 1)
SET(LNUM_COMPLEX 1)  #~2DO: complex.h not available on Cygwin and MSVC

# Generate luaconf.h
CONFIGURE_FILE(src/luaconf.h.cmake luaconf.h)
INCLUDE_DIRECTORIES(${CMAKE_BINARY_DIR})

# Compiler defines.
ADD_DEFINITIONS(--std=gnu99) # for complex.h
  #~2DO - msvc?
#ADD_DEFINITIONS(-D_GNU_SOURCE) # prevent implict decl warnings on c99
#~2DO above not effective on Cygwin.  -D_POSIX_SOURCE? -D_XOPEN_SOURCE?

# END LNUM PATCH

# Determine install host
IF ( WIN32 AND NOT CYGWIN)
	ADD_DEFINITIONS ( -DLUA_BUILD_AS_DLL )
	IF(MSVC)
		ADD_DEFINITIONS(-D_CRT_SECURE_NO_DEPRECATE )
		SET(DEF_FILE src/lua.def)
		SET(DLL_RC_FILE src/lua_dll.rc)
		SET(LUA_RC_FILE src/lua.rc)
		SET(LUAC_RC_FILE src/lua_simple.rc)
	ENDIF()
ELSE ( )
	ADD_DEFINITIONS ( -DLUA_USE_POSIX -DLUA_USE_DLOPEN )
	SET ( LIBS m dl )
ENDIF ( )

# Add Readline support when available
FIND_PATH(READLINE_INCLUDE_DIR readline/readline.h)
FIND_LIBRARY(READLINE_LIBRARY NAMES readline)
IF ( READLINE_LIBRARY )
	INCLUDE_DIRECTORIES ( ${READLINE_INCLUDE_DIR} )
	ADD_DEFINITIONS ( -DLUA_USE_READLINE )
	SET ( LIBS ${LIBS} ${READLINE_LIBRARY} )
ENDIF ( )

# Add Curses support when available
INCLUDE(FindCurses)
IF ( CURSES_LIBRARY )
	INCLUDE_DIRECTORIES ( ${CURSES_INCLUDE_DIR} )
	SET ( LIBS ${LIBS} ${CURSES_LIBRARY} )
ENDIF ( )

# Build Libraries
SET ( SRC_LIBLUA src/lapi.c src/lcode.c src/ldebug.c src/ldo.c src/ldump.c src/lfunc.c src/lgc.c src/llex.c src/lmem.c src/lobject.c src/lopcodes.c src/lparser.c src/lstate.c src/lstring.c src/ltable.c src/ltm.c src/lundump.c src/lvm.c src/lzio.c src/lauxlib.c src/lbaselib.c src/ldblib.c src/liolib.c src/lmathlib.c src/loslib.c src/ltablib.c src/lstrlib.c src/loadlib.c src/linit.c)

# LNUM PATCH - ADD LINE
SET ( SRC_LIBLUA ${SRC_LIBLUA} src/lnum.c )

ADD_LIBRARY ( liblua SHARED ${SRC_LIBLUA} ${DEF_FILE} ${DLL_RC_FILE})
TARGET_LINK_LIBRARIES ( liblua ${LIBS})
SET_TARGET_PROPERTIES ( liblua PROPERTIES OUTPUT_NAME lua CLEAN_DIRECT_OUTPUT 1 )

ADD_LIBRARY ( liblua_static ${SRC_LIBLUA} )
TARGET_LINK_LIBRARIES ( liblua_static ${LIBS})

INCLUDE_DIRECTORIES ( src )

# Build Executables
SET ( SRC_LUA src/lua.c )
SET ( SRC_LUAC src/luac.c src/print.c )

ADD_EXECUTABLE ( lua ${SRC_LUA} ${LUA_RC_FILE})
ADD_EXECUTABLE ( luac ${SRC_LUAC} ${LUAC_RC_FILE})
TARGET_LINK_LIBRARIES ( lua liblua )
TARGET_LINK_LIBRARIES ( luac liblua_static )

# Install
INSTALL ( TARGETS lua luac liblua RUNTIME DESTINATION ${INSTALL_BIN} LIBRARY DESTINATION ${INSTALL_LIB} ARCHIVE DESTINATION ${INSTALL_LIB})

# LNUM PATCH - CHANGED LINE
INSTALL ( FILES src/lua.h ${CMAKE_BINARY_DIR}/luaconf.h src/luaconf_internal.h src/lualib.h src/lauxlib.h etc/lua.hpp DESTINATION ${INSTALL_INC} )

INSTALL ( FILES etc/strict.lua DESTINATION ${INSTALL_LMOD} )
INSTALL ( DIRECTORY doc etc test DESTINATION ${INSTALL_DATA} PATTERN ".svn" EXCLUDE )
INSTALL ( FILES README COPYRIGHT HISTORY DESTINATION ${INSTALL_DATA} )

# LNUM PATCH - ADDED LINE
INSTALL ( FILES README.LNUM DESTINATION ${INSTALL_DATA} )

