cmake_minimum_required (VERSION 3.0)
cmake_policy(VERSION 3.0)

project(LearnOpenGL)

set(CMAKE_CXX_STANDARD 17) # this does nothing for MSVC, use target_compile_options below
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build (Debug or Release)" FORCE)
ENDIF(NOT CMAKE_BUILD_TYPE)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")

link_directories(${CMAKE_SOURCE_DIR}/lib)

# find the required packages
find_package(GLM REQUIRED)
message(STATUS "GLM included at ${GLM_INCLUDE_DIR}")
find_package(GLFW3 REQUIRED)
message(STATUS "Found GLFW3 in ${GLFW3_INCLUDE_DIR}")
find_package(ASSIMP REQUIRED)
message(STATUS "Found ASSIMP in ${ASSIMP_INCLUDE_DIR}")
# find_package(SOIL REQUIRED)
# message(STATUS "Found SOIL in ${SOIL_INCLUDE_DIR}")
# find_package(GLEW REQUIRED)
# message(STATUS "Found GLEW in ${GLEW_INCLUDE_DIR}")

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wall")
find_package(OpenGL REQUIRED)
add_definitions(${OPENGL_DEFINITIONS})
find_package(X11 REQUIRED)
# note that the order is important for setting the libs
# use pkg-config --libs $(pkg-config --print-requires --print-requires-private glfw3) in a terminal to confirm
set(LIBS ${GLFW3_LIBRARY} X11 Xrandr Xinerama Xi Xxf86vm Xcursor GL dl pthread freetype ${ASSIMP_LIBRARY})
set (CMAKE_CXX_LINK_EXECUTABLE "${CMAKE_CXX_LINK_EXECUTABLE} -ldl")


set(CHAPTERS
  ch1
  ch2
  ch3
)

set(ch1
  hello_window
  triangle
  hello_triangle
  trianglesMakeBox
  drawTwoTriangles
)

set(ch2
  uniforms
  shaders_class
  textures
  combined_textures
)

set(ch3
  transform
)

configure_file(configuration/root_directory.h.in configuration/root_directory.h)
include_directories(${CMAKE_BINARY_DIR}/configuration)

# first create relevant static libraries requried for other projects
add_library(STB_IMAGE "src/stb_image.cpp")
set(LIBS ${LIBS} STB_IMAGE)

add_library(GLAD "src/glad.c")
set(LIBS ${LIBS} GLAD)

macro(makeLink src dest target)
  add_custom_command(TARGET ${target} POST_BUILD COMMAND ${CMAKE_COMMAND} -E create_symlink ${src} ${dest}  DEPENDS  ${dest} COMMENT "mklink ${src} -> ${dest}")
endmacro()

function(create_project_from_sources chapter excercise)
	file(GLOB SOURCE
            "src/${chapter}/${excercise}/*.h"
            "src/${chapter}/${excercise}/*.cpp"
            "src/${chapter}/${excercise}/*.vs"
            "src/${chapter}/${excercise}/*.fs"
            "src/${chapter}/${excercise}/*.gs"
            "src/${chapter}/${excercise}/*.cs"
  )

	if (excercise STREQUAL "")
		SET(replaced "")
		string(REPLACE "/" "_" replaced ${chapter})
		set(NAME ${replaced})
	else()
		set(NAME "${chapter}__${excercise}")
	endif()

  add_executable(${NAME} ${SOURCE})
  target_link_libraries(${NAME} ${LIBS})
  set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${chapter}")
  # copy shader files to build directory
  file(GLOB SHADERS
           "src/${chapter}/${excercise}/*.vs"
           # "src/${chapter}/${excercise}/*.frag"
           "src/${chapter}/${excercise}/*.fs"
           "src/${chapter}/${excercise}/*.gs"
     "src/${chapter}/${excercise}/*.cs"
  )
  foreach(SHADER ${SHADERS})
    file(COPY ${SHADER} DESTINATION ${CMAKE_SOURCE_DIR}/bin/${chapter})
  endforeach(SHADER)
	# copy dlls
	file(GLOB DLLS "dlls/*.dll")
endfunction()

# then create a project file per tutorial
foreach(CHAPTER ${CHAPTERS})
  foreach(EXCERCISE ${${CHAPTER}})
    create_project_from_sources(${CHAPTER} ${EXCERCISE})
  endforeach(EXCERCISE)
endforeach(CHAPTER)

include_directories(${CMAKE_SOURCE_DIR}/includes)
