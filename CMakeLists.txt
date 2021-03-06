cmake_minimum_required (VERSION 2.6)
project (cint)
set(cint_VERSION "2.8")

#set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O2 -DNDEBUG")
set(CMAKE_BUILD_TYPE RELWITHDEBINFO)
set(CMAKE_VERBOSE_MAKEFILE OFF)

if (${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION} GREATER 2.8)
  include(GNUInstallDirs)
elseif (${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION} EQUAL 2.8 AND
    ${CMAKE_PATCH_VERSION} GREATER 4)
  include(GNUInstallDirs)
else ()
  set(CMAKE_INSTALL_LIBDIR lib CACHE PATH "lib")
  set(CMAKE_INSTALL_INCLUDEDIR include CACHE PATH "include")
endif (${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION} GREATER 2.8)
#set(CMAKE_INCLUDE_CURRENT_DIR ON)

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")
find_package(QUADMATH)
if(QUADMATH_FOUND)
  set(HAVE_QUADMATH_H 1 CACHE INTERNAL "Have QUADMATH")
endif()

find_package(BLAS REQUIRED)

include(CheckFunctionExists)
set(CMAKE_REQUIRED_INCLUDES math.h)
set(CMAKE_REQUIRED_LIBRARIES m)
check_function_exists(expl HAVE_EXPL)
check_function_exists(sqrtl HAVE_SQRTL)
#set(CMAKE_REQUIRED_INCLUDES quadmath.h)
#set(CMAKE_REQUIRED_LIBRARIES quadmath)
#check_function_exists(fabsq HAVE_QUADMATH_H)

include_directories(
  ${PROJECT_BINARY_DIR}/include
  ${PROJECT_SOURCE_DIR}/src
  ${PROJECT_BINARY_DIR}/src)

configure_file(
  "${PROJECT_SOURCE_DIR}/src/config.h.in"
  "${PROJECT_BINARY_DIR}/src/config.h")

configure_file(
  "${PROJECT_SOURCE_DIR}/include/cint.h.in"
  "${PROJECT_BINARY_DIR}/include/cint.h")

set(cintSrc 
  src/c2f.c src/cart2sph.c src/cint1e.c src/cint2e.c src/cint_bas.c
  src/fblas.c src/g1e.c src/g2e.c src/misc.c src/optimizer.c
  src/rys_roots.c
  src/cint2c2e.c src/g2c2e.c src/cint3c2e.c src/g3c2e.c src/optimizer_3c2e.c 
  src/cint3c1e.c src/g3c1e.c src/breit.c src/cint4c1e.c src/g4c1e.c
  src/autocode/auto_intor1.c src/autocode/auto_intor2.c
  src/autocode/auto_intor3.c src/autocode/grad1.c
  src/autocode/grad2.c src/autocode/grad3.c src/autocode/auto_hess.c
  src/autocode/auto_3c2e.c src/autocode/auto_3c1e.c
  src/autocode/gaunt1.c src/autocode/breit1.c)

option(BUILD_SHARED_LIBS "build shared libraries" 1)
option(ENABLE_EXAMPLE "build examples" 0)
option(ENABLE_TEST "build tests" 0)
if(QUICK_TEST)
  set(RUN_QUICK_TEST --quick)
endif()

if(ENABLE_STATIC)
  set(BUILD_SHARED_LIBS 0)
endif()

add_library(cint ${cintSrc})
set_target_properties(cint PROPERTIES
  VERSION ${cint_VERSION}
  LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
target_link_libraries(cint ${BLAS_LIBRARIES})
if(QUADMATH_FOUND)
  target_link_libraries(cint quadmath)
endif()

install(TARGETS cint DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR})
install(FILES ${PROJECT_BINARY_DIR}/include/cint.h DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_INCLUDEDIR})

if(ENABLE_EXAMPLE)
  enable_language(Fortran)
  find_package(OpenMP)
  if(OPENMP_FOUND)
    set(HAVE_OPENMP 1)
  else ()
    set(OpenMP_C_FLAGS " ")
  endif()
  add_subdirectory(examples)
endif()

if(BUILD_SHARED_LIBS AND ENABLE_TEST)
  find_package(PythonInterp)
  message(STATUS "Found python  ${PYTHON_EXECUTABLE}")
  if(PYTHONINTERP_FOUND)
    enable_testing()
    add_test(NAME cinttest
      COMMAND ${PYTHON_EXECUTABLE} ${PROJECT_SOURCE_DIR}/testsuite/test_cint.py ${RUN_QUICK_TEST})
    add_test(NAME cint3c2etest
      COMMAND ${PYTHON_EXECUTABLE} ${PROJECT_SOURCE_DIR}/testsuite/test_3c2e.py ${RUN_QUICK_TEST})
  endif()
endif()
