#/*
# Tutorial Framework
#*/

cmake_minimum_required(VERSION 2.6)

project(LTHS)

if(WIN32)
	set(CMAKE_MODULE_PATH "$ENV{OGRE_HOME}/CMake/;${CMAKE_MODULE_PATH}")
	set(OGRE_SAMPLES_INCLUDEPATH
		$ENV{OGRE_HOME}/Samples/include
	)
endif(WIN32)

if(UNIX)
	set(CMAKE_MODULE_PATH "/usr/lib/OGRE/cmake/;${CMAKE_MODULE_PATH}")
	set(OGRE_SAMPLES_INCLUDEPATH
		/usr/share/OGRE/samples/Common/include/
	)
endif(UNIX)

if (CMAKE_BUILD_TYPE STREQUAL "")
	set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Choose the type of build, options are: None (CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif ()
 
set(CMAKE_DEBUG_POSTFIX "_d")
 
set(CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_BINARY_DIR}/dist")
 
find_package(OGRE REQUIRED)
 
if(NOT "${OGRE_VERSION_NAME}" STREQUAL "Cthugha")
  message(SEND_ERROR "You need Ogre 1.7 Cthugha to build this.")
endif()
 
find_package(OIS REQUIRED)
 
if(NOT OIS_FOUND)
    message(SEND_ERROR "Failed to find OIS.")
endif()
 
# Find Boost
if (NOT OGRE_BUILD_PLATFORM_IPHONE)
	if (WIN32 OR APPLE)
		set(Boost_USE_STATIC_LIBS TRUE)
	else ()
		# Statically linking boost to a dynamic Ogre build doesn't work on Linux 64bit
		set(Boost_USE_STATIC_LIBS ${OGRE_STATIC})
	endif ()
	if (MINGW)
		# this is probably a bug in CMake: the boost find module tries to look for
		# boost libraries with name libboost_*, but CMake already prefixes library
		# search names with "lib". This is the workaround.
		set(CMAKE_FIND_LIBRARY_PREFIXES ${CMAKE_FIND_LIBRARY_PREFIXES} "")
	endif ()
	set(Boost_ADDITIONAL_VERSIONS "1.42" "1.42.0" "1.41.0" "1.41" "1.40.0" "1.40" "1.39.0" "1.39" "1.38.0" "1.38" "1.37.0" "1.37" )
	# Components that need linking (NB does not include header-only components like bind)
	set(OGRE_BOOST_COMPONENTS thread date_time)
	find_package(Boost COMPONENTS ${OGRE_BOOST_COMPONENTS} QUIET)
	if (NOT Boost_FOUND)
		# Try again with the other type of libs
		set(Boost_USE_STATIC_LIBS NOT ${Boost_USE_STATIC_LIBS})
		find_package(Boost COMPONENTS ${OGRE_BOOST_COMPONENTS} QUIET)
	endif()
	find_package(Boost QUIET)
 
	# Set up referencing of Boost
	include_directories(${Boost_INCLUDE_DIR})
	add_definitions(-DBOOST_ALL_NO_LIB)
	set(OGRE_LIBRARIES ${OGRE_LIBRARIES} ${Boost_LIBRARIES})
endif()
 
set(SRC_ROOT "../../src")

set(HDRS
	${SRC_ROOT}/BaseApplication.h
	${SRC_ROOT}/TutorialApplication.h
)
 
set(SRCS
	${SRC_ROOT}/BaseApplication.cpp
	${SRC_ROOT}/TutorialApplication.cpp
)
 
include_directories( ${OIS_INCLUDE_DIRS}
	${OGRE_INCLUDE_DIRS}
	${OGRE_SAMPLES_INCLUDEPATH}
)
 
add_executable(LTHS WIN32 ${HDRS} ${SRCS})
 
set_target_properties(LTHS PROPERTIES DEBUG_POSTFIX _d)
 
target_link_libraries(LTHS ${OGRE_LIBRARIES} ${OIS_LIBRARIES})
 
# post-build copy for win32
if(WIN32 AND NOT MINGW)
	add_custom_command( TARGET LTHS PRE_BUILD
		COMMAND if not exist .\\dist\\bin mkdir .\\dist\\bin )
	add_custom_command( TARGET LTHS POST_BUILD
		COMMAND copy \"$(TargetPath)\" .\\dist\\bin )
endif(WIN32 AND NOT MINGW)
 
# set the executable output path for Unix and MinGW instead of post-build copy
if(MINGW OR UNIX)
	set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/dist/bin)
endif(MINGW OR UNIX)
 
# if win32, copy the Ogre DLLs and the media over
if(WIN32)
	install(TARGETS LTHS 
		RUNTIME DESTINATION bin
		CONFIGURATIONS All)
 
	install(DIRECTORY $ENV{OGRE_HOME}/media
		DESTINATION ../
	)
 
	install(FILES $ENV{OGRE_HOME}/bin/release/OgreMain.dll
		$ENV{OGRE_HOME}/bin/release/RenderSystem_Direct3D9.dll
		$ENV{OGRE_HOME}/bin/release/RenderSystem_GL.dll
		$ENV{OGRE_HOME}/bin/release/Plugin_OctreeSceneManager.dll
		$ENV{OGRE_HOME}/bin/release/cg.dll
		$ENV{OGRE_HOME}/bin/release/OIS.dll
		$ENV{OGRE_HOME}/bin/release/Plugin_CgProgramManager.dll
		$ENV{OGRE_HOME}/bin/release/Plugin_ParticleFX.dll
		$ENV{OGRE_HOME}/bin/release/Plugin_BSPSceneManager.dll
		$ENV{OGRE_HOME}/bin/release/Plugin_CgProgramManager.dll
		$ENV{OGRE_HOME}/bin/release/Plugin_PCZSceneManager.dll
		$ENV{OGRE_HOME}/bin/release/Plugin_OctreeZone.dll
		$ENV{OGRE_HOME}/bin/release/resources.cfg
		$ENV{OGRE_HOME}/bin/release/plugins.cfg
		DESTINATION bin
		CONFIGURATIONS Release RelWithDebInfo
	)
 
	install(FILES $ENV{OGRE_HOME}/bin/debug/OgreMain_d.dll
		$ENV{OGRE_HOME}/bin/debug/RenderSystem_Direct3D9_d.dll
		$ENV{OGRE_HOME}/bin/debug/RenderSystem_GL_d.dll
		$ENV{OGRE_HOME}/bin/debug/Plugin_OctreeSceneManager_d.dll
		$ENV{OGRE_HOME}/bin/debug/cg.dll
		$ENV{OGRE_HOME}/bin/debug/OIS_d.dll
		$ENV{OGRE_HOME}/bin/debug/Plugin_CgProgramManager_d.dll
		$ENV{OGRE_HOME}/bin/debug/Plugin_ParticleFX_d.dll
		$ENV{OGRE_HOME}/bin/debug/Plugin_BSPSceneManager_d.dll
		$ENV{OGRE_HOME}/bin/debug/Plugin_CgProgramManager_d.dll
		$ENV{OGRE_HOME}/bin/debug/Plugin_PCZSceneManager_d.dll
		$ENV{OGRE_HOME}/bin/debug/Plugin_OctreeZone_d.dll
		$ENV{OGRE_HOME}/bin/debug/resources_d.cfg
		$ENV{OGRE_HOME}/bin/debug/plugins_d.cfg
		DESTINATION bin
		CONFIGURATIONS Debug
	)
endif(WIN32)
 
# if Linux, copy the Ogre configuration files over
if(UNIX AND NOT APPLE)
	install(FILES ${SRC_ROOT}/cfg/linux/plugins.cfg
		${SRC_ROOT}/cfg/linux/resources.cfg
		DESTINATION bin
		CONFIGURATIONS Release RelWithDebInfo Debug
	)
endif(UNIX AND NOT APPLE)
