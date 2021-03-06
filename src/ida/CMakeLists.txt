# ---------------------------------------------------------------
# Programmer:  Daniel R. Reynolds @ SMU
#              Radu Serban @ LLNL
# ---------------------------------------------------------------
# LLNS/SMU Copyright Start
# Copyright (c) 2017, Southern Methodist University and 
# Lawrence Livermore National Security
#
# This work was performed under the auspices of the U.S. Department 
# of Energy by Southern Methodist University and Lawrence Livermore 
# National Laboratory under Contract DE-AC52-07NA27344.
# Produced at Southern Methodist University and the Lawrence 
# Livermore National Laboratory.
#
# All rights reserved.
# For details, see the LICENSE file.
# LLNS/SMU Copyright End
# ---------------------------------------------------------------
# CMakeLists.txt file for the IDA library

INSTALL(CODE "MESSAGE(\"\nInstall IDA\n\")")

# Add variable ida_SOURCES with the sources for the IDA library
SET(ida_SOURCES
  ida.c
  ida_bbdpre.c
  ida_direct.c
  ida_ic.c
  ida_io.c
  ida_spils.c
  )

# Add variable shared_SOURCES with the common SUNDIALS sources which will
# also be included in the IDA library
SET(shared_SOURCES
  ${sundials_SOURCE_DIR}/src/sundials/sundials_nvector.c
  ${sundials_SOURCE_DIR}/src/sundials/sundials_matrix.c
  ${sundials_SOURCE_DIR}/src/sundials/sundials_linearsolver.c
  ${sundials_SOURCE_DIR}/src/sundials/sundials_math.c
  ${sundials_SOURCE_DIR}/src/sundials/sundials_band.c
  ${sundials_SOURCE_DIR}/src/sundials/sundials_dense.c
  ${sundials_SOURCE_DIR}/src/sundials/sundials_direct.c
  ${sundials_SOURCE_DIR}/src/sundials/sundials_iterative.c
  ${sundials_SOURCE_DIR}/src/sundials/sundials_version.c
  ${sundials_SOURCE_DIR}/src/nvec_ser/nvector_serial.c
  )

# Add variable sunmatrix_SOURCES with the common SUNMatrix sources which will
# also be included in the IDA library
SET(sunmatrix_SOURCES
  ${sundials_SOURCE_DIR}/src/sunmat_band/sunmatrix_band.c
  ${sundials_SOURCE_DIR}/src/sunmat_dense/sunmatrix_dense.c
  ${sundials_SOURCE_DIR}/src/sunmat_sparse/sunmatrix_sparse.c
  )

# Add variable sunlinsol_SOURCES with the common SUNLinearSolver sources which will
# also be included in the IDA library
SET(sunlinsol_SOURCES
  ${sundials_SOURCE_DIR}/src/sunlinsol_band/sunlinsol_band.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_dense/sunlinsol_dense.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_spbcgs/sunlinsol_spbcgs.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_spfgmr/sunlinsol_spfgmr.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_spgmr/sunlinsol_spgmr.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_sptfqmr/sunlinsol_sptfqmr.c
  ${sundials_SOURCE_DIR}/src/sunlinsol_pcg/sunlinsol_pcg.c
  )

# IF(PETSC_FOUND)
#    # use MPI_MPICC as the compiler
#    SET(CMAKE_C_COMPILER ${MPI_MPICC})
#    LIST(APPEND ida_SOURCES ida_petsc.c)
# ENDIF()

# Add variable ida_HEADERS with the exported IDA header files
SET(ida_HEADERS
  ida.h
  ida_bbdpre.h
  ida_direct.h
  ida_spils.h
  )

# Add prefix with complete path to the IDA header files
ADD_PREFIX(${sundials_SOURCE_DIR}/include/ida/ ida_HEADERS)

# Add source directories to include directories for access to
# implementation only header files.
INCLUDE_DIRECTORIES(.)
INCLUDE_DIRECTORIES(../sundials)

# Define C preprocessor flag -DBUILD_SUNDIALS_LIBRARY 
ADD_DEFINITIONS(-DBUILD_SUNDIALS_LIBRARY)

# Build the static library
IF(BUILD_STATIC_LIBS)

  # Add the build target for the static IDA library
  ADD_LIBRARY(sundials_ida_static STATIC 
    ${ida_SOURCES} ${shared_SOURCES} ${sunmatrix_SOURCES} ${sunlinsol_SOURCES})

  # Set the library name and make sure it is not deleted
  SET_TARGET_PROPERTIES(sundials_ida_static
    PROPERTIES OUTPUT_NAME sundials_ida CLEAN_DIRECT_OUTPUT 1)

  # Install the IDA library
  INSTALL(TARGETS sundials_ida_static DESTINATION lib)

ENDIF(BUILD_STATIC_LIBS)

# Build the shared library
IF(BUILD_SHARED_LIBS)

  # Add the build target for the IDA library
  ADD_LIBRARY(sundials_ida_shared SHARED
    ${ida_SOURCES} ${shared_SOURCES} ${sunmatrix_SOURCES} ${sunlinsol_SOURCES})

  IF(UNIX)
    TARGET_LINK_LIBRARIES(sundials_ida_shared m)
  ENDIF()

  # Set the library name and make sure it is not deleted
  SET_TARGET_PROPERTIES(sundials_ida_shared
    PROPERTIES OUTPUT_NAME sundials_ida CLEAN_DIRECT_OUTPUT 1)

  # Set VERSION and SOVERSION for shared libraries
  SET_TARGET_PROPERTIES(sundials_ida_shared
    PROPERTIES VERSION ${idalib_VERSION} SOVERSION ${idalib_SOVERSION})

  # Install the IDA library
  INSTALL(TARGETS sundials_ida_shared DESTINATION lib)

ENDIF(BUILD_SHARED_LIBS)

# Install the IDA header files
INSTALL(FILES ${ida_HEADERS} DESTINATION include/ida)

# Install the IDA implementation header file
INSTALL(FILES ida_impl.h DESTINATION include/ida)

#
MESSAGE(STATUS "Added IDA module")
