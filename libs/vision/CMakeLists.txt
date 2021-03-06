# Lists of directories with source files:
#  See "DeclareMRPTLib.cmake" for explanations
# -------------------------------------------------
LIST(APPEND vision_EXTRA_SRCS		"${MRPT_SOURCE_DIR}/libs/vision/src/maps/*.cpp" "${MRPT_SOURCE_DIR}/libs/vision/include/mrpt/maps/*.h" "${MRPT_SOURCE_DIR}/libs/vision/include/mrpt/slam/*.h")
LIST(APPEND vision_EXTRA_SRCS_NAME 	"maps" "maps" "maps-bcks-compat")

LIST(APPEND vision_EXTRA_SRCS		"${MRPT_SOURCE_DIR}/libs/vision/src/obs/*.cpp" "${MRPT_SOURCE_DIR}/libs/vision/include/mrpt/slam/CObservation*.h")
LIST(APPEND vision_EXTRA_SRCS_NAME 	"observations" "observations")

LIST(APPEND vision_EXTRA_SRCS		"${MRPT_SOURCE_DIR}/libs/vision/src/pnp/*.cpp" "${MRPT_SOURCE_DIR}/libs/vision/include/mrpt/vision/pnp_algos.h")
LIST(APPEND vision_EXTRA_SRCS_NAME 	"pnp" "pnp")

IF(CMAKE_MRPT_HAS_SIFT_HESS)
	LIST(APPEND vision_EXTRA_SRCS
		"${MRPT_SOURCE_DIR}/libs/vision/src/sift-hess/imgfeatures.c"
		"${MRPT_SOURCE_DIR}/libs/vision/src/sift-hess/sift.c"
		"${MRPT_SOURCE_DIR}/libs/vision/src/sift-hess/utils.c"
		"${MRPT_SOURCE_DIR}/libs/vision/include/mrpt/sift-hess/*.h")
	LIST(APPEND vision_EXTRA_SRCS_NAME 	"sift-hess" "sift-hess" "sift-hess" "sift-hess")
ENDIF()

IF(CMAKE_MRPT_HAS_OPENCV)

	include(ExternalProject)

	ExternalProject_Add(libcvd
		SOURCE_DIR ${MRPT_SOURCE_DIR}/otherlibs/libcvd
		CMAKE_ARGS
			-DCMAKE_POSITION_INDEPENDENT_CODE=ON
			-DCVD_SSE2=${CMAKE_MRPT_HAS_SSE2}
			-DCVD_TESTS=OFF
			-DCVD_PROGS=OFF
			-DCVD_EXAMPLES=OFF
		INSTALL_COMMAND ""
	)

	set(CVD_DIR "${MRPT_SOURCE_DIR}/otherlibs/libcvd")
	set(CVD_BIN_DIR "${MRPT_BINARY_DIR}/libs/vision/libcvd-prefix/src/libcvd-build")

	if(MSVC)
		set(CVD_LIB "${CVD_BIN_DIR}/$<CONFIG>/CVD$<$<CONFIG:Debug>:_debug>.lib")
	else()
		set(CVD_LIB "${CVD_BIN_DIR}/libCVD.a")
	endif()
	INCLUDE_DIRECTORIES(${CVD_DIR})

	INCLUDE_DIRECTORIES(${OpenCV_INCLUDE_DIR} ${OpenCV_INCLUDE_DIRS} ${OPENCV_INCLUDE_DIRS} ${OPENCV_INCLUDEDIR})
ENDIF()

#---------------------------------------------
# Macro declared in "DeclareMRPTLib.cmake":
#---------------------------------------------
define_mrpt_lib(
	# Lib name
	vision
	# Dependencies:
	mrpt-obs
	)

IF(CMAKE_MRPT_HAS_OPENCV)
	add_dependencies(mrpt-vision libcvd)
ENDIF()

IF(BUILD_mrpt-vision)
	IF(CMAKE_MRPT_HAS_SIFT_HESS)
		IF (MRPT_ENABLE_PRECOMPILED_HDRS AND MSVC)
			# Ignore precompiled headers in sift-hess sources:
			set_source_files_properties(${sift-hess_FILES} PROPERTIES COMPILE_FLAGS "/Y- /wd4244")
		ENDIF ()

		# Avoid some annoying warnings:
		IF (CMAKE_COMPILER_IS_GNUCXX)
			set_source_files_properties(${sift-hess_FILES} PROPERTIES COMPILE_FLAGS "-Wno-unused-function")
		ENDIF ()
	ENDIF()

	# Ignore precompiled headers in faster sources:
	IF (MRPT_ENABLE_PRECOMPILED_HDRS AND MSVC)
		set_source_files_properties(${faster_FILES} PROPERTIES COMPILE_FLAGS "/Y-")
	ENDIF ()

	IF(CMAKE_MRPT_HAS_OPENCV)
		target_link_libraries(mrpt-vision PRIVATE ${OpenCV_LIBRARIES} ${CVD_LIB})
	ENDIF()

ENDIF()
