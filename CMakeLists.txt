
################################################################################
#
# CMakeLists.txt
#
# Author: Anthony Cabrera
# Contact: cabreraam AT ieee DOT org
# Description: CMakeLists.txt replacement for SNaCFE.x/SNBone
#
################################################################################

cmake_minimum_required(VERSION 3.18)

if(NOT PROJECT_NAME STREQUAL "test-suite")

	project(snbone)
	message(STATUS "${PROJECT_NAME} - snbone not in test-suite") 
	set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

else()
	message(STATUS "${PROJECT_NAME} - snbone in test-suite")
endif()

enable_language(Fortran)

add_subdirectory(src_fortran)
# TODO
#add_subdirectory(src_makemesh)
# TODO
#add_subdirectory(src_processmesh)

