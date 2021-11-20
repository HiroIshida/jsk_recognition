# http://ros.org/doc/groovy/api/catkin/html/user_guide/supposed.html
cmake_minimum_required(VERSION 2.8.3)
project(jsk_pcl_ros)

# Use ccache if installed to make it fast to generate object files
find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
endif()

# check c++11 / c++0x
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-std=c++14" COMPILER_SUPPORTS_CXX14)
check_cxx_compiler_flag("-std=c++11" COMPILER_SUPPORTS_CXX11)
check_cxx_compiler_flag("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX14)
  set(CMAKE_CXX_FLAGS "-std=c++14")
elseif(COMPILER_SUPPORTS_CXX11)
  set(CMAKE_CXX_FLAGS "-std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
  set(CMAKE_CXX_FLAGS "-std=c++0x")
else()
  message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

find_package(catkin REQUIRED COMPONENTS
  cv_bridge
  dynamic_reconfigure
  eigen_conversions
  # genmsg
  # geometry_msgs
  image_geometry
  image_transport
  interactive_markers
  # jsk_footstep_msgs
  jsk_pcl_ros_utils
  # jsk_recognition_msgs
  jsk_recognition_utils
  # jsk_rviz_plugins # kinfu.h uses jsk_rviz_plugins/OverlayText.h, but jsk_recognition should not depends on jsk_visualization
  jsk_topic_tools
  # kdl_conversions
  kdl_parser
  laser_assembler
  # nodelet
  # octomap_msgs
  octomap_server
  pcl_msgs
  pcl_ros
  # sensor_msgs
  tf
  # tf2_ros
  tf_conversions
  )
find_package(moveit_ros_perception)
find_package(PkgConfig)
pkg_check_modules(ml_classifiers ml_classifiers QUIET)
# only run in hydro
find_package(PCL REQUIRED)
find_package(OpenMP)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")

# For kinfu
#find_package(PCL QUIET COMPONENTS gpu_kinfu_large_scale)

pkg_check_modules(yaml_cpp yaml-cpp REQUIRED)
if(${yaml_cpp_VERSION} VERSION_LESS "0.5.0")
## indigo yaml-cpp : 0.5.0 /  hydro yaml-cpp : 0.3.0
  add_definitions("-DUSE_OLD_YAML")
endif()

# generate the dynamic_reconfigure config file
generate_dynamic_reconfigure_options(
  cfg/VoxelGridLargeScale.cfg
  cfg/PlaneSupportedCuboidEstimator.cfg
  cfg/InteractiveCuboidLikelihood.cfg
  cfg/ExtractParticlesTopNBase.cfg
  cfg/HeightmapMorphologicalFiltering.cfg
  cfg/HeightmapConverter.cfg
  cfg/HeightmapToPointCloud.cfg
  cfg/HeightmapTimeAccumulation.cfg
  cfg/GeometricConsistencyGrouping.cfg
  cfg/UniformSampling.cfg
  cfg/BorderEstimator.cfg
  cfg/HintedStickFinder.cfg
  cfg/HintedPlaneDetector.cfg
  cfg/PeopleDetection.cfg
  cfg/TorusFinder.cfg
  cfg/NormalDirectionFilter.cfg
  cfg/RegionGrowingMultiplePlaneSegmentation.cfg
  cfg/LineSegmentCollector.cfg
  cfg/LineSegmentDetector.cfg
  cfg/ParticleFilterTracking.cfg
  cfg/BilateralFilter.cfg
  cfg/ICPRegistration.cfg
  cfg/OrganizedPassThrough.cfg
  cfg/EuclideanClustering.cfg
  cfg/HSIColorFilter.cfg
  cfg/RGBColorFilter.cfg
  cfg/ImageRotate.cfg
  cfg/RegionGrowingSegmentation.cfg
  cfg/ColorBasedRegionGrowingSegmentation.cfg
  cfg/OrganizedMultiPlaneSegmentation.cfg
  cfg/MultiPlaneExtraction.cfg
  cfg/NormalEstimationIntegralImage.cfg
  cfg/EnvironmentPlaneModeling.cfg
  cfg/ColorHistogram.cfg
  cfg/ColorHistogramClassifier.cfg
  cfg/ColorHistogramFilter.cfg
  cfg/ColorHistogramMatcher.cfg
  cfg/ColorHistogramVisualizer.cfg
  cfg/ClusterPointIndicesDecomposer.cfg
  cfg/GridSampler.cfg
  cfg/OrganizedEdgeDetector.cfg
  cfg/EdgeDepthRefinement.cfg
  cfg/ParallelEdgeFinder.cfg
  cfg/EdgebasedCubeFinder.cfg
  cfg/MultiPlaneSACSegmentation.cfg
  cfg/BoundingBoxFilter.cfg
  cfg/ResizePointsPublisher.cfg
  cfg/RearrangeBoundingBox.cfg
  cfg/LINEMODDetector.cfg
  cfg/SupervoxelSegmentation.cfg
  cfg/FeatureRegistration.cfg
  cfg/FisheyeSphere.cfg
  cfg/MovingLeastSquareSmoothing.cfg
  cfg/OctreeVoxelGrid.cfg
  cfg/OctreeChangePublisher.cfg
  cfg/ExtractTopPolygonLikelihood.cfg
  cfg/PointcloudDatabaseServer.cfg
  cfg/TargetAdaptiveTracking.cfg
  cfg/PPFRegistration.cfg
  cfg/PrimitiveShapeClassifier.cfg
  cfg/Kinfu.cfg
  cfg/PointcloudScreenpoint.cfg
  cfg/OrganizedStatisticalOutlierRemoval.cfg
  )

find_package(OpenCV REQUIRED core imgproc)

include_directories(include ${catkin_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS} ${PCL_INCLUDE_DIRS})
link_directories(${catkin_LIBRARY_DIRS})
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -z defs")
endif()
macro(jsk_pcl_nodelet _nodelet_cpp _nodelet_class _single_nodelet_exec_name)
  jsk_nodelet(${_nodelet_cpp} ${_nodelet_class} ${_single_nodelet_exec_name}
    jsk_pcl_nodelet_sources jsk_pcl_nodelet_executables)
endmacro(jsk_pcl_nodelet _nodelet_cpp _nodelet_class _single_nodelet_exec_name)

macro(jsk_pcl_nodelet_upstream _upstream _nodelet_class _single_nodelet_exec_name)
  message("DEPRECATION WARNING: nodelet '${_nodelet_class}' is moved to upstream package '${_upstream}'.")
  jsk_nodelet(dummy.cpp ${_nodelet_class} ${_single_nodelet_exec_name} jsk_pcl_nodelet_dummy_sources jsk_pcl_nodelet_executables)
endmacro(jsk_pcl_nodelet_upstream _upstream _nodelet_class _single_nodelet_exec_name)

if ($ENV{TRAVIS_JOB_ID})
  add_definitions("-O0")
else ($ENV{TRAVIS_JOB_ID})
  add_definitions("-O2 -g")
endif ($ENV{TRAVIS_JOB_ID})

jsk_pcl_nodelet(src/depth_image_creator_nodelet.cpp
  "jsk_pcl/DepthImageCreator" "depth_image_creator")

# bench/ directory

get_property(dirs DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY INCLUDE_DIRECTORIES)
message("flags: ${CMAKE_CXX_FLAGS}")

catkin_package(
    DEPENDS PCL
    CATKIN_DEPENDS pcl_ros ${PCL_MSGS} jsk_recognition_utils
    INCLUDE_DIRS include
    LIBRARIES ${jsk_pcl_ros_install_libraries}
)

# download and install sample data
add_custom_target(${PROJECT_NAME}_install_sample_data ALL COMMAND python$ENV{ROS_PYTHON_VERSION} ${PROJECT_SOURCE_DIR}/scripts/install_sample_data.py)
# download and install trained data
add_custom_target(${PROJECT_NAME}_install_trained_data ALL COMMAND python$ENV{ROS_PYTHON_VERSION} ${PROJECT_SOURCE_DIR}/scripts/install_trained_data.py)

# From https://github.com/jsk-ros-pkg/jsk_recognition/pull/2345
# Install header files directly into ${CATKIN_PACKAGE_INCLUDE_DESTINATION}.
# If the package has setup.py and modules under src/${PROJECT_NAME}/,
# install python executables directly into ${CATKIN_PACKAGE_BIN_DESTINATION}.
# However, if it doesn't have setup.py, directories including python executables
# should be installed recursively into ${CATKIN_PACKAGE_SHARE_DESTINATION}.
# Also, other directories like 'launch' or 'sample' must be installed
# recursively into ${CATKIN_PACKAGE_SHARE_DESTINATION}.
# Be careful that 'launch' and 'launch/' are different: the former is directory
# and the latter is each content.
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
)
install(TARGETS ${jsk_pcl_ros_install_libraries}
  ${jsk_pcl_nodelet_executables}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)
