SET(BOOST_MAJOR_VERSION 1)
SET(BOOST_MINOR_VERSION 70)
SET(BOOST_SUBMINOR_VERSION 0)
SET(BOOST_VERSION ${BOOST_MAJOR_VERSION}.${BOOST_MINOR_VERSION}.${BOOST_SUBMINOR_VERSION})

SET(BOOST_PROJECT_DIR "${EXTERNAL_PROJECT_DIR}/boost")
SET(BOOST_OUTPUT_DIR "${EXTERNAL_PROJECT_DIR}/lib/boost/${KLAYGE_PLATFORM_NAME}")
SET(REL_PATH "External/Downloads/boost_${BOOST_MAJOR_VERSION}_${BOOST_MINOR_VERSION}_${BOOST_SUBMINOR_VERSION}.patched.7z")
SET(PACKAGE_FILE_ID "FAB753C1749EFA033C19B479668D8E6D93FBD2BE")

DOWNLOAD_PACKAGE("boost" ${REL_PATH} ${PACKAGE_FILE_ID} "${BOOST_PROJECT_DIR}/LICENSE_1_0.txt")

IF(KLAYGE_COMPILER_MSVC
	OR (KLAYGE_COMPILER_GCC AND (NOT KLAYGE_PLATFORM_WINDOWS OR NOT (KLAYGE_COMPILER_VERSION EQUAL 81)))
	OR (KLAYGE_COMPILER_CLANG AND (NOT (KLAYGE_PLATFORM_DARWIN OR KLAYGE_PLATFORM_IOS)) AND (NOT (KLAYGE_COMPILER_VERSION LESS 70))))
	SET(WITH_FILESYSTEM OFF)
ELSE()
	SET(WITH_FILESYSTEM ON)
ENDIF()

set(KLAYGE_WITH_BOOST_FILESYSTEM ${WITH_FILESYSTEM} CACHE INTERNAL "" FORCE)

IF(KLAYGE_PLATFORM_WINDOWS_STORE)
	ADD_DEFINITIONS(-DBOOST_NO_ANSI_APIS)
ENDIF()
IF(KLAYGE_COMPILER_MSVC)
	ADD_DEFINITIONS(-D_CRT_SECURE_NO_DEPRECATE)
	ADD_DEFINITIONS(-D_SCL_SECURE_NO_DEPRECATE)
ENDIF()
IF(KLAYGE_PLATFORM_WINDOWS)
	IF((KLAYGE_COMPILER_GCC AND (KLAYGE_ARCH_NAME STREQUAL "x64")) OR KLAYGE_COMPILER_CLANG)
		ADD_DEFINITIONS(-DBOOST_USE_WINDOWS_H)
	ENDIF()
ENDIF()

set(Boost_INCLUDE_DIR "${EXTERNAL_PROJECT_DIR}/boost" CACHE INTERNAL "" FORCE)
if(KLAYGE_PLATFORM_WINDOWS OR (KLAYGE_PREFERRED_LIB_TYPE STREQUAL "STATIC"))
	set(boost_lib_dir ${EXTERNAL_PROJECT_DIR}/lib/boost/${KLAYGE_PLATFORM_NAME})
else()
	set(boost_lib_dir ${KLAYGE_BIN_DIR})
endif()
set(is_static_lib FALSE)
if(KLAYGE_PREFERRED_LIB_TYPE STREQUAL "STATIC")
	set(is_static_lib TRUE)
endif()
GenerateLibFileName("boost_filesystem${KLAYGE_OUTPUT_SUFFIX}${CMAKE_DEBUG_POSTFIX}" ${is_static_lib} debug_filesystem_name)
GenerateLibFileName("boost_filesystem${KLAYGE_OUTPUT_SUFFIX}" ${is_static_lib} optimized_filesystem_name)
set(Boost_FILESYSTEM_LIBRARY
	debug ${boost_lib_dir}/${debug_filesystem_name} optimized ${boost_lib_dir}/${optimized_filesystem_name}
)

if(KLAYGE_COMPILER_MSVC)
	if(KLAYGE_PLATFORM_WINDOWS_STORE)
		set(KLAYGE_FILESYSTEM_LIBRARY "kernel32")
	else()
		set(KLAYGE_FILESYSTEM_LIBRARY "")
	endif()
