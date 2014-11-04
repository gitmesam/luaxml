project ( luaxml C )
cmake_minimum_required ( VERSION 2.8 )

option(USE_LUA51 "Use Lua (PUC-Rio Lua) version 5.1 includes (default)" ON)
option(USE_LUA52 "Use Lua (PUC-Rio Lua) version 5.2 includes")
option(USE_LUA53 "Use Lua (PUC-Rio Lua) version 5.3 includes")
option(USE_LUAJIT "Use LuaJIT includes instead of PUC-Rio Lua ones (recommended, if you're using LuaJIT, but disabled by default)")

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        FORCE)
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")


if(USE_LUAJIT)
    set(USE_LUA53 OFF)
    set(USE_LUA52 OFF)
    set(USE_LUA51 OFF)
elseif(USE_LUA53)
    set(USE_LUA51 OFF)
    set(USE_LUA52 OFF)
    set(USE_LUAJIT OFF)
elseif(USE_LUA52)
    set(USE_LUA51 OFF)
    set(USE_LUA53 OFF)
    set(USE_LUAJIT OFF)
elseif(USE_LUA51)
    set(USE_LUA53 OFF)
    set(USE_LUA52 OFF)
    set(USE_LUAJIT OFF)
endif()

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
	if (USE_LUA53)
		set(_lua_module_dir "${_lua_lib_dir}/lua/5.3")
		set(_lua_shared_dir "${_lua_lib_dir}/../share/lua/5.3")
	elseif (USE_LUA52)
		set(_lua_module_dir "${_lua_lib_dir}/lua/5.2")
		set(_lua_shared_dir "${_lua_lib_dir}/../share/lua/5.2")
	else ()
		set(_lua_module_dir "${_lua_lib_dir}/lua/5.1")
		set(_lua_shared_dir "${_lua_lib_dir}/../share/lua/5.1")
	endif ()
endif()


get_filename_component(_lua_lib_dir ${LUA_LIBRARY} PATH)

add_library(LuaXML_lib MODULE ${SRC})
set_target_properties(LuaXML_lib PROPERTIES PREFIX "")
install(TARGETS LuaXML_lib DESTINATION "${_lua_module_dir}")
install(FILES LuaXML.lua DESTINATION "${_lua_shared_dir}")

#install_lua_module ( LuaXML_lib LuaXML_lib${WORKAROUND}.c )
#install_lua_module ( LuaXml LuaXml.lua )
