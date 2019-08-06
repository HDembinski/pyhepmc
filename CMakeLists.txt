project(pyhepmc_ng CXX)
#Should be fixed 
cmake_minimum_required(VERSION 2.8.12)
set(CMAKE_VERBOSE_MAKEFILE ON)
find_package (Python COMPONENTS  Development Interpreter)
option(USE_INSTALLED_HEPMC3 "Use pyhepmc as a standalone package" ON)


#This is needed for standalone compilation
if (USE_INSTALLED_HEPMC3)
#take project vestion from setup.py
file(STRINGS setup.py  pyhepmc_version_defines REGEX "__version__")
string(REPLACE "'" ";" pyhepmc_version_split ${pyhepmc_version_defines})
list(GET pyhepmc_version_split 1 pyhepmc_version)
set(${PROJECT_NAME}_VERSION ${pyhepmc_version})
set(THIS_PROJECT_NAME ${PROJECT_NAME})
set(THIS_PROJECT_NAME_VERSION ${pyhepmc_version})
#
find_package(HepMC3 REQUIRED)
include_directories(${HEPMC3_INCLUDE_DIR})
include(GNUInstallDirs)
if (HEPMC3_ROOTIO_LIB)
  set (ROOT_FIND_COMPONENTS Core RIO Tree)
  list(APPEND CMAKE_PREFIX_PATH $ENV{ROOTSYS})
  list(APPEND CMAKE_PREFIX_PATH ${ROOT_DIR})
  find_package(ROOT REQUIRED COMPONENTS Core RIO Tree)
  message(STATUS "ROOT_USE_FILE ${ROOT_USE_FILE}")
  include(${ROOT_USE_FILE})

  message(STATUS "ROOT includes: ${ROOT_INCLUDE_DIRS}")
  message(STATUS "ROOT libraries: ${ROOT_LIBRARIES}")
  message(STATUS "ROOT_FOUND: ${ROOT_FOUND}")

  if(ROOT_FOUND)
    include_directories(${ROOT_INCLUDE_DIRS})
    add_definitions(-DHEPMC3_ROOTIO)
    set(HEPMC3_ROOTIO_LIBRARY ${HEPMC3_ROOTIO_LIB} )
  endif()
endif()

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("/Zc:__cplusplus" COMPILER_SUPPORTS_ZCPLUSPLUS)
if(COMPILER_SUPPORTS_ZCPLUSPLUS)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Zc:__cplusplus")
endif()

CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
else()
message(FATAL "C++11 support is required") 
endif()
else()
set(THIS_PROJECT_NAME pyhepmc)
set(${THIS_PROJECT_NAME_VERSION} ${PROJECT_VERSION})
endif()
#<-This is needed for standalone compilation 
message(STATUS "pyhepmc v${THIS_PROJECT_NAME_VERSION}")

#Taken from CmakelIsts in pybind project
set(PYBIND11_HEADERS
  include/pybind11/detail/class.h
  include/pybind11/detail/common.h
  include/pybind11/detail/descr.h
  include/pybind11/detail/init.h
  include/pybind11/detail/internals.h
  include/pybind11/detail/typeid.h
  include/pybind11/attr.h
  include/pybind11/buffer_info.h
  include/pybind11/cast.h
  include/pybind11/chrono.h
  include/pybind11/common.h
  include/pybind11/complex.h
  include/pybind11/options.h
  include/pybind11/eigen.h
  include/pybind11/embed.h
  include/pybind11/eval.h
  include/pybind11/functional.h
  include/pybind11/numpy.h
  include/pybind11/operators.h
  include/pybind11/pybind11.h
  include/pybind11/pytypes.h
  include/pybind11/stl.h
  include/pybind11/stl_bind.h
)

Python_add_library(${THIS_PROJECT_NAME} MODULE ${PYBIND11_HEADERS} src/main.cpp src/hepevt_wrapper.h)
target_include_directories(${THIS_PROJECT_NAME} PUBLIC ${Python_INCLUDE_DIRS} )
target_compile_definitions(${THIS_PROJECT_NAME} PRIVATE "-DVERSION_INFO=\"${THIS_PROJECT_NAME_VERSION}\"" )
SET_TARGET_PROPERTIES(${THIS_PROJECT_NAME} PROPERTIES OUTPUT_NAME  cpp)
#set(lib_type MODULE)
#
target_link_libraries(${THIS_PROJECT_NAME} PUBLIC ${Python_LIBRARIES}  HepMC3  HepMC3search)

install(TARGETS ${THIS_PROJECT_NAME} DESTINATION ${Python_SITEARCH}/${THIS_PROJECT_NAME})
install(FILES pyhepmc_ng/__init__.py   pyhepmc_ng/view.py DESTINATION  ${Python_SITEARCH}/${THIS_PROJECT_NAME})
