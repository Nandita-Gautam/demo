### VERSION INFO
set(OpenPose_VERSION_MAJOR 1)
set(OpenPose_VERSION_MINOR 7)
set(OpenPose_VERSION_PATCH 0)
set(OpenPose_VERSION ${OpenPose_VERSION_MAJOR}.${OpenPose_VERSION_MINOR}.${OpenPose_VERSION_PATCH})

### OS-DEPENDENT FLAGS
if (WIN32)
  set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
else()
  set(CMAKE_MACOSX_RPATH 1)
endif()

### CMAKE HEADERS
if (${CMAKE_VERSION} VERSION_GREATER 3.9.0)
  cmake_policy(SET CMP0048 NEW)
  project(OpenPose VERSION ${OpenPose_VERSION})
elseif (${CMAKE_VERSION} VERSION_GREATER 3.0.0)
  cmake_policy(SET CMP0048 NEW)
  project(OpenPose VERSION ${OpenPose_VERSION})
else (${CMAKE_VERSION} VERSION_GREATER 3.9.0)
  project(OpenPose)
endif (${CMAKE_VERSION} VERSION_GREATER 3.9.0)
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)

### Extra functionality
include(cmake/Utils.cmake)
if (NOT WIN32 AND NOT UNIX AND NOT APPLE)
  message(FATAL_ERROR "Unsupported operating system. Only Windows, Mac and Unix systems supported.")
endif (NOT WIN32 AND NOT UNIX AND NOT APPLE)

### CMAKE_BUILD_TYPE
set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build.")
set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Release" "Debug" "MinSizeRel" "RelWithDebInfo")

