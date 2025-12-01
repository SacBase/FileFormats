CMAKE_MINIMUM_REQUIRED (VERSION 3.20)

PROJECT (sac-imageformats)

# Where the compiled sac modules result
SET (DLL_BUILD_DIR "${PROJECT_BINARY_DIR}/lib")

# For what targets we build modules
SET (TARGETS    seq seq_checks mt_pth CACHE STRING "Build for these targets")
SET (SAC2C_EXEC                       CACHE STRING "A path to sac2c compiler")
OPTION (BUILDGENERIC "Do not use architecture specific optimisations (useful for package builds)" OFF)

# Check whether sac2c is operational
LIST (APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake-common")
INCLUDE ("cmake-common/check-sac2c.cmake")
INCLUDE ("cmake-common/check-sac2c-feature-support.cmake")
INCLUDE ("cmake-common/misc-macros.cmake")

# Add our custom Find*.cmake
LIST (APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# Ensure libYAML is available
FIND_PACKAGE (LibYAML REQUIRED)

# Ensure cJSON is available
FIND_PACKAGE (cJSON REQUIRED)
SET (SAC2C_EXTRA_INC "-I${CJSON_INCLUDE_DIR}"
	 CACHE STRING "Extra include files that should be used by sac2c")
SET (SAC2C_CPP_INC "-Xc" "-L${CJSON_LIBRARIES}" "-Xl" "-L${CJSON_LIBRARIES}")

# Check if sac2c suppports building generically
IF (BUILDGENERIC)
    CHECK_SAC2C_SUPPORT_FLAG ("generic" "-generic")
    IF (HAVE_FLAG_generic)
        LIST (APPEND SAC2C_CPP_INC "-generic")
        MESSAGE (STATUS "Building with *no* system-specific optimisations")
    ELSE ()
        MESSAGE (STATUS "Generic-build disabled as sac2c does not support this")
        SET (BUILDGENERIC OFF)
    ENDIF ()
ENDIF ()

# For every target run CMakeLists.txt in src
FOREACH (TARGET IN ITEMS ${TARGETS})
    ADD_SUBDIRECTORY (src src-${TARGET})
ENDFOREACH ()

# This build target is responsible for generating the package sac2crc file
CREATE_SAC2CRC_TARGET ("imageformats" "${DLL_BUILD_DIR}" "${DLL_BUILD_DIR}" "")