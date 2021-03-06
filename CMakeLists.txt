CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
PROJECT( ZYPPER C CXX )
SET( PACKAGE "zypper" )

# where to look first for cmake modules, before ${CMAKE_ROOT}/Modules/ is checked
SET( CMAKE_MODULE_PATH
  ${ZYPP_PREFIX}/share/cmake/Modules
  ${ZYPPER_SOURCE_DIR}/cmake/modules
  ${CMAKE_MODULE_PATH}
)
INCLUDE(ZyppCommon)

OPTION (ENABLE_BUILD_TRANS "Build translation files by default?" ON)

# VERSION
INCLUDE( ${ZYPPER_SOURCE_DIR}/VERSION.cmake )
SET( VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}" )
ADD_DEFINITIONS( -D_FILE_OFFSET_BITS=64 )
ADD_DEFINITIONS( -DVERSION="${VERSION}" )

SET( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O2 -Wall -Woverloaded-virtual -Wnon-virtual-dtor -fstack-protector -std=c++11 -fvisibility-inlines-hidden" )
SET( CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -g -O2 -Wall -fstack-protector" )

GENERATE_PACKAGING(${PACKAGE} ${VERSION})

FIND_PACKAGE( Zypp REQUIRED )
IF( ZYPP_FOUND )
  INCLUDE_DIRECTORIES(${ZYPP_INCLUDE_DIR})
ELSE( ZYPP_FOUND )
  MESSAGE( FATAL_ERROR "libzypp not found" )
ENDIF( ZYPP_FOUND )

FIND_PACKAGE( Gettext REQUIRED )
IF( GETTEXT_FOUND )
  INCLUDE_DIRECTORIES(${GETTEXT_INCLUDE_DIR})
ELSE( GETTEXT_FOUND )
  MESSAGE( FATAL_ERROR "gettext not found" )
ENDIF( GETTEXT_FOUND )

FIND_PACKAGE( Readline REQUIRED )
IF( READLINE_FOUND )
  INCLUDE_DIRECTORIES(${READLINE_INCLUDE_DIR})
ELSE( READLINE_FOUND )
  MESSAGE( FATAL_ERROR "readline not found" )
ENDIF( READLINE_FOUND )

FIND_PACKAGE( Augeas REQUIRED )
IF( AUGEAS_FOUND )
  INCLUDE_DIRECTORIES(${AUGEAS_INCLUDE_DIR})
  FIND_PACKAGE(LibXml2)
  IF (LIBXML2_FOUND)
    INCLUDE_DIRECTORIES(${LIBXML2_INCLUDE_DIR})
  ENDIF( LIBXML2_FOUND)
ELSE( AUGEAS_FOUND )
  MESSAGE( FATAL_ERROR "augeas not found" )
ENDIF( AUGEAS_FOUND )

MACRO(ADD_TESTS)
  FOREACH( loop_var ${ARGV} )
    SET_SOURCE_FILES_PROPERTIES( ${loop_var}_test.cc COMPILE_FLAGS "-DBOOST_TEST_DYN_LINK -DBOOST_TEST_MAIN -DBOOST_AUTO_TEST_MAIN=\"\" " )
    ADD_EXECUTABLE( ${loop_var}_test ${loop_var}_test.cc )
    TARGET_LINK_LIBRARIES( ${loop_var}_test ${ZYPP_LIBRARY} boost_unit_test_framework zypper_lib zypper_test_utils)
    ADD_TEST( ${loop_var}_test ${CMAKE_CURRENT_BINARY_DIR}/${loop_var}_test --catch_system_errors=no)
  ENDFOREACH( loop_var )
ENDMACRO(ADD_TESTS)

ADD_SUBDIRECTORY( src )
ADD_SUBDIRECTORY( doc )
ADD_SUBDIRECTORY( tests EXCLUDE_FROM_ALL )

IF ( ENABLE_BUILD_TRANS )
  ADD_SUBDIRECTORY( po )
ELSE ( ENABLE_BUILD_TRANS )
  ADD_SUBDIRECTORY( po EXCLUDE_FROM_ALL )
ENDIF ( ENABLE_BUILD_TRANS )

INCLUDE(CTest)
ENABLE_TESTING()


# documentation too important to be hidden in doc:
INSTALL(
  FILES COPYING HACKING
  DESTINATION ${INSTALL_PREFIX}/share/doc/packages/${PACKAGE}
)

# system-wide zypper.conf
INSTALL(
  FILES zypper.conf
  DESTINATION ${SYSCONFDIR}/zypp
)

# zypper-log tool
INSTALL(
  PROGRAMS tools/zypper-log
  DESTINATION ${INSTALL_PREFIX}/sbin
)

# zypper-aptitude compat tool
INSTALL(
  PROGRAMS tools/aptitude tools/apt-get tools/apt
  DESTINATION ${INSTALL_PREFIX}/bin
)
INSTALL(
  DIRECTORY tools/apt-packagemap.d/
  DESTINATION ${SYSCONFDIR}/zypp/apt-packagemap.d
)