### FLAGS
if (WIN32)
  set(CMAKE_SUPPRESS_REGENERATION ON)
  string(REPLACE "/D_WINDOWS" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  string(REPLACE "/DWIN32" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
  string(REPLACE "/W3" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  string(REPLACE "/GR" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  string(REPLACE "/O2" "/Ox" CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")
  string(REPLACE "/Ob2" "" CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /Ot /Oi /Gy /Z7")
  set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS_RELEASE} /LTCG:incremental /OPT:REF /OPT:ICF")
elseif (UNIX)
  add_definitions(-std=c++11)
  set(CMAKE_CXX_FLAGS_RELEASE "-O3")
elseif (APPLE)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  set(CMAKE_CXX_FLAGS_RELEASE "-O3")
endif (WIN32)

### PROJECT OPTIONS
set(DL_FRAMEWORK CAFFE CACHE STRING "Select Deep Learning Framework.")
if (WIN32)
  set_property(CACHE DL_FRAMEWORK PROPERTY STRINGS CAFFE)
else (WIN32)
  set_property(CACHE DL_FRAMEWORK PROPERTY STRINGS CAFFE NV_CAFFE)
endif (WIN32)

include(CMakeDependentOption)
if (${DL_FRAMEWORK} MATCHES "CAFFE" OR ${DL_FRAMEWORK} MATCHES "NV_CAFFE")
  CMAKE_DEPENDENT_OPTION(BUILD_CAFFE "Build Caffe as part of OpenPose." ON "DL_FRAMEWORK" ON)
  add_definitions(-DUSE_CAFFE)
  if (${DL_FRAMEWORK} MATCHES "NV_CAFFE")
    MESSAGE(STATUS "Using NVIDIA NVCaffe")
    add_definitions(-DNV_CAFFE)
  endif (${DL_FRAMEWORK} MATCHES "NV_CAFFE")
endif (${DL_FRAMEWORK} MATCHES "CAFFE" OR ${DL_FRAMEWORK} MATCHES "NV_CAFFE")

set(GPU_MODE CPU_ONLY CACHE STRING "Select the acceleration GPU library or CPU otherwise.")
set_property(CACHE GPU_MODE PROPERTY STRINGS CUDA OPENCL CPU_ONLY)

if (${GPU_MODE} MATCHES "CPU_ONLY")
  add_definitions(-DUSE_CPU_ONLY)
  message(STATUS "Building CPU Only.")
  add_definitions(-DCPU_ONLY)
endif (${GPU_MODE} MATCHES "CPU_ONLY")

if (${GPU_MODE} MATCHES "OPENCL")
  find_package(OpenCL)
endif (${GPU_MODE} MATCHES "OPENCL")

if (${USE_MKL})
  add_definitions(-DUSE_MKL)
  message(STATUS "Building with MKL support.")
endif (${USE_MKL})

if (PROFILER_ENABLED)
  add_definitions(-DPROFILER_ENABLED)
endif (PROFILER_ENABLED)

option(WITH_3D_RENDERER "Add OpenPose 3D renderer module (it requires FreeGLUT library)." OFF)
if (UNIX AND NOT APPLE)
  option(WITH_CERES "Add Ceres support for advanced 3-D reconstruction." OFF)
endif (UNIX AND NOT APPLE)
option(WITH_FLIR_CAMERA "Add FLIR (formerly Point Grey) camera code (requires Spinnaker SDK already installed)." OFF)

option(WITH_OPENCV_WITH_OPENGL "Much faster GUI display, but you must also enable OpenGL support in OpenCV by configuring OpenCV using CMake with WITH_OPENGL=ON flag." OFF)

if (WIN32 OR APPLE)
  set(WITH_EIGEN NONE CACHE STRING "Select the Eigen mode: NONE if not required, BUILD to let OpenPose download it, or FIND to let CMake find it (e.g., if you installed it manually).")
  set_property(CACHE WITH_EIGEN PROPERTY STRINGS NONE BUILD FIND)
elseif (UNIX AND NOT APPLE)
  set(WITH_EIGEN NONE CACHE STRING "Select the Eigen mode: NONE if not required, BUILD to let OpenPose download it, or FIND to let CMake find it (e.g., if you installed it manually or used `sudo apt-get` to install it).")
  set_property(CACHE WITH_EIGEN PROPERTY STRINGS NONE BUILD FIND)
endif (WIN32 OR APPLE)

option(DOWNLOAD_BODY_25_MODEL "Download body 25-keypoint (body COCO and 6-keypoint foot) model." ON)
option(DOWNLOAD_BODY_COCO_MODEL "Download body 18-keypoint COCO model." OFF)
option(DOWNLOAD_BODY_MPI_MODEL "Download body 15-keypoint MPI model." OFF)
option(DOWNLOAD_FACE_MODEL "Download face model." ON)
option(DOWNLOAD_HAND_MODEL "Download hand model." ON)

option(BUILD_EXAMPLES "Build OpenPose examples." ON)
option(BUILD_DOCS "Build OpenPose documentation." OFF)
option(BUILD_PYTHON "Build OpenPose python." ON)
if (WIN32)
  option(BUILD_BIN_FOLDER "Copy all required 3rd-party DLL files into {build_directory}/bin. Disable to save some memory." ON)
endif (WIN32)

option(BUILD_UNITY_SUPPORT "Build OpenPose as a Unity plugin." OFF)
option(BUILD_SHARED_LIBS "Build as shared lib." ON)
option(PROFILER_ENABLED "If enabled, OpenPose will be able to print out speed information at runtime." OFF)

if (${GPU_MODE} MATCHES "OPENCL" OR (UNIX OR APPLE))
  unset(CMAKE_THREAD_LIBS_INIT CACHE)
  find_package(Threads)
endif (${GPU_MODE} MATCHES "OPENCL" OR (UNIX OR APPLE))

set(DOWNLOAD_SERVER "http://posefs1.perception.cs.cmu.edu/OpenPose/" CACHE STRING "Server from which the models and 3rdparty libraries will be downloaded from.")
mark_as_advanced(DOWNLOAD_SERVER)

### FIND REQUIRED PACKAGES
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules")

if (${GPU_MODE} MATCHES "OPENCL")
  find_package(OpenCL)
endif (${GPU_MODE} MATCHES "OPENCL")

if (WITH_OPENCV_WITH_OPENGL)
  add_definitions(-DUSE_OPENCV_WITH_OPENGL)
endif (WITH_OPENCV_WITH_OPENGL)
if (WITH_3D_RENDERER)
  add_definitions(-DUSE_3D_RENDERER)
endif (WITH_3D_RENDERER)
if (WITH_CERES)
  add_definitions(-DUSE_CERES)
endif (WITH_CERES)
if (WITH_FLIR_CAMERA)
  add_definitions(-DUSE_FLIR_CAMERA)
endif (WITH_FLIR_CAMERA)

if (WIN32)
  if ("${CMAKE_VERSION}" VERSION_GREATER 3.6.3)
    set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT OpenPoseDemo)
  endif ("${CMAKE_VERSION}" VERSION_GREATER 3.6.3)
  set_property(GLOBAL PROPERTY USE_FOLDERS ON)
  set(FIND_LIB_PREFIX ${CMAKE_SOURCE_DIR}/3rdparty/windows)

  message(STATUS "Downloading windows dependencies...")
  set(OP_WIN_URL "${DOWNLOAD_SERVER}3rdparty/windows")
  set(OP_WIN_DIR "${CMAKE_SOURCE_DIR}/3rdparty/windows")

  download_zip("opencv_420_v14_15_2020_04_04.zip" ${OP_WIN_URL} ${FIND_LIB_PREFIX} 59da628c17fe0f4755d22ff86dbb8c06)
  download_zip("caffe3rdparty_16_2020_11_14.zip" ${OP_WIN_URL} ${FIND_LIB_PREFIX} 83C17DC4473985B6AB3A29FD9AB93409)
  if (${GPU_MODE} MATCHES "OPENCL")
    download_zip("caffe_opencl_2018_02_13.zip" ${OP_WIN_URL} ${FIND_LIB_PREFIX} 3ac3e1acf5ee6a4e57920be73053067a)
  elseif (${GPU_MODE} MATCHES "CPU_ONLY")
    download_zip("caffe_cpu_2018_05_27.zip" ${OP_WIN_URL} ${FIND_LIB_PREFIX} 87E8401B6DFBAC5B8E909DD20E3B3390)
  else (${GPU_MODE} MATCHES "OPENCL")
    download_zip("caffe_16_2020_11_14.zip" ${OP_WIN_URL} ${FIND_LIB_PREFIX} 33030DBEC9911533566E239199BB4716)
  endif (${GPU_MODE} MATCHES "OPENCL")
  if (WITH_3D_RENDERER)
    download_zip("freeglut_2018_01_14.zip" ${OP_WIN_URL} ${FIND_LIB_PREFIX} BB182187285E06880F0EDE3A39530091)
  endif (WITH_3D_RENDERER)
  message(STATUS "Windows dependencies downloaded.")

  set(OPENCV_VS_VERSION "15")
  find_library(OpenCV_LIBS_RELEASE opencv_world420 HINTS ${FIND_LIB_PREFIX}/opencv/x64/vc${OPENCV_VS_VERSION}/lib)
  find_library(OpenCV_LIBS_DEBUG opencv_world420d HINTS ${FIND_LIB_PREFIX}/opencv/x64/vc${OPENCV_VS_VERSION}/lib)
  find_library(GFLAGS_LIBRARY_RELEASE gflags HINTS ${FIND_LIB_PREFIX}/caffe3rdparty/lib)
  find_library(GFLAGS_LIBRARY_DEBUG gflagsd HINTS ${FIND_LIB_PREFIX}/caffe3rdparty/lib)
  find_library(GLOG_LIBRARY_RELEASE glog HINTS ${FIND_LIB_PREFIX}/caffe3rdparty/lib)
  find_library(GLOG_LIBRARY_DEBUG glogd HINTS ${FIND_LIB_PREFIX}/caffe3rdparty/lib)

  if (BUILD_CAFFE)
    unset(Caffe_INCLUDE_DIRS CACHE)
    unset(Caffe_LIB_RELEASE CACHE)
    unset(Caffe_Proto_LIB_RELEASE CACHE)
  endif (BUILD_CAFFE)

  if (${GPU_MODE} MATCHES "OPENCL")
    set(VCXPROJ_FILE_GPU_MODE "_CL")
    find_library(Caffe_LIB_RELEASE caffe HINTS ${FIND_LIB_PREFIX}/caffe_opencl/lib)
    find_library(Caffe_Proto_LIB_RELEASE caffeproto HINTS ${FIND_LIB_PREFIX}/caffe_opencl/lib)
    find_library(Caffe_LIB_DEBUG caffe-d HINTS ${FIND_LIB_PREFIX}/caffe_opencl/lib)
    find_library(Caffe_Proto_LIB_DEBUG caffeproto-d HINTS ${FIND_LIB_PREFIX}/caffe_opencl/lib)
  elseif (${GPU_MODE} MATCHES "CPU_ONLY")
    set(VCXPROJ_FILE_GPU_MODE "_CPU")
    find_library(Caffe_LIB_RELEASE caffe HINTS ${FIND_LIB_PREFIX}/caffe_cpu/lib)
    find_library(Caffe_Proto_LIB_RELEASE caffeproto HINTS ${FIND_LIB_PREFIX}/caffe_cpu/lib)
    find_library(Caffe_LIB_DEBUG caffe-d HINTS ${FIND_LIB_PREFIX}/caffe_cpu/lib)
    find_library(Caffe_Proto_LIB_DEBUG caffeproto-d HINTS ${FIND_LIB_PREFIX}/caffe_cpu/lib)
  else (${GPU_MODE} MATCHES "OPENCL")
    set(VCXPROJ_FILE_GPU_MODE "")
    find_library(Caffe_LIB_RELEASE caffe HINTS ${FIND_LIB_PREFIX}/caffe/lib)
    find_library(Caffe_Proto_LIB_RELEASE caffeproto HINTS ${FIND_LIB_PREFIX}/caffe/lib)
    find_library(Caffe_LIB_DEBUG caffe-d HINTS ${FIND_LIB_PREFIX}/caffe/lib)
    find_library(Caffe_Proto_LIB_DEBUG caffeproto-d HINTS ${FIND_LIB_PREFIX}/caffe/lib)
  endif (${GPU_MODE} MATCHES "OPENCL")

  find_library(BOOST_SYSTEM_LIB_RELEASE boost_system-vc142-mt-x64-1_74 HINTS ${FIND_LIB_PREFIX}/caffe3rdparty/lib)
  find_library(BOOST_SYSTEM_LIB_DEBUG boost_system-vc142-mt-gd-x64-1_74 HINTS ${FIND_LIB_PREFIX}/caffe3rdparty/lib)
  find_library(BOOST_FILESYSTEM_LIB_RELEASE boost_filesystem-vc142-mt-x64-1_74 HINTS ${FIND_LIB_PREFIX}/caffe3rdparty/lib)
  find_library(BOOST_FILESYSTEM_LIB_DEBUG boost_filesystem-vc142-mt-gd-x64-1_74 HINTS ${FIND_LIB_PREFIX}/caffe3rdparty/lib)
  if (WITH_3D_RENDERER)
    find_library(GLUT_LIBRARY freeglut HINTS ${FIND_LIB_PREFIX}/freeglut/lib)
  endif (WITH_3D_RENDERER)
  if (WITH_FLIR_CAMERA)
    find_library(SPINNAKER_LIB spinnaker_v140 HINTS ${FIND_LIB_PREFIX}/spinnaker/lib)
  endif (WITH_FLIR_CAMERA)
  set(Caffe_LIBS_RELEASE "${Caffe_LIB_RELEASE};${Caffe_Proto_LIB_RELEASE}")
  set(Caffe_LIBS_DEBUG "${Caffe_LIB_DEBUG};${Caffe_Proto_LIB_DEBUG}")
  set(OpenCV_INCLUDE_DIRS "3rdparty/windows/opencv/include")
  if (${GPU_MODE} MATCHES "OPENCL")
    unset(Caffe_INCLUDE_DIRS CACHE)
    set(Caffe_INCLUDE_DIRS "3rdparty/windows/caffe_opencl/include;3rdparty/windows/caffe_opencl/include2" CACHE FILEPATH "Caffe_INCLUDE_DIRS")
  elseif (${GPU_MODE} MATCHES "CPU_ONLY")
    set(Caffe_INCLUDE_DIRS "3rdparty/windows/caffe_cpu/include;3rdparty/windows/caffe_cpu/include2" CACHE FILEPATH "Caffe_INCLUDE_DIRS")
  else (${GPU_MODE} MATCHES "OPENCL")
    set(Caffe_INCLUDE_DIRS "3rdparty/windows/caffe/include;3rdparty/windows/caffe/include2" CACHE FILEPATH "Caffe_INCLUDE_DIRS")
  endif (${GPU_MODE} MATCHES "OPENCL")
  set(WINDOWS_INCLUDE_DIRS "3rdparty/windows/caffe3rdparty/include")
  if (WITH_3D_RENDERER)
    set(GLUT_INCLUDE_DIRS "3rdparty/windows/freeglut/include")
  endif (WITH_3D_RENDERER)
  if (WITH_FLIR_CAMERA)
    set(SPINNAKER_INCLUDE_DIRS "3rdparty/windows/spinnaker/include")
  endif (WITH_FLIR_CAMERA)
  set(Caffe_FOUND 1)

  if (BUILD_PYTHON)
    if (NOT BUILD_BIN_FOLDER)
      message(FATAL_ERROR "BUILD_BIN_FOLDER must be turned on to as well to build python library")
    endif (NOT BUILD_BIN_FOLDER)
  endif (BUILD_PYTHON)

  if (BUILD_BIN_FOLDER)
    set(BIN_FOLDER ${CMAKE_BINARY_DIR}/bin)
    file(MAKE_DIRECTORY ${BIN_FOLDER})
    if (${GPU_MODE} MATCHES "CUDA")
      file(GLOB CAFFE_DLL "${CMAKE_SOURCE_DIR}/3rdparty/windows/caffe/bin/*.dll")
    elseif (${GPU_MODE} MATCHES "OPENCL")
      file(GLOB CAFFE_DLL "${CMAKE_SOURCE_DIR}/3rdparty/windows/caffe_opencl/bin/*.dll")
    elseif (${GPU_MODE} MATCHES "CPU_ONLY")
      file(GLOB CAFFE_DLL "${CMAKE_SOURCE_DIR}/3rdparty/windows/caffe_cpu/bin/*.dll")
    endif ()
    file(COPY ${CAFFE_DLL} DESTINATION ${BIN_FOLDER})
    file(GLOB CAFFE_3RD_PARTY_DLL "${CMAKE_SOURCE_DIR}/3rdparty/windows/caffe3rdparty/lib/*.dll")
    file(COPY ${CAFFE_3RD_PARTY_DLL} DESTINATION ${BIN_FOLDER})
    file(GLOB OPENCV_DLL "${CMAKE_SOURCE_DIR}/3rdparty/windows/opencv/x64/vc${OPENCV_VS_VERSION}/bin/*.dll")
    file(COPY ${OPENCV_DLL} DESTINATION ${BIN_FOLDER})
    if (WITH_3D_RENDERER OR WITH_3D_ADAM_MODEL)
      file(GLOB GLUT_DLL "${CMAKE_SOURCE_DIR}/3rdparty/windows/freeglut/bin/*.dll")
      file(COPY ${GLUT_DLL} DESTINATION ${BIN_FOLDER})
    endif (WITH_3D_RENDERER OR WITH_3D_ADAM_MODEL)
    if (WITH_FLIR_CAMERA)
      file(GLOB SPINNAKER_DLL "${CMAKE_SOURCE_DIR}/3rdparty/windows/spinnaker/bin/*")
      file(COPY ${SPINNAKER_DLL} DESTINATION ${BIN_FOLDER})
    endif (WITH_FLIR_CAMERA)
  endif (BUILD_BIN_FOLDER)
endif (WIN32)

### PROJECT INCLUDES
include_directories(
  include
  ${Protobuf_INCLUDE_DIRS}
  ${GFLAGS_INCLUDE_DIR}
  ${GLOG_INCLUDE_DIR}
  ${OpenCV_INCLUDE_DIRS})

if (USE_ASIO)
  include_directories(${CMAKE_SOURCE_DIR}/3rdparty/asio/include/)
  add_definitions(-DUSE_ASIO)
  add_definitions(-DASIO_STANDALONE)
endif (USE_ASIO)

if (NOT ${WITH_EIGEN} MATCHES "NONE")
  include_directories(${EIGEN3_INCLUDE_DIRS})
  set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries} Eigen3::Eigen)
endif (NOT ${WITH_EIGEN} MATCHES "NONE")

if (APPLE)
  include_directories("/usr/local/opt/openblas/include")
endif (APPLE)

if (USE_MKL)
  include_directories("${MKL_PATH}/include/")
endif (USE_MKL)

if (Caffe_FOUND)
  include_directories(${Caffe_INCLUDE_DIRS})
endif (Caffe_FOUND)

if (${GPU_MODE} MATCHES "CUDA")
  include_directories(${CUDA_INCLUDE_DIRS})
elseif (${GPU_MODE} MATCHES "OPENCL")
  include_directories(${OpenCL_INCLUDE_DIRS})
endif (${GPU_MODE} MATCHES "CUDA")

# 3D
if (WITH_3D_RENDERER)
  include_directories(${GLUT_INCLUDE_DIRS})
endif (WITH_3D_RENDERER)
if (WITH_CERES)
  include_directories(${CERES_INCLUDE_DIRS})
endif (WITH_CERES)
if (WITH_FLIR_CAMERA)
  include_directories(SYSTEM ${SPINNAKER_INCLUDE_DIRS}) # To remove its warnings, equivalent to -isystem
endif (WITH_FLIR_CAMERA)
if (WITH_3D_ADAM_MODEL)
  include_directories(include/adam) # TODO: TEMPORARY - TO BE REMOVED IN THE FUTURE
  include_directories(${CERES_INCLUDE_DIRS})
  include_directories(${EIGEN3_INCLUDE_DIRS})
  set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries} Eigen3::Eigen)
  include_directories(${IGL_INCLUDE_DIRS})
  include_directories(${LIBIGL_INCLUDE_DIRS})
  include_directories(${OPENGL_INCLUDE_DIR} ${GLUT_INCLUDE_DIRS} ${GLEW_INCLUDE_DIRS})
