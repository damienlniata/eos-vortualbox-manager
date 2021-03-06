# project name
project (com.github.damienlniata.eos-virtualbox-manager)

# the oldest stable cmake version we support
cmake_minimum_required (VERSION 2.6)

# tell cmake where its modules can be found in our project directory
list (APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
list (APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/src)

# where we install data directory (if we have any)
set (HOMEDIR $ENV{HOME})
set (HOME_DATADIR ${HOMEDIR}/.local/share/eradio)
set (DATADIR "${CMAKE_INSTALL_PREFIX}/share")

# what to call that directory where we install data too
set (PKGDATADIR "${DATADIR}/com.github.damienlniata.eos-virtualbox-manager")

set (EXEC_NAME "com.github.damienlniata.eos-virtualbox-manager")
set (RELEASE_NAME "Virtualbox Manager")
set (VERSION "0.1")
set (VERSION_INFO "First Release")

set (CONFIG_FILE ${CMAKE_CURRENT_SOURCE_DIR}/src/config.vala)
set (CONFIG_FILE_REL src/config.vala)
configure_file (${CMAKE_CURRENT_SOURCE_DIR}/src/config.vala.cmake ${CONFIG_FILE})

# we're about to use pkgconfig to make sure dependencies are installed so let's find pkgconfig first
find_package(PkgConfig)

set (PROJECT_DEPS
    gtk+-3.0
    granite
    gee-0.8
    libnotify
)

pkg_check_modules(DEPS REQUIRED ${PROJECT_DEPS})

add_definitions(${DEPS_CFLAGS})
link_libraries(${DEPS_LIBRARIES} m)
link_directories(${DEPS_LIBRARY_DIRS})

# make sure we have vala
find_package(Vala REQUIRED)
# make sure we use vala
include(ValaVersion)
# make sure it's the desired version of vala
ensure_vala_version("0.16" MINIMUM)

# files we want to compile
include(ValaPrecompile)
vala_precompile(VALA_C ${EXEC_NAME}
    ${CONFIG_FILE_REL}
    src/main.vala
    src/application/vbox-application.vala
    src/ui/windows/vbox-main-window.vala
    src/utils/vbox-utils.vala

# tell what libraries to use when compiling
PACKAGES
    ${PROJECT_DEPS}
)

# Translation files
set (GETTEXT_PACKAGE "${CMAKE_PROJECT_NAME}")
add_definitions (-DGETTEXT_PACKAGE=\"${GETTEXT_PACKAGE}\")
add_subdirectory (po)

# tell cmake what to call the executable we just made
add_executable(${EXEC_NAME} ${VALA_C})

# install the binaries we just made
install (TARGETS ${EXEC_NAME} RUNTIME DESTINATION bin)

# install our .desktop file so the Applications menu will see it
install (FILES ${CMAKE_CURRENT_SOURCE_DIR}/data/com.github.damienlniata.eos-virtualbox-manager.desktop DESTINATION ${DATADIR}/applications/)

# install our .appdata.xml file so AppCenter will see it
install (FILES ${CMAKE_CURRENT_SOURCE_DIR}/data/com.github.damienlniata.eos-virtualbox-manager.appdata.xml DESTINATION ${DATADIR}/metainfo/)