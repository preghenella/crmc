CMAKE_MINIMUM_REQUIRED (VERSION 2.6)
PROJECT (crmc)

######################################ONLY EDIT THIS######################################

# Enable/Disable models to be built
OPTION (__CRMCSTATIC__ "Build with static library" OFF)   #if ON should not combined DPMJET/PHOJET/PYTHIA because they use different version of pythia (for dynamic library no problem)

OPTION (__QGSJET01__   "Build with model" OFF)
OPTION (__GHEISHA__    "Build with model" OFF)
OPTION (__PYTHIA__     "Build with model" OFF)
OPTION (__HIJING__     "Build with model" OFF)
OPTION (__SIBYLL__     "Build with model" OFF)
OPTION (__PHOJET__     "Build with model" OFF)
OPTION (__DPMJET__     "Build with model" OFF)
OPTION (__QGSJETII03__ "Build with model" OFF)
OPTION (__QGSJETII04__ "Build with model" OFF)
######################################ONLY EDIT THIS######################################

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set (CMAKE_INSTALL_PREFIX $ENV{PWD} CACHE PATH "default install path" FORCE )
endif()

SET(PROJECT_BINARY_DIR ${CMAKE_INSTALL_PREFIX})

MESSAGE(STATUS "source ${PROJECT_SOURCE_DIR}")
MESSAGE(STATUS "binary ${PROJECT_BINARY_DIR}")
MESSAGE(STATUS "build ${CMAKE_BINARY_DIR}")

## The version number. Only nummeric PLEASE!
SET (CRMC_VERSION_MAJOR 1)
SET (CRMC_VERSION_MINOR 5)
SET (CRMC_VERSION_PATCH 3)

##Compiler options
# hotfix for lxbatch not sure why he selects wrong compiler
#SET (CMAKE_Fortran_COMPILER gfortran)
#SET (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -lgfortran")

# comprehensible compiler languages (default c cxx)
ENABLE_LANGUAGE (Fortran)

#add some warning to the compiler flags
FOREACH(lang C CXX Fortran)
   IF(CMAKE_${lang}_COMPILER_ID STREQUAL GNU)
     SET (CMAKE_${lang}_FLAGS "${CMAKE_${lang}_FLAGS} -Wunused")
     SET (CMAKE_${lang}_FLAGS "${CMAKE_${lang}_FLAGS} -O -Wuninitialized")
     #SET (CMAKE_${lang}_FLAGS "${CMAKE_${lang}_FLAGS} -Wall")
   ENDIF()
ENDFOREACH()

## RPATH (where to find library)
# use, i.e. don't skip the full RPATH for the build tree
SET(CMAKE_SKIP_BUILD_RPATH  FALSE)

# when building, don't use the install RPATH already
# (but later on when installing)
SET(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)

SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

# the RPATH to be used when installing, but only if it's not a system directory
LIST(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_INSTALL_PREFIX}/lib" isSystemDir)
IF("${isSystemDir}" STREQUAL "-1")
  SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
ENDIF("${isSystemDir}" STREQUAL "-1")

