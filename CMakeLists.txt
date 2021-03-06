#----------------------------------------------------------------------------
# Setup the project
cmake_minimum_required(VERSION 2.6 FATAL_ERROR)
project(HepMCEx03)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules)

#----------------------------------------------------------------------------
# Find Geant4 package, activating all available UI and Vis drivers by default
# You can set WITH_GEANT4_UIVIS to OFF via the command line or ccmake/cmake-gui
# to build a batch mode only executable
#
option(WITH_GEANT4_UIVIS "Build example with Geant4 UI and Vis drivers" ON)
if(WITH_GEANT4_UIVIS)
  find_package(Geant4 REQUIRED ui_all vis_all)
else()
  find_package(Geant4 REQUIRED)
endif()

#----------------------------------------------------------------------------
# Setup Geant4 include directories and compile definitions
#
include(${Geant4_USE_FILE})

#----------------------------------------------------------------------------
# Find HepMC (required package)
#
find_package(HepMC REQUIRED)

#----------------------------------------------------------------------------
# Find ROOT (required package)
#
find_package(ROOT REQUIRED)
#----------------------------------------------------------------------------
# Find Pythia8 (required package)
#
find_package(Pythia8 REQUIRED)
if(PYTHIA8_FOUND)
  message(STATUS "G4 Examples: Pythia8 found. --> HepMCEx03 example with Pythia8 enabled.") 
  add_definitions(-DG4LIB_USE_PYTHIA8) 
else()
  message(STATUS "G4 Examples: Pythia8 -- NOT found.") 
  set(PYTHIA8_LIBRARY "")
  set(PYTHIA8_HEPMC_LIBRARY "")
  set(PYTHIA8_HAPDF_LIBRARY "")
endif()

#----------------------------------------------------------------------------
# Locate sources and headers for this project
#
include_directories(${PROJECT_SOURCE_DIR}/include 
                    ${Geant4_INCLUDE_DIR}
                    ${HEPMC_INCLUDE_DIR}
                    ${PYTHIA8_INCLUDE_DIR}
                    ${ROOT_INCLUDE_DIR})
file(GLOB sources ${PROJECT_SOURCE_DIR}/src/*.cc)
file(GLOB headers ${PROJECT_SOURCE_DIR}/include/*.hh)

#----------------------------------------------------------------------------
# Generate dictionary for input/output via HepMC ROOT files
#
ROOT_GENERATE_DICTIONARY(HepMC include/HepMCG4Classes.hh  LINKDEF include/HepMCG4LinkDef.h )
add_library(HepMCdict SHARED ${sources} HepMC.cxx)
target_link_libraries(HepMCdict ${HEPMC_LIBRARIES} ${ROOT_LIBRARIES})


#----------------------------------------------------------------------------
# Add the executable, and link it to the Geant4 libraries
#
add_executable(HepMCEx03 HepMCEx03.cc ${sources} ${headers})
target_link_libraries(HepMCEx03 ${Geant4_LIBRARIES}
                      ${HEPMC_LIBRARIES}
                       ${PYTHIA8_LIBRARY}
                       ${PYTHIA8_HEPMC_LIBRARY} ${PYTHIA8_HAPDF_LIBRARY}
                       HepMCdict ${ROOT_LIBRARIES})

#----------------------------------------------------------------------------
# Copy all scripts to the build directory, i.e. the directory in which we
# build HepMCEx03. This is so that we can run the executable directly because it
# relies on these scripts being in the current working directory.
#
set(HepMCEx03_SCRIPTS
    hepmc_ascii.in hepmc_root.in
    vis.mac hepmc_pythia8.in
  )

foreach(_script ${HepMCEx03_SCRIPTS})
  configure_file(
    ${PROJECT_SOURCE_DIR}/${_script}
    ${PROJECT_BINARY_DIR}/${_script}
    COPYONLY
    )
endforeach()

#----------------------------------------------------------------------------
# Install the executable to 'bin' directory under CMAKE_INSTALL_PREFIX
#
install(TARGETS HepMCEx03 DESTINATION bin)

