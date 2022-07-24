cmake_minimum_required(VERSION 3.17)
project(LearnOpenGL)
LINK_DIRECTORIES(${CMAKE_SOURCE_DIR}/lib )
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
set(LIBS glfw3 GL X11 pthread Xrandr Xi dl)

add_library(GLAD "src/glad.c")
set(LIBS ${LIBS} GLAD)

set(CHAPTERS
  ch1
)

set(ch1
  hello_window
  triangle
  hello_triangle
)

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
  endforeach(EXCERCISE)
endforeach(CHAPTER)

include_directories(include)
include_directories(${CMAKE_SOURCE_DIR}/includes)