## find packages
SET (CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/CMakeModules" CACHE PATH "Module Path" FORCE)

SET (CRMC_SOURCES src/crmcMain.cc src/CRMC.cc src/CRMCinterface.cc src/CRMCoptions.cc src/OutputPolicyLHE.cc src/OutputPolicyNone.cc src/CRMCtrapfpe.c)


FIND_PACKAGE (Root)
if (Root_FOUND) 
  INCLUDE_DIRECTORIES ("${ROOT_INCLUDE_DIR}")
  LIST(APPEND CRMC_SOURCES src/OutputPolicyROOT.cc)
  add_definitions(-DWITH_ROOT)
  MESSAGE("Build ROOT Output Interface")
else (Root_FOUND)
  MESSAGE("Cannot Build ROOT Output Interface")
endif (Root_FOUND)

# SET(Boost_DEBUG TRUE)
FIND_PACKAGE (Boost 1.35 REQUIRED COMPONENTS filesystem iostreams system program_options)

FIND_PACKAGE (HepMC REQUIRED COMPONENTS HepMC HepMCfio)
INCLUDE_DIRECTORIES ("${HepMC_INCLUDE_DIRS}")
LIST(APPEND CRMC_SOURCES src/OutputPolicyHepMC.cc)
add_definitions(-DWITH_HEPMC)
MESSAGE("Build HEPMC Output Interface")

FIND_PACKAGE (Fastjet)
if (FASTJET_FOUND)
  include_directories("${Fastjet_INCLUDE_DIRS}")
  add_definitions(-DWITH_FASTJET)
  MESSAGE("Include fastjet filter")
endif (FASTJET_FOUND)


# prepare crmc to use same size HEPMC common block than HepMC was compiled with
STRING ( STRIP ${HepMC_HEPEVT_SIZE} Replace_String )
FILE(READ ${PROJECT_SOURCE_DIR}/src/epos.inc-HEPMC-TEMPLATE template)
STRING( REGEX REPLACE "HEPEVT_SIZE_REPLACE" "${Replace_String}" template2 "${template}")
FILE(WRITE ${PROJECT_SOURCE_DIR}/src/epos.inc "${template2}")

FILE(READ ${PROJECT_SOURCE_DIR}/src/CRMCinterface.h-HEPMC-TEMPLATE template3)
STRING( REGEX REPLACE "HEPEVT_SIZE_REPLACE" "${Replace_String}" template4 "${template3}")
FILE(WRITE ${PROJECT_SOURCE_DIR}/src/CRMCinterface.h "${template4}")

FILE(READ ${PROJECT_SOURCE_DIR}/src/pythia/pythia6115.f-HEPMC-TEMPLATE template5)
STRING( REGEX REPLACE "HEPEVT_SIZE_REPLACE" "${Replace_String}" template6 "${template5}")
FILE(WRITE ${PROJECT_SOURCE_DIR}/src/pythia/pythia6115.f "${template6}")

FILE(READ ${PROJECT_SOURCE_DIR}/src/pythia/pythia6215.f-HEPMC-TEMPLATE template7)
STRING( REGEX REPLACE "HEPEVT_SIZE_REPLACE" "${Replace_String}" template8 "${template7}")
FILE(WRITE ${PROJECT_SOURCE_DIR}/src/pythia/pythia6215.f "${template8}")

FILE(READ ${PROJECT_SOURCE_DIR}/src/pythia/pythia-6.4.28.f-HEPMC-TEMPLATE template9)
STRING( REGEX REPLACE "HEPEVT_SIZE_REPLACE" "${Replace_String}" template10 "${template9}")
FILE(WRITE ${PROJECT_SOURCE_DIR}/src/pythia/pythia-6.4.28.f "${template10}")

FILE(READ ${PROJECT_SOURCE_DIR}/src/hijing/hipyset1.35.f-HEPMC-TEMPLATE template11)
STRING( REGEX REPLACE "HEPEVT_SIZE_REPLACE" "${Replace_String}" template12 "${template11}")
FILE(WRITE ${PROJECT_SOURCE_DIR}/src/hijing/hipyset1.35.f "${template12}")

FILE(READ ${PROJECT_SOURCE_DIR}/src/dpmjet/3.0-6/pythia6115dpm3v1.f-HEPMCTEMPLATE template13)
STRING( REGEX REPLACE "HEPEVT_SIZE_REPLACE" "${Replace_String}" template14 "${template13}")
FILE(WRITE ${PROJECT_SOURCE_DIR}/src/dpmjet/3.0-6/pythia6115dpm3v1.f "${template14}")




## configure a header file to pass some of the CMake settings to the source code
CONFIGURE_FILE (
  "${PROJECT_SOURCE_DIR}/src/CRMCconfig.h.in"
  "${PROJECT_BINARY_DIR}/src/CRMCconfig.h"
  )

INCLUDE_DIRECTORIES ("${PROJECT_SOURCE_DIR}/src")

#add epos (must!)
ADD_SUBDIRECTORY (src/epos)

SET (TABS "")

# data tables for epos
LIST(APPEND TABS epos.inics)
LIST(APPEND TABS epos.inics.lhc)
LIST(APPEND TABS epos.inidi)
LIST(APPEND TABS epos.iniev)
LIST(APPEND TABS epos.inirj)
LIST(APPEND TABS epos.inirj.lhc)
LIST(APPEND TABS epos.initl)

IF (__QGSJET01__)
ADD_SUBDIRECTORY ("${PROJECT_SOURCE_DIR}/src/qgsjet")
LIST(APPEND TABS qgsjet.dat)
LIST(APPEND TABS qgsjet.ncs)
SET(STATIC_LIBS ${STATIC_LIBS} Qgsjet01)
get_property(HELPER SOURCE src/models.F PROPERTY COMPILE_FLAGS)
set_property(SOURCE src/models.F PROPERTY COMPILE_FLAGS "${HELPER} -D __QGSJET01__") #set properties APPEND has a bug
ENDIF (__QGSJET01__)

IF (__GHEISHA__)
ADD_SUBDIRECTORY ("${PROJECT_SOURCE_DIR}/src/gheisha")
SET(STATIC_LIBS ${STATIC_LIBS} Gheisha)
get_property(HELPER SOURCE src/models.F PROPERTY COMPILE_FLAGS)
set_property(SOURCE src/models.F PROPERTY COMPILE_FLAGS "${HELPER} -D __GHEISHA__")
ENDIF (__GHEISHA__)

IF (__PYTHIA__)
ADD_SUBDIRECTORY ("${PROJECT_SOURCE_DIR}/src/pythia")
SET(STATIC_LIBS ${STATIC_LIBS} Pythia)
get_property(HELPER SOURCE src/models.F PROPERTY COMPILE_FLAGS)
set_property(SOURCE src/models.F PROPERTY COMPILE_FLAGS "${HELPER} -D __PYTHIA__")
ENDIF (__PYTHIA__)

IF (__HIJING__)
ADD_SUBDIRECTORY ("${PROJECT_SOURCE_DIR}/src/hijing")
SET(STATIC_LIBS ${STATIC_LIBS} Hijing)
get_property(HELPER SOURCE src/models.F PROPERTY COMPILE_FLAGS)
set_property(SOURCE src/models.F PROPERTY COMPILE_FLAGS "${HELPER} -D __HIJING__")
ENDIF (__HIJING__)

IF (__SIBYLL__)
ADD_SUBDIRECTORY ("${PROJECT_SOURCE_DIR}/src/sibyll")
SET(STATIC_LIBS ${STATIC_LIBS} Sibyll)
get_property(HELPER SOURCE src/models.F PROPERTY COMPILE_FLAGS)
set_property(SOURCE src/models.F PROPERTY COMPILE_FLAGS "${HELPER} -D __SIBYLL__")
ENDIF (__SIBYLL__)

IF (__PHOJET__)
ADD_SUBDIRECTORY ("${PROJECT_SOURCE_DIR}/src/phojet")
SET(STATIC_LIBS ${STATIC_LIBS} Phojet)
get_property(HELPER SOURCE src/models.F PROPERTY COMPILE_FLAGS)
set_property(SOURCE src/models.F PROPERTY COMPILE_FLAGS "${HELPER} -D __PHOJET__")
ENDIF (__PHOJET__)

IF (__DPMJET__)
LIST(APPEND TABS dpmjet.dat)
LIST(APPEND TABS phojet_fitpar.dat)
ADD_SUBDIRECTORY ("${PROJECT_SOURCE_DIR}/src/dpmjet/3.0-6")
SET(STATIC_LIBS ${STATIC_LIBS} Dpmjet)
get_property(HELPER SOURCE src/models.F PROPERTY COMPILE_FLAGS)
set_property(SOURCE src/models.F PROPERTY COMPILE_FLAGS "${HELPER} -D __DPMJET__")
ENDIF (__DPMJET__)

#put definition outside "if" because it is always is crmc.param file
SET (QGSJETII_TABS "03")
IF (__QGSJETII03__)
ADD_SUBDIRECTORY ("${PROJECT_SOURCE_DIR}/src/qgsjetII")
SET(STATIC_LIBS ${STATIC_LIBS} QgsjetII03)
get_property(HELPER SOURCE src/models.F PROPERTY COMPILE_FLAGS)
set_property(SOURCE src/models.F PROPERTY COMPILE_FLAGS "${HELPER} -D __QGSJETII03__")
LIST(APPEND TABS qgsdat-II-03.lzma)
LIST(APPEND TABS sectnu-II-03)
ENDIF (__QGSJETII03__)

IF (__QGSJETII04__)
ADD_SUBDIRECTORY ("${PROJECT_SOURCE_DIR}/src/qgsjetII-04")
SET(STATIC_LIBS ${STATIC_LIBS} QgsjetII04)
get_property(HELPER SOURCE src/models.F PROPERTY COMPILE_FLAGS)
set_property(SOURCE src/models.F PROPERTY COMPILE_FLAGS "${HELPER} -D __QGSJETII04__")
SET (QGSJETII_TABS "04")
LIST(APPEND TABS qgsdat-II-04.lzma)
LIST(APPEND TABS sectnu-II-04)
ENDIF (__QGSJETII04__)

IF (__CRMCSTATIC__)
#has to be after all other models since variables must be set for fortran precompiler
add_library(CrmcFortranStatic STATIC src/crmc-aaa.f src/models.F)
INSTALL (TARGETS CrmcFortranStatic
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib/static
        )
TARGET_LINK_LIBRARIES (CrmcFortranStatic ${STATIC_LIBS})
ELSE (__CRMCSTATIC__)
#Dummy library which includes epos.inc file so HEPMC will find the symbol for hepevt_
#with fPIC linker option it should always point to right symbol also for other libs
add_library(HepEvtDummy SHARED src/dummy.f)
INSTALL (TARGETS HepEvtDummy
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib/static
        )
ENDIF (__CRMCSTATIC__)



#include directories for headers
INCLUDE_DIRECTORIES ("${PROJECT_BINARY_DIR}/src")
INCLUDE_DIRECTORIES ("${Boost_INCLUDE_DIRS}")
# the executable
ADD_EXECUTABLE(crmc ${CRMC_SOURCES})

# linking of modules
TARGET_LINK_LIBRARIES (crmc ${CMAKE_DL_LIBS})
TARGET_LINK_LIBRARIES (crmc ${HepMC_LIBRARIES})
TARGET_LINK_LIBRARIES (crmc ${Boost_LIBRARIES})
if (Root_FOUND)
  TARGET_LINK_LIBRARIES (crmc ${ROOT_LIBRARIES})
endif (Root_FOUND)
IF (__CRMCSTATIC__)
  TARGET_LINK_LIBRARIES (crmc CrmcBasic)
  TARGET_LINK_LIBRARIES (crmc CrmcFortranStatic)
ELSE (__CRMCSTATIC__)
  TARGET_LINK_LIBRARIES (crmc HepEvtDummy)
ENDIF (__CRMCSTATIC__)

## installation
# configure the parameter file
install(CODE "SET (CRMCROOT \"${PROJECT_BINARY_DIR}\")")
install(CODE "SET (QIITAB \"${QGSJETII_TABS}\")")
install(CODE "
             IF(NOT EXISTS ${PROJECT_BINARY_DIR}/crmc.param)
             MESSAGE (STATUS \"(Re)creating crmc.param\")
             CONFIGURE_FILE (\"${PROJECT_SOURCE_DIR}/src/crmc.param.in\" \"${PROJECT_BINARY_DIR}/crmc.param\" @ONLY IMMEDIATE)
             ENDIF(NOT EXISTS ${PROJECT_BINARY_DIR}/crmc.param)
             ")

# install tabs
SET (TABSDIR "")
foreach(theTab ${TABS})
  list(APPEND TABSDIR ${PROJECT_SOURCE_DIR}/tabs/${theTab} )
endforeach()
INSTALL (FILES ${TABSDIR} DESTINATION ${PROJECT_BINARY_DIR}/tabs)
#    install(CODE "EXECUTE_PROCESS(COMMAND ${CMAKE_COMMAND} -E copy_directory ${PROJECT_SOURCE_DIR}/tabs ${PROJECT_BINARY_DIR}/tabs)")

# install executable
INSTALL (PROGRAMS ${CMAKE_BINARY_DIR}/crmc DESTINATION bin)

# enable dashboard scripting
include (CTest)
ADD_TEST (crmcRuns bin/crmc -o hepmc -f ctest.hepmc -T 1)
#ADD_TEST (crmcCheckFile md5sum "ctest.hepmc")
#SET_TESTS_PROPERTIES (crmcCheckFile PROPERTIES PASS_REGULAR_EXPRESSION ".*777cb7e60b7c8104393a4fb22b137538  ctest.hepmc")
ADD_TEST (crmcRemoveFile rm "ctest.hepmc")

