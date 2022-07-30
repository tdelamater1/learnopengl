cmake_minimum_required(VERSION 3.17)
project(LearnOpenGL)
LINK_DIRECTORIES(${CMAKE_SOURCE_DIR}/lib )
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
set(LIBS glfw3 GL X11 pthread Xrandr Xi dl)

add_library(GLAD "src/glad.c")
add_library(STB "src/stb_image.cpp")
set(LIBS ${LIBS} GLAD STB)

set(CHAPTERS
  ch1
  ch2
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
)

configure_file(configuration/root_directory.h.in configuration/root_directory.h)
include_directories(${CMAKE_BINARY_DIR}/configuration)

foreach(CHAPTER ${CHAPTERS})
  foreach(EXCERCISE ${${CHAPTER}})
    file(GLOB SOURCE
      "src/${CHAPTER}/${EXCERCISE}/*.h"
       "src/${CHAPTER}/${EXCERCISE}/*.cpp"
    )
    add_executable(${EXCERCISE} ${SOURCE})
    target_link_libraries(${EXCERCISE} ${LIBS})
    set_target_properties(${EXCERCISE}
      PROPERTIES
      RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${CHAPTER}"
    )
    # copy shader files to build directory
    file(GLOB SHADERS 
      "src/${CHAPTER}/${EXCERCISE}/*.fs"
      "src/${CHAPTER}/${EXCERCISE}/*.vs"
    )
    foreach(SHADER ${SHADERS})
      file(COPY ${SHADER} DESTINATION ${CMAKE_SOURCE_DIR}/bin/${CHAPTER})
    endforeach(SHADER)
  endforeach(EXCERCISE)
endforeach(CHAPTER)

include_directories(include)
include_directories(${CMAKE_SOURCE_DIR}/includes)
