### CMakeList for MC Booster

cmake_minimum_required(VERSION 2.8)

# project name
project(MCBooster)

# The version number.
set (MCBooster_VERSION_MAJOR 1)
set (MCBooster_VERSION_MINOR 0)
set (MCBooster_VERSION_PATCH 0)

# warn user if system is not UNIX
if(NOT UNIX)
  message(FATAL_ERROR "This is an unsupported system.")
endif()

# configure a header file to pass some of the CMake settings
# to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/mcbooster/MCBooster.h.in"
  "${PROJECT_SOURCE_DIR}/mcbooster/MCBooster.h"
 )
mark_as_advanced(MCBooster_VERSION_MAJOR MCBooster_VERSION_MINOR MCBooster_VERSION_PATCH)

SET(MCBooster_CMAKE_DIR "${PROJECT_SOURCE_DIR}/cmake")
SET(CMAKE_MODULE_PATH "${MCBooster_CMAKE_DIR}" ${CMAKE_MODULE_PATH})
SET(CMAKE_VERBOSE_MAKEFILE  ON)

# get TCLAP
find_package(TCLAP REQUIRED)
include_directories(${TCLAP_INCLUDE_DIR})

# get THRUST
find_package(Thrust 1.8 REQUIRED)
include_directories(${THRUST_INCLUDE_DIR})

# get ROOT
find_package(ROOT REQUIRED)
include_directories(${ROOT_INCLUDE_DIR})
link_directories(${ROOT_LIBRARY_DIR})

#cuda
find_package(CUDA 6.5) 

#generate API documentation with Doxygen
find_package(Doxygen)
option(BUILD_DOCUMENTATION "Create and install the HTML based API documentation (requires Doxygen)" NO)

#options to gcc
set(CMAKE_CXX_FLAGS " -march=native -fPIC -O3 -fopenmp -lgomp" CACHE STRING "compile flags" FORCE)

#including directories

include_directories(include; ${PROJECT_SOURCE_DIR})

if(CUDA_FOUND)

	#set cuda flags
	
	SET(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS}; --std=c++11 ; --cudart ;  static ; -O3 ; --relocatable-device-code=false; -Xptxas -dlcm=cg)
	SET(CUDA_SEPARABLE_COMPILATION OFF)
	SET(CUDA_VERBOSE_BUILD ON)
	
	# Detect CUDA architecture and get best NVCC flags
	
	INCLUDE(${CMAKE_CURRENT_SOURCE_DIR}/cmake/FindCudaArch.cmake)
	SELECT_NVCC_ARCH_FLAGS(NVCC_FLAGS_EXTRA)
	LIST(APPEND CUDA_NVCC_FLAGS ${NVCC_FLAGS_EXTRA})
	
endif(CUDA_FOUND)

if((CUDA_FOUND) AND (CUDA_GPU_DETECT_OUTPUT))

	SET(BUILD_CUDA_TARGETS  ON)

else()

	SET(BUILD_CUDA_TARGETS  OFF)

endif()

# messages
MESSAGE(STATUS "Build CUDA/NVCC-based targets: ${BUILD_CUDA_TARGETS}")
MESSAGE(STATUS "TCLAP Include path: ${TCLAP_INCLUDE_DIR}")
MESSAGE(STATUS "Thrust Include path: ${THRUST_INCLUDE_DIR}")
MESSAGE(STATUS "ROOT Include path: ${ROOT_INCLUDE_DIR}")
MESSAGE(STATUS "ROOT Library path: ${ROOT_LIBRARY_DIR}")
MESSAGE(STATUS "ROOT Libraries: ${ROOT_LIBRARIES}")
MESSAGE(STATUS "project source dir: " ${PROJECT_SOURCE_DIR} )
MESSAGE(STATUS "project build dir: " ${PROJECT_BINARY_DIR} )
MESSAGE(STATUS "CUDA libraries: " ${CUDA_LIBRARIES}  )
MESSAGE(STATUS "nvcc cflags: " ${CUDA_NVCC_FLAGS} )
MESSAGE(STATUS "gcc cflags: " ${CMAKE_CXX_FLAGS} )
MESSAGE(STATUS "Install diretory: " ${CMAKE_INSTALL_PREFIX}  )

if(BUILD_CUDA_TARGETS)

