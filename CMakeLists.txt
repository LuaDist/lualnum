# Copyright (C) 2007-2012 LuaDist.
# Submitted by David Manura (based on CMakeLists.txt for Lua 5.1.4 package).
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.

project ( lualnum C )
cmake_minimum_required ( VERSION 2.8 )
include ( cmake/dist.cmake )
include ( lua )
# BEGIN END LNUM PATCH

# Apply LNUM patch and remove luaconf.h.
file ( READ src/lua.h LUAHSTR )
string ( REGEX MATCH LUA_LNUM LUAHSTR_LNUM ${LUAHSTR} )
if ( NOT LUAHSTR_LNUM )
  execute_process ( COMMAND patch -p 1 -i lua514-lnum-20090417-custom.patch WORKING_DIRECTORY 
    ${CMAKE_SOURCE_DIR} )
endif ( NOT LUAHSTR_LNUM )
file ( REMOVE ${CMAKE_SOURCE_DIR}/src/luaconf.h )

# Select number features.
set ( LNUM_DOUBLE 1 )
#SET(LNUM_FLOAT 1)
#SET(LNUM_LDOUBLE 1)
set ( LNUM_INT32 1 )
#SET(LNUM_INT64 1)
set ( LNUM_COMPLEX 1 )
#~2DO: complex.h not available on Cygwin and MSVC

# Generate luaconf.h
configure_file ( src/luaconf.h.cmake luaconf.h )
include_directories ( ${CMAKE_BINARY_DIR} )

# Compiler defines.
add_definitions ( --std=gnu99 )
# for complex.h
#~2DO - msvc?
#ADD_DEFINITIONS(-D_GNU_SOURCE) # prevent implict decl warnings on c99
#~2DO above not effective on Cygwin.  -D_POSIX_SOURCE? -D_XOPEN_SOURCE?

# END LNUM PATCH

# Determine install host
if ( WIN32 AND NOT CYGWIN )
  add_definitions ( -DLUA_BUILD_AS_DLL )
  if ( MSVC )
    add_definitions ( -D_CRT_SECURE_NO_DEPRECATE )
    set ( DEF_FILE src/lua.def )
    set ( DLL_RC_FILE src/lua_dll.rc )
    set ( LUA_RC_FILE src/lua.rc )
    set ( LUAC_RC_FILE src/lua_simple.rc )
  endif ( )
else ( )
  add_definitions ( -DLUA_USE_POSIX -DLUA_USE_DLOPEN )
  set ( LIBS m dl )
endif ( )

# Add Readline support when available
find_path ( READLINE_INCLUDE_DIR readline/readline.h )
find_library ( READLINE_LIBRARY NAMES readline )
if ( READLINE_LIBRARY )
  include_directories ( ${READLINE_INCLUDE_DIR} )
  add_definitions ( -DLUA_USE_READLINE )
  set ( LIBS ${LIBS} ${READLINE_LIBRARY} )
endif ( )

# Add Curses support when available
include ( FindCurses )
if ( CURSES_LIBRARY )
  include_directories ( ${CURSES_INCLUDE_DIR} )
  set ( LIBS ${LIBS} ${CURSES_LIBRARY} )
endif ( )

# Build Libraries
set ( SRC_LIBLUA src/lapi.c src/lcode.c src/ldebug.c src/ldo.c src/ldump.c src/lfunc.c 
  src/lgc.c src/llex.c src/lmem.c src/lobject.c src/lopcodes.c src/lparser.c src/lstate.c 
  src/lstring.c src/ltable.c src/ltm.c src/lundump.c src/lvm.c src/lzio.c src/lauxlib.c 
  src/lbaselib.c src/ldblib.c src/liolib.c src/lmathlib.c src/loslib.c src/ltablib.c 
  src/lstrlib.c src/loadlib.c src/linit.c )

# LNUM PATCH - ADD LINE
set ( SRC_LIBLUA ${SRC_LIBLUA} src/lnum.c )

add_library ( liblua ${SRC_LIBLUA} ${DEF_FILE} ${DLL_RC_FILE} )
target_link_libraries ( liblua ${LIBS} )
set_target_properties ( liblua PROPERTIES OUTPUT_NAME lua CLEAN_DIRECT_OUTPUT 1 )

add_library ( liblua_static ${SRC_LIBLUA} )
target_link_libraries ( liblua_static ${LIBS} )

include_directories ( src )

# Build Executables
set ( SRC_LUA src/lua.c )
set ( SRC_LUAC src/luac.c src/print.c )

add_executable ( lua ${SRC_LUA} ${LUA_RC_FILE} )
add_executable ( luac ${SRC_LUAC} ${LUAC_RC_FILE} )
target_link_libraries ( lua liblua )
target_link_libraries ( luac liblua_static )

# Install
install_executable ( lua luac )
install_library ( liblua )
install_header ( src/lua.h ${CMAKE_BINARY_DIR}/luaconf.h src/luaconf_internal.h src/lualib.h 
  src/lauxlib.h etc/lua.hpp )
install_lua_module ( strict etc/strict.lua )
install_doc ( doc/ )
install_test ( tect/ ) 
install_foo ( etc/ )
install_data ( README COPYRIGHT HISTORY README.LNUM  )

