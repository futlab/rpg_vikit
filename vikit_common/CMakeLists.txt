SET(PROJECT_NAME vikit_common)
PROJECT(${PROJECT_NAME})
CMAKE_MINIMUM_REQUIRED (VERSION 3.10)
SET(CMAKE_BUILD_TYPE Release) # Release, RelWithDebInfo
SET(CMAKE_VERBOSE_MAKEFILE OFF)
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/CMakeModules/")
OPTION(USE_ROS "Build with Catkin" FALSE) # Set False if you want to build this package without Catkin

# Set build flags. Set IS_ARM on odroid board as environment variable
SET(CMAKE_CXX_FLAGS "-Wall -D_LINUX -D_REENTRANT -march=native -Wno-unused-variable -Wno-unused-but-set-variable -Wno-unknown-pragmas")
IF(DEFINED ENV{ARM_ARCHITECTURE})
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfpu=neon -march=armv7-a")
ELSE()
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mmmx -msse -msse -msse2 -msse3 -mssse3")
ENDIF()
IF(CMAKE_COMPILER_IS_GNUCC)
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
ELSE()
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
ENDIF()
SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS} -O3 -fsee -fomit-frame-pointer -fno-signed-zeros -fno-math-errno -funroll-loops")

# Add plain cmake packages
find_package(OpenCV REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(Sophus REQUIRED)


IF(USE_ROS)
  FIND_PACKAGE(catkin REQUIRED COMPONENTS roscpp cmake_modules)
  LIST(APPEND INCLUDE_DIRECTORIES ${catkin_INCLUDE_DIRS})
  catkin_package(
    DEPENDS Eigen OpenCV Sophus
    CATKIN_DEPENDS roscpp
    INCLUDE_DIRS include
    LIBRARIES ${PROJECT_NAME}
  )
ENDIF()

# Set Sourcefiles
list(APPEND SOURCEFILES src/atan_camera.cpp
                        src/omni_camera.cpp
                        src/math_utils.cpp
                        src/vision.cpp
                        src/performance_monitor.cpp
                        src/robust_cost.cpp
                        src/user_input_thread.cpp
                        src/pinhole_camera.cpp
                        src/homography.cpp
                        src/img_align.cpp)

# Create vikit library
add_library(vikit_common ${SOURCEFILES})
set_target_properties(vikit_common PROPERTIES
    CXX_STANDARD 17
    CXX_EXTENSIONS OFF
    POSITION_INDEPENDENT_CODE ON
)

add_library(vikit::common ALIAS vikit_common)
target_link_libraries(${PROJECT_NAME}
    Eigen3::Eigen
    ${OpenCV_LIBS}
    ${Sophus_LIBRARIES}
    ${catkin_LIBRARIES}
)

# Include dirs
target_include_directories(vikit_common
    PUBLIC include
    PRIVATE ${OpenCV_INCLUDE_DIRS}
    PRIVATE ${Sophus_INCLUDE_DIRS}
)


if(USE_ROS)
  TARGET_LINK_LIBRARIES(vikit_common ${catkin_LIBRARIES})
endif()

# Tests
add_executable(test_vk_common_camera test/test_camera.cpp)
TARGET_LINK_LIBRARIES(test_vk_common_camera vikit::common)

add_executable(test_vk_common_triangulation test/test_triangulation.cpp)
TARGET_LINK_LIBRARIES(test_vk_common_triangulation vikit::common)

add_executable(test_vk_common_patch_score test/test_patch_score.cpp)
TARGET_LINK_LIBRARIES(test_vk_common_patch_score vikit::common)


################################################################################
# Create the vikit_commonConfig.cmake file for other cmake projects.
if(NOT USE_ROS)
    install(
        TARGETS vikit_common
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include
    )

    export(TARGETS vikit_common NAMESPACE vikit:: FILE vikit_commonTargets.cmake)

    configure_file(vikit_commonConfig.cmake.in vikit_commonConfig.cmake @ONLY)
    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/vikit_commonConfig.cmake"
            DESTINATION lib/cmake/vikit_common
    )


  # GET_TARGET_PROPERTY( FULL_LIBRARY_NAME ${PROJECT_NAME} LOCATION )
  # SET(vikit_common_LIBRARIES ${FULL_LIBRARY_NAME} )
  # SET(vikit_common_LIBRARY_DIR ${PROJECT_BINARY_DIR} )
  # SET(vikit_common_INCLUDE_DIR "${PROJECT_SOURCE_DIR}/include")
  # CONFIGURE_FILE( ${CMAKE_CURRENT_SOURCE_DIR}/vikit_commonConfig.cmake.in
  #     ${CMAKE_CURRENT_BINARY_DIR}/vikit_commonConfig.cmake @ONLY IMMEDIATE )
  # export( PACKAGE vikit_common )

  # INSTALL(DIRECTORY include/vikit DESTINATION ${CMAKE_INSTALL_PREFIX}/include FILES_MATCHING PATTERN "*.h" )
  # INSTALL(TARGETS ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_PREFIX}/lib )
endif()
