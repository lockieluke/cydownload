cmake_minimum_required(VERSION 3.24)

project(cydownload LANGUAGES CXX)

if (CMAKE_VERSION VERSION_LESS "3.7.0")
    set(CMAKE_INCLUDE_CURRENT_DIR ON)
endif ()

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_OSX_DEPLOYMENT_TARGET 12.3)

# Perform build type specific actions
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0 -g")
else ()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
    set(BUILD_SHARED_LIBS OFF)
endif ()

set(CMAKE_MACOSX_RPATH TRUE)
set(CMAKE_INSTALL_RPATH "@executable_path/../Frameworks")
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

find_package(Qt6 COMPONENTS Core Gui Network Widgets DBus REQUIRED)
qt_standard_project_setup()

find_package(ZLIB REQUIRED)
find_package(BZip2 REQUIRED)

qt_add_executable(${PROJECT_NAME} MACOSX_BUNDLE
        src/main.cpp
        src/mainwindow.cpp
        src/dialogrepo.cpp
        src/helpers.cpp
        src/downloader.cpp
        src/logger.cpp
        src/repodialog.cpp
)

target_include_directories(${PROJECT_NAME} PRIVATE ${ZLIB_INCLUDE_DIRS} ${BZip2_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} PRIVATE Qt6::Core Qt6::Gui Qt6::Network Qt6::Widgets Qt6::DBus BZip2::BZip2 ZLIB::ZLIB)

set_target_properties(${PROJECT_NAME} PROPERTIES
        MACOSX_BUNDLE_GUI_IDENTIFIER "me.lockie.${PROJECT_NAME}"
        MACOSX_BUNDLE_BUNDLE_NAME ${PROJECT_NAME}
        MACOSX_BUNDLE_BUNDLE_VERSION "0.1"
        MACOSX_BUNDLE_SHORT_VERSION_STRING "0.1"
        MACOSX_BUNDLE_LONG_VERSION_STRING "0.1"
        INSTALL_RPATH "@executable_path/../Frameworks"
        BUILD_WITH_INSTALL_RPATH TRUE
        MACOSX_RPATH TRUE
#        XCODE_ATTRIBUTE_DEPLOYMENT_POSTPROCESSING YES
#        XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY "Mac Developer"
#        XCODE_ATTRIBUTE_CODE_SIGN_STYLE "Automatic"
#        XCODE_ATTRIBUTE_DEVELOPMENT_TEAM ${DEVELOPMENT_TEAM_ID}
        XCODE_ATTRIBUTE_LD_RUNPATH_SEARCH_PATHS "@executable_path/../Frameworks"
        XCODE_ATTRIBUTE_FRAMEWORK_SEARCH_PATHS "@executable_path/../Frameworks"
)

# Bundle dependencies
if(CMAKE_GENERATOR MATCHES "Xcode")
    set(APP_PATH "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_BUILD_TYPE}/cydownload.app" CACHE PATH "Application bundle directory" FORCE)
else()
    set(APP_PATH "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_CFG_INTDIR}/${PROJECT_NAME}.app")
endif()

install(TARGETS ${PROJECT_NAME}
        BUNDLE DESTINATION . COMPONENT Runtime
        RUNTIME DESTINATION bin COMPONENT Runtime
)

message(STATUS "APP_PATH: ${APP_PATH}")
install(
        CODE "include(BundleUtilities)
        include(InstallRequiredSystemLibraries)
        message(STATUS \"APP_PATH: ${APP_PATH}\")
        fixup_bundle(${APP_PATH} \"\" \"\")

        # Sign the frameworks
        file(GLOB_RECURSE FRAMEWORKS \"${APP_PATH}/Contents/Frameworks/*\")
        foreach(FRAMEWORK \${FRAMEWORKS})
            execute_process(COMMAND codesign --deep --force --sign \"${DEVELOPMENT_TEAM_ID}\" \"\${FRAMEWORK}\")
        endforeach()

        execute_process(COMMAND codesign --deep --force --sign \"${DEVELOPMENT_TEAM_ID}\" ${APP_PATH})
"
)
set(CPACK_GENERATOR "Bundle")
set(CPACK_BUNDLE_NAME ${PROJECT_NAME})
set(CPACK_BUNDLE_STARTUP_COMMAND "${APP_PATH}/Contents/MacOS/${PROJECT_NAME}")
include(CPack)