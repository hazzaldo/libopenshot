####################### CMakeLists.txt (libopenshot) #########################
# @brief CMake build file for libopenshot (used to generate makefiles)
# @author Jonathan Thomas <jonathan@openshot.org>
#
# @section LICENSE
#
# Copyright (c) 2008-2019 OpenShot Studios, LLC
# <http://www.openshotstudios.com/>. This file is part of
# OpenShot Library (libopenshot), an open-source project dedicated to
# delivering high quality video editing and animation solutions to the
# world. For more information visit <http://www.openshot.org/>.
#
# OpenShot Library (libopenshot) is free software: you can redistribute it
# and/or modify it under the terms of the GNU Lesser General Public License
# as published by the Free Software Foundation, either version 3 of the
# License, or (at your option) any later version.
#
# OpenShot Library (libopenshot) is distributed in the hope that it will be
# useful, but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with OpenShot Library. If not, see <http://www.gnu.org/licenses/>.
################################################################################

# Collect and display summary of options/dependencies
include(FeatureSummary)

include(GNUInstallDirs)

# Enable IN_LIST in older CMake
if (POLICY CMP0057)
  cmake_policy(SET CMP0057 NEW)
endif()

# Juce requires either DEBUG or NDEBUG to be defined on MacOS.
# -DNDEBUG is set by cmake for all release configs, so add
# -DDEBUG for debug builds. We'll do this for all OSes, even
# though only MacOS requires it.
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DDEBUG")
# Make sure we've picked some build type, default to debug
if(NOT DEFINED CMAKE_BUILD_TYPE OR CMAKE_BUILD_TYPE STREQUAL "")
    set(CMAKE_BUILD_TYPE "Debug")
endif()

################ WINDOWS ##################
# Set some compiler options for Windows
# required for libopenshot-audio headers
if (WIN32)
  add_definitions( -DIGNORE_JUCE_HYPOT=1 )
  set(CMAKE_CXX_FLAGS " ${CMAKE_CXX_FLAGS} -include cmath")
endif()

if (APPLE)
	# If you still get errors compiling with GCC 4.8, mac headers need to be patched: http://hamelot.co.uk/programming/osx-gcc-dispatch_block_t-has-not-been-declared-invalid-typedef/
	set_property(GLOBAL PROPERTY JUCE_MAC "JUCE_MAC")
	set(EXTENSION "mm")
	set(JUCE_PLATFORM_SPECIFIC_DIR build/macosx/platform_specific_code)
	set(JUCE_PLATFORM_SPECIFIC_LIBRARIES "-framework Carbon -framework Cocoa -framework CoreFoundation -framework CoreAudio -framework CoreMidi -framework IOKit -framework AGL -framework AudioToolbox -framework QuartzCore -lobjc -framework Accelerate")

	# Prevent compiling with __cxx11
	add_definitions(-D_GLIBCXX_USE_CXX11_ABI=0)
endif()

################ IMAGE MAGICK ##################
# Set the Quantum Depth that ImageMagick was built with (default to 16 bits)
if(ENABLE_MAGICK)
  IF (MAGICKCORE_QUANTUM_DEPTH)
    add_definitions( -DMAGICKCORE_QUANTUM_DEPTH=${MAGICKCORE_QUANTUM_DEPTH} )
  ELSE (MAGICKCORE_QUANTUM_DEPTH)
    add_definitions( -DMAGICKCORE_QUANTUM_DEPTH=16 )
  ENDIF (MAGICKCORE_QUANTUM_DEPTH)
  IF (MAGICKCORE_HDRI_ENABLE)
    add_definitions( -DMAGICKCORE_HDRI_ENABLE=${MAGICKCORE_HDRI_ENABLE} )
  ELSE (MAGICKCORE_HDRI_ENABLE)
    add_definitions( -DMAGICKCORE_HDRI_ENABLE=0 )
  ENDIF (MAGICKCORE_HDRI_ENABLE)

  # Find the ImageMagick++ library
  find_package(ImageMagick COMPONENTS Magick++ MagickWand MagickCore)
  if (ImageMagick_FOUND)
    # Include ImageMagick++ headers (needed for compile)
    include_directories(${ImageMagick_INCLUDE_DIRS})

    # define a global var (used in the C++)
    add_definitions( -DUSE_IMAGEMAGICK=1 )
    list(APPEND CMAKE_SWIG_FLAGS "-DUSE_IMAGEMAGICK=1")

    set(HAVE_IMAGEMAGICK TRUE CACHE BOOL "Building with ImageMagick support" FORCE)
    mark_as_advanced(HAVE_IMAGEMAGICK)
  endif()
