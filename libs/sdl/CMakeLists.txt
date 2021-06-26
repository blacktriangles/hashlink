
add_library(sdl.hdll SHARED
    sdl.c
    gl.c
)

set_as_hdll(sdl)

target_include_directories(sdl.hdll
    PRIVATE
    ${INCLUDES_BASE_DIR}/gl
)

target_link_libraries(sdl.hdll
    libhl
    SDL2
)

if(WIN32)
    target_link_libraries(sdl.hdll
        winmm
        opengl32
    )
endif()

if(APPLE)
    find_package(OPENGL REQUIRED)
    target_include_directories(sdl.hdll
        PRIVATE
        ${OPENGL_INCLUDE_DIR}
    )
    target_link_libraries(sdl.hdll
        libhl
        ${OPENGL_gl_LIBRARY}
    )
endif()


install(
    TARGETS sdl.hdll
    DESTINATION ${HDLL_DESTINATION}
)
