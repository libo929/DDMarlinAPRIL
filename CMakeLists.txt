########################################################
# cmake file for building DDMarlinAPRIL
# @author Jan Engels, DESY
CMAKE_MINIMUM_REQUIRED(VERSION 2.8.2 FATAL_ERROR)
########################################################


# project name
PROJECT( DDMarlinAPRIL )


# project version# continuing from MarlinAPRIL version 2.0.0 revision 1934
SET( DDMarlinAPRIL_VERSION_MAJOR 0 )
SET( DDMarlinAPRIL_VERSION_MINOR 9 )
SET( DDMarlinAPRIL_VERSION_PATCH 1 )


### CMAKE ###################################################################

# load default settings from ILCSOFT_CMAKE_MODULES
FIND_PACKAGE( ILCUTIL COMPONENTS ILCSOFT_CMAKE_MODULES REQUIRED )

INCLUDE( ilcsoft_default_settings )


### SOURCE ##################################################################

#include directories
INCLUDE_DIRECTORIES( ./include )
INSTALL_DIRECTORY( ./include DESTINATION . FILES_MATCHING PATTERN "*.h" )

AUX_SOURCE_DIRECTORY( ./src/ DDMarlinAPRIL_SRCS )

### DEPENDENCIES ############################################################

FIND_PACKAGE( Marlin 1.0 REQUIRED )
FIND_PACKAGE( MarlinUtil 01.10 REQUIRED )
FIND_PACKAGE( PandoraSDK 03.00.00 REQUIRED )
#FIND_PACKAGE( LCContent 03.00.00 REQUIRED )
FIND_PACKAGE( APRILContent REQUIRED )
FIND_PACKAGE( DD4hep REQUIRED )
FIND_PACKAGE( ROOT REQUIRED )
FIND_PACKAGE( MarlinTrk REQUIRED )

IF (APRILContent_FOUND)
	LIST(APPEND CMAKE_MODULE_PATH ${APRILContent_ROOT}/cmake)
ELSE()
	MESSAGE(FATAL_ERROR "APRILContent is not found.")
ENDIF()

FIND_PACKAGE( mlpack REQUIRED )

FOREACH( pkg PandoraSDK Marlin MarlinUtil APRILContent DD4hep ROOT MarlinTrk mlpack)
    IF( ${pkg}_FOUND )
        INCLUDE_DIRECTORIES( SYSTEM ${${pkg}_INCLUDE_DIRS} )
        LINK_LIBRARIES( ${${pkg}_LIBRARIES} )
        ADD_DEFINITIONS ( ${${pkg}_DEFINITIONS} )
    ENDIF()
ENDFOREACH()

IF( PANDORA_MONITORING )
    FIND_PACKAGE( PandoraMonitoring 03.00.00 REQUIRED )
    IF( PandoraMonitoring_FOUND )
        INCLUDE_DIRECTORIES( SYSTEM ${PandoraMonitoring_INCLUDE_DIRS} )
        LINK_LIBRARIES( ${PandoraMonitoring_LIBRARIES} )
        ADD_DEFINITIONS ( ${PandoraMonitoring_DEFINITIONS} )
        ADD_DEFINITIONS( "-DMONITORING" )
    ENDIF()
ENDIF()

SET( CMAKE_CXX_FLAGS "-fopenmp ${CMAKE_CXX_FLAGS}" )

### DOCUMENTATION ###########################################################

OPTION( INSTALL_DOC "Set to OFF to skip build/install Documentation" OFF )

IF( INSTALL_DOC AND EXISTS "${PROJECT_SOURCE_DIR}/doc/CMakeLists.txt" )
    ADD_SUBDIRECTORY( ./doc )
ELSE()
    MESSAGE( STATUS "INSTALL_DOC set to OFF" )
    SET( INSTALL_DOC OFF )
ENDIF()


### LIBRARY #################################################################

# add library
ADD_SHARED_LIBRARY( DDMarlinAPRIL ${DDMarlinAPRIL_SRCS} )


### INSTALL #################################################################

# install library
INSTALL_SHARED_LIBRARY( DDMarlinAPRIL DESTINATION lib )

# install header files
INSTALL_DIRECTORY( ./include DESTINATION . )

# display some variables and write them to cache
DISPLAY_STD_VARIABLES()
