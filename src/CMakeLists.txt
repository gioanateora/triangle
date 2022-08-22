list(APPEND triangle_src_sources ${CMAKE_CURRENT_SOURCE_DIR}/triangle.c)

list(APPEND triangle_src_headers ${CMAKE_CURRENT_SOURCE_DIR}/mytriangle.hpp)
list(APPEND triangle_src_headers ${CMAKE_CURRENT_SOURCE_DIR}/triangle.h)

list(APPEND triangle_src_includes ${CMAKE_CURRENT_SOURCE_DIR})

set(triangle_src_sources ${triangle_src_sources} PARENT_SCOPE)
set(triangle_src_headers ${triangle_src_headers} PARENT_SCOPE)
set(triangle_src_includes ${triangle_src_includes} PARENT_SCOPE)