endif (WITH_3D_ADAM_MODEL)

# Windows includes
if (WIN32)
  include_directories(${WINDOWS_INCLUDE_DIRS})
endif (WIN32)

### COLLECT ALL 3RD-PARTY LIBRARIES TO BE LINKED AGAINST
if (UNIX OR APPLE)
  set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries} ${OpenCV_LIBS} ${GLOG_LIBRARY})
elseif (WIN32)
  set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries}
      debug ${OpenCV_LIBS_DEBUG} optimized ${OpenCV_LIBS_RELEASE}
      debug ${GFLAGS_LIBRARY_DEBUG} optimized ${GFLAGS_LIBRARY_RELEASE}
      debug ${GLOG_LIBRARY_DEBUG} optimized ${GLOG_LIBRARY_RELEASE})
endif (UNIX OR APPLE)

# G Flags (for demos)
if (UNIX OR APPLE)
  set(examples_3rdparty_libraries ${OpenPose_3rdparty_libraries} ${GFLAGS_LIBRARY})
endif (UNIX OR APPLE)

# Deep net Framework
if (${DL_FRAMEWORK} MATCHES "CAFFE" OR ${DL_FRAMEWORK} MATCHES "NV_CAFFE")
  if (UNIX OR APPLE)
    set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries} ${Caffe_LIBS})
  elseif (WIN32)
    set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries}
      debug ${Caffe_LIBS_DEBUG} optimized ${Caffe_LIBS_RELEASE})
  endif (UNIX OR APPLE)
