if (NOT CMAKE_VERSION VERSION_LESS "3.12")
	find_package(Python3 COMPONENTS Interpreter Development REQUIRED)
	include_directories(${Python3_INCLUDE_DIRS} ${Python3_INCLUDE_DIRS}/..)
else()
	find_package (PythonLibs ${PYTHON_VERSION_STRING} EXACT) # Maps PythonLibs to the PythonInterp version of the main cmake
	include_directories(${PYTHON_INCLUDE_DIRS} ${PYTHON_INCLUDE_DIRS}/..)
endif()

find_package(Qt5Widgets REQUIRED)

# generate windows .rc file for this binary
if (WIN32)
	include(${CMAKE_SOURCE_DIR}/cmake/win/win_rc.cmake)
	generate_win_rc_file(hyperiond)
endif()

add_executable(hyperiond
	console.h
	hyperiond.h
	systray.h
	hyperiond.cpp
	systray.cpp
	main.cpp
	${hyperiond_WIN_RC_PATH}
)

# promote hyperiond as GUI app
if (WIN32)
	target_link_options(hyperiond PUBLIC /SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup)
endif()

target_link_libraries(hyperiond
	commandline
	hyperion
	effectengine
	jsonserver
	flatbufserver
	protoserver
	webserver
	ssdp
	database
	python
	resources
	Qt5::Widgets
)

if (NOT CMAKE_VERSION VERSION_LESS "3.12")
	target_link_libraries( hyperiond ${Python3_LIBRARIES} )
else()
	target_link_libraries( hyperiond ${PYTHON_LIBRARIES} )
endif()

if (ENABLE_AVAHI)
	target_link_libraries(hyperiond bonjour)
endif ()

if (ENABLE_V4L2)
	target_link_libraries(hyperiond v4l2-grabber)
endif ()

if (ENABLE_QTC)
	target_link_libraries(hyperiond qtc-grabber)
endif ()

if(NOT WIN32)
	install ( TARGETS hyperiond DESTINATION "share/hyperion/bin" COMPONENT "Hyperion" )
	install ( DIRECTORY ${CMAKE_SOURCE_DIR}/bin/service DESTINATION "share/hyperion/" COMPONENT "Hyperion" )
	install ( FILES ${CMAKE_SOURCE_DIR}/effects/readme.txt DESTINATION "share/hyperion/effects" COMPONENT "Hyperion" )
	install ( FILES ${CMAKE_SOURCE_DIR}/resources/icons/hyperion-icon-32px.png DESTINATION "share/hyperion/icons" COMPONENT "Hyperion" )

	# Desktop file for hyperiond
	install ( FILES ${CMAKE_SOURCE_DIR}/cmake/desktop/hyperiond_128.png DESTINATION "share/hyperion/desktop" COMPONENT "Hyperion" )
	install ( FILES ${CMAKE_SOURCE_DIR}/cmake/desktop/hyperiond.desktop DESTINATION "share/hyperion/desktop" COMPONENT "Hyperion" )
else()
	install ( TARGETS hyperiond DESTINATION "bin" COMPONENT "Hyperion" )
	install ( FILES ${CMAKE_SOURCE_DIR}/effects/readme.txt DESTINATION "effects" COMPONENT "Hyperion" )

	#set( CMAKE_INSTALL_UCRT_LIBRARIES TRUE )
	#set( CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS_SKIP TRUE )
	include( InstallRequiredSystemLibraries )
endif()

if(CMAKE_HOST_UNIX)
	install( CODE "EXECUTE_PROCESS(COMMAND ln -sf \"../share/hyperion/bin/hyperiond\" \"${CMAKE_BINARY_DIR}/symlink_hyperiond\" )" COMPONENT "Hyperion" )
	install( FILES ${CMAKE_BINARY_DIR}/symlink_hyperiond DESTINATION "bin" RENAME hyperiond COMPONENT "Hyperion" )
	install( CODE "FILE (REMOVE ${CMAKE_BINARY_DIR}/symlink_hyperiond )" COMPONENT "Hyperion" )
endif()

# Copy dependencies (not for OSX)
include(${CMAKE_SOURCE_DIR}/cmake/Dependencies.cmake)

if (NOT WIN32) # Unix
	DeployUnix("hyperiond")
elseif(WIN32) # Windows
	DeployWindows("hyperiond")
endif ()