endif()

################# LIBOPENSHOT-AUDIO ###################
# Find JUCE-based openshot Audio libraries
find_package(OpenShotAudio 0.2.0 REQUIRED)

# Include Juce headers (needed for compile)
include_directories(${LIBOPENSHOT_AUDIO_INCLUDE_DIRS})

################# BLACKMAGIC DECKLINK ###################
# Find BlackMagic DeckLinkAPI libraries
if (ENABLE_BLACKMAGIC)

  find_package(BlackMagic)

  if (BLACKMAGIC_FOUND)
    # Include Blackmagic headers (needed for compile)
    include_directories(${BLACKMAGIC_INCLUDE_DIR})

    # define a global var (used in the C++)
    add_definitions( -DUSE_BLACKMAGIC=1 )
    list(APPEND CMAKE_SWIG_FLAGS "-DUSE_BLACKMAGIC=1")
  endif()

endif()

###############  PROFILING  #################
#set(PROFILER "/usr/lib/libprofiler.so.0.3.2")
#set(PROFILER "/usr/lib/libtcmalloc.so.4")

if(CMAKE_VERSION VERSION_LESS 3.3)
  # IWYU wasn't supported internally in 3.2
  set(ENABLE_IWYU FALSE)
endif()

if(ENABLE_IWYU)
	find_program(IWYU_PATH NAMES "iwyu"
		DOC "include-what-you-use source code scanner executable")
	if(IWYU_PATH)
		if(IWYU_OPTS)
			separate_arguments(IWYU_OPTS)
			list(APPEND _iwyu_opts "-Xiwyu" ${IWYU_OPTS})
		endif()
		set(CMAKE_CXX_INCLUDE_WHAT_YOU_USE ${IWYU_PATH} ${_iwyu_opts})
	else()
		set(ENABLE_IWYU FALSE)
	endif()
endif()
add_feature_info("IWYU (include-what-you-use)" ENABLE_IWYU "Scan all source files with 'iwyu'")

# Main library sources
set(OPENSHOT_SOURCES
  AudioBufferSource.cpp
  AudioReaderSource.cpp
  AudioResampler.cpp
  CacheBase.cpp
  CacheDisk.cpp
  CacheMemory.cpp
  ChunkReader.cpp
  ChunkWriter.cpp
  Color.cpp
  Clip.cpp
  ClipBase.cpp
  Coordinate.cpp
  CrashHandler.cpp
  DummyReader.cpp
  ReaderBase.cpp
  RendererBase.cpp
  WriterBase.cpp
  EffectBase.cpp
  EffectInfo.cpp
  FFmpegReader.cpp
  FFmpegWriter.cpp
  Fraction.cpp
  Frame.cpp
  FrameMapper.cpp
  Json.cpp
  KeyFrame.cpp
  OpenShotVersion.cpp
  ZmqLogger.cpp
  PlayerBase.cpp
  Point.cpp
  Profiles.cpp
  QtHtmlReader.cpp
  QtImageReader.cpp
  QtPlayer.cpp
  QtTextReader.cpp
  Settings.cpp
  Timeline.cpp)

# Video effects
set(EFFECTS_SOURCES
  effects/Bars.cpp
  effects/Blur.cpp
  effects/Brightness.cpp
  effects/ChromaKey.cpp
  effects/ColorShift.cpp
  effects/Crop.cpp
  effects/Deinterlace.cpp
  effects/Hue.cpp
  effects/Mask.cpp
  effects/Negate.cpp
  effects/Pixelate.cpp
  effects/Saturation.cpp
  effects/Shift.cpp
  effects/Wave.cpp)

# Qt video player components
set(QT_PLAYER_SOURCES
  Qt/AudioPlaybackThread.cpp
  Qt/PlayerDemo.cpp
  Qt/PlayerPrivate.cpp
  Qt/VideoCacheThread.cpp
  Qt/VideoPlaybackThread.cpp
  Qt/VideoRenderer.cpp
  Qt/VideoRenderWidget.cpp)

