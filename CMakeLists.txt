################################################################################
# user build settings
SET(TRACE TRUE)
SET(HAVE_G2O TRUE)
#SET(HAVE_MRPT TRUE)
#TRUE

SET(DEBUG_OUTPUT TRUE) # Only relevant if build without ROS

################################################################################
MESSAGE ("This is message.lep")
#UNCOMMENT
#SET(DEFAULT_HAS_MRPT ON)
#UNCOMMENT
#SET(HAS_MRPT ${DEFAULT_HAS_MRPT} CACHE BOOL "Build the representation api which uses the MRPT library")

SET(PROJECT_NAME lepsvo)
PROJECT(${PROJECT_NAME})
CMAKE_MINIMUM_REQUIRED (VERSION 2.8.3)
SET(CMAKE_BUILD_TYPE Release) # Release, RelWithDebInfo -> Set in cmake-gui option
SET(CMAKE_VERBOSE_MAKEFILE OFF)
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/CMakeModules/")

# Set definitions

#IF(HAVE_MRPT)
# ADD_DEFINITIONS(-DHAS_MRPT)
#ENDIF()

IF(TRACE)
    ADD_DEFINITIONS(-DSVO_TRACE)
ENDIF()

IF(DEBUG_OUTPUT)
    ADD_DEFINITIONS(-DSVO_DEBUG_OUTPUT)
ENDIF()

# Set build flags, set ARM_ARCHITECTURE environment variable on Odroid
SET(CMAKE_CXX_FLAGS "-Wall -D_LINUX -D_REENTRANT -march=native -Wno-unused-variable -Wno-unused-but-set-variable -Wno-unknown-pragmas -Wno-reorder")
IF(DEFINED ENV{ARM_ARCHITECTURE})
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfpu=neon -march=armv7-a")
ELSE()
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mmmx -msse -msse -msse2 -msse3 -mssse3")
ENDIF()
IF(CMAKE_COMPILER_IS_GNUCC)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
ELSE()
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    #SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O3 -mtune=native -march=native")
ENDIF()
SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O3 -fsee -fomit-frame-pointer -fno-signed-zeros -fno-math-errno -funroll-loops")

# Add plain cmake packages 2.4.13
FIND_PACKAGE(OpenCV 3 REQUIRED)
FIND_PACKAGE(Eigen REQUIRED)
FIND_PACKAGE(Sophus REQUIRED)
#FIND_PACKAGE(fast REQUIRED)
FIND_PACKAGE(Pangolin REQUIRED)
FIND_PACKAGE(Boost REQUIRED COMPONENTS thread system filesystem)

#IF(HAVE_MRPT)
#SET(MRPT_DONT_USE_DBG_LIBS 1) #use release libraries for linking even if "Debug" CMake build
#   FIND_PACKAGE(MRPT REQUIRED base opengl gui)
#ENDIF()

FIND_LIBRARY(YAML_CPP_LIBRARIES yaml-cpp)
if(NOT YAML_CPP_LIBRARIES)
    # If yaml-cpp not found in the system, try finding it as a user CMake-generated project
    FIND_PACKAGE(yaml-cpp REQUIRED)
    INCLUDE_DIRECTORIES(${YAML_CPP_INCLUDE_DIRS})
endif(NOT YAML_CPP_LIBRARIES)

#FIND_PACKAGE(vikit_common REQUIRED)
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

# Include dirs
INCLUDE_DIRECTORIES(
        include
        ${Eigen3_INCLUDE_DIRS}
        ${OpenCV_INCLUDE_DIRS}
        ${Sophus_INCLUDE_DIRS}
        ${Boost_INCLUDE_DIRS}
        # ${fast_INCLUDE_DIRS}
        ${YAML_CPP_INCLUDE_DIRS}
        ${PROJECT_SOURCE_DIR}/3rdparty/line_descriptor/include/
        ${Pangolin_INCLUDE_DIRS}
)
# Set link libraries
LIST(APPEND LINK_LIBS
        ${OpenCV_LIBS}
        #${Sophus_LIBRARIES}
        ${Boost_LIBRARIES}
        #  ${fast_LIBRARIES}
        ${YAML_CPP_LIBRARIES}
        ~/3rdpartylib/Sophus/build/libSophus.so
        ${PROJECT_SOURCE_DIR}/3rdparty/line_descriptor/lib/liblinedesc.so
        )

