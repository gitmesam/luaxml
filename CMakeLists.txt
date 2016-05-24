project ( luaxml C )
cmake_minimum_required ( VERSION 2.8 )

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        FORCE)
endif()

SET(INSTALL_CMOD share/lua/cmod CACHE PATH "Directory to install Lua binary modules (configure lua via LUA_CPATH)")
SET(INSTALL_LMOD share/lua/lmod CACHE PATH "Directory to install pure Lua modules (configure lua via LUA_LPATH)")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")

find_package(Lua REQUIRED)

get_filename_component(_lua_lib_dir ${LUA_LIBRARY} PATH)

if ( MSVC )
  add_definitions ( -TP )
  # treat as C++ for C99-ish features
  set ( WORKAROUND _cpp )
endif ( MSVC )

set(SRC LuaXML_lib${WORKAROUND}.c)

include_directories(${LUA_INCLUDE_DIR})

if(WIN32)
	# Win32 modules need to be linked to the Lua library.
	set(_lua_module_dir "${_lua_lib_dir}")
	# Windows sprintf()/strtod() handle NaN/inf differently. Not supported.
	add_definitions(-DDISABLE_INVALID_NUMBERS)
else()
		set(_lua_module_dir "${INSTALL_CMOD}")
		set(_lua_shared_dir "${INSTALL_LMOD}")
endif()


get_filename_component(_lua_lib_dir ${LUA_LIBRARY} PATH)

add_library(LuaXML_lib MODULE ${SRC})
set_target_properties(LuaXML_lib PROPERTIES PREFIX "")
install(TARGETS LuaXML_lib DESTINATION "${_lua_module_dir}")
install(FILES LuaXml.lua DESTINATION "${_lua_shared_dir}")