# Get list of MOC'able headers
file(GLOB_RECURSE OPENSHOT_QT_HEADERS ${CMAKE_SOURCE_DIR}/include/Qt/*.h)

# Disable RPATH
set(CMAKE_MACOSX_RPATH 0)

############### CREATE LIBRARY #################
# Create shared openshot library
add_library(openshot SHARED)

target_sources(openshot PRIVATE
  ${OPENSHOT_SOURCES}
  ${EFFECTS_SOURCES}
  ${QT_PLAYER_SOURCES}
  ${OPENSHOT_QT_HEADERS}
  )

# Set SONAME and other library properties
set_target_properties(openshot PROPERTIES
  AUTOMOC ON
  VERSION ${PROJECT_VERSION}
  SOVERSION ${PROJECT_SO_VERSION}
  INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib"
  )

# Add optional ImageMagic-dependent sources
if(ImageMagick_FOUND)
  target_sources(openshot PRIVATE
    ImageReader.cpp
    ImageWriter.cpp
    TextReader.cpp)
endif()

# BlackMagic related files
if(BLACKMAGIC_FOUND)
  target_sources(openshot PRIVATE
    DecklinkInput.cpp
    DecklinkReader.cpp
    DecklinkOutput.cpp
    DecklinkWriter.cpp)
endif()

# Location of our includes, both internally and when installed
target_include_directories(openshot
  PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${CMAKE_BINARY_DIR}/include
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/libopenshot>)


################### JSONCPP #####################
# Include jsoncpp headers (needed for JSON parsing)
if (USE_SYSTEM_JSONCPP)
  message(STATUS "Looking for system jsoncpp")
  # Either an installed config or our find module will
  # create the IMPORTED target jsoncpp_lib if successful
  find_package(jsoncpp)
endif ()

if (NOT jsoncpp_FOUND AND NOT DISABLE_BUNDLED_JSONCPP)
  message(STATUS "Using embedded jsoncpp (not found or USE_SYSTEM_JSONCPP disabled)")
  if (NOT TARGET jsoncpp_lib)
    add_library(jsoncpp_lib INTERFACE)
    target_include_directories(jsoncpp_lib INTERFACE
      "${PROJECT_SOURCE_DIR}/thirdparty/jsoncpp")
    target_sources(jsoncpp_lib INTERFACE "${PROJECT_SOURCE_DIR}/thirdparty/jsoncpp/jsoncpp.cpp")
    # Because this satisfies the requirement, an installed JsonCpp is optional
    set_package_properties(jsoncpp PROPERTIES TYPE OPTIONAL)
  endif ()
  add_feature_info("jsoncpp (embedded)" TRUE "jsoncpp will be compiled from the bundled sources")
endif ()

if (jsoncpp_FOUND)
  # JsonCpp is actually required, even though we probe for it optionally
  # (This tells feature_summary() to bail if it's not found, later)
  set_package_properties(jsoncpp PROPERTIES TYPE REQUIRED)
endif ()

# If we found any usable JsonCpp, use it. Otherwise, bail.
if (TARGET jsoncpp_lib)
  target_link_libraries(openshot PUBLIC jsoncpp_lib)
endif ()

################# QT5 ###################
# Find QT5 libraries
set(_qt_components Core Gui Widgets)
find_package(Qt5 COMPONENTS ${_qt_components} REQUIRED)

foreach(_qt_comp IN LISTS _qt_components)
  if(TARGET Qt5::${_qt_comp})
    target_link_libraries(openshot PUBLIC Qt5::${_qt_comp})
  endif()
endforeach()

# Keep track of Qt version, to embed in our version header
set(QT_VERSION_STR ${Qt5Core_VERSION_STRING} CACHE STRING "Qt version linked with" FORCE)
mark_as_advanced(QT_VERSION_STR)

################### FFMPEG #####################
# Find FFmpeg libraries (used for video encoding / decoding)
find_package(FFmpeg REQUIRED COMPONENTS avcodec avdevice avformat avutil swscale)

set(all_comps avcodec avdevice avformat avfilter avutil postproc swscale swresample avresample)
set(version_comps avcodec avformat avutil)

foreach(ff_comp IN LISTS all_comps)
  if(TARGET FFmpeg::${ff_comp})
    target_link_libraries(openshot PUBLIC FFmpeg::${ff_comp})
    # Keep track of some FFmpeg lib versions, to embed in our version header
    if(${ff_comp} IN_LIST version_comps)
      string(TOUPPER ${ff_comp} v_name)
      set(${v_name}_VERSION_STR ${${ff_comp}_VERSION} CACHE STRING "${ff_comp} version used" FORCE)
      mark_as_advanced(${v_name}_VERSION_STR)
    endif()
  endif()

endforeach()

################### OPENMP #####################
# Check for OpenMP (used for multi-core processing)

# OpenMP is required by FFmpegReader/Writer
find_package(OpenMP REQUIRED)

if(NOT TARGET OpenMP::OpenMP_CXX)
    # Older CMake versions (< 3.9) don't create find targets.
    add_library(OpenMP_TARGET INTERFACE)
    add_library(OpenMP::OpenMP_CXX ALIAS OpenMP_TARGET)
    target_compile_options(OpenMP_TARGET INTERFACE ${OpenMP_CXX_FLAGS})
    target_link_libraries(OpenMP_TARGET INTERFACE ${OpenMP_CXX_FLAGS})
endif()

target_link_libraries(openshot PUBLIC OpenMP::OpenMP_CXX)

################### ZEROMQ #####################
# Find ZeroMQ library (used for socket communication & logging)
find_package(ZeroMQ REQUIRED) # Creates libzmq target

# Some platforms package the header-only cppzmq C++ bindings separately,
# others (Ubuntu) bundle them in with libzmq itself
find_package(cppzmq QUIET) # Creates cppzmq target

# Link ZeroMQ library
if (TARGET libzmq)
  target_link_libraries(openshot PUBLIC libzmq)
endif()
# Include cppzmq headers, if not bundled into libzmq
if (TARGET cppzmq)
  target_link_libraries(openshot PUBLIC cppzmq)
endif()

################### RESVG #####################
# Migrate some legacy variable names
if(DEFINED RESVGDIR AND NOT DEFINED RESVG_ROOT)
  set(RESVG_ROOT ${RESVGDIR})
endif()
if(DEFINED ENV{RESVGDIR} AND NOT DEFINED RESVG_ROOT)
  set(RESVG_ROOT $ENV{RESVGDIR})
endif()

# Find resvg library (used for rendering svg files)
find_package(RESVG)

# Include resvg headers (optional SVG library)
if (TARGET RESVG::resvg)
  #include_directories(${RESVG_INCLUDE_DIRS})
  target_link_libraries(openshot PUBLIC RESVG::resvg)

  target_compile_definitions(openshot PUBLIC "-DUSE_RESVG=1")
  list(APPEND CMAKE_SWIG_FLAGS "-DUSE_RESVG=1")

  set(HAVE_RESVG TRUE CACHE BOOL "Building with Resvg support" FORCE)
  mark_as_advanced(HAVE_RESVG)

endif()

###############  LINK LIBRARY  #################
# Link remaining dependency libraries
target_link_libraries(openshot PUBLIC
  ${LIBOPENSHOT_AUDIO_LIBRARIES}
  ${PROFILER})

if(ImageMagick_FOUND)
  target_link_libraries(openshot PUBLIC ${ImageMagick_LIBRARIES})
endif()

if(BLACKMAGIC_FOUND)
  target_link_libraries(openshot PUBLIC ${BLACKMAGIC_LIBRARY_DIR})
endif()

if(WIN32)
  # Required for exception handling on Windows
  target_link_libraries(openshot PUBLIC "imagehlp" "dbghelp" )
endif()


############### CLI EXECUTABLES ################
# Create test executable
add_executable(openshot-example examples/Example.cpp)

# Define path to test input files
set(TEST_MEDIA_PATH "${PROJECT_SOURCE_DIR}/src/examples/")
if (WIN32)
  string(REPLACE "/" "\\\\" TEST_MEDIA_PATH TEST_MEDIA_PATH)
endif()
target_compile_definitions(openshot-example PRIVATE
	-DTEST_MEDIA_PATH="${TEST_MEDIA_PATH}" )

# Link test executable to the new library
target_link_libraries(openshot-example openshot)

add_executable(openshot-html-test examples/ExampleHtml.cpp)
target_link_libraries(openshot-html-test openshot Qt5::Gui)

############### PLAYER EXECUTABLE ################
# Create test executable
add_executable(openshot-player Qt/demo/main.cpp)

# Link test executable to the new library
target_link_libraries(openshot-player openshot)

############### TEST BLACKMAGIC CAPTURE APP ################
if (BLACKMAGIC_FOUND)
  # Create test executable
  add_executable(openshot-blackmagic
    examples/ExampleBlackmagic.cpp)

  # Link test executable to the new library
  target_link_libraries(openshot-blackmagic openshot)
endif()

############### INCLUDE SWIG BINDINGS ################
add_subdirectory(bindings)

############### INSTALL HEADERS & LIBRARY ################

# Install primary library
install(TARGETS openshot
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(DIRECTORY ${CMAKE_SOURCE_DIR}/include/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libopenshot
  FILES_MATCHING PATTERN "*.h")

############### CPACK PACKAGING ##############
if(MINGW)
  set(CPACK_GENERATOR "NSIS")
endif()
if(UNIX AND NOT APPLE)
  set(CPACK_GENERATOR "DEB")
endif()
#if(UNIX AND APPLE)
#  set(CPACK_GENERATOR "DragNDrop")
#endif()
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Jonathan Thomas") #required

include(CPack)
