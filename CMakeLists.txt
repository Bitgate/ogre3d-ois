cmake_minimum_required(VERSION 2.8)
project(OIS)

set(OIS_MAJOR_VERSION 1)
set(OIS_MINOR_VERSION 4)
set(OIS_PATCH_VERSION 0)

set(OIS_VERSION ${OIS_MAJOR_VERSION}.${OIS_MINOR_VERSION}.${OIS_PATCH_VERSION})
set(OIS_COMPATIBILITY_VERSION ${OIS_MAJOR_VERSION}.${OIS_MINOR_VERSION}.0)
set(OIS_SOVERSION ${OIS_MAJOR_VERSION}.${OIS_MINOR_VERSION}.0)

set(OIS_LIBRARY_NAME OIS)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/includes/OISPrereqs.h.in ${CMAKE_CURRENT_SOURCE_DIR}/includes/OISPrereqs.h @ONLY)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING
        "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
        FORCE)
endif()

option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(OIS_BUILD_DEMOS "Build demo applications" ON)
set(CMAKE_MACOSX_RPATH 0)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/includes
    ${CMAKE_CURRENT_SOURCE_DIR}/includes/linux
    ${CMAKE_CURRENT_SOURCE_DIR}/includes/SDL
    ${CMAKE_CURRENT_SOURCE_DIR}/includes/iphone
    ${CMAKE_CURRENT_SOURCE_DIR}/includes/mac
    ${CMAKE_CURRENT_SOURCE_DIR}/includes/win32
)

set(ois_source
    ${ois_source}
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OISInputManager.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OISObject.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OISEffect.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OISJoyStick.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OISKeyboard.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OISForceFeedback.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OISException.cpp"
)

if (UNIX AND NOT APPLE)

    find_package(X11 REQUIRED)
    include_directories(SYSTEM ${X11_INCLUDE_DIR})

    set(ois_source
        ${ois_source}
        "${CMAKE_CURRENT_SOURCE_DIR}/src/linux/EventHelpers.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/linux/LinuxForceFeedback.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/linux/LinuxInputManager.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/linux/LinuxJoyStickEvents.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/linux/LinuxKeyboard.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/linux/LinuxMouse.cpp"
    )
endif()

if (WIN32)
    # I'm assuming that this will be set under all normal circumstances
    # if not, the user can set it themselves
    set(DXSDK_DIR "$ENV{DXSDK_DIR}" CACHE PATH "Location of the DirectX SDK on your system.")

	# Attempt the most common Direct X directory and check for the required included files.
	if(NOT DXSDK_DIR)
		if(EXISTS "C:/Program Files (x86)/Microsoft DirectX SDK (June 2010)/Include/d3dx9.h")
			set(DXSDK_DIR "C:/Program Files (x86)/Microsoft DirectX SDK (June 2010)")
		endif()
	endif()

    if ("${DXSDK_DIR}" STREQUAL "")
        MESSAGE(FATAL_ERROR "Could not locate DirectX SDK on this system.\nYou can download DirectX 9 at: https://www.microsoft.com/en-us/download/details.aspx?id=6812.")
    else()
        MESSAGE("Found DirectX SDK at ${DXSDK_DIR}")
    endif()

    if(BUILD_SHARED_LIBS)
        add_definitions(-DOIS_DYNAMIC_LIB)
    endif(BUILD_SHARED_LIBS)

    include_directories("${DXSDK_DIR}/Include")

    set(ois_source
        ${ois_source}
        "${CMAKE_CURRENT_SOURCE_DIR}/src/win32/Win32ForceFeedback.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/win32/Win32InputManager.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/win32/Win32JoyStick.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/win32/Win32KeyBoard.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/win32/Win32Mouse.cpp"
    )
endif()

if (APPLE)
    set(CMAKE_MACOSX_RPATH 1)
    find_library(COCOA_LIBRARY Cocoa)
    find_library(FOUNDATION_LIBRARY Foundation)
    find_library(IOKIT_LIBRARY IOKit)
    find_path(COCOA_INCLUDE_DIR Cocoa/Cocoa.h)
    find_library(CARBON_LIBRARY Carbon)

    add_definitions(-x objective-c++)

    include_directories(SYSTEM ${COCOA_INCLUDE_DIR})
    set(ois_source
        ${ois_source}
        "${CMAKE_CURRENT_SOURCE_DIR}/src/mac/MacHelpers.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/mac/MacHIDManager.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/mac/MacInputManager.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/mac/MacJoyStick.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/mac/MacKeyboard.cpp"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/mac/MacMouse.cpp"

        "${CMAKE_CURRENT_SOURCE_DIR}/src/mac/CocoaInputManager.mm"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/mac/CocoaJoyStick.mm"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/mac/CocoaKeyboard.mm"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/mac/CocoaMouse.mm"
    )
    set(OIS_BUILD_DEMOS true)
