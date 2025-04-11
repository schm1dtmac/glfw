
add_library(glfw ${GLFW_LIBRARY_TYPE}
                 "${GLFW_SOURCE_DIR}/include/GLFW/glfw3.h"
                 "${GLFW_SOURCE_DIR}/include/GLFW/glfw3native.h"
                 internal.h platform.h
                 context.c init.c input.c monitor.c platform.c window.c)

# The time, thread and module code is shared between all backends on a given OS,
# including the null backend, which still needs those bits to be functional
target_sources(glfw PRIVATE cocoa_time.h cocoa_time.c posix_thread.h
                                posix_module.c posix_thread.c)

enable_language(OBJC)
target_compile_definitions(glfw PRIVATE _GLFW_COCOA)
target_compile_options(glfw
PRIVATE
    -O3
    -march=native
    -fomit-frame-pointer
)
target_sources(glfw PRIVATE cocoa_platform.h cocoa_init.m
                            cocoa_monitor.m cocoa_window.m
                            nsgl_context.m)
set(GLFW_LIB_NAME glfw3)
set(GLFW_LIB_NAME_SUFFIX "")

set_target_properties(glfw PROPERTIES
                      OUTPUT_NAME ${GLFW_LIB_NAME}
                      VERSION ${GLFW_VERSION_MAJOR}.${GLFW_VERSION_MINOR}
                      SOVERSION ${GLFW_VERSION_MAJOR}
                      C_STANDARD 99
                      C_EXTENSIONS OFF
                      INTERPROCEDURAL_OPTIMIZATION ON
                      DEFINE_SYMBOL _GLFW_BUILD_DLL
                      FOLDER "GLFW3")

target_include_directories(glfw PUBLIC
                           "$<BUILD_INTERFACE:${GLFW_SOURCE_DIR}/include>"
                           "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>")
target_include_directories(glfw PRIVATE
                           "${GLFW_SOURCE_DIR}/src"
                           "${GLFW_BINARY_DIR}/src")
target_link_libraries(glfw PRIVATE Threads::Threads)

target_link_libraries(glfw PRIVATE "-framework Cocoa"
                                   "-framework IOKit"
                                   "-framework CoreFoundation"
                                   "-framework QuartzCore"
                                   "-framework CoreVideo")

set(glfw_PKG_DEPS "")
set(glfw_PKG_LIBS "-framework Cocoa -framework IOKit -framework CoreFoundation -framework QuartzCore -framework CoreVideo")

list(JOIN glfw_PKG_DEPS " " deps)
list(JOIN glfw_PKG_LIBS " " libs)

set(GLFW_PKG_CONFIG_REQUIRES_PRIVATE "${deps}" CACHE INTERNAL
    "GLFW pkg-config Requires.private")
set(GLFW_PKG_CONFIG_LIBS_PRIVATE "${libs}" CACHE INTERNAL
    "GLFW pkg-config Libs.private")

configure_file("${GLFW_SOURCE_DIR}/CMake/glfw3.pc.in" glfw3.pc @ONLY)