#INCLUDE_DIRECTORIES(${vikit_common_INCLUDE_DIRS})
#LIST(APPEND LINK_LIBS ${vikit_common_LIBRARIES})

# Set sourcefiles
LIST(APPEND SOURCEFILES
        src/frame_handler_mono.cpp
        src/frame_handler_base.cpp
        src/frame.cpp
        #src/point.cpp
        src/feature.cpp
        src/feature3D.cpp
        src/feature3D_impl.cpp
        src/map.cpp
        src/pose_optimizer.cpp
        src/initialization.cpp
        src/matcher.cpp
        src/reprojector.cpp
        src/feature_alignment.cpp
        src/feature_detection.cpp
        src/depth_filter.cpp
        src/config.cpp
        #src/sparse_img_align.cpp
        src/camera_model.cpp
        src/sparse_align.cpp
        src/debug.cpp
        src/math_utils.cpp
        src/homography.cpp
        src/robust_cost.cpp
        src/fast_10_score.cpp
        src/fast_nonmax_3x3.cpp
        src/fast_10.cpp
        src/faster_corner_10_sse.cpp

        src/slamviewer.cpp
        #src/five-point.cpp
        #src/precomp.cpp
        #src/modelest.cpp
        )

#IF(HAVE_MRPT)
#   LIST(APPEND SOURCEFILES src/sceneRepresentation.cpp)
#ENDIF()

IF(HAVE_G2O)
    ADD_DEFINITIONS(-DUSE_BUNDLE_ADJUSTMENT)
ENDIF()
# Add g2o if available
IF(HAVE_G2O)
    FIND_PACKAGE(G2O REQUIRED)

    message("g2o is found: ${G2O_INCLUDE_DIR}")

    INCLUDE_DIRECTORIES(
            ${G2O_INCLUDE_DIR}
            /usr/include/suitesparse # for cholmod
    )

    #LIST(APPEND LINK_LIBS
    # ${G2O_CORE_LIBRARY} ${G2O_STUFF_LIBRARY} ${G2O_SOLVER_CHOLMOD} ${G2O_SOLVER_CSPARSE} ${G2O_SOLVER_DENSE}
    # ${G2O_SOLVER_PCG} ${G2O_TYPES_SBA} cholmod cxsparse
    #)
    LIST(APPEND LINK_LIBS  g2o_core  g2o_solver_csparse  g2o_csparse_extension g2o_types_sba g2o_solver_dense g2o_stuff g2o_parser g2o_solver_pcg  cholmod cxsparse )
    LIST(APPEND SOURCEFILES src/bundle_adjustment.cpp)
ENDIF(HAVE_G2O)


# Code below is just for accessibility issues in IDE editor
# List all files (headers) contained by svo library
FILE(GLOB_RECURSE all_include_files RELATIVE "${CMAKE_SOURCE_DIR}" *.h *.hpp)

# Visualize the files of this directory in IDE creating an custom empty target
add_custom_target( lepsvo_includes DEPENDS ${all_include_files} SOURCES ${all_include_files} )
#TARGET_LINK_LIBRARIES(svo ${LINK_LIBS}   ${Pangolin_LIBRARIES})

# Create svo library
ADD_LIBRARY(lepsvo SHARED ${SOURCEFILES})
#TARGET_LINK_LIBRARIES(lepsvo ${LINK_LIBS} ${MRPT_LIBS})
#UNCOMMENT ${MRPT_LIBS} add THIS LIKE lepsvo ${LINK_LIBS} ${MRPT_LIBS}
TARGET_LINK_LIBRARIES(lepsvo ${LINK_LIBS}   ${Pangolin_LIBRARIES})

################################################################################
#uncomment
#IF(HAVE_MRPT)
ADD_EXECUTABLE( run_pipeline  	    app/run_pipeline.cpp)
TARGET_LINK_LIBRARIES( run_pipeline	        lepsvo)

ADD_EXECUTABLE( run_pipeline_tum  	 app/run_pipeline_tum.cpp)
TARGET_LINK_LIBRARIES( run_pipeline_tum	        lepsvo)

#ENDIF()


