# Copyright 2011,2012 Free Software Foundation, Inc.
#
# This file is part of GNU Radio
#
# GNU Radio is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3, or (at your option)
# any later version.
#
# GNU Radio is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with GNU Radio; see the file COPYING.  If not, write to
# the Free Software Foundation, Inc., 51 Franklin Street,
# Boston, MA 02110-1301, USA.

########################################################################
# Setup library
########################################################################
include(GrPlatform) #define LIB_SUFFIX

########################################################################
# Find boost
########################################################################
if(UNIX AND EXISTS "/usr/lib")
    list(APPEND BOOST_LIBRARYDIR "/usr/lib")
endif(UNIX AND EXISTS "/usr/lib")
if(UNIX AND EXISTS "/usr/lib64")
    list(APPEND BOOST_LIBRARYDIR "/usr/lib64") #fedora 64-bit fix
endif(UNIX AND EXISTS "/usr/lib64")
set(Boost_ADDITIONAL_VERSIONS
    "1.35.0" "1.35" "1.36.0" "1.36" "1.37.0" "1.37" "1.38.0" "1.38" "1.39.0" "1.39"
    "1.40.0" "1.40" "1.41.0" "1.41" "1.42.0" "1.42" "1.43.0" "1.43" "1.44.0" "1.44"
    "1.45.0" "1.45" "1.46.0" "1.46" "1.47.0" "1.47" "1.48.0" "1.48" "1.49.0" "1.49"
    "1.50.0" "1.50" "1.51.0" "1.51" "1.52.0" "1.52" "1.53.0" "1.53" "1.54.0" "1.54"
    "1.55.0" "1.55" "1.56.0" "1.56" "1.57.0" "1.57" "1.58.0" "1.58" "1.59.0" "1.59"
    "1.60.0" "1.60" "1.61.0" "1.61" "1.62.0" "1.62" "1.63.0" "1.63" "1.64.0" "1.64"
    "1.65.0" "1.65" "1.66.0" "1.66" "1.67.0" "1.67" "1.68.0" "1.68" "1.69.0" "1.69"
    "1.74.0" "1.74" "1.81.0" "1.81"
)
find_package(Boost "1.74" REQUIRED COMPONENTS filesystem system)
add_definitions(-DBOOST_NO_CXX11_SCOPED_ENUMS)

include_directories(${Boost_INCLUDE_DIR})
link_directories(${Boost_LIBRARY_DIRS})

list(APPEND fsk4_sources
    demod_ff_impl.cc
    generic_f_impl.cc
    apco25_f_impl.cc
    rdlap_f_impl.cc
)

set(fsk4_sources "${fsk4_sources}" PARENT_SCOPE)
if(NOT fsk4_sources)
	MESSAGE(STATUS "No C++ sources... skipping lib/")
	return()
endif(NOT fsk4_sources)

add_library(gnuradio-fsk4 SHARED ${fsk4_sources})
message("adding boost libraries ${Boost_LIBRARIES}")
target_link_libraries(gnuradio-fsk4 ${Boost_LIBRARIES} ${GNURADIO_ALL_LIBRARIES})
set_target_properties(gnuradio-fsk4 PROPERTIES DEFINE_SYMBOL "gnuradio_fsk4_EXPORTS")

if(APPLE)
    set_target_properties(gnuradio-fsk4 PROPERTIES
        INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib"
    )
endif(APPLE)

########################################################################
# Install built library files
########################################################################
install(TARGETS gnuradio-fsk4
    LIBRARY DESTINATION lib${LIB_SUFFIX} # .so/.dylib file
    ARCHIVE DESTINATION lib${LIB_SUFFIX} # .lib file
    RUNTIME DESTINATION bin              # .dll file
)

########################################################################
# Build and register unit test
########################################################################
include(GrTest)

include_directories(${CPPUNIT_INCLUDE_DIRS})

list(APPEND test_fsk4_sources
    ${CMAKE_CURRENT_SOURCE_DIR}/test_fsk4.cc
    ${CMAKE_CURRENT_SOURCE_DIR}/qa_fsk4.cc
)

add_executable(test-fsk4 ${test_fsk4_sources})

target_link_libraries(
  test-fsk4
  ${GNURADIO_RUNTIME_LIBRARIES}
  ${Boost_LIBRARIES}
  ${CPPUNIT_LIBRARIES}
  gnuradio-fsk4
)

GR_ADD_TEST(test_fsk4 test-fsk4)