endif (${DL_FRAMEWORK} MATCHES "CAFFE" OR ${DL_FRAMEWORK} MATCHES "NV_CAFFE")

# CPU vs. GPU
if (USE_MKL)
  set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries} ${MKL_LIBS})
endif (USE_MKL)
if (${GPU_MODE} MATCHES "OPENCL")
  set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries} ${CMAKE_THREAD_LIBS_INIT} ${OpenCL_LIBRARIES})
endif (${GPU_MODE} MATCHES "OPENCL")

# Boost
if (WIN32)
  if (${GPU_MODE} MATCHES "CPU_ONLY" OR ${GPU_MODE} MATCHES "OPENCL" OR BUILD_PYTHON)
    set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries}
        debug ${BOOST_SYSTEM_LIB_RELEASE} optimized ${BOOST_SYSTEM_LIB_RELEASE})
    set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries}
        debug ${BOOST_FILESYSTEM_LIB_RELEASE} optimized ${BOOST_FILESYSTEM_LIB_RELEASE})
  endif ()
endif (WIN32)

# 3-D
if (WITH_3D_ADAM_MODEL)
  set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries}
      ${OPENGL_LIBRARIES} ${GLUT_LIBRARY} ${GLEW_LIBRARY} ${FREE_IMAGE_LIBRARY})