endif()

source_group("OIS" FILES ${ois_source})

add_library(OIS ${ois_source})

# This is neccessary to get the dll import/export to work without modifying the
# source to use cmake's method of setting the dll import/export definitions
if (WIN32)
    SET_TARGET_PROPERTIES(OIS PROPERTIES COMPILE_FLAGS "-DOIS_NONCLIENT_BUILD")
endif()

set_target_properties(OIS PROPERTIES SOLUTION_FOLDER "libOIS")

if (APPLE)
    target_link_libraries(OIS ${COCOA_LIBRARY})
    target_link_libraries(OIS ${IOKIT_LIBRARY})
    target_link_libraries(OIS ${FOUNDATION_LIBRARY})
endif()

if (WIN32)
    if (CMAKE_CL_64)
        set(DIRECTX_ARCH x64)
    else()
        set(DIRECTX_ARCH x86)
    endif()

    target_link_libraries(OIS "${DXSDK_DIR}/Lib/${DIRECTX_ARCH}/dinput8.lib" "${DXSDK_DIR}/Lib/${DIRECTX_ARCH}/dxguid.lib")
endif()

if(UNIX)
    if (NOT APPLE)
        add_dependencies(OIS X11)
        target_link_libraries(OIS X11)
    endif()

    set_target_properties(OIS PROPERTIES
        VERSION ${OIS_VERSION}
        SOVERSION ${OIS_SOVERSION}
    )
endif()

if (OIS_BUILD_DEMOS)
    project(OISConsoleApp)
    source_group("ConsoleApp" FILES "${CMAKE_CURRENT_SOURCE_DIR}/demos/OISConsoleDemo/OISConsole.cpp")
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/includes)
    if (WIN32)
        add_executable(ConsoleApp "${CMAKE_CURRENT_SOURCE_DIR}/demos/OISConsoleDemo/OISConsole.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/demos/OISConsoleDemo/CommandLine.rc")
    else()
        add_executable(ConsoleApp "${CMAKE_CURRENT_SOURCE_DIR}/demos/OISConsoleDemo/OISConsole.cpp")
    endif()
    set_target_properties(ConsoleApp PROPERTIES SOLUTION_FOLDER "OIS Console Demo")
    add_dependencies(ConsoleApp OIS)
    target_link_libraries(ConsoleApp "OIS")
    if (APPLE)
        target_link_libraries(ConsoleApp ${CARBON_LIBRARY})
    endif()

    project(OISFFConsoleApp)
    source_group("FFConsoleApp" FILES "${CMAKE_CURRENT_SOURCE_DIR}/demos/FFConsoleDemo/FFConsoleDemo.cpp")
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/includes)
    if (WIN32)
        add_executable(FFConsoleApp "${CMAKE_CURRENT_SOURCE_DIR}/demos/FFConsoleDemo/FFConsoleDemo.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/demos/FFConsoleDemo/FF.rc")
    else()
        add_executable(FFConsoleApp "${CMAKE_CURRENT_SOURCE_DIR}/demos/FFConsoleDemo/FFConsoleDemo.cpp")
    endif()
    set_target_properties(FFConsoleApp PROPERTIES SOLUTION_FOLDER "OIS FF Console Demo")
    add_dependencies(FFConsoleApp OIS)
    target_link_libraries(FFConsoleApp "OIS")
    if (APPLE)
        target_link_libraries(FFConsoleApp ${CARBON_LIBRARY})
    endif()

endif()

set(LIB_INSTALL_DIR "lib")
set(OIS_LIB_DIRECTORY ${CMAKE_INSTALL_PREFIX}/${LIB_INSTALL_DIR})

file(GLOB PUBLIC_HEADERS "${CMAKE_SOURCE_DIR}/includes/*.h")

install(FILES ${PUBLIC_HEADERS}
  DESTINATION include/ois
)

install(TARGETS OIS
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  FRAMEWORK DESTINATION Library/Frameworks
)

if(UNIX)
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/OIS.pc.in ${CMAKE_BINARY_DIR}/OIS.pc @ONLY)
    install(FILES ${CMAKE_BINARY_DIR}/OIS.pc DESTINATION ${OIS_LIB_DIRECTORY}/pkgconfig)
endif()
