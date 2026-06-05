cmake_minimum_required(VERSION 3.12)
project(wihotspot-gui)

include(FindPkgConfig)
pkg_check_modules(GTK gtk+-3.0 REQUIRED)
pkg_check_modules(X11 x11 REQUIRED)
pkg_check_modules(AVAHI avahi-client REQUIRED)
include_directories(${GTK_INCLUDE_DIRS})
include_directories(${X11_INCLUDE_DIRS})
include_directories(${AVAHI_INCLUDE_DIRS})

add_definitions(-DNO_QR=1)

set (CMAKE_C_STANDARD 99)
set (CMAKE_CXX_STANDARD 11)

add_executable(${PROJECT_NAME} ui/main.c ui/h_prop.c ui/h_prop.h ui/ui.c ui/ui.h ui/read_config.cpp ui/read_config.h ui/util.c ui/util.h ui/about_ui.c ui/about_ui.h ui/mdns.c ui/mdns.h)

target_link_libraries(${PROJECT_NAME} ${GTK_LIBRARIES} ${X11_LIBRARIES} ${AVAHI_LIBRARIES})

include_directories(/usr/include)

set (source "${CMAKE_SOURCE_DIR}/ui/glade")
set (destination "${CMAKE_CURRENT_BINARY_DIR}/glade")

add_custom_command(
        TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E create_symlink ${source} ${destination}
        DEPENDS ${destination}
        COMMENT "symbolic link resources folder from ${source} => ${destination}"
)

set (appdir "/usr/share")
set (appbin "/usr/bin")

if (EXISTS "$ENV{DESTDIR}" AND NOT EXISTS "${CMAKE_INSTALL_PREFIX}")
    set (CMAKE_INSTALL_PREFIX "$ENV{DESTDIR}")
endif()

add_custom_target(install_build
        COMMAND mkdir -p ${CMAKE_INSTALL_PREFIX}${appdir}/${PROJECT_NAME}
        COMMAND mkdir -p ${CMAKE_INSTALL_PREFIX}${appdir}/${PROJECT_NAME}/glade
        COMMAND install -Dm755 ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME} ${CMAKE_INSTALL_PREFIX}${appdir}/${PROJECT_NAME}/${PROJECT_NAME}
        COMMAND install -Dm755 ${CMAKE_CURRENT_BINARY_DIR}/glade/* ${CMAKE_INSTALL_PREFIX}${appdir}/${PROJECT_NAME}/glade/
        COMMAND ln -rs ${CMAKE_INSTALL_PREFIX}${appdir}/${PROJECT_NAME}/${PROJECT_NAME} ${CMAKE_INSTALL_PREFIX}${appbin}/${PROJECT_NAME}

        COMMAND install -Dm644 ${CMAKE_SOURCE_DIR}/desktop/hotspot.png ${CMAKE_INSTALL_PREFIX}${appdir}/${PROJECT_NAME}/hotspot.png
        COMMAND install -Dm644 ${CMAKE_SOURCE_DIR}/desktop/wifihotspot.desktop ${CMAKE_INSTALL_PREFIX}${appdir}/${PROJECT_NAME}/${PROJECT_NAME}.desktop
        COMMAND install -Dm644 ${CMAKE_SOURCE_DIR}/desktop/wifihotspot.desktop ${CMAKE_INSTALL_PREFIX}${appdir}/applications/${PROJECT_NAME}.desktop
        DEPENDS ${PROJECT_NAME}
        COMMENT Installing files...

        COMMENT Remove old files if exist
        COMMAND rm -rf ${CMAKE_INSTALL_PREFIX}${appdir}/wihotspot
        COMMAND rm -f ${CMAKE_INSTALL_PREFIX}${appdir}/applications/wihotspot.desktop
        )

add_custom_target(uninstall_build
        COMMAND rm -rf ${CMAKE_INSTALL_PREFIX}${appdir}/${PROJECT_NAME}
        COMMAND rm -f ${CMAKE_INSTALL_PREFIX}${appbin}/${PROJECT_NAME}
        COMMAND rm -f ${CMAKE_INSTALL_PREFIX}${appdir}/applications/${PROJECT_NAME}.desktop
        COMMAND rm -rf ${CMAKE_INSTALL_PREFIX}${appdir}/${PROJECT_NAME}
        COMMENT Uninstalling files...

        COMMENT Remove old files if exist
        COMMAND rm -rf ${CMAKE_INSTALL_PREFIX}${appdir}/wihotspot
        COMMAND rm -f ${CMAKE_INSTALL_PREFIX}${appdir}/applications/wihotspot.desktop
        )

install(TARGETS ${PROJECT_NAME})