#make cuda targets

    cuda_add_executable(MCBoosterGenerateSample
	${CMAKE_CURRENT_SOURCE_DIR}/src/GenerateSample.cu
	)
	
	target_link_libraries(MCBoosterGenerateSample
	${ROOT_LIBRARIES}
	rt
	)
	
	cuda_add_executable(MCBoosterCUDA
	${CMAKE_CURRENT_SOURCE_DIR}/src/Generate.cu
	)
	
	target_link_libraries(MCBoosterCUDA
	${ROOT_LIBRARIES}
	rt
	)
	
	cuda_add_executable(PerfTest_MCBoosterCUDA
	${CMAKE_CURRENT_SOURCE_DIR}/src/PerformanceTest.cu
	)
	
	target_link_libraries(PerfTest_MCBoosterCUDA
	${ROOT_LIBRARIES}
	rt	
	)
	
	cuda_add_executable(MCBoosterOpenMP_NVCC
	${CMAKE_CURRENT_SOURCE_DIR}/src/Generate.cu
	OPTIONS -DTHRUST_DEVICE_SYSTEM=THRUST_DEVICE_SYSTEM_OMP
	)
	
	target_link_libraries(MCBoosterOpenMP_NVCC
	${ROOT_LIBRARIES}
	rt
	)
	
endif(BUILD_CUDA_TARGETS)



add_executable(MCBoosterOpenMP_GCC
${CMAKE_CURRENT_SOURCE_DIR}/src/Generate.cpp
)
set_target_properties( MCBoosterOpenMP_GCC PROPERTIES COMPILE_FLAGS "-std=c++11 -DTHRUST_DEVICE_SYSTEM=THRUST_DEVICE_SYSTEM_OMP" )

target_link_libraries(MCBoosterOpenMP_GCC
${ROOT_LIBRARIES}
rt
)


#cuda_add_executable(PerfTest_MCBoosterOpenMP 
#${CMAKE_CURRENT_SOURCE_DIR}/src/PerformanceTest.cu
#OPTIONS -DTHRUST_DEVICE_SYSTEM=THRUST_DEVICE_SYSTEM_OMP
#)


#target_link_libraries(PerfTest_MCBoosterOpenMP
#${ROOT_LIBRARIES}
#rt
#)

install(DIRECTORY include
  DESTINATION include)

install(DIRECTORY 
	src
  DESTINATION src)
  
install(DIRECTORY 
	cmake
  DESTINATION cmake)

install(FILES
  CMakeLists.txt
  COPYING
  Doxyfile
  Doxyfile.in
  MCBooster.h.in
  DESTINATION ./)
  
if(BUILD_CUDA_TARGETS)
	install(TARGETS MCBoosterCUDA DESTINATION bin)
	install(TARGETS PerfTest_MCBoosterCUDA DESTINATION bin)
	install(TARGETS MCBoosterOpenMP_NVCC DESTINATION bin)
endif()

install(TARGETS MCBoosterOpenMP_GCC DESTINATION bin)
#install(TARGETS PerfTest_MCBoosterOpenMP DESTINATION bin)


if(BUILD_DOCUMENTATION)

    if(NOT DOXYGEN_FOUND)
        message(FALTAL_ERROR "Doxygen is needed to build the documentation.")
    endif()

    set(doxyfile_in ${CMAKE_CURRENT_SOURCE_DIR}/doc/Doxyfile.in)
    set(doxyfile ${CMAKE_CURRENT_BINARY_DIR}/doc/Doxyfile)

    configure_file(${doxyfile_in} ${doxyfile} @ONLY)

    add_custom_target(doc
        COMMAND ${DOXYGEN_EXECUTABLE} ${doxyfile}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        COMMENT "Generating API documentation with Doxygen"
        VERBATIM)

    install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doc DESTINATION doc)
    
endif()

# build a CPack driven installer package
include (InstallRequiredSystemLibraries)
set (CPACK_SOURCE_IGNORE_FILES  build/*)
set (CPACK_RESOURCE_FILE_LICENSE  
     "${CMAKE_CURRENT_SOURCE_DIR}/COPYING")
set (CPACK_RESOURCE_FILE_README  
     "${CMAKE_CURRENT_SOURCE_DIR}/README.md")
set (CPACK_PACKAGE_VERSION_MAJOR "${MCBooster_VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${MCBooster_VERSION_MINOR}")
set (CPACK_PACKAGE_VERSION_PATCH "${MCBooster_VERSION_PATCH}")
include (CPack)


