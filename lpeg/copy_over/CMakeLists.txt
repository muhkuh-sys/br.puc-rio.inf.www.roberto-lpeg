CMAKE_MINIMUM_REQUIRED(VERSION 3.22)

PROJECT("lpeg")

ENABLE_TESTING()
FIND_PACKAGE(Python3 REQUIRED COMPONENTS Interpreter)
SET(PYTHON_EXECUTABLE "${Python3_EXECUTABLE}")

FIND_PACKAGE(lua5.4 CONFIG REQUIRED)
GET_TARGET_PROPERTY(LUA_INCLUDE_DIR org.lua.lua::lua5.4 INTERFACE_INCLUDE_DIRECTORIES)

# Build the Lua module.
ADD_LIBRARY(TARGET_lpeg MODULE
            lpcap.c
            lpcode.c
            lpcset.c
            lpprint.c
            lptree.c
            lpvm.c)
TARGET_INCLUDE_DIRECTORIES(TARGET_lpeg
                           PRIVATE ${LUA_INCLUDE_DIR})
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	TARGET_LINK_LIBRARIES(TARGET_lpeg lua5.4)
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Set the name of the output file to "lpeg".
SET_TARGET_PROPERTIES(TARGET_lpeg PROPERTIES PREFIX "" OUTPUT_NAME "lpeg")

# On mingw link all compiler libraries static.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	SET_PROPERTY(TARGET TARGET_lpeg PROPERTY LINK_FLAGS "-static -static-libgcc -static-libstdc++")
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Install the lua module.
INSTALL(TARGETS TARGET_lpeg
        EXPORT EXPORT_package
        LIBRARY DESTINATION ${INSTALL_DIR_LUA_MODULES}
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)

# Add tests for this module.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	# Here are the MinGW specific tests.
	ADD_TEST(NAME lpeg_MinGW_DLL_dependencies
	         COMMAND "${PYTHON_EXECUTABLE}" ${TEST_SCRIPT_FOLDER}/mingw_dll_dependencies.py -u lua5.1 -u lua5.2 -u lua5.3 -u lua5.4 $<TARGET_FILE:TARGET_lpeg>)
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Install all LUA scripts.
INSTALL(FILES re.lua
        DESTINATION ${INSTALL_DIR_LUA_SCRIPTS})