else()
	if(KLAYGE_COMPILER_GCC AND (NOT KLAYGE_PLATFORM_WINDOWS OR NOT (KLAYGE_COMPILER_VERSION EQUAL 81)))
		set(KLAYGE_FILESYSTEM_LIBRARY "stdc++fs")
	elseif(KLAYGE_COMPILER_CLANG AND KLAYGE_PLATFORM_LINUX)
		if(KLAYGE_COMPILER_VERSION LESS 70)
			set(KLAYGE_FILESYSTEM_LIBRARY "c++experimental")
		else()
			set(KLAYGE_FILESYSTEM_LIBRARY "c++fs")
		endif()
	else()
		if(KLAYGE_PLATFORM_WINDOWS OR (KLAYGE_PREFERRED_LIB_TYPE STREQUAL "STATIC"))
			set(boost_lib_dir ${EXTERNAL_PROJECT_DIR}/lib/boost/${KLAYGE_PLATFORM_NAME})
		else()
			set(boost_lib_dir ${KLAYGE_BIN_DIR})
		endif()
		set(KLAYGE_FILESYSTEM_LIBRARY ${Boost_FILESYSTEM_LIBRARY})
	endif()
endif()
set(KLAYGE_FILESYSTEM_LIBRARY "${KLAYGE_FILESYSTEM_LIBRARY}" CACHE INTERNAL "" FORCE)

MACRO(SETUP_BOOST_LIBRARY LIB_NAME)
	SOURCE_GROUP("Source Files" FILES ${SOURCE_FILES})

	INCLUDE_DIRECTORIES(${BOOST_PROJECT_DIR}/)
	LINK_DIRECTORIES(${BOOST_PROJECT_DIR}/lib/${KLAYGE_PLATFORM_NAME})
	LINK_DIRECTORIES(${KLAYGE_BIN_DIR})

	ADD_LIBRARY(${LIB_NAME} ${KLAYGE_PREFERRED_LIB_TYPE}
		${SOURCE_FILES}
	)

	SET_TARGET_PROPERTIES(${LIB_NAME} PROPERTIES
		ARCHIVE_OUTPUT_DIRECTORY ${BOOST_OUTPUT_DIR}
		ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${BOOST_OUTPUT_DIR}
		ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${BOOST_OUTPUT_DIR}
		ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO ${BOOST_OUTPUT_DIR}
		ARCHIVE_OUTPUT_DIRECTORY_MINSIZEREL ${BOOST_OUTPUT_DIR}
		RUNTIME_OUTPUT_DIRECTORY ${KLAYGE_BIN_DIR}
		RUNTIME_OUTPUT_DIRECTORY_DEBUG ${KLAYGE_BIN_DIR}
		RUNTIME_OUTPUT_DIRECTORY_RELEASE ${KLAYGE_BIN_DIR}
		RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${KLAYGE_BIN_DIR}
		RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL ${KLAYGE_BIN_DIR}
		LIBRARY_OUTPUT_DIRECTORY ${KLAYGE_BIN_DIR}
		LIBRARY_OUTPUT_DIRECTORY_DEBUG ${KLAYGE_BIN_DIR}
		LIBRARY_OUTPUT_DIRECTORY_RELEASE ${KLAYGE_BIN_DIR}
		LIBRARY_OUTPUT_DIRECTORY_RELWITHDEBINFO ${KLAYGE_BIN_DIR}
		LIBRARY_OUTPUT_DIRECTORY_MINSIZEREL ${KLAYGE_BIN_DIR}
		PROJECT_LABEL ${LIB_NAME}
		DEBUG_POSTFIX ${CMAKE_DEBUG_POSTFIX}
		OUTPUT_NAME boost_${LIB_NAME}${KLAYGE_OUTPUT_SUFFIX}
		FOLDER "${EXTERNAL_VS_FOLDER_PREFIX}/boost"
	)

	TARGET_LINK_LIBRARIES(${LIB_NAME} ${EXTRA_LINKED_LIBRARIES})
ENDMACRO()

IF(WITH_FILESYSTEM)
	ADD_SUBDIRECTORY(filesystem)
ENDIF()