endif (WITH_3D_ADAM_MODEL)
if (WITH_3D_RENDERER)
  set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries} ${GLUT_LIBRARY} ${OPENGL_LIBRARIES})
endif (WITH_3D_RENDERER)
if (WITH_CERES)
  set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries} ${CERES_LIBRARIES})
endif (WITH_CERES)
if (WITH_FLIR_CAMERA)
  set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries} ${SPINNAKER_LIB})
endif (WITH_FLIR_CAMERA)

# Pthread
if (UNIX OR APPLE)
  set(OpenPose_3rdparty_libraries ${OpenPose_3rdparty_libraries} pthread)
endif (UNIX OR APPLE)

### ADD SUBDIRECTORIES
if (Caffe_FOUND)
  add_subdirectory(src)
  if (BUILD_EXAMPLES)
    add_subdirectory(examples)
  endif (BUILD_EXAMPLES)
endif (Caffe_FOUND)

### DOWNLOAD MODELS
# Download the models if the flag is set
message(STATUS "Download the models.")

# URL to the models
set(OPENPOSE_URL "${DOWNLOAD_SERVER}models/")

download_model("BODY_25" ${DOWNLOAD_BODY_25_MODEL} pose/body_25/pose_iter_584000.caffemodel 78287B57CF85FA89C03F1393D368E5B7) # Body (BODY_25)
download_model("body (COCO)" ${DOWNLOAD_BODY_COCO_MODEL} pose/coco/pose_iter_440000.caffemodel 5156d31f670511fce9b4e28b403f2939) # Body (COCO)
download_model("body (MPI)" ${DOWNLOAD_BODY_MPI_MODEL} pose/mpi/pose_iter_160000.caffemodel 2ca0990c7562bd7ae03f3f54afa96e00) # Body (MPI)
download_model("face" ${DOWNLOAD_FACE_MODEL} face/pose_iter_116000.caffemodel a6df79b6d2c9f2cd45d4b0c9b38e4970) # Face
download_model("hand" ${DOWNLOAD_HAND_MODEL} hand/pose_iter_102000.caffemodel 8a2cfc5e8c1b0a4b1e8f6e3da7a59a34) # Hand

### END
