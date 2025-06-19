PROJECT(qucstrans CXX C)
CMAKE_MINIMUM_REQUIRED(VERSION 3.10)
cmake_policy(VERSION 3.10)

SET(QUCS_NAME "qucs-s")

# use top VERSION file
file (STRINGS ${PROJECT_SOURCE_DIR}/../VERSION QUCS_VERSION)

if(DEFINED CI_VERSION)
    set(PROJECT_VERSION "${CI_VERSION}")
else()
    set(PROJECT_VERSION "${QUCS_VERSION}")
endif()

message(STATUS "Configuring ${PROJECT_NAME} (GUI): VERSION ${PROJECT_VERSION}")

set(PROJECT_VENDOR "Qucs team. This program is licensed under the GNU GPL")
set(PROJECT_COPYRIGHT_YEAR "2014")
set(PROJECT_DOMAIN_FIRST "qucs")
set(PROJECT_DOMAIN_SECOND "org")


add_compile_definitions(HAVE_CONFIG_H)
SET(CMAKE_POSITION_INDEPENDENT_CODE ON)

# configure the header config.h
CONFIGURE_FILE (
    "${PROJECT_SOURCE_DIR}/../config.h.cmake"
    "${PROJECT_BINARY_DIR}/config.h"
)

INCLUDE_DIRECTORIES("${PROJECT_BINARY_DIR}")

find_package(Qt6 REQUIRED COMPONENTS Core Gui Widgets )
include_directories(
      ${Qt6Core_INCLUDE_DIRS}
      ${Qt6Gui_INCLUDE_DIRS}
      ${Qt6Widgets_INCLUDE_DIRS}
      )


set(QT_VERSION ${Qt6Core_VERSION})

if (${QT_VERSION} VERSION_LESS "6.7.0")
    set(CMAKE_CXX_STANDARD 17)
else()
    set(CMAKE_CXX_STANDARD 20)
endif()
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    if (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
        add_compile_definitions(_CRT_SECURE_NO_WARNINGS _SCL_SECURE_NO_WARNINGS)
        add_compile_options(/permissive- /Zc:__cplusplus /Zc:preprocessor /MP /Od /vmg)
        add_compile_options(/wd4244 /wd4267 /wd4312)
    else()
        add_compile_options(-Wall -Wextra -O0 -g)
        if (CMAKE_CXX_COMPILER_ID MATCHES "^AppleClang$|^Clang$")
            add_compile_options(-Wno-ignored-attributes)
        endif()
    endif()
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
    if (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
        add_compile_definitions(_CRT_SECURE_NO_WARNINGS _SCL_SECURE_NO_WARNINGS)
        string(REGEX REPLACE "/W1" "/w" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
        add_compile_options(/permissive- /Zc:__cplusplus /Zc:preprocessor /MP /vmg)
        add_link_options(/OPT:REF /OPT:ICF)
    else()
        add_compile_options(-w)
    endif()
endif()
ADD_DEFINITIONS(${QT_DEFINITIONS})

SET(QUCSTRANS_SRCS
  helpdialog.cpp
  main.cpp
  optionsdialog.cpp
  qucstrans.cpp
)

SET(QUCSTRANS_HDRS
  c_microstrip.h
  coax.h
  coplanar.h
  microstrip.h
  rectwaveguide.h
  transline.h
  stripline.h
  units.h
)

SET(QUCSTRANS_MOC_HDRS
  helpdialog.h
  optionsdialog.h
  qucstrans.h
)


SET( LIB_SRC
  c_microstrip.cpp
  coax.cpp
  coplanar.cpp
  microstrip.cpp
  rectwaveguide.cpp
  transline.cpp
  stripline.cpp
)

SET(RESOURCES qucstrans_.qrc)

QT6_WRAP_CPP( QUCSTRANS_MOC_SRCS ${QUCSTRANS_MOC_HDRS} )
QT6_ADD_RESOURCES(RESOURCES_SRCS ${RESOURCES})

ADD_LIBRARY(transcalc STATIC ${LIB_SRC} )

IF(APPLE)
  # set information on Info.plist file
  SET(MACOSX_BUNDLE_INFO_STRING "${PROJECT_NAME} ${PROJECT_VERSION}")
  SET(MACOSX_BUNDLE_BUNDLE_VERSION "${PROJECT_NAME} ${PROJECT_VERSION}")
  SET(MACOSX_BUNDLE_LONG_VERSION_STRING "${PROJECT_NAME} ${PROJECT_VERSION}")
  SET(MACOSX_BUNDLE_SHORT_VERSION_STRING "${PROJECT_VERSION}")
  SET(MACOSX_BUNDLE_COPYRIGHT "${PROJECT_COPYRIGHT_YEAR} ${PROJECT_VENDOR}")
  SET(MACOSX_BUNDLE_GUI_IDENTIFIER "${PROJECT_DOMAIN_SECOND}.${PROJECT_DOMAIN_FIRST}")
  SET(MACOSX_BUNDLE_BUNDLE_NAME "${PROJECT_NAME}")
  SET(MACOSX_BUNDLE_ICON_FILE qucstrans.icns)

  # set where in the bundle to put the icns file
  SET_SOURCE_FILES_PROPERTIES(${CMAKE_CURRENT_SOURCE_DIR}/../qucs/bitmaps/qucstrans.icns PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
  # include the icns file in the target
  SET(QUCSTRANS_SRCS ${QUCSTRANS_SRCS} ${CMAKE_CURRENT_SOURCE_DIR}/../qucs/bitmaps/qucstrans.icns)

ENDIF(APPLE)


ADD_EXECUTABLE(${QUCS_NAME}trans MACOSX_BUNDLE WIN32
  ${QUCSTRANS_SRCS}
  ${QUCSTRANS_HDRS}
  ${QUCSTRANS_MOC_SRCS}
  ${RESOURCES_SRCS} )

TARGET_LINK_LIBRARIES( ${QUCS_NAME}trans Qt6::Core Qt6::Gui Qt6::Widgets transcalc )
SET_TARGET_PROPERTIES(${QUCS_NAME}trans PROPERTIES POSITION_INDEPENDENT_CODE TRUE)

ADD_SUBDIRECTORY( examples )

INSTALL(TARGETS ${QUCS_NAME}trans
    BUNDLE DESTINATION ${DEPLOY_BIN_DIR} COMPONENT Runtime
    RUNTIME DESTINATION ${DEPLOY_BIN_DIR} COMPONENT Runtime
    )

if(WIN32 AND APPLE)
    set(deploy_tool_options_arg "")
    if(WIN32)
        set(deploy_tool_options_arg --svg --no-translations --no-opengl-sw --no-system-d3d-compiler --no-network --no-compiler-runtime)
    endif(WIN32)
    
    qt_generate_deploy_app_script(
        TARGET ${QUCS_NAME}trans
        OUTPUT_SCRIPT qucs_trans_deploy_script
        NO_UNSUPPORTED_PLATFORM_ERROR
        DEPLOY_TOOL_OPTIONS ${deploy_tool_options_arg}
    )
    install(SCRIPT ${qucs_trans_deploy_script})
endif()
