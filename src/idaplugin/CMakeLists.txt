##
## CMake build script for the IDA Pro plugin.
##

# Target name *MUST* be the same as plugin source directory.
set(TARGET_NAME "idaplugin")
set(FINAL_PLUGIN_NAME "retdec")

# Windows.
if(MSVC)
	add_definitions(-D__NT__) #-DWIN32
	set(IDASDK_LIBRARIES "${IDA_SDK_DIR}/lib/x64_win_vc_32/ida.lib")
elseif(APPLE)
	ADD_DEFINITIONS (-D__MAC__)
	set(IDASDK_LIBRARIES "${IDA_SDK_DIR}/lib/x64_mac_gcc_32/libida.dylib")
# Linux.
else()
	add_definitions(-D__LINUX__)

	# From idasdk/readme.txt:
	# There are no ida.a libraries for Linux, since you can link directly
	# to the shared library of the IDA kernel (libida.so or libida.dylib).
	# This shared library comes with IDA itself (not with the SDK).
	set(IDASDK_LIBRARIES "")
endif()

# Make our includes work.
include_directories("..")

# Make includes from IDA SDK work.
include_directories(SYSTEM
	"${IDA_SDK_DIR}/include"
)

# Build.
set(IDAPLUGIN_SOURCES
	code_viewer.cpp
	config_generator.cpp
	decompiler.cpp
	defs.cpp
	idaplugin.cpp
	plugin_config.cpp
)

add_library(${TARGET_NAME} SHARED ${IDAPLUGIN_SOURCES})

target_link_libraries(${TARGET_NAME}
	retdec
	jsoncpp
	${IDASDK_LIBRARIES}
)
if(WIN32)
	# shlwapi.dll for PathRemoveFileSpec() used in utilsl/filesystem_path.h
	target_link_libraries(${TARGET_NAME} shlwapi)
endif()
if(MSYS)
	target_link_libraries(${TARGET_NAME} ws2_32)
endif()

# Set a proper name of the library that represents the plugin (e.g. retdec.dylib
# on macOS, retdec.so on Linux and retdec.dll on Windows).
set_target_properties(${TARGET_NAME} PROPERTIES PREFIX "")
set_target_properties(${TARGET_NAME} PROPERTIES OUTPUT_NAME "${FINAL_PLUGIN_NAME}")

if(IDA_PATH)
	install(TARGETS ${TARGET_NAME}
		LIBRARY DESTINATION "${IDA_PATH}/plugins/"
		RUNTIME DESTINATION "${IDA_PATH}/plugins/"
	)
endif()
