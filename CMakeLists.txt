cmake_minimum_required(VERSION 2.8.3)
project(bta_ros)

set(2DSENSOR)
## check required modules
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake_modules")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D PLAT_LINUX" )
find_package(bta REQUIRED)

if (2DSENSOR)
	find_package(GStreamer REQUIRED )
	find_package(GLIB REQUIRED)
	find_package(GObject REQUIRED)
endif ()
#find_package(Boost REQUIRED COMPONENTS system thread)

#find_package(OpenCV REQUIRED)

## Find catkin macros and libraries
find_package(catkin REQUIRED COMPONENTS
  dynamic_reconfigure
  roscpp
  sensor_msgs
  std_msgs
  pcl_ros
  pcl_conversions
  image_transport
  camera_info_manager
  nodelet
)

generate_dynamic_reconfigure_options(
  cfg/bta_ros.cfg
)

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if you package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
  INCLUDE_DIRS include
  LIBRARIES bta_ros sensor2d
 CATKIN_DEPENDS dynamic_reconfigure roscpp sensor_msgs std_msgs pcl_ros pcl_conversions image_transport camera_info_manager nodelet
  DEPENDS bta GStreamer GLIB GObject
)

###########
## Build ##
###########

include_directories(include
  ${catkin_INCLUDE_DIRS}
  ${bta_INCLUDE_DIR}
  ${PCL_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)



add_library(bta_ros
src/bta_ros.cpp
)
target_link_libraries(bta_ros bta_ros ${bta_LIBRARIES} ${catkin_LIBRARIES} )
add_dependencies(bta_ros ${PROJECT_NAME}_gencfg) 

add_library(BtaRosNodelet
src/bta_ros_nodelet.cpp
)
target_link_libraries(BtaRosNodelet bta_ros ${catkin_LIBRARIES} ${Boost_LIBRARIES} )

add_executable(bta_ros_node src/bta_ros_node.cpp)

target_link_libraries(bta_ros_node bta_ros ${catkin_LIBRARIES} )

if (2DSENSOR)
	include_directories(include
	  ${GSTREAMER_INCLUDE_DIRS} 
	  ${GSTREAMER_APP_INCLUDE_DIRS} 
	  ${GLIB_INCLUDE_DIRS} 
	  ${GOBJECT_INCLUDE_DIRS}
	)

	add_library(sensor2d
	src/sensor2D.cpp
	)
	target_link_libraries(sensor2d 
		sensor2d 
		${GOBJECT_LIBRARIES} 
		${GSTREAMER_LIBRARIES} 
		${GSTREAMER_APP_LIBRARIES} 
		${GLIB_LIBRARIES} 
		${catkin_LIBRARIES} 
	)

	add_library(Sensor2DNodelet
	src/sensor2D_nodelet.cpp
	)
	target_link_libraries(Sensor2DNodelet 
		sensor2d 
		${catkin_LIBRARIES} 
		${Boost_LIBRARIES} 
	)

	add_executable(sensor2d_node src/sensor2d_node.cpp)

	target_link_libraries(sensor2d_node sensor2d ${catkin_LIBRARIES} )
endif ()


#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables and/or libraries for installation
 install(TARGETS bta_ros bta_ros_node BtaRosNodelet
   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
 )
if (2DSENSOR)
 install(TARGETS sensor2d Sensor2DNodelet sensor2d_node
   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
 )
endif ()
## Mark cpp header files for installation
 install(DIRECTORY include/${PROJECT_NAME}/
   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
   PATTERN ".svn" EXCLUDE
 )

## Mark other files for installation (e.g. launch and bag files, etc.)
	install(FILES nodelet_plugins.xml
		DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
	)


#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
# catkin_add_gtest(${PROJECT_NAME}-test test/test_sentis_m100.cpp)
# if(TARGET ${PROJECT_NAME}-test)
#   target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
# endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)
